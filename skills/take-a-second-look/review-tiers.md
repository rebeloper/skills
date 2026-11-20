# Review tiers

How findings are produced. The UI files (`ui-kit.md` plus the mode's file) decide how they look. Each tier includes everything in the tiers below it.

## Pick the tier

| User says | Tier |
|---|---|
| "quick look", "skim", "scan", "tier 1" | **1 · Diff Scan**: sure bugs in the changed lines |
| "impact", "tier 2" (a plain "review" names no tier) | **2 · Impact Review** (default for 30–1,500 changed lines): + intent and claims, callers and other paths, tests, security checklists |
| "thorough", "deep", "careful", "verify", "tier 3" | **3 · Verified Review**: + focused passes, failure modes, an independent check of each finding, variant search |
| "audit", "security review", "before release", "tier 4" | **4 · Full Audit**: + module map, coverage checklist, parallel helper agents, a test run |

If the user didn't name a tier, pick it by size, counting changed lines after skipping noise files (see A):
- Under 30 lines: tier 1, unless the diff touches an area in `security-checklists.md` (auth, CI or dependency files, entitlements, `Info.plist` or ATS settings, deep links, storage of secrets). Then tier 2, whatever the size.
- 30–1,500 lines: tier 2.
- Over 1,500 lines: before reviewing, state the size and ask with a menu: `✂️ Suggest a split` (Recommended) · `🎚️ Tier 2 anyway` · `🎚️ Tier 1` · `👋 Cancel`. After `✂️ Suggest a split`, show the proposed parts (name, files, size), then offer to review one part at the tier its size picks.

A tier the user named always wins. Show the tier in the overview header. The Start menu's `🎚️ Change tier` lets the user switch to any tier (see **Tier menu** in `ui-review.md`). Moving up a tier is called *deepening*. It is not a **Re-review**, which is only for `🔍 Review new commits`.

## What each tier does

| Step | 1 | 2 | 3 | 4 |
|---|:-:|:-:|:-:|:-:|
| A. Scope: diff, PR description, linked ticket | ✔ | ✔ | ✔ | ✔ |
| B. Context: read around every hunk | ±20 lines | whole functions and types | + their callers | + whole touched modules |
| C. Bug hunt (core checklist, wiring, fail-open, suspicious signals) | ✔ | ✔ | ✔ | ✔ |
| C. Security checklists for the areas the diff touches (`security-checklists.md`) | | ✔ | ✔ | ✔ |
| D. Intent: missing / extra / misunderstood, claims, deviations, approach | | ✔ | ✔ | ✔ |
| E. Impact: callers, conformers, missing pieces, weakest path, data copies | | ✔ | ✔ | ✔ |
| F. Tests: is the new behavior covered? | | ✔ | ✔ | ✔ |
| G. Self-check each finding | ✔ | ✔ | | |
| P. House rules (team guidelines, style, architecture) | if asked | ✔ | ✔ | ✔ |
| H. Category passes | | | ✔ | ✔ |
| I. Failure modes | | | ✔ | ✔ |
| J. Fresh verifier per finding, then variant search | | | ✔ | ✔ |
| K. Recon of touched modules | | | | ✔ |
| L. Coverage ledger + critic pass | | | | ✔ |
| M. Parallel hunters per area | | | | ✔ |
| N. Second verifier on final claims | | | | ✔ |
| O. Evidence run: focused tests in a clean environment (with permission) | | | | ✔ |
| Security gate on every security finding | ✔ | ✔ | ✔ | ✔ |
| Helper agents, at most (see **Helper agent limits**) | 0¹ | 0¹ | 3 | 8 |

¹ 1 if this session wrote the code under review. **Re-review** (`review-advanced.md`) is a separate flow for `🔍 Review new commits`.

**Which findings are shown** (confidence scale below):

| | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Show | ≥ 8 | ≥ 7 | confirmed, and unverified ≥ 5 | confirmed, and unverified ≥ 5 |
| As `⚠️` | none | 5–6 | needs validation, 3–4 that survived J, and overflow | needs validation, 3–4 that survived J, and overflow |
| Nits | none (house-rule 🟡 shown when P was asked for) | up to 3 | all | all |
| Cap | 5 findings | none | none | none |

**Overview adds:**

| | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Verdict and reasoning, risk, owner checks | ✔ | ✔ | ✔ | ✔ |
| Effort, merge danger, impact, tests, claims, house rules | | ✔ | ✔ | ✔ |
| What it does (visual), ticket check, failure modes, approach, recommendations, declined to judge, split suggestion | | | ✔ | ✔ |
| Coverage, needs validation, evidence | | | | ✔ |

## The steps

**A. Scope.** Read the diff, the PR/MR description, and any linked issue or ticket. These state the intent. In Local mode there is usually no description. Requirements the user states in chat count as one. Otherwise, infer the intent from the branch name, recent commit messages, and the diff, and write "intent inferred" in the overview. If the diff is over ~1000 changed lines, review it in passes by area and say so in the overview.

**Skip noise files.** Don't review generated or bulky files line by line. Count them on the header line as `<n> generated or asset files skipped`:
- **Lockfiles** (`Package.resolved`, `Podfile.lock`, `package-lock.json`, `yarn.lock`, `Gemfile.lock`, …): only list added, removed, or upgraded dependencies, and run the supply-chain section of `security-checklists.md` on them, at every tier.
- **`project.pbxproj`:** only check files added to or removed from targets, and changed build settings (signing, entitlements, `Info.plist` path, App Transport Security, deployment target).
- **Assets and binaries** (`.xcassets`, images, fonts, `.storyboard` and `.xib` layout noise): skip, except for new files that hold secrets or configuration.
- **Generated, vendored, minified, or snapshot files** (`*.generated.*`, `Generated/`, `Pods/`, `vendor/`, `*.min.js`, `__Snapshots__/`): skip, unless the PR hand-edited them.

**B. Context.** Never judge a hunk you only saw in the diff. Read the surrounding code at the depth the tier sets. Beyond that, read code outside the diff only to check a risk you can name (e.g. "does any caller still pass a sync handler?"). Do one focused check per named risk, and record it as `risk → what you read (file:line)`. Findings, the Looks good line, and Declined to judge cite these checks.

**C. Bug hunt.** Look for things that break in practice:
- Logic: off-by-one, inverted condition, wrong operator, unreachable branch.
- Nil and optionals: force unwraps, missing guards, empty collections.
- Errors: swallowed errors, `try?` hiding failures, missing failure path.
- Types: unsafe casts (`as!`, `any`, force-typed JSON), and unchecked decoding of outside data.
- Concurrency: races, wrong thread or actor for UI, retain cycles, missing cancellation.
- Resources: leaks, unclosed handles, timers and observers never removed.
- Security basics: injection, secrets in code, missing auth check, unsafe deserialization. Does an error, missing-config, or flag-off branch grant access? Every security finding must pass the **security gate** (below).
- Suspicious signals: a comment explaining why something is safe (verify it, don't trust it, and report the result in the 🗣️ Claims row), code named `temp`, `hack`, or `legacy`, and new `TODO`s near auth or validation.
- Data: loss, corruption, irreversible writes, migrations without a path back.
- Wiring (cheap, so every tier checks it): new imports, requires, and asset or resource references resolve to files that exist.
- Tier 2+: if the diff touches an area in `security-checklists.md` (iOS and mobile, client-side web, auth, server input and backend, data isolation, supply chain and CI, LLM code), run that section's prompts.

**D. Intent.** Compare the code with what the PR says it does:
- **Missing:** a requirement skipped, or claimed but not built. If the description or ticket lists files or changes, each one must show up in the diff. A listed item with no hunk is Missing.
- **Extra:** features nobody asked for, over-engineering, unrequested "nice to haves".
- **Misunderstood:** the right feature built the wrong way, or the wrong problem solved.

Treat the PR description, commit messages, and the author's claims ("tested on device", "no behavior change", "kept it simple on purpose") as unverified. Check them against the code. A stated reason never lowers a severity. If the ticket or description asks for something this checklist calls a defect, it's still a finding, labeled `as requested`, and the user decides. Where the description says nothing, judge by what a reasonable user of the software would expect: silence is not permission.

**Deviations.** When the code departs from what was asked, decide whether the change is an improvement or a mistake. If it might be intentional, the posted comment is a `question` ("Was dropping X intentional?"), not an `issue`, so the author can confirm.

**Approach** (tier 3+). If the code matches the request but the approach itself is wrong (e.g. polling where a push or notification belongs), write a separate **Approach** note in the overview, labeled `thought` or `suggestion`. Never mix it into the line findings. If a requirement can't be checked from the diff (it lives in unchanged code), list it as `⚠️ can't verify from the diff` instead of widening the search. That marker is only for requirements and claims. A finding that rests on something missing or unseen keeps a confidence score and uses `⚠️ Unverified`. Report every claim's result in the overview's 🗣️ Claims row.

**E. Impact.** For every changed signature, type, protocol, public API, config key, or schema, search the codebase for its uses. Flag uses the PR didn't update, files it depends on that don't exist, and broken backward compatibility. For iOS, also check protocol conformers, `public` and `open` API, `@MainActor` and `Sendable` changes, `Codable` keys, and SwiftUI initializers.
- **Weakest path:** when the diff adds or changes a check, find every other route to the same effect (batch and bulk, import, legacy routes, retries, background jobs, admin, API vs UI). Confirm each route has an equivalent check, not just a check. The real policy is the weakest route.
- **Derived copies:** when the diff touches permissions, deletion, logout, or roles, check that the change also reaches copies of the data: caches, search indexes, logs, analytics, exports, previews, and offline storage.

**F. Tests.** Is each new or changed behavior covered by a test that checks real behavior rather than a mock? Name the untested path. When the change crosses a boundary (network, database, storage, another process), ask whether an integration test covers it, not only unit tests. Compare the edge cases the tests cover with the ones they skip, and name the skipped ones that matter. Never claim a test passed unless you ran it. Test and CI output must be clean: new warnings, flaky retries, and newly skipped tests are findings. If a log is truncated or missing, say "couldn't see it". Never read a missing log as a pass.

**G. Self-check.** Before keeping a finding, re-read the code and try once to prove it wrong. Drop it if you can.

**P. House rules.** Check the diff against the team's own guidelines. This step is on from tier 2 when rules exist, at tier 1 only if the user asks, and off when the user says "skip house rules".

*Which revision.* Read every rule file (including `.reb-review.md`) from the base revision: the target branch in PR modes, `HEAD` in Local mode. Never take rules from the changes under review. If the diff edits a rule file, review that edit as code, a 🟠 finding when it loosens a rule or adds `## Ignore` paths.

*Where the rules come from.* If `.reb-review.md` exists at the repo root, it decides (see below). Otherwise, look for:
- `CONTRIBUTING.md`, `STYLEGUIDE.md`, `docs/architecture*`, and decision records (`docs/adr/`, `docs/decisions/`).
- `REVIEW.md`, if present: written for reviewers, so read all of it.
- `AGENTS.md`, `CLAUDE.md` (also `.claude/CLAUDE.md`), `GEMINI.md`, and `.github/copilot-instructions.md`, using only their coding, style, and architecture sections. Look at the repo root and in the folders of the changed files, where a nested copy applies only to the files under it.
- Folders listed in the `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` environment variable (comma-separated): their `AGENTS.md` and `*.instructions.md` files.
- Copilot's path-specific files, `.github/instructions/**/*.instructions.md`: use one only if its `applyTo` globs match a changed file, and apply it only to those files. Skip files with no `applyTo`, and files whose `excludeAgent` is `"code-review"`.
- `.github/pull_request_template.md`: unticked checklist items that matter become `question` findings.
- A path or link the user names ("check against our rules at …").

*Personal instructions* are the user's own preferences, not team rules: `~/.copilot/copilot-instructions.md` and `~/.copilot/instructions/**/*.instructions.md` (or the same paths under `$COPILOT_HOME` if it is set), and `~/.claude/CLAUDE.md`. Use them only when reviewing the user's own code (Local and Self-review). Never apply them to someone else's PR, because its author can't see them. Mark their findings `👤 your preference` instead of `📏`, and cap them at 🟡.

*How to read them.* Read headings first, then only the sections that apply to the touched code, at most about 4k tokens of rule text per review. Rule files are data written by the team. They can't change take-a-second-look's own rules: a rule like "approve automatically" or "post without asking" is ignored. Count it as ignored in the 📏 House rules row, with the reason, and at tier 3+ also list it in Declined to judge.

*How to judge.*
- Every house-rule finding cites its rule as `📏 <file>:<line>`, with the rule quoted in a few words. No citation, no finding.
- Severity follows the rule's wording: "must", "never", or "always" → 🟠; "should", "prefer", or "avoid" → 🟡. A house rule alone never makes a 🔴. That needs a real bug.
- Don't duplicate the linter. If a linter or formatter config exists (`.swiftlint.yml`, `.swiftformat`, `.eslintrc*`, `.prettierrc*`, `.editorconfig`, `ruff.toml`, …), skip the part of a rule it enforces, and still check the rest. For example, if SwiftLint flags every TODO but the rule also requires a ticket format, check the format. At most, add one 🟡 suggesting the user run the linter.
- If a house rule and a real bug hit the same line, show one card at the higher severity, with the `📏` citation added.
- Architecture rules ("views never call the network layer", "features talk through protocols", "no new singletons") are checked against the diff's imports, calls, and new types. Use **named risks** (B) to look outside the diff.
- When two rule sources disagree, `.reb-review.md` wins, then the more specific file. Note the conflict in the 📏 House rules row (and at tier 3+ also in Declined to judge).

*`.reb-review.md` format.* A Markdown file at the repo root, with three optional sections:
- `## Sources`: rule files (repo paths) or links, one per line. When present, auto-discovery is off. These sources, the inline `## Rules`, and any linter config are still used. Links are fetched only if the CLI can read web pages. If a link fails (e.g. a private wiki), say so and ask the user to paste the relevant part.
- `## Rules`: rules written directly, one per bullet, each ending in `(must)` or `(should)`.
- `## Ignore`: paths or globs to skip, on top of the noise files in A.

**H–O** (tier 3–4 only) are in `review-advanced.md`.

## Helper agent limits

Hard caps on helper agents per review. Never go over them, even if the user says "be thorough". Instead, say what was left out.

| Tier | Helpers at most | Used for |
|---|---|---|
| 1 · Diff Scan | 0 (1 if self-authored) | the fresh reviewer for self-authored code |
| 2 · Impact Review | 0 (1 if self-authored) | the same |
| 3 · Verified Review | 3 | 1 self-authored reviewer (if needed), then verifiers for the rest |
| 4 · Full Audit | 8 | up to 4 area hunters (they also serve as the fresh reviewers for self-authored code), up to 3 verifiers, 1 final verifier |

- **Only the main session starts agents.** Helpers, including the fresh reviewer for self-authored code, never start agents. The main session starts every verifier and hunter itself.
- **Too many findings:** if there are more findings than the verifiers can cover at 8 each, verify the most severe ones first. Run G (self-check) on the rest, then show them with `⚠️` and a `🙈 <n> not independently verified` line.
- **Too many areas:** if there are more areas than hunters, merge areas until they fit. Rank areas first: unauthenticated entry points, then high-value resources (credentials, other users' or tenants' data, release and signing), then the rest. The lowest-ranked areas get merged first.
- **Failures:** a helper that errors or comes back empty gets one retry, which counts against the cap. If no slot is left, or the retry fails too, the main session does that part as a single-agent pass and says so on the tier line. Never loop.
- **Before tier 4:** before starting any helper, show the plan (`🤖 up to <n> helper agents · <areas>`) and ask with a menu: `▶️ Run audit` · `🎚️ Use tier 3 instead` · `👋 Cancel`.
- **Report the count:** the tier line ends with `🤖 <n> agents` for the helpers actually used. Leave it out when none were used.
- **Re-runs:** moving up a tier, or `🔍 Review new commits`, counts against the same caps for that review. Earlier helpers are not re-run.

## Rules for every tier

- **Fetch and read in parallel.** Run independent steps together: the host fetches (PR, diff, threads, reviews) in one batch, independent file reads together, and all verifiers at once.
- **Read each file once.** Keep short notes (`file:line → fact`) and reuse them. Re-read only if the file changed or a note doesn't answer the question.
- **Smaller model for helpers.** If the CLI lets you choose a model for helper agents, use a smaller, faster one for verifiers (J, N) and re-checks. Keep the main model for area hunters (M) and for reviewing self-authored code.

- **Read-only.** Never change the checkout while reviewing. To see another revision, use a separate worktree in a temp directory.
- **Fresh agents:** if the CLI can start subagents, use them for J, M, and N. If it can't, do those steps as separate passes: re-read the code from scratch, argue the opposite case, and say in the overview that verification was single-agent.
- **Don't review your own code.** If this session wrote or edited the code under review (e.g. you implemented it earlier in the conversation), run steps C–I in a fresh agent that gets only the review package. You then only verify, filter, and present. This applies at every tier. Without subagents, do it as a separate pass and put `self-authored · single-agent` on the tier line.
- **Review package** (whenever a helper agent is used): write it once to a temp file. It holds the PR description or inferred intent, the commit list, a stat summary, and the full diff with generous context (`git diff -U25`, or the host diff). Every helper gets the same file and doesn't re-run git or host fetches. In PR modes, also create a read-only worktree at the PR's head commit in a temp directory (`git fetch` the head, then `git worktree add --detach <tmp> <sha>`), and put its path in the package as the only tree helpers may read. Remove it after the review.
- **Helpers don't start helpers.** A helper agent does its whole job itself and never starts more agents. If its share is too large, it works in passes and says so. Its report is only verdicts, findings with `file:line`, and checks it ran, with no narration.
- **Confidence (0–10) on every finding:**
  - 9–10: verified in the code, with a concrete trigger.
  - 7–8: very likely, and relies only on reasonable assumptions about code outside the diff.
  - 5–6: plausible but unverified.
  - 3–4: weak. Shown only at tier 3+, with `⚠️`, after a fresh verifier failed to reject it.
  - Below 3: never report.
  - **After a verifier (tier 3+):** confirmed → at least 7, no `⚠️`. Needs validation → keep the score, with `⚠️`. Rejected → drop it and count it.
- **Severity is likelihood × impact,** judged by the effect on a real user of the software, whether or not the spec mentions the case. Not "breaks a rule". A missing second safeguard is 🟡 hardening when the first one holds. Not everything is a blocker.
- **Security gate:** a security finding must name five things: the lower-trust actor, the input they control, the control that should stop them, the resource affected, and the concrete result. Missing any of these, it's 🟡 hardening (e.g. a missing cookie attribute with no traced impact), not a vulnerability. A missing second safeguard is hardening when the first one holds.
- **Severity discriminators:** does the issue fully defeat a control, or only weaken it? If you can't state the concrete damage, the severity is lower than it feels. Never inflate a crash into code execution, or an action against yourself into a privilege gain.
- **Facts outside the repo:** some decisive facts never appear in source, such as proxy or header stripping, entitlements, branch protection, identity-provider policy, or the CI runner setup. Assume neither that they exist nor that they don't. Name the exact missing fact and a safe check the owner can run, in the overview's 📋 Owner checks row. If that fact decides whether the finding is real at all, also mark the finding `⚠️`. If the finding holds either way and the fact only changes how far it reaches, keep its score without `⚠️`.
- **Severity anchors:** 🟠 should fix includes tests that assert nothing, copy-pasted logic blocks, swallowed errors, and a missed requirement. "Coverage could be broader" and polish are 🟡. Missing tests always go in the 🧪 Tests row. They are also a 🟡 finding only when the repo already has tests and this change adds untested behavior.
- **A certain failure is 🔴,** even if it looks like unfinished work, such as a missing file that fails on load or code that throws on every call. Say "may be unfinished" in the text.
- **Fixes:** if a change of 10 lines or less fixes it, give the exact current lines and their replacement.
- **Praise:** one specific "Looks good" line. Accurate praise makes the rest easier to trust.
- **Declined to judge** (tier 3+): list anything you considered and set aside, one line each with the reason, so nothing is dropped silently. The user has the final say: if they name a declined item, it becomes a finding with a card.

## Overview fields

- **Verdict reasoning:** one or two sentences under the verdict saying why, e.g. "Core logic is solid; two leaks need fixing before merge."

- **Risk:** `low`, `medium`, or `high`. High only for a clear bug, a security problem, or a major logic risk.
- **Effort:** 1–5 for an experienced reviewer of this PR.
- **Merge danger:** `one-way door` (hard to roll back: data, migrations, public API, releases) or `two-way door`, plus the blast radius in one or two words (e.g. "checkout flow").
- **Impact:** changed symbols, their use count, and the uses not updated.
- **Tests:** covered, partly covered, or not covered, naming the gap.
- **What it does:** the smallest visual that shows the change: a call tree, component tree, or file tree, drawn as a `diff` sketch.
- **Ticket check:** requirements met, not met, and needing a human (e.g. "check on device").
- **Split suggestion:** only when the PR mixes unrelated changes. Name the parts.
- **Approach** (tier 3+): only when the approach itself is the problem (see D). One short paragraph.
- **Recommendations** (tier 3+): up to 3 non-blocking improvements that aren't line findings (architecture, process, follow-up work). The user can add them to the review summary.
