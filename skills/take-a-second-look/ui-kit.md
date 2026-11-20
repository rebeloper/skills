# take-a-second-look UI kit

Every screen is one of these templates. Keep the order, headings, emoji, and separators exactly as shown and only fill in the slots marked `<...>`. Chat renders GitHub-flavored markdown, so headings, tables, blockquotes, and `diff` blocks are the visual system. The fences below show the raw markdown. Print each template as plain markdown, without the outer fence, so it renders.

**Tokens**

| Token | Use |
|---|---|
| 🔴 Blocker · 🟠 Should fix · 🟡 Nit | Finding severity |
| ⛔ Not ready · ⚠️ Fix first · ✅ Ready | Verdict |
| 💬 open · ✔️ resolved · 🕸️ outdated | Thread state |
| ✅ valid · 🤔 partly · ❌ disagree | Your take on a comment (author) |
| ✅ addressed · 🤔 partly · ❌ not addressed | Your take on a reply or fix (reviewer) |
| `---` | Separates every card |

This file holds the parts every mode uses. Reviews (Local, PR review, Self-review) also read `ui-review.md`. Discussion also reads `ui-discussion.md`.

## Comment labels (Conventional Comments)

Every comment and reply that gets posted or copied starts with a label line in the Conventional Comments format:

```
**<label>[ (<decoration>)]:** <subject>

<discussion>
```

| Finding | Posted label |
|---|---|
| 🔴 Blocker | `issue (blocking)`, or `todo (blocking)` for a small required chore |
| 🟠 Should fix | `issue (non-blocking)` for a defect, `suggestion (non-blocking)` for an improvement |
| 🟡 Nit | `nitpick (non-blocking)` |
| Something you need answered, or a deviation that may be intentional | `question` |
| Something done well | `praise`. Post at most one, and only if it's specific. |

Other labels can be used when they fit: `thought` for an idea that isn't a request, `chore` for process work such as a changelog entry, and `note` for information with no action needed. A small fix can take the `(if-minor)` decoration, meaning "fix it only if it's quick".

A house-rule comment ends with a link to the rule: `Rule: <rule file>#L<line>` (or the `.reb-review.md` rule text). The emoji severity is for display in chat only. Posted text starts with the label line, not an emoji.

Replies use the same format. If the user's reply contains a question back to the reviewer, label it `question`. Otherwise, a plain reply needs no label.

In Discussion mode as the author, read the reviewer's label: `nitpick` and `(non-blocking)` mean the user may decline. `question` needs an answer, not a code change. `(blocking)` must be fixed or argued against before resolving.

## Menus

A menu is one single-choice question. Label each option with an emoji and a verb. Put the recommended option first and mark it "(Recommended)". Print `---` before the Start menu. Finding cards already open with one.

**Use the CLI's own question tool.** Most agent CLIs have a tool for asking the user a multiple-choice question, under different names. If one is available, use it for every menu. Fill in whatever it supports: a short title (`Start`, `<i>/<total>`, `Finish`), a description for each option, and a preview of what the option will do. Leave out what it doesn't support. If the tool limits the number of options, the menus here already fit within 4.

**If there is no such tool,** print a numbered list and end your turn. Accept the number, the label, or free text, and never move on without an answer:

```
**<title> · <question>**
1. 📤 Post (Recommended) · <preview, one line>
2. 📋 Copy · <preview, one line>
3. 🗑️ Drop
4. ⏹️ Stop here · Skips findings <i>–<total>
```

In the numbered form, a preview becomes one line after ` · `. Leave it out when the card above already shows the full text.

Free text always works: reword a comment, ask a question, or type "undo". Answer, then show the same menu again.

## Code view

Code is shown the way a host's diff view shows it, with line numbers and inline pinned comments. The fence is ```` ```diff ```` so the terminal colors `+` lines green and `-` lines red. The color comes from each line's first character, so the sign always comes first.

````
```diff
@@ -38,9 +38,15 @@ final class ProfileViewModel
    38   38 ┃     func load() {
-   39      ┃         api.fetchProfile(id: userID) { result in
+        39 ┃         isLoading = true
+        40 ┃         api.fetchProfile(id: userID) { result in
+        41 ┃             self.timer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { _ in
            ┃  ╭─ 🔴 @you · L41 · new side · Blocker · inline · a1b2c3d
            ┃  │ issue (blocking): Retain cycle in the refresh timer
            ┃  │
            ┃  │ The timer's closure captures self strongly.
            ┃  │
            ┃  │ Suggested change
            ┃  │ -            self.timer = ... { _ in
            ┃  │ +            self.timer = ... { [weak self] _ in
            ┃  ╰─
+        42 ┃                 self.refresh()
```
````

- **Columns:** each line is the sign (`+`, `-`, or a space), a space, the old line number right-aligned in 4 characters, a space, the new line number right-aligned in 4, then ` ┃ ` (heavy bar) and the code exactly as written. Removed lines leave the new number blank, and added lines leave the old number blank.
- **Hunk header:** the first line is the raw `@@ ... @@ <enclosing type or func>` header, starting in column 0.
- **Range:** show 5-15 lines around the finding. Keep every `+`/`-` line near it. Where you cut far context, add a trim row: `     ⋯    ⋯ ┃`, with `⋯` in both number columns. A blank code line ends at `┃`, with no trailing space.
- **Pinned comments:** a comment box goes directly under the line it is pinned to. For a range, that is the last line of the range, as all three hosts do. A comment on the old side goes under the `-` line.
- **Comment box lines:** these start with 12 spaces and then `┃  `, so the `┃` lines up with the code column and the box stays uncolored. The box has a left border only: `╭─` header, `│ ` body, `╰─` close. Never add a right border, because emoji widths would misalign it.
  - The header line is `<severity> @<who> · L<start>[–<end>] · <new | old> side · <severity word> · <inline> · <sha7>`. In Thread cards, use `@<who> · <relative time>` instead.
  - Wrap body text at 72 characters. Markdown does not render inside the fence, so write plain text with no `**` or backticks.
  - A suggestion is the line `Suggested change`, followed by `-` and `+` lines.
- **Usernames:** outside a fence, write every username as inline code, `` `@<name>` ``, so it shows in the theme's code color. Inside a fence, write it as plain `@<name>`.
- **Existing threads:** in every code view, show open threads by others pinned to their lines, in the same box format, so nobody repeats a point already made. Collapse a long thread to its first and last comment plus `│ … <n> more`.
- **Threads:** put every comment in one box, oldest first. Separate replies with `├─ @<who> · <time>`. The draft reply goes last, after `├┄ ✏️ draft · @<you> · <✔️ will resolve | stays open>`, and its body uses `┆ ` instead of `│ `.

## Copy line

Copy the exact text that would have been posted, including any suggestion in the host's format, then print:

```
📋 Copied · paste at `<file>` line <L>[–<end>] · <new | old> side
🔗 <link to that line in the PR>
```

- Clipboard command, picked by OS: macOS `pbcopy`, Windows `clip.exe`, Linux `wl-copy`, or `xclip -selection clipboard` if that's missing. Write the text to a temp file first and pipe the file in (`pbcopy < file`), so the shell never interprets it.
- If no clipboard command works, print the text in a fenced block for manual copying and say so.
- Link: from the host file's **Line link** section.
- In Discussion mode, copy the reply and link to the thread's comment URL instead.

## Undo and Finish menu

**Undo:** undo only reaches back one action at a time.
- After 📤 Post: delete that comment (host file). If the action also resolved a thread, unresolve it. Confirm with `↩️ Removed <file>:<line>`.
- After 🔧 Fix (Local, Self-review): reverse exactly the lines that fix changed, and leave every other line in the file alone. Never run `git checkout`, `git restore`, or `git stash` on the file, because it holds the user's own uncommitted work. Confirm with `↩️ Reverted <file>:<line>`.
- After 💬 Reply or ✔️ Accept (Discussion, reviewer side): delete the reply (host file), and unresolve the thread if the action resolved it.
- After a verdict: use the host file's verdict undo. Where the host can't undo it (a GitHub review), say so and offer a follow-up comment.
- For the Discussion author flow, see `SKILL.md`.

**Finish menu** (PR review: after the last card, or right after the overview when there are no findings. Discussion as reviewer: after a Re-review): `⛔ Request changes` · `💬 Comment` · `✅ Approve` · `👋 Leave as is`. Recommend from the findings, not from what was posted: `Request changes` if any 🔴 is still open, `Approve` only if there are no 🔴 or 🟠, otherwise `Comment`. Before the menu, print the review summary that will post with it, in full, as a blockquote under `**Review summary**`, so the user sees the exact text. This submits the overall verdict. The inline comments are already posted.

## Done card

If nothing was fixed, committed, posted, copied, replied, or resolved, skip the card and print one line: `👋 Nothing changed.`

```
---
## ✅ Done
<🔧 n fixed> · <📦 n committed> · <🚀 pushed> · <📤 n posted> · <📋 n copied> · <💬 n replied> · <✔️ n resolved> · <⏭️ n skipped>
<Leave out any count that is 0 or doesn't apply to the mode.>
<URL(s) if posted>
<Notes, one per line, when they apply: fixes merged or dropped by Fix all, fixes skipped for missing facts, `📦 stash restored on <branch>`, `💾 queue saved`.>

<Ending: Local → "Changes are uncommitted: `git diff` to see them." · Self-review → "Changes are uncommitted. Commit and push when ready." · Discussion author → the push and send result.>
```
