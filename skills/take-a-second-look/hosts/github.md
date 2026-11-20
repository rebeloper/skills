# GitHub (gh CLI)

## Setup

`gh` must be logged in (`gh auth status`). If it isn't, ask the user to run `gh auth login` in their terminal. For GitHub Enterprise, use `--hostname <host>` and build every link from the PR's `url` (its host), never from `github.com`.
Get the repo with `gh repo view --json nameWithOwner -q .nameWithOwner` (or from the PR URL) and split it into `O`/`R`. `<N>` is the PR number.

Write every comment, reply, and summary body to a temp file and pass it with `-F body=@file`, so the shell never interprets it.

**Shell safety:** file paths, branch names, and titles come from the PR and can contain shell syntax (`$(…)`, backticks). Never type them into a command line, not even in double quotes. Write them into a file with a file-writing tool or a quoted heredoc (`<<'EOF'`), and build JSON bodies from those files with `jq` or `python3`. Send JSON with `--input <file>`.

## Who am I

- Me: `gh api user -q .login`
- PR author: `.author.login` from **Read the PR**.
- Waiting on me:
  - **My PR:** unresolved `reviewThreads` started by others where the last comment isn't mine, plus conversation comments and non-empty review summaries by others that I haven't answered.
  - **Their PR:** open threads I started or commented in where the last comment isn't mine, plus any thread whose latest comment @mentions me.
  Use **Read threads** for the data.

## Read the PR

```bash
gh pr view <N> -R "$O/$R" --json number,title,body,author,baseRefName,headRefName,headRefOid,additions,deletions,changedFiles,statusCheckRollup,url
gh pr diff <N> -R "$O/$R"
gh api -H "Accept: application/vnd.github.raw" "repos/$O/$R/contents/<url-encoded path>?ref=<headRefOid>"   # full file, any size; URL-encode the path with python3 from the path file
```

## Post an inline comment

```json
{ "body": "<comment markdown>", "commit_id": "<headRefOid>",
  "path": "<path>", "line": <line>, "side": "RIGHT" }
```

```bash
gh api "repos/$O/$R/pulls/<N>/comments" --input comment.json -q '"\(.id) \(.html_url)"'   # build comment.json with jq or python3
```

- Use `"side": "LEFT"` for the old side. For a range, add `"start_line": <first>`.
- `line` must be inside a diff hunk, or GitHub returns 422. Post a finding outside the hunks as a PR conversation comment instead (`repos/$O/$R/issues/<N>/comments`, see **Reply, resolve**).
- Keep the returned id for undo.
- **Suggestion:** a ```` ```suggestion ```` block holding the replacement for lines `start_line`–`line`. GitHub shows it as a one-click "Commit suggestion".

## Submit the verdict

```bash
gh api "repos/$O/$R/pulls/<N>/reviews" -f commit_id=<headRefOid> \
  -f event=<REQUEST_CHANGES|COMMENT|APPROVE> -F body=@summary.md -q .html_url
```

`REQUEST_CHANGES` and `COMMENT` need a body. On your own PR, only `COMMENT` works.

## Conversation comment

```bash
gh api "repos/$O/$R/issues/<N>/comments" -F body=@comment.md -q '"\(.id) \(.html_url)"'   # PR-level comment (PR-level findings, replies to conversation comments and review summaries)
```

## Read threads

```bash
gh api graphql --paginate -F o="$O" -F r="$R" -F n=<N> -f query='
query($o:String!,$r:String!,$n:Int!,$endCursor:String){repository(owner:$o,name:$r){pullRequest(number:$n){
  reviewThreads(first:100,after:$endCursor){pageInfo{hasNextPage endCursor}
    nodes{id isResolved isOutdated path line startLine diffSide
    comments(first:100){totalCount nodes{author{login} body url createdAt diffHunk originalCommit{abbreviatedOid}}}}}}}}'
gh api --paginate "repos/$O/$R/issues/<N>/comments?per_page=100"   # conversation comments: .user.login .body .html_url .id
gh api --paginate "repos/$O/$R/pulls/<N>/reviews?per_page=100"     # review summaries: .user.login .state .body .html_url .commit_id
```

A review summary with a non-empty body from someone else counts as a conversation item, and its reply is a conversation comment.

`--paginate` follows `reviewThreads` pages. If a thread's `totalCount` is over 100, say that only its first 100 replies were read.

`reviewThreads` are line comments and can be resolved. `diffHunk` on the first comment feeds the Thread card's code view. Conversation comments come from the REST call and can only be replied to.

## Reply, resolve

```bash
# reply in a thread
gh api graphql -F t=<threadId> -F b=@reply.md -f query='
mutation($t:ID!,$b:String!){addPullRequestReviewThreadReply(input:{pullRequestReviewThreadId:$t,body:$b}){comment{url fullDatabaseId}}}'

# resolve / unresolve
gh api graphql -F t=<threadId> -f query='mutation($t:ID!){resolveReviewThread(input:{threadId:$t}){thread{isResolved}}}'
gh api graphql -F t=<threadId> -f query='mutation($t:ID!){unresolveReviewThread(input:{threadId:$t}){thread{isResolved}}}'

# reply to a PR conversation comment
gh api "repos/$O/$R/issues/<N>/comments" -F body=@reply.md -q '"\(.id) \(.html_url)"'
```

## Undo

```bash
gh api -X DELETE "repos/$O/$R/pulls/comments/<id>"     # inline comment or thread reply (fullDatabaseId)
gh api -X DELETE "repos/$O/$R/issues/comments/<id>"    # PR conversation comment
```

If the undone action also resolved a thread, unresolve it. A submitted review (Approve, Request changes, Comment) can't be deleted. Say so, and offer a follow-up comment instead.

## New commits since me

```bash
OLD=$(gh api --paginate "repos/$O/$R/pulls/<N>/reviews?per_page=100" -q '.[]|select(.user.login=="<me>")|.commit_id' | tail -1)
gh api -H "Accept: application/vnd.github.diff" "repos/$O/$R/compare/$OLD...<headRefOid>"
```

If I never submitted a review, use the PR commit just before my first comment (`gh pr view <N> --json commits`). If `OLD` is no longer reachable because of a force-push, say so and offer a full review instead.

## Reviews

```bash
gh pr view <N> -R "$O/$R" --json latestReviews,reviewRequests \
  -q '.latestReviews[]|"\(.author.login) \(.state)"'      # APPROVED | CHANGES_REQUESTED | COMMENTED
```

## Commit link

`<PR url>/commits/<sha>`

## Line link (Copy)

`<PR url>/files#diff-<sha256 of path>R<line>`, using `L<line>` for the old side. Get the hash from the path file, not the command line: `tr -d '\n' < path.txt | shasum -a 256`.
