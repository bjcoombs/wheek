# Deliberately not adopted

Practices from the AI-Native SDLC Playbook and from the maintainer's enterprise conventions that wheek deliberately does not adopt. The playbook is written for engineering leaders in large regulated organisations, and most of its cost is coordination cost. This team has one human and no coordination problem. Skipping these is the design, not a shortfall.

**The operative residue.** Everything a cold session needs from this file is six lines, and all six are already stated in `CLAUDE.md` at the rule they govern: Task Master is off; there is no CI, no hooks and no branch protection; merging stays human; `/6hats` is shadowed by a user-level duplicate and behaves unpredictably until that is removed; the CI-fixture evaluation and the three deny-only `PreToolUse` gates are issues to open, not designs to park in a norms file. The rest of this file is the record of what was considered and declined, for a future session tempted to re-add it.

- **The committed artifact chain (`intent.md`, `spec.md`, `plan.md` as files).** Adopted in substance, rejected in form. The chain stops intent being distorted as it passes between an originator, a product owner, an analyst and an engineer; here they are one person. Intent and spec live in one GitHub issue (via `docs/intent-template.md`), the plan is a comment on that issue, the diff and evidence live in the PR. Three surfaces, all version-controlled and already linked.

- **`REVIEW.md` as a separate review policy file.** Justified when many reviewers must apply identical depth. One reviewer running `/code-review` on every PR gets that for free, and a second policy file read only at review time goes stale. The policy is the evidence gate and the autonomy ladder in `CLAUDE.md`, read at the start of every session.

- **Continuous evals in CI (20-50 recorded tasks, daily runs, gating config changes on pass rate).** Guards against silent agent-quality regression — a real risk, but not wheek's, and a 20-50 case suite would exceed the size of the codebase. The held-out test set is this project's eval suite.

- **Hooks as approval gates, today.** The one genuinely deterministic control in the playbook, and it is not in place: zero hooks are configured on this machine, and the global CLAUDE.md's "integrated hooks system" claim is unbacked. `CLAUDE.md` says plainly that the sacred-directory and data rules are discipline, which is weaker, and names the three gates worth one issue. That claim in `~/.claude/CLAUDE.md` is also false for every other project and should be fixed there.

- **Managed settings pushed via MDM.** Assumes an admin pushing policy to engineers who might switch it off. There is one engineer and he wrote the policy.

- **Per-environment permission tiers, scoped short-lived tokens, allowlisted MCP deploy capabilities, a named release manager.** There is no production; the deployment target is a Raspberry Pi with a USB microphone next to a hutch. The principle worth keeping — the agent may act up to the gate and cannot pass it — survives as the Ben-only tier.

- **Branch protection requiring an approving review.** Not enabled, deliberately: a required-approval rule on a solo repo blocks the only person who can approve, and the workarounds (self-approval, admin bypass) are worse than not having it. The same end is reached socially — `main` changes only through a squash-merged PR, and a human merges.

- **Task Master, `/tm`, and the epic-tag pattern.** Installed on this machine but not configured for wheek: no `.taskmaster/config.json`, no models, no keys, so every AI-powered command fails today, and enabling it means API cost beyond the Claude Code subscription. The epic-tag-to-many-PRs pattern solves decomposition of large multi-stream programmes, which a repo with two commits does not have. Ruled out at the rule in `CLAUDE.md` so a future session does not half-enable it. The empty `.taskmaster/tasks/` directory left in `wheek-main` by a tooling probe has been removed.

- **The `/issues` marathon and autonomous merge.** Needs three labels that do not exist, Agent Teams enabled, and CI for its merge criteria to mean anything. More importantly, auto-merging on green CI is the wrong default for changes whose correctness is a held-out number a person should read.

- **`/fix-develop`, the CI half of `/fix-pr`, and `@claude` PR-comment fix loops.** No `.github/workflows/` exists and the GitHub app that would service comment loops is not verified installed. Local `/code-review` before merge covers the same ground with tooling confirmed present.

- **`/marathon` and `/pr-review-merge` as commands.** They do not exist in that form — both are library skills requiring a caller-supplied adapter, as are the internal `assess-findings` and `assess-pr` steps. Telling a future agent to type a command that does not resolve wastes a turn and erodes trust in the rest of the file.

- **Stage 6 control bands (1σ log, 2σ diagnose, 3σ propose) and agent-authored intent from a breach.** Sigma needs a stable baseline over many samples; wheek will have a handful of model cards and a daily event count from one microphone. Three tiers over that is false precision, the exact failure this project exists to avoid. An intent artefact with no human author at any point is what the honesty mandate rules out. What survives is the staleness rule: an artefact older than the current model is not a current result.

- **`/loop` and `/schedule` for automated retraining or watchdogs.** Both available, both deliberately unused. Scheduled retraining with nobody reading the numbers is how a stale or cheating model quietly becomes the published one.

- **A metrics dashboard, cycle-time measurement, or a per-stage metrics layer.** Correct criticism of the playbook, wrong medicine here. Cycle time for a team of one measures nothing. The only measurement wheek needs is per-class precision and recall on held-out data, and that already gates every model PR.

- **A dedicated verifier subagent.** Sensible at scale; here `/code-review` on the diff plus the evidence block covers the same ground, and one more agent role is one more thing to keep current.

- **Skills as institutional knowledge (`.claude/skills/<name>/`).** The playbook's case is consistent policy application across many teams. One repo, one maintainer, and the knowledge fits in `CLAUDE.md`. If a wheek-specific skill is ever warranted the obvious candidate is a launch skill for `/run`.

- **Escalation to named policy owners; brand, security, compliance and UX skills as spec-time constraints.** There are no policy owners. The one constraint that genuinely applies at design time — does this change what the project can honestly claim, and on what evidence — is a line in the intent template instead.

- **Parallel sessions at scale.** Worktree isolation is kept in full; it genuinely stops sessions colliding on files. The picture of one engineer steering many simultaneous sessions is not adopted: two worktrees at once is comfortable here, and beyond that steering costs more than it saves.

- **A GitHub project board.** The DIP board in the maintainer's global conventions is Senapt-specific down to its field values, and an empty board on a public hobby repo is a maintenance task with no reader. A handful of open issues is the tracker.

- **Six Hats for implementation work.** Reserved for decision analysis, per the existing convention. The user-level `~/.claude/commands/6hats.md` shadows the plugin version and references a "Purple Hat" the plugin lacks; that duplicate should be removed before anyone relies on `/6hats` behaving predictably.

- **The enterprise register and stack rules that do not transfer.** Dropped wholesale: the SQL-first principle and Flyway migration immutability (no database, and none until one is earned), MCP database access, Kubernetes safety protocols and `stern`/`k9s`, Jira and `acli`, Maven build commands, and Java/Log4j conventions. The professional tone is kept — it is a public repo and precision about what a model proves is the product — but impact here is per-class precision, recall and clip counts on three brothers, not customers affected.

- **Gates that cannot fail.** Any step whose only outcome is Ben agreeing with Ben was removed, including the draft's end-of-flow retro ritual; its three triggers survive without the ceremony. The plan gate is real because an agent writes the plan and a human approves it. A gate that cannot fail teaches that gates are decorative.

**Deliberately left open, to become issues rather than rules.** A read budget for the held-out test set across many PRs; a Stage 1 analogue of the background-only test; whether a logged confidence figure is calibrated and what an abstain threshold would be; what a behavioural claim over the event log must state (n, effect size, null); recording the model version on every event-log row; a labelling protocol for clips the labeller cannot classify; a minimum n per class per pig before a per-pig figure is published; and a backup rule for the audio, which exists on one machine and is irreplaceable. Each is a real hole the critics found. None is answered by inventing a rule nobody has tested.
