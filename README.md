# rebeloper skills

Review your code and learn from it, with you typing the code.

Eight free agent skills. One reviews code and shows the result as cards, tables and diffs in the chat. Seven teach: they explain, quiz, mentor and coach, and you write the code yourself.

## Skills

| Skill | What it does | Start it with |
|---|---|---|
| `take-a-second-look` | Code review in the chat: local changes, a branch, or a PR/MR on GitHub, GitLab or Bitbucket. Also works through review comments. | "review my local changes", "review PR 231" |
| `explain` | Explains one concept at a depth you pick: `five`, `junior` (default), `non-dev`, `teammate`. | `/explain teammate <topic>` |
| `teach-me` | Turns a resource (repo, path, web page, chat, topic) into a paced, step-by-step lesson. | `/teach-me <url or topic>` |
| `mentor-me` | A long-haul learning workspace in plain markdown, across many sessions. | `/mentor-me <skill you want to build>` |
| `quiz-me` | A live multiple-choice quiz. Wrong picks get a hint, never the answer. | `/quiz-me <topic>` |
| `review-with-me` | Audits code with a review engine you pick, then leads you to each finding through questions. | `/review-with-me <pr, branch, path…> [--engine <name>]` |
| `now-i-do-it` | Saves your uncommitted change, reverts it, and paces you to rebuild it by hand. | `/now-i-do-it` |
| `wait-what` | That last message did not land. Re-pitch it. | `/wait-what` |

`take-a-second-look` starts on its own when you ask for a review. The other seven start only when you call them by name: `/explain` in most CLIs, or however your CLI invokes a skill.

### Who writes the fix

The learning skills never write your code. You type it.

`take-a-second-look` is different: it is a review tool, so in Local review and Self-review it applies fixes to your files (it never commits or pushes unless you ask). If you want to write the fixes yourself, run `/review-with-me` instead. It uses `take-a-second-look` as its default engine, changes no files, and leads you to each finding with questions so you write the fix.

## Install

```bash
npx skills add rebeloper/skills -g
```

This uses the open [skills](https://skills.sh) installer, which supports Claude Code, Codex, Cursor, and many more. It asks which agents to install for. Pick them up front with `-a`, e.g. `-a claude-code`. Leave out `-g` to install into the current project only. Add `-s <skill>` to install one skill on its own.

Update with `npx skills update -g`, and remove with `npx skills remove -g` (leave out `-g` for a project install).

<details>
<summary>Manual install</summary>

```bash
git clone https://github.com/rebeloper/skills.git rebeloper-skills
cd rebeloper-skills
mkdir -p ~/.claude/skills
for d in skills/*/; do ln -s "$PWD/${d%/}" ~/.claude/skills/; done
```

For GitHub Copilot CLI use `~/.copilot/skills`. For CLIs that read `~/.agents/skills`, use that. Pull the repo to update.

</details>

### Requirements

- `git`
- For `take-a-second-look` on a Git host:
  - **GitHub:** [`gh`](https://cli.github.com), logged in with `gh auth login`
  - **GitLab:** [`glab`](https://gitlab.com/gitlab-org/cli), logged in with `glab auth login` (GitLab 17.2+)
  - **Bitbucket Cloud:** `curl`, plus an Atlassian API token set in your shell as `BITBUCKET_EMAIL` and `BITBUCKET_API_TOKEN`, or saved as two lines (email, token) in `~/.config/reb-review/bitbucket`
  - `python3` or `jq`, to build JSON bodies safely (GitLab and Bitbucket)

## take-a-second-look

### Modes

`take-a-second-look` picks the mode from what you say and who wrote the PR. It shows its pick and the reason, so you can redirect it.

| Situation | Mode | A finding becomes |
|---|---|---|
| No PR mentioned | **Local review** | a fix applied to your files |
| Someone else's PR | **PR review** | a comment you post, copy, or drop |
| Your PR, nothing waiting on you | **Self-review** | a fix in your branch, nothing posted |
| Any PR with threads waiting on you | **Discussion** | a reply, plus a fix commit if it's your PR |

**Discussion** works from both sides:

- **Your PR:** it checks each comment against the code before agreeing or pushing back, and makes one commit per fix. It queues the replies, pushes at the end, then posts every reply with a link to its commit.
- **Their PR:** it checks whether the author's reply or new commits actually fix what you asked. It shows that line then and now, and offers a review of only the new commits.

### Review tiers

| Tier | Name | Adds |
|---|---|---|
| 1 | **Diff Scan** | sure bugs in the changed lines |
| 2 | **Impact Review** (default for most PRs) | intent vs the PR description (claims checked, not trusted), callers and other paths the PR didn't update, test coverage, security checklists for touched areas |
| 3 | **Verified Review** | focused passes (security, performance, compatibility…), failure modes, an independent check of each finding, variant search · up to 3 helper agents |
| 4 | **Full Audit** | module map, coverage checklist, parallel reviewers, focused tests in a clean environment (only with your OK) · up to 8 helper agents, asks first |

Without a tier named, small diffs (under 30 lines) get Diff Scan, most get Impact Review, and very large ones (over 1,500 lines) ask first. Generated files, lockfiles, assets, and `project.pbxproj` noise are skipped. Say "quick look", "thorough review", or "audit", or switch tiers from the start menu. Every finding carries a confidence score. Comments follow [Conventional Comments](https://conventionalcomments.org) (`issue (blocking):`, `nitpick (non-blocking):`, …).

### House rules

From tier 2, `take-a-second-look` also checks the diff against your team's own guidelines: style, architecture, and conventions. It finds them on its own in `CONTRIBUTING.md`, `STYLEGUIDE.md`, `REVIEW.md`, `docs/architecture*`, decision records, `AGENTS.md`/`CLAUDE.md`/`GEMINI.md`, Copilot's instruction files (`.github/copilot-instructions.md` and path-specific `.github/instructions/*.instructions.md`), and your PR template. When you review your own code, your personal instructions (`~/.copilot/copilot-instructions.md`, `~/.copilot/instructions/`, `~/.claude/CLAUDE.md`) count too, as 🟡 preferences. They're never applied to other people's PRs. Every finding cites the rule it breaks. "Must" rules become 🟠, "should" rules 🟡. Anything your linter already enforces is skipped.

To choose the sources yourself, add a `.reb-review.md` at the repo root:

```markdown
## Sources
- docs/ios-guidelines.md
- https://wiki.example.com/mobile/architecture

## Rules
- Views never call `APIClient` directly; go through a view model. (must)
- New async code uses async/await, not completion handlers. (should)
- No new singletons. (must)

## Ignore
- Sources/Generated/**
```

All three sections are optional. Say "skip house rules" to turn the check off for a review.

### Usage

```
review my local changes
quick look at my changes before I commit
review PR 231
thorough review of https://gitlab.com/acme/app/-/merge_requests/88
go through the comments on my PR 231
```

### Safety

- Nothing reaches your Git host until you pick an action (Post, Reply, Accept, Agree, a verdict, or Push & send) on a card or screen that shows the exact text. Type `undo` to remove your last post, where the host allows it.
- Reviews never change your checkout. Fixes stay local unless you ask to commit, or, in Discussion mode on your own PR, until you choose **Push & send**.
- It never force-pushes. It runs someone else's PR code only with your explicit OK.

## Moving from okay or second-look

Both projects moved here. Uninstall the old ones so they don't run beside the new skills:

- **okay plugin:** `/plugin uninstall okay@okay`, then `/plugin marketplace remove okay`.
- **second-look skill:** `npx skills remove -g second-look`.

Then rename these, if you have them:

- `.second-look.md` → `.reb-review.md` (repo root)
- `~/.config/second-look/bitbucket` → `~/.config/reb-review/bitbucket`
- `.okay/` → `.reb/` (`now-i-do-it` session files), and the matching `.gitignore` line

## License

[MIT](LICENSE)
