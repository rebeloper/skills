# Review tiers: advanced steps

Read this file only at tier 3–4, or for a **Re-review**. Everything in `review-tiers.md` still applies, including the **Helper agent limits**.

## Steps H–O

**H. Category passes.** One focused pass per category that applies to the touched code: security (use the matching sections of `security-checklists.md`; deep specialist areas such as HTTP desync, memory safety, kernel, or cloud IAM get named in Declined to judge when touched, not hunted), performance, API and backward compatibility, error handling, maintainability (duplication, premature abstraction, YAGNI), structure (each file has one clear job; did this change create a very large new file or grow an existing one a lot? Judge only what this change added), and docs.

**I. Failure modes.** For each risky change: what fails, where (`file:function`), the realistic trigger, and what would detect it (a named test or signal, or "nothing"). Note whether this PR covers it. Always ask about the default posture: what happens when config is missing, a flag is off, a dependency is down, crypto or a certificate fails, or a migration is half done? A path that fails open is a finding.

**J. Fresh verifier.** Give each finding to a fresh agent that tries to disprove it. Pass the review package and the finding, never your reasoning. The verdict is one of:
- **confirmed:** the trace is complete.
- **needs validation:** state the exact unresolved fact.
- **rejected:** dropped, and counted on the tier line (`· <n> rejected`) at tier 3, or in the 🗺️ Coverage row at tier 4.

The agent that found a finding never verifies it. One fresh verifier may take up to 8 findings. Beyond that, batch them by area, at most 8 per verifier, within the caps in **Helper agent limits**.

**Variants.** After a strong finding is confirmed, search for the same root cause by name, by structure, and by logic: within the diff at tier 3, across the touched modules at tier 4. One root cause with many entry points is one finding listing them all. Separate missing controls are separate findings. Apply the same rule to remove duplicates in hunter reports (M) before verifying.

If a verifier confirms part of a finding and leaves the rest open, split it into two cards: the confirmed part, and a `⚠️` card for the open part.

At tier 3+, the verdict decides the marker: **needs validation** always shows `⚠️`, whatever the score. **Confirmed** never does.

**K. Recon.** Before hunting, map the touched modules: what they own, where outside input enters, and which trust boundaries the change crosses.

**L. Coverage ledger.** List every changed file against every category from H, and mark each cell checked, not applicable, or not done. A critic pass then looks for gaps. Fill the gaps before reporting.

**M. Parallel hunters.** Split the ledger by area and give each area to its own fresh agent, within the caps in **Helper agent limits**.

**N. Second verifier.** A different fresh agent re-checks every final `file:line` claim and quoted line against the files.

**O. Evidence run.** Build, then run the focused tests that answer a specific doubt from the review, and show the before and after. Don't run whole suites, race detectors, or repeat loops. Recommend those in the overview instead. Run in a clean environment: in the temp worktree, with a fresh home and only `PATH` passed through (`env -i PATH="$PATH" HOME=<tmp> …`), so no tokens or host logins reach the code. Block network where possible, and set time and memory limits. Running PR code is running someone else's code: ask first, and prefer a sandbox with no network. For a PR you didn't write, never run it without an explicit yes.

## Re-review

Used only for `🔍 Review new commits` in Discussion (moving up a tier is *deepening*, not a Re-review). It is not a fresh review of the whole PR:
1. **Each earlier finding gets a verdict**, in order: ✅ addressed or ❌ not addressed, with `file:line` evidence. "Attempted" is not addressed: the specific defect must no longer be reachable. A partial fix is ❌, with what is still reachable.
2. **New code:** every new hunk in the new commits gets a normal review at the current tier and its usual severities, whether it's a fix, breakage the fix caused, or unrelated new work.
3. **Out of scope:** issues noticed in code these commits didn't change are listed as non-blocking, never as a new finding round.
