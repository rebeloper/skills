# take-a-second-look UI: reviews

Screens for Local, PR review, and Self-review. Shared parts (tokens, comment labels, menus, code view, copy line, Done card) are in `ui-kit.md`. Same rules: keep the order, headings, emoji, and separators, and only fill in the `<...>` slots.

## PR header

```
## 🔀 <PR title> · <#n on GitHub/Bitbucket, !n on GitLab>
🧭 <Their PR | Your PR> · <reason> → <PR review | Self-review | Discussion>
`<head>` → `<base>` · `@<author>` · <files> files · +<add> −<del> · <checks: ✅ passing | ❌ failing | ⏳ pending> <· <n> generated or asset files skipped>

> <one-line summary of what the PR does, in your words>
```

## Overview

Local mode replaces the PR header with this header line:

```
## 🔍 Local changes · `<branch>`
🧭 <reason> → Local
<n> file(s) · +<add> −<del> · <uncommitted | staged | vs `<base>`> <· intent inferred, when there's no description> <· <n> generated or asset files skipped>
```

Then:

````
> <⛔ **Not ready** | ⚠️ **Fix first** | ✅ **Ready**>: <n> blockers · <n> should fix · <n> nits
> <1–2 sentences: why this verdict>

🎚️ Tier <n> · <Diff Scan | Impact Review | Verified Review | Full Audit> · 🌡️ Risk <low | medium | high> <tier 2+: · ⏱️ Effort <1-5>/5> <· self-authored · single-agent, when that applies> <· 🤖 <n> agents, when any were used>

<tier 3+: **What it does**
```diff
<smallest call / component / file tree sketch of the change>
```>

| | |
|---|---|
| 🚪 Merge danger | <one-way | two-way> door · blast radius: <1-2 words> |
| 🧭 Impact | <new symbols: `<symbol>` new · <n> uses. Symbols left unused by this change: `<symbol>` · now unused. Changed symbols:> `<symbol>` <change> · <n> uses · <⚠️ n not updated | ✅ all updated | ⚠️ exported, outside callers unknown (when 0 uses in the repo)> |
| 🧪 Tests | <✅ covered | 🟡 partly: <gap> | ❌ not covered: <gap>> |
| 🗣️ Claims | <tier 2+, when the description, the user, or a code comment claims something: each claim → ✅ holds · ❌ contradicted by `<file>:<line>` · ⚠️ can't verify from the repo (e.g. manual testing)> |
| 👥 Reviewers | <PR modes, when others reviewed: `@a` ⛔ changes requested · `@b` ✅ approved · `@c` 💬 commented · <n> open threads> |
| 📏 House rules | <when step P ran: `<source>`, `<source>` <· 👤 personal instructions, in Local and Self-review> · <n> followed · <n> broken · <n> skipped (linter covers) · <n> ignored (<why>)> |
| 📋 Owner checks | <when a finding depends on a fact outside the repo: the fact → a safe check the owner can run> |
| 🎫 Ticket | <tier 3+, when a ticket, PR description, or requirements the user stated in chat exist: n met · n met but broken · n missing · n need a human> |
| 🗺️ Coverage | <tier 4: n files × n categories checked · n rejected · n need validation> |

| # | | Where | Issue | Conf. |
|---|---|---|---|---|
| 1 | 🔴 | `<file>:<line>` | <issue, under 60 chars> | 9 |
| 2 | 🟠 | `<file>:<line>` | <issue> | 6 ⚠️ |

<if any were hidden: 🙈 <n> more below the tier's confidence bar · a higher tier (`🎚️ Change tier`) shows them>
<if verifiers couldn't cover all findings: 🙈 <n> not independently verified (marked ⚠️)>

**Looks good:** <one specific line on what is solid>
````

At tier 1, the field table keeps only the 📋 Owner checks and 📏 House rules rows, and only when they have content. At every tier, leave out any row that doesn't apply. At tier 3+, add after the table: **Failure modes** (a table: what · where · trigger · detected by), **Approach** when it applies, **Recommendations** (up to 3 bullets), **Declined to judge** (one line each, ending with `Name one to make it a finding.`), and **Split suggestion** when it applies. At tier 4, also add **Needs validation** (finding · the exact open question) and **Evidence** (what was run and the result). When verification was single-agent, say so on the tier line.

Number findings by severity: all 🔴 first, then 🟠, then 🟡. Within a severity, order by kind: security, then data loss, then crash or always-fails, then wrong result or a breaking API change, then a missed requirement, then everything else. Break ties by the earlier failure point (a crash on load comes before a crash on call), then by the larger blast radius. Mark unverified findings with ⚠️ in the Conf. column only, never in the Issue text. The overview has no finding details. Those come one at a time in the walkthrough.

Pluralize counts correctly ("1 file", "2 files"). Untracked files count as changed files, and all their lines count as additions.

Verdict rule: any 🔴 → ⛔ Not ready. Only 🟠 → ⚠️ Fix first. Only 🟡 or nothing → ✅ Ready.

If there are no findings, print only the header and `> ✅ **Ready**: nothing found`, followed by one line on what you checked. Skip the walkthrough. In PR review, go straight to the **Finish menu**.

## Start and tier menus

**Start menu** (after the overview). Title `Start`, question `How do you want to go through the <n> findings?`. `▶️ Walk through` is the recommended option (preview: the first card's title). `👋 Done` ends without changes. The `🔧 Fix all now` preview says how many fixes, which files, and which fixes will first ask for facts:
Local, Self-review: `▶️ Walk through` · `🔧 Fix all now` · `🎚️ Change tier` · `👋 Done`
PR review: `▶️ Walk through` · `🔒 Keep private` · `🎚️ Change tier` · `👋 Done`

`🔒 Keep private` walks through the cards with `📋 Copy` · `🗑️ Drop` · `⏹️ Stop here` only. Nothing is posted, and there is no Finish menu. `👋 Done` ends without posting anything.

The `🎚️ Change tier` preview shows the current tier. Picking it opens the **Tier menu**.

**Tier menu:** all four tiers as options, in order. Mark the current one "(current)" and recommend none. Each option's description says what the tier adds and its rough cost:

```
**Tier · Which depth?**
1. 🎚️ 1 · Diff Scan · sure bugs in the changed lines · ~1 min
2. 🎚️ 2 · Impact Review (current) · + intent and claims, callers and other paths, tests, security checklists · a few min
3. 🎚️ 3 · Verified Review · + focused passes, failure modes, an independent check of each finding, variant search · longer, up to 3 agents
4. 🎚️ 4 · Full Audit · + module map, coverage checklist, parallel helper agents, a test run · longest, most expensive, up to 8 agents, asks first
```

- **Higher tier:** re-review, keeping the findings already made. Only the steps the new tier adds run again.
- **Lower tier:** don't re-review. Re-filter the existing findings with that tier's confidence bar, nit rule, and cap.
- **Current tier:** close the menu.

In all three cases, print the new overview and the Start menu again.

`Fix all now` skips the walkthrough. Apply fixes in dependency order. If one fix makes another moot or changes it, merge or drop that one and say so on the Done card. If any fix needs facts only the user has (e.g. where a missing module lives), ask for all of them in one question first. Fixes the user can't answer for are skipped and listed on the Done card.

## Finding card

Show one card at a time, then the finding menu. Wait for the answer before showing the next card.

````
---

### 🔴 <i>/<total> · <short title>
`<file>:<line>` · <added | changed | removed> in this <PR | change> · confidence <n>/10 <tier 3+: · <✅ verified | ⚠️ needs validation: <open question>>> <· 👥 already raised by `@<who>`, when an open thread covers it>

<Code view of the hunk. In PR review mode, the comment box sits inside it, under the pinned line.>

<1-3 sentences: what breaks, for which input, and why.>
<House-rule finding: add `📏 <rule file>:<line> · "<rule, in a few words>"`.>
<If the finding or its fix rests on something you could not check, end with `⚠️ Unverified: <assumption>` (e.g. the DB driver's placeholder syntax).>

<Local and Self-review:>
**Fix**
```diff
-<current line>
+<fixed line>
```

<PR review mode: if the finding is outside every diff hunk, it can't be pinned. Mark it 📝 PR-level and show the comment after the code view as a blockquote with the same metadata line. 📤 Post posts it right away as a PR-level comment (host file, conversation comment).>
````

The box shows a suggestion as `Suggested change`. The posted body uses the host's suggestion format (see the host file). Include a suggestion only when the fix replaces exactly the pinned lines. Otherwise describe the fix in the comment text.

If no local diff fits (a missing file, or a change needed in callers elsewhere), write `**Fix:** <one sentence>` instead of the Fix diff. The `🔧 Fix` preview then shows that sentence. If the fix needs facts only the user has, such as where a module lives, picking Fix asks for them first.

**Finding menu.** Each option's preview shows exactly what will happen:
Local: `🔧 Fix` (preview: the diff) · `⏭️ Skip` · `⏹️ Stop here`
PR review: `📤 Post` (preview: the exact comment, with the suggestion as it will post) · `📋 Copy` · `🗑️ Drop` · `⏹️ Stop here`
PR review, already raised (👥): `👍 Agree in their thread` (preview: the short reply, e.g. "+1, also seeing this") · `📋 Copy` · `🗑️ Drop` · `⏹️ Stop here`
Self-review: same as Local.

Recommend Fix or Post for 🔴 and 🟠. Don't recommend anything for 🟡. The `⏹️ Stop here` preview reads `Skips findings <i>–<total>`.

Act as soon as the user picks, confirm with one line, then show the next card:
- 🔧 Fix → apply it, then `✔️ Fixed <file>:<line>`
- 📤 Post → post this one comment now (host file), then `✔️ Posted <file>:<line> · <comment url>`
- 📋 Copy → see **Copy line** in `ui-kit.md`.
- `Stop here` ends the walkthrough. The current finding and the rest count as skipped.

**Undo** and the **Finish menu** are in `ui-kit.md`.
