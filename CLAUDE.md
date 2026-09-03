# Wheek

Guinea pig vocalisation classifier and behaviour logger.

## North star

Give a household with guinea pigs honest, timestamped evidence of what their pigs
are saying, to whom, and when — using published ethology for meaning and machine
learning only for detection.

Three questions the system exists to answer:

1. **What call was that?** Classify wheek, chut, rumble, chatter, purr, other.
2. **Who made it?** Identify which individual pig called, within a call type.
3. **What does the pattern mean?** Detect contagion (one wheek triggering others)
   and separation calling (call rate when pigs are apart vs together).

## Honest framing (non-negotiable)

This is **not** a translator. It is a classifier plus a behaviour dictionary.

- The model contributes reliable, timestamped evidence: which call, from which
  pig, at what time, in what sequence.
- The sound-to-meaning mapping comes from published ethology, not from the model.
- Every user-facing output states its confidence. No output claims to decode intent.
- If accuracy cannot be measured on held-out data, it is not reported as a result.

Language rule: say "classified as a wheek (0.87 confidence)", never "she said
'feed me'". Meanings are shown as a documented ethogram lookup, clearly labelled
as such.

## Domain vocabulary

Use the correct ethological terms throughout code, labels, and docs.

| Call | Description | Documented meaning |
|---|---|---|
| `wheek` | Loud rising squeal | Food/attention begging; contact call |
| `chut` | Soft repeated tutting | Contented pottering |
| `rumble` | Low vibrating growl | Dominance, strutting, courtship |
| `chatter` | Rapid teeth chattering | Annoyance, warning |
| `purr` | Low continuous vibration | Contentment or unease, context-dependent |
| `other` | Anything else, incl. background | — |

Label directories and CSV values use exactly these lowercase keys.

## Architecture

Hierarchical, two stage. Stage 2 runs per call type so "one pig wheeks more"
cannot leak into individual ID.

```
mic → 1s chunks → YAMNet embedding (1024-d)
                    ├─ Stage 1: call-type classifier  → wheek / chut / ...
                    └─ Stage 2: per-call-type pig ID  → pig A / B / C
                                              ↓
                                     CSV event log → analysis
```

- **Features**: YAMNet embeddings (transfer learning; no training from scratch).
- **Classifiers**: start with logistic regression. Reach for anything heavier only
  when a held-out score says the simple model is the bottleneck.
- **Deployment**: Raspberry Pi with USB mic by the cage, or a laptop left running.
- **Storage**: append-only CSV. Timestamp, call type, confidence, pig, confidence,
  channel balance. Plain files, greppable, no database until one is earned.

## Evidence standards

The project's value is honesty about accuracy. The operative, checkable form of these
standards is the **Evidence gate** in the delivery norms below; this section states why
they exist and what the limits are.

- **Nothing is a result without a held-out number.** If accuracy cannot be measured on
  held-out data, it is not reported as a result.
- **Guard against the proximity cheat**: record ground truth with each pig in varied
  positions; prefer stereo so left/right balance is an explicit, inspectable feature
  rather than a hidden shortcut. The background-only screen in the gate is what enforces
  this.
- **Expect and state limits**: three same-age brothers share body size and vocal tract
  dimensions, so ID is the hard case. 70-85% on three siblings is a good result. Below
  ~50% is a coin flip and must be reported as such.
- **Voices drift.** Young pigs change pitch as they grow. Re-record and retrain every
  couple of months; a result older than the artefact it describes is not a current
  result.

## Engineering principles

- Simplest thing that works. Logistic regression over a fine-tuned network until
  measurement says otherwise.
- Every script runs standalone from the command line with `--help`.
- Retraining is one command. Adding clips must never require code changes.
- Thresholds are configuration, not constants buried in code.
- No silent failures: if the mic drops or a model is missing, say so loudly.
- Python, with dependencies pinned. Install instructions that work from cold.

## Scope

**In scope**: recording and labelling clips, training both stages, live capture,
CSV logging, contagion and separation analysis, accuracy reporting.

**Out of scope for now**: real-time speech synthesis, mobile apps, a "what is my
pig saying" chat interface, any claim of translation, subscription anything.

## Data and privacy

Audio recorded in a home. Raw clips never enter the repository in any form. The full
rules, including what may be committed as derived data, are under **Data rules** in the
delivery norms below.

## Reference

- Google YAMNet — AudioSet classifier with a 1024-d embedding for transfer learning
- Individual differences in infant guinea pig isolation whistles — identity lives
  in *sets* of acoustic parameters, not any single one
- Automatic acoustic identification of individuals across recording conditions
  (Stowell et al.) — small training sets viable; check for confounds
- Hierarchical acoustic identification of individuals — call type first, then ID
## Delivery norms

How a change reaches `main`. `CONTRIBUTING.md` is the public prose version and is not
normative: where they differ, this file wins. `docs/not-adopted.md` records what was
deliberately declined and why.

**Precedence.** For this repository this file supersedes `~/.claude/CLAUDE.md`. Each
override is repeated at its own rule below: default branch `main` not `develop`; issue
and PR bodies use the structures here; PR titles lowercase conventional; Fibonacci
sizing stops at 13; Task Master unused; the repo root is `~/dev/github.com/bjcoombs/`.

### Session start

Three checks, in `wheek-main`, before touching anything:

```bash
cd /Users/ben/dev/github.com/bjcoombs/wheek/wheek-main
git status --short     # must print nothing
git worktree list      # must match the work in flight
```

Then name the issue this session is for. No file is edited until it has one.

### Where data and models live

Audio, trained artefacts and metrics live outside every git tree, in the group
directory shared by `wheek-main` and all worktrees:

```
/Users/ben/dev/github.com/bjcoombs/wheek/data/     clips and labels
/Users/ben/dev/github.com/bjcoombs/wheek/models/   trained artefacts
```

Code resolves them from `WHEEK_DATA_DIR` and `WHEEK_MODEL_DIR`. The default: take
`git rev-parse --git-common-dir`, **resolve it to an absolute path**, then take its
parent's parent and append `data` / `models`. Both `wheek-main` and every worktree land
on the same two paths — a worktree's common dir is `wheek-main/.git`. Never anchor to
the working directory or `--show-toplevel`, which do not.

A worktree starts with no audio and no models. When the directory is absent or empty,
say so in the PR and write the Evidence block as `not run — no data available`. Never
fabricate a number, never omit the block.

**Layout** (the first recording-tool issue implements it; it does not exist yet):

```
data/clips/<call-type>/<session-id>-<nnn>.wav    call type from the vocabulary above
data/labels.csv                                  clip filename, pig, labeller, date
```

**Recording session:** one continuous recording period, one microphone placement, one
date. Its id is `YYYY-MM-DD-<placement>-<nn>` where `<placement>` is a single lowercase
token with no hyphen naming the mic position, and `<nn>` is a two-digit counter within
that date and placement. In `2026-07-14-cage-01` the placement is `cage`. Placements in
use are listed in `docs/splits.json`; a new one is added there in the same PR that
first records it. The id is stamped into the filename at capture time and cannot be
reconstructed later, so the capture tool must write it.

**Splits manifest:** `docs/splits.json` holds the session assignment — session ids only,
no audio, no clip content — which is what makes "the test set is read once" checkable
across PRs. `background_only` lists sessions recorded with the pigs absent; they are
used for the cheat screen and never for training. The first model PR proposes the
initial assignment and Ben approves it; after that, changing it is propose-and-wait and
the PR says why. `updated` is `YYYY-MM-DD` and is set in every PR that changes the file.

### Autonomy ladder

**Act alone:** read and run anything in the repo; create, work in and remove a
worktree; write and refactor code and tests; run `/code-review`, `/simplify`,
`/security-review`, `/run`; commit and push to a feature branch it owns, including
`--force-with-lease` when rebasing onto a moved `main`; open, update and comment on a
PR; create and comment on issues under existing labels; write `docs/metrics/*.csv`
produced by its own verification run, and `docs/model-cards/`; append or delete a line
in Recurring corrections, and edit `docs/intent-template.md`, in the same PR as the
change that prompted it.

In `wheek-main` specifically, these and only these: `git status`, `git log`, `git
diff`, `git fetch`, `git checkout main`, `git pull origin main`, `git branch`, `git
worktree add`, `git worktree list`, `git worktree remove`.

**Propose in one line and wait for Ben:** adding, removing or upgrading a dependency;
changing the CSV event-log schema, the call-type vocabulary, a documented ethogram
meaning, or a threshold default; deleting a test or rewriting what one asserts, where
restructuring a test that still asserts the same thing is act-alone; changing an eval
threshold or fixture, or `docs/splits.json`; editing `CLAUDE.md` outside Recurring
corrections, `README.md` or `CONTRIBUTING.md`; adding `.github/workflows/`, `Makefile`,
`pyproject.toml` or `.claude/settings.json`; creating a label or changing repo settings
via `gh`. One proposal may cover a set that belongs together — the bootstrap PR below
is one approval, not four.

**Ben only:** merging any PR; any command in `wheek-main` outside the list above;
`git push --force`; `git add -f`; writing an accuracy figure into `README.md`, which
approval to edit it does not unlock; deciding or editing ground-truth labels; changing
`LICENSE`; anything putting raw audio or identifying detail into the public repo.

**Never, by any agent:** starting a live microphone recording. The mic is in a family
living room. Ben records; agents process what exists. A script that opens an audio
input stream is tested against files, or by Ben.

Unsure: treat it as the tier above and ask in one line.

### The unit of work

One GitHub issue, one branch, one worktree, one PR — with two lanes out. Recording,
labelling, listening back and reading the event log need none of it. A 1-point change
with no behaviour in it, a typo or a dead link, goes straight to a branch and a one-line
PR body, no issue.

Everything else is sized in Fibonacci points. The number is a confidence signal and it
picks the next action:

| Points | Meaning | Action |
|---|---|---|
| 1-2 | Understood, mechanical | Implement directly |
| 3 | Understood, minor unknowns | Short plan comment, then implement |
| 5-8 | Probable, real unknowns | Plan on the issue, Ben's go before code |
| 13 | Not understood | No worktree. Write the issue, run `/understand`, then split into issues of 8 or less |

The scale stops at 13, overriding the global file's 21: anything larger is split before
a branch exists. A 13 is usually not one piece of work but a spike whose deliverable is
a number.

### 0. Pick the work

GitHub Issues is the only tracker, via `gh`. Task Master is not used on wheek,
overriding the global CLAUDE.md's "Task Master Workflow (ALWAYS FOLLOW)": do not run
`task-master` here and do not create `.taskmaster` content.

The issue body follows `docs/intent-template.md` — those five headings, not the global
issue structure. Ambiguous problem: `/understand` first. Contested trade-off: `/6hats`
or `huddle`, synthesis into the issue. References to other issues use full GitHub URLs
in body text, per the global rule; the short `#12` form appears only in a PR title,
where GitHub needs it to auto-close.

### 1. Plan

3 points or larger starts in plan mode: read, produce a strategy, edit nothing. The plan
is a comment on the issue; there is no `plan.md`. It names the files that will change,
the work order, what could break, the command that verifies it, and for a model change
which metrics get re-run on which split. Naming the metric and the split before the work
starts is the point — choosing afterwards is how the flattering split gets picked.

Stopping rule: someone who never saw the conversation could implement it from the plan.

At 5+ it waits for Ben's go before the first file edit. That go covers the whole plan;
anything discovered mid-flight that was not in it goes back to Ben in one line first.

### 2. Isolate

`wheek-main` stays on `main`, clean, always — no edits, no commits, no other branch
checked out. The default branch is `main`, not `develop`.

```bash
cd /Users/ben/dev/github.com/bjcoombs/wheek/wheek-main
git checkout main && git pull origin main
git branch 12-session-split
git worktree add ../worktree/12-session-split 12-session-split
cd ../worktree/12-session-split
```

Branch and directory `<issue-number>-<brief-description>`, lowercase and hyphenated;
`<pr-number>-verification` when reviewing a PR. Scratch files, PR bodies and metric
dumps go to the scratchpad or `/tmp`, never inside the repo.

### 3. Build, then review the diff

One command per session that answers pass or fail, run before claiming anything works.
Until a test suite exists it is the script running from a cold shell, printing `--help`,
producing what the plan predicted.

Bug fix: write the failing test first, watch it fail, then fix the code. Never edit a
test to make a fix pass; a diff touching both a test and the code it covers says why in
the PR body, as does any change to an eval threshold or fixture. A wrong result that
reached `main` gets the test that would have caught it, in the same PR.

Commit in small steps: `type: Brief factual description` — type lowercase, description
starting with a capital. Types `feat` `fix` `refactor` `docs` `test` `chore`. No emojis,
no Claude attribution, no co-author or generated-with trailers, anywhere.

Then, on the diff: `/code-review` always, at `high` when the diff touches training,
evaluation, thresholds or the event-log schema; `/simplify` when the diff outgrew the
plan; `/security-review` for dependencies, file paths, `subprocess`, network, or loading
a model or config from disk. Every finding is fixed or listed under Known limitations.

If a review fix changes code that affects training, re-train and update the provenance.
Shipping numbers from a superseded commit fails Ben's merge check.

### 4. Pull request

Title: lowercase conventional, carrying `(fixes #12)` — not the global `Fix:` form. Body
sections in order, superseding the global PR structure, each omitted only by writing
"Not applicable": Summary, Changes, Evidence, Risk, Known limitations, Follow-ups.

Evidence carries the literal output of the verification command, plus the metrics block
for a model change, or "No model behaviour changed", or "not run — no data available".

An agent opens, updates and pushes to a PR without asking. An agent does not merge.

### 5. Merge and clean up

Done, for the agent: criteria met as written, verification output pasted, the gate below
satisfied, findings fixed or listed, docs true including "not measured yet". Ben then
squash-merges, after reading the diff against the plan and confirming the Evidence block
came from the code in that diff, not an earlier run.

```bash
gh pr merge <n> --squash --delete-branch
cd /Users/ben/dev/github.com/bjcoombs/wheek/wheek-main
git pull origin main
git worktree remove ../worktree/12-session-split
```

A stale worktree is how this layout rots. Remove it in the same sitting.

Skipping a step in flow 0-5 is allowed when the PR body names the step and says why.
Step 2, the evidence gate, the data rules and the Ben-only tier never bend.

### Evidence gate

A PR touching features, training data, splits, thresholds or evaluation does not merge
until its Evidence section satisfies all of this.

1. **Split by recording session, never by clip**, per `docs/splits.json`. Adjacent
   chunks share room tone, mic placement and pig position. Name the held-out sessions.
2. **Test set read once, at PR time.** Tune against validation. If the change was
   iterated against the test set, say so and report the number as validation, not
   held-out.
3. **Per-class precision and recall, with n per class**, never accuracy alone, plus a
   confusion matrix for Stage 1. Overall accuracy hides a model that never predicts
   `rumble`.
4. **Majority-class baseline on the same split**, and **chance beside every ID number** —
   three pigs is 1/3.
5. **Background-only screen, mandatory for any Stage 2 change.** The measured quantity is
   the **modal-class share**: of empty-cage clips, the fraction assigned to whichever pig
   the model names most often. Not accuracy — an empty-cage clip has no true pig. Under
   an honest model it sits near 1/3. Use 100+ clips from `background_only` sessions and
   more than one mic placement; report n, the placements, the full predicted
   distribution and the share, even when it passes.

   Void threshold `1/3 + 2*sqrt((2/9)/n)`, computed from the exact fractions and rounded
   to three decimals: **0.428 at n=100, 0.410 at n=150, 0.400 at n=200**. At or below it
   passes. Above it, the model is reading room noise, mic proximity or channel balance
   rather than voice: the ID result is void, is reported as void, and the change is
   reworked. Do not tune until it passes.
6. **Provenance in every metrics block and model card**: split, training commit, training
   date, and training clip counts per class per pig. The card is committed at
   `docs/model-cards/<model-id>.md` where `<model-id>` is `<stage>-<YYYY-MM-DD>`; the
   artefact itself stays in `models/`.
7. **A published number later found wrong is corrected in place**, dated, with an issue
   opened.

Numbers with no split, no n and no date are not evidence.

Metrics block — the only copy; `CONTRIBUTING.md` points here. Figures are illustrative,
but the shape is not: every field shown is required by the gate above.

```
Split: docs/splits.json @ a1b2c3d — held out 2026-07-14-cage-01, 2026-07-22-shelf-01
Commit: a1b2c3d   Trained: 2026-08-30   Test read: once
Training clips per class per pig: wheek A 41 B 38 C 44 / chut A 22 B 19 C 25 / ...

Stage 1 call type, n=412
                  precision  recall     n
  wheek                0.91    0.88   102
  chut                 0.74    0.69    88
  rumble               0.66    0.71    54
  chatter              0.61    0.58    41
  purr                 0.55    0.49    37
  other                0.83    0.87    90
  majority-class baseline accuracy: 0.25
  confusion matrix: docs/metrics/stage1-2026-08-30-confusion.csv

Stage 2 pig ID within wheek, n=102, chance 1/3
  overall 0.74   per pig: A 0.78 / B 0.71 / C 0.72

Background-only screen: n=140 clips, 3 sessions, placements cage + shelf
  predicted A 44 / B 51 / C 45
  modal-class share 0.364, void threshold 0.413 — passes
```

### Data rules

`data/`, `models/`, `*.wav` and `*.csv` are gitignored, with `docs/metrics/**/*.csv` and
`evals/fixtures/**/*.csv` un-ignored as the two committed derived paths. Whatever lands
there is named in the PR body.

Raw home audio never enters the repository in any form, fixtures included. Incidental
human speech is not a dataset: discard it, never commit it, never train on it. The event
log — when a household was noisy, and who was in the room — is not a metrics file and is
not committed. The repo is public: no addresses, no household detail, no other voices.

Recording is passive. Nobody separates the pigs to manufacture separation calls. If a
protocol would make their day worse, that data is not collected.

An outside contributor has no access to the audio, so a model-affecting change from
outside is code plus a described method. Its PR body says "Evidence to be produced by
maintainer — no access to source audio"; Ben runs the evaluation, posts the numbers as a
PR comment before merging, and applies his "produced by the code in this diff" check to
that run.

### Not present yet

Everything here is absent today, and each line names what retires it. Nothing above
hedges on these.

- **No `Makefile`:** do not run `make` while none exists. Once one is committed that
  instruction is spent and its targets join step 3.
- **No test suite, `wheek` package, `pyproject.toml`, `docs/metrics/`,
  `docs/model-cards/` or `evals/fixtures/`:** `pytest` and `python -m wheek.<script>
  --help` are the shape step 3 will take, not commands that run today. The bootstrap PR
  picks the environment tool, pins dependencies, adds `Makefile` and
  `.github/workflows/ci.yml`, creates those directories, and writes the real commands
  into step 3 — one propose-and-wait approval covering the set. Actions are enabled and
  the `gh` token has `workflow` scope.
- **No `wheek/data/` or `wheek/models/` directory:** created by the first recording
  session, not by a PR.
- **No CI and no hooks.** The global CLAUDE.md's "integrated hooks system" is not backed
  by any hook config here, so these rules are discipline, which is weaker. Two issues to
  open rather than designs to park here: a fixture evaluation over held-out sessions
  only, and three deny-only `PreToolUse` gates via `update-config`.

### Tooling that exists

Typed as slash commands: `/understand`, `/code-review`, `/simplify`, `/security-review`,
`/run`, `/loop`, `/schedule`, `/init`. Also `/6hats`, whose plugin version is shadowed by
a differing user-level `~/.claude/commands/6hats.md` — treat its behaviour as
unpredictable until that duplicate is removed.

Invoked by name as skills: `huddle`, `assess`, `deslop`, `semantic-compress`,
`skill-forge`, `ghsync`, `ghreport`. Installed, none exercised here yet.

`marathon`, `pr-review-merge`, `/tm`, `/issues` and `/fix-develop` do not apply.

### Recurring corrections

Append one line when the same mistake is caught twice, in the same PR as the fix; delete
one that stopped earning its place. The one part of this file an agent edits without
asking.

- (none yet)
