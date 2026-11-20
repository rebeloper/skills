---
name: take-a-second-look
description: Use when the user asks to review code, including local uncommitted or staged changes, a branch before commit or push, or a pull request or merge request on GitHub, GitLab, or Bitbucket by number or URL, or asks to work through, answer, reply to, or resolve review comments on a PR or MR.
---

# take-a-second-look

Code review in chat that looks and feels like a GUI. Four modes, one visual language.

**REQUIRED:** Read `review-tiers.md` before reviewing, and `review-advanced.md` only at tier 3–4 or for a Re-review. Before writing any output, read `ui-kit.md`, plus `ui-review.md` for Local, PR review, and Self-review, or `ui-discussion.md` for Discussion. Don't read the other mode's file. Screens several modes share (Finish menu, Undo, Done card) are in `ui-kit.md`. Read `security-checklists.md` sections only when the diff touches their area. Every screen you print is one of the UI files' templates, filled in. For PR review, Self-review, and Discussion, read the host file first (see **Host**). Local mode needs no host: skip host detection.

## Host

Find the host from the PR/MR URL if the user gave one, otherwise from `git remote get-url origin`:

| Remote contains | Read |
|---|---|
| `github.com`, or a host where `gh auth status --hostname <host>` succeeds (GitHub Enterprise) | `hosts/github.md` |
| `gitlab.com`, or a host where `glab auth status --hostname <host>` succeeds | `hosts/gitlab.md` |
| `bitbucket.org` | `hosts/bitbucket.md` |

Check only the remote's own host, never a global login. If nothing matches, ask with a menu. Use the host's own words everywhere: "PR #12" on GitHub and Bitbucket, "MR !12" on GitLab. Every host file covers the same actions: read, post inline, suggestion format, verdict, read threads, reply/resolve, undo, and line link. If a host can't do something, leave it out of the menus.

## Pick the mode

Decide in this order. The first match wins.

1. **The user named the mode** ("reply to the comments", "review it as a reviewer", "just my local changes"): use it.
2. **No PR/MR mentioned:** **Local**. If the working tree is clean, review the branch against its base. If that's empty too, say there's nothing to review.
3. **A PR/MR is mentioned,** or the user refers to "my PR" or "the comments" without a number. Without a number or URL, look up the open PR for the current branch (`gh pr view`, `glab mr view`, or Bitbucket's pull requests filtered by source branch). If there's none, or more than one, ask with a menu. Then get the user's username, the PR author, and the threads waiting on the user (host file, **Who am I**):
   - The user asked to *review* it ("review PR 12"): **PR review** on someone else's PR, **Self-review** on their own. If threads are waiting, offer Discussion after the Done card.
   - Any threads waiting on the user, on either side: **Discussion**.
   - Someone else's PR, nothing waiting: **PR review**.
   - The user's PR, nothing waiting: **Self-review**.

Show the pick and the reason on one line under the header, e.g. `🧭 Their PR · 2 replies waiting on you → Discussion`. If that's wrong, the user says so and you switch. If the host can't be reached to check the author, ask with a menu.

## Local

1. Collect: `git status --short`, `git diff HEAD`, untracked files (`git ls-files --others --exclude-standard`, read them). In a repo with no commits yet (`git rev-parse --verify HEAD` fails), use `git diff --cached`, `git diff`, and the untracked files instead. If the user asks for the whole repo, diff from the empty tree: `git diff $(git hash-object -t tree /dev/null) HEAD`. For a branch: `git diff <base>...HEAD`, where `<base>` is the first of these that `git rev-parse --verify` accepts: the one the user named, `origin/HEAD`, `main`, `master`. If none exists, ask with a menu.
2. Review at the chosen tier (`review-tiers.md`; without one, the tier is picked by size).
3. Print the **Overview**, then the **Start menu**. If the user changes the tier there, re-review and show the new overview.
4. Walk through the findings: show one **Finding card** with its **Finding menu**, apply the fix if the user picks it, then move to the next card.
5. Show a **Done card**.

## PR review

1. Read the PR/MR and its diff (host file). Do not check out the branch unless the user asks.
2. Review at the chosen tier (`review-tiers.md`; without one, the tier is picked by size). Print the **PR header**, the **Overview**, and the **Start menu**. If the user changes the tier there, re-review and show the new overview.
3. Walk through the findings one **Finding card** at a time. Each pick in the **Finding menu** takes effect immediately: the comment is posted, copied, or dropped.
   - **Other reviewers:** read the existing threads and reviews first (host file, **Read threads** and **Reviews**). If a finding was already raised in an open thread, mark the card `👥 already raised by @x`, and the menu offers `👍 Agree in their thread` instead of `📤 Post`. Never post a duplicate.
4. Show the **Finish menu** to submit the overall verdict, then a **Done card**.

## Self-review

The user's own PR before anyone reviews it. Follow **PR review**, with these changes:
- The Finding menu is the Local one: 🔧 Fix · ⏭️ Skip · ⏹️ Stop here.
- Fixes go to the working tree. Use the same **Branch safety** steps as Discussion (below). Before the first fix, compare `git rev-parse HEAD` with the PR's head commit. If they differ (unpushed commits, or a stale or diverged branch), say so and ask: fast-forward to the PR head, fix anyway (each fix located by content, never by line number alone), or cancel.
- There is no Finish menu and nothing is posted. The Done card ends with: "Changes are uncommitted. Commit and push when ready."

## Discussion

Replying to PR/MR conversations that wait on the user, from either side. The role decides the take and the menu:

| | Author (the user's PR) | Reviewer (someone else's PR) |
|---|---|---|
| Waiting on the user | open threads from others where the last word isn't the user's | threads the user started or joined where someone replied after them, plus threads that @mention them |
| Take | ✅ valid · 🤔 partly · ❌ disagree | ✅ addressed · 🤔 partly · ❌ not addressed |
| Menu | see **Thread action bar** in `ui-discussion.md` | see **Thread action bar** in `ui-discussion.md` |

1. Fetch the threads (host file). Keep only the ones waiting on the user, unless they ask for all of them.
2. Print the **Inbox**. Blocking threads come first.
3. Walk through one **Thread card** at a time, then its **Thread action bar**. Before taking a side, check against the code as it is now, not against what anyone claims:
   - **Author:** check the comment against the code. If it cites a house rule, read that rule (step P in `review-tiers.md`) and check that it really says so. Never agree because a reviewer said so, and never push back without a technical reason. If a comment is unclear, the draft reply asks a question.
   - **Reviewer:** check whether their reply or new commits actually address your point. Show what changed on that line since your comment.
4. **Author: commit per fix, send after the push.**
   - **Branch safety**, before the first fix, in this order:
     1. Record where the user is: the current branch, or the commit if HEAD is detached.
     2. If the tree has uncommitted changes, ask whether to stash them *before* switching anything, because they must never be mixed into fix commits. If the user agrees, run `git stash push -u -m "reb-review: <PR>"` and remember it. If they decline, skip the fix on any file that has local changes (queue the reply only), and say so.
     3. Get onto the PR's head: compare by commit, not only by branch name. If the head branch is checked out in another worktree, offer to work there instead. If the local branch is behind, ahead of, or diverged from the PR head, say so and ask: fast-forward, or cancel.
   - 🔧 Fix applies the change and commits only that fix's files: `git commit -m "fix: <short title>" -m "Review: <thread url>" -- <paths>`. Before committing, check that `git diff --cached` holds nothing else. If a commit hook fails, show its output and stop on that card.
   - Every pick queues the reply and resolve instead of posting them. This covers fixes, answers, and pushback alike.
   - "undo" before the push removes the last queued item. If that item made its own commit and `git rev-parse HEAD` is still that commit, run `git reset --mixed HEAD~1`, then reverse exactly the lines that fix changed, so nothing stays staged or half-applied. If the item made no commit, only remove it from the queue. If the commit is shared with other queued replies, ask first, because those replies link to it. If HEAD moved, don't reset. Tell the user instead.
   - After the last card, show the **Send screen**. On `🚀 Push & send all`, first fetch and check that the remote head is still the PR head you started from. If it moved (e.g. a reviewer committed a suggestion), don't push: save the queue, show the new commits, and ask whether to rebase (then update every queued commit link to the new SHAs) or keep local. Then push to the PR's own remote and branch, never force: `git push <remote> HEAD:<headRefName>`. If the push is rejected, send nothing, save the queue (as Keep local does), and explain why.
   - Once the push succeeds, post each queued reply with its commit link (host file, **Commit link**), then resolve its thread. Mark each item as sent the moment it posts. If an error stops the loop, save the unsent items to the queue file and stop. Sending again later posts only those.
   - On `💾 Keep local`, save the queue to `$(git rev-parse --git-common-dir)/reb-review/<host>-<owner>-<repo>-<N>.json` (`mkdir -p` first). It lives inside git's own folder, so it's never committed, and every worktree shares it. The next Discussion session on this PR starts by offering to send it, after checking that each queued commit is still on the branch.
   - **At the end** (after Push & send, Keep local, Stop, or a rejected push): switch back to the recorded branch or commit, then `git stash pop` the remembered stash there, and say so on the Done card. If the pop conflicts, stop and show the conflict without resolving it yourself.
5. **Reviewer:** replies and resolves post as soon as they're picked.
6. **Same ask, several threads** (author): fix it once with one commit, then queue a reply in each thread linking that commit. **Contradicting asks:** mark both threads ⚔️ and show both on one card, so the reply can address both reviewers.
7. **Reviewer, at the end:** if the author pushed commits after the user's last review, offer `🔍 Review new commits`: a **Re-review** (`review-advanced.md`) of just those commits (host file, **New commits since me**), with a verdict on each of the user's earlier findings, then the **Finish menu** to update the verdict.
8. Show a **Done card**: replied, resolved, fixed, committed, pushed, skipped.

## Rules

- Nothing reaches the host until the user picks an action (Post, Reply, Accept, Agree, a verdict, or Push & send) on a card or screen that shows the exact text. Posting is public, and "undo" removes the last post where the host allows it.
- Every finding has `file:line`, checked against the real file. For a removed line, cite the old side: `<file>:<old line> (removed)`.
- Severity is one of: 🔴 Blocker, 🟠 Should fix, 🟡 Nit. Do not invent others.
- Posted and copied comments follow Conventional Comments (`issue (blocking): ...`, `nitpick (non-blocking): ...`). See **Comment labels** in `ui-kit.md`.
- Give fixes as `diff` blocks with the smallest change that works.
- Write replies in the user's voice: short and specific, with no "Great catch!" filler.
- You can't approve or request changes on your own PR/MR, unless the host allows it (GitLab can, per project; see the host file). Otherwise offer only 💬 Comment there.
