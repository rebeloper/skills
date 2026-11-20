# take-a-second-look UI: Discussion

Screens for Discussion mode. Shared parts (tokens, comment labels, menus, code view, copy line, Undo, Finish menu, Done card) are in `ui-kit.md`. Same rules: keep the order, headings, emoji, and separators, and only fill in the `<...>` slots.

## Inbox

```
## 💬 Discussion · <#n | !n> <PR title>
🧭 <Your PR · you're the author | Their PR · you're the reviewer> · <n> waiting on you <reviewer: · <n> new commits since your review>

| # | | Who | Where | Latest | State |
|---|---|---|---|---|---|
| 1 | 🔴 | `@<who>` | `<file>:<line>` | "<latest message, under 60 chars>" | <author: open | reviewer: 🔧 pushed a fix | 💬 answered | ↩️ pushed back> |
| 2 | 💬 | `@<who>` | `<file>:<line>` | "..." | ... |
| 3 | 💬 | `@<who>` | PR thread | "..." | ... |
```

Keep "Latest" under 60 chars, and start it with the label if one was used (e.g. `question: why not…`). The second column is 🔴 for `(blocking)` threads, 🕸️ for outdated ones, and 💬 otherwise. Order: blocking first, then the rest by age, then PR-level comments.

## Thread card

````
---

### 💬 <i>/<total> · `@<who>` on `<file>:<line>`
<💬 open | 🕸️ outdated> · <n> comment(s) · 🔗 `<commit the thread is on, 7 chars>`

<Code view of the comment's diffHunk, with the whole thread and the draft reply inside it, under the pinned line.>

<Reviewer, if the code at that line changed since your comment, add:>
**Since your comment** · `<old sha7>` → `<head sha7>`
```diff
<the same lines then vs now, in the code view format>
```

**Take:** <author: ✅ valid | 🤔 partly | ❌ disagree · reviewer: ✅ addressed | 🤔 partly | ❌ not addressed>: <one sentence on why. Mention the label if it changes the answer, e.g. "non-blocking nit, fine to decline".>

<Author, if the take is valid or partly, add:>
**Fix**
```diff
-<current line>
+<fixed line>
```
````

If the thread is outdated, say so on the status line, and diff the old hunk against the current file before taking a position.

**PR-level thread** (a conversation comment or review summary, with no file or line): the heading is `### 💬 <i>/<total> · `@<who>` on the PR`, the status line drops the commit, and the card shows the whole thread as a quoted box (same box format, with no code lines around it) instead of a code view.

**Code view without a diff hunk:** on GitLab and Bitbucket the host returns no hunk. Build it from the MR or PR diff around the comment's line (host file, **Read threads**).

## Thread action bar

A menu (see **Menus** in `ui-kit.md`). Each preview shows exactly what will happen (the reply text, or the fix diff plus the reply):

Author: `🔧 Fix + reply + resolve` · `💬 Reply` · `⏭️ Skip` · `⏹️ Stop here`
Reviewer: `✔️ Accept + resolve` · `💬 Reply` · `⏭️ Skip` · `⏹️ Stop here`

Copy is free text ("copy the reply"). For the author, `⏹️ Stop here` goes straight to the **Send screen** with what's queued so far.

Recommend Fix when the author's take is valid, and Accept when the reviewer's take is addressed. Otherwise recommend Reply. As a reviewer, ✅ addressed means the defect can no longer happen. "Attempted" doesn't count, and 🤔 partly is treated as not addressed: never recommend Accept for it. As a reviewer, Accept posts a short reply (e.g. "Looks good now, thanks.") only if the user keeps the draft. They can clear it to resolve silently.

Reviewer: act on the pick right away. Replies and resolves post immediately, followed by `✔️ Replied <file>:<line> · <url>`.
Author: a fix commits right away, followed by `✔️ Committed <sha7> · fix: <title>`. The reply is queued (`🕓 Queued reply · <file>:<line>`) and posts after the push, from the **Send screen**. Copy reply follows **Copy line** in `ui-kit.md`. With free text, the user can rewrite the reply (use their wording as given), say "resolve only", or say "undo".

## Re-review screen

Reviewer side, after `🔍 Review new commits` (method: **Re-review** in `review-advanced.md`):

```
## 🔍 Re-review · <#n | !n> · `<old sha7>` → `<head sha7>`
<n> new commits · <n> files · +<add> −<del>

| # | Your earlier finding | Verdict | Evidence |
|---|---|---|---|
| 1 | <title> | <✅ addressed | ❌ not addressed: <what is still reachable>> | `<file>:<line>` |

<New breakage from the fix, and new code found in these commits: one Finding card each (`ui-kit.md` code view, same severities), shown one at a time with the reviewer's Finding menu: 📤 Post · 📋 Copy · 🗑️ Drop · ⏹️ Stop here.>

<Out of scope (unchanged code only): one line each, non-blocking.>
```

Then the **Finish menu** (`ui-kit.md`) to update the verdict.

## Send screen

Author side of Discussion, after the last card:

```
## 🚀 Ready to push · <#n | !n>
<n> commits · <n> replies · <n> resolves · `<local branch>` → `<remote>/<PR head branch>`

| # | Commit | Thread | Reply |
|---|---|---|---|
| 1 | `a1b2c3d` fix: guard bad avatar URL | `ImageLoader.swift:22` · `@maria` | "Restored the guard; throws .badURL now." |
| 2 | (none) | `ProfileView.swift:8` · `@tom` | "question: would a subview help here? …" |
```

Below the table, show each reply in full as a blockquote, including its `Fixed in <sha7>` line, so the user sees the exact text before sending. The menu is `🚀 Push & send all` (Recommended) · `✏️ Edit queue` · `💾 Keep local`. `✏️ Edit queue` reopens any card the user names. Replies post as `<reply text>` followed by a blank line and `Fixed in [<sha7>](<commit link>)` for fix replies.
