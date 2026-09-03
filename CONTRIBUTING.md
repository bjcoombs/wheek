# Contributing to wheek

wheek classifies guinea pig calls and logs them with timestamps. It is not a translator,
and the difference matters: the software's job is to produce evidence a person can check,
and the meanings come from published ethology, not from the model.

Contributions are welcome. This document explains how work gets done here and why the
rules are shaped the way they are. [CLAUDE.md](CLAUDE.md) is the normative version —
where the two differ, it wins, and it holds the exact numbers.

## The short version

1. Open an issue using the five headings in [docs/intent-template.md](docs/intent-template.md).
2. Agree the approach before writing much code, especially for anything that touches
   training or evaluation.
3. Work on a branch. Keep commits small and conventionally formatted.
4. Open a PR whose body carries a Summary, Changes, Evidence, Risk, Known limitations
   and Follow-ups.
5. The maintainer squash-merges.

## Setting up

There is nothing to install yet — the repository currently holds documentation. When the
first code lands, this section will name the environment tool and the one command that
runs the tests.

The audio and trained models are not in the repository and never will be. They live
alongside it on the maintainer's machine, resolved through `WHEEK_DATA_DIR` and
`WHEEK_MODEL_DIR`. This means an outside contributor cannot run the model evaluation, and
that is handled explicitly below rather than pretended away.

## How work is organised

One issue, one branch, one PR. Work is sized in Fibonacci points — 1, 2, 3, 5, 8, 13 —
where the number is as much a statement of confidence as of effort. A 3 is understood
well enough to start. An 8 has real unknowns and gets a written plan first. A 13 is not
understood, and is usually a spike whose deliverable is a number rather than code; it
gets split before anyone opens a branch.

Recording clips, labelling them, listening back and reading the event log need no issue,
no branch and no PR. That work happens outside the repository and is not taxed with
process.

The maintainer works in git worktrees under a `wheek/wheek-main` + `wheek/worktree/`
layout. You do not need to reproduce it — a normal fork and clone is fine.

## What a good pull request looks like

**Title:** `type: Brief factual description (fixes #12)`. The type prefix is lowercase
and one of `feat`, `fix`, `refactor`, `docs`, `test`, `chore`.

**Body:** Summary, Changes, Evidence, Risk, Known limitations, Follow-ups. Write "Not
applicable" rather than dropping a section.

**Evidence** is the section that matters most. It carries the literal output of whatever
command demonstrates the change works. For anything touching a model, it carries the
metrics block defined in [CLAUDE.md](CLAUDE.md) — the same shape every time, so numbers
from different PRs can be compared.

**Tone:** factual and neutral, describing systems rather than people. No emojis. Findings
from review are either fixed or written down under Known limitations; they are not
dropped quietly.

## The evidence rules, and why they exist

The exact thresholds live in [CLAUDE.md](CLAUDE.md). What follows is why they are there,
because a contributor who understands the reasoning will apply it to cases the rules did
not anticipate.

**Split by recording session, never by clip.** Two clips cut from the same recording
share room tone, microphone placement and the position of the animals. Put one in train
and one in test and the model can score well by recognising the afternoon, not the call.
This is the single easiest way to publish a number that is wrong.

**Read the test set once.** Tune against validation. Every extra look at the test set
turns it into training data by a slower route. If a change was iterated against it, the
number gets reported as validation — still useful, just not what it would otherwise be
claimed to be.

**Report per-class precision and recall, never accuracy alone.** A classifier that never
once predicts `rumble` can still post a healthy overall accuracy. Per-class numbers make
that visible; a single number hides it.

**State chance next to every identification number.** Three pigs means chance is 1/3. A
74% identification rate is a real result; 38% is close to noise. Without chance printed
beside it, a reader cannot tell which they are looking at.

**Screen the identification model against empty-cage audio.** This is the one the project
exists to get right. Feed the model recordings with no pigs in them and see which pig it
names. It has no correct answer available, so an honest model spreads its guesses near
chance. If it confidently picks one pig from room noise, it has learned the room, the
microphone or the channel balance rather than the voice — and the identification result
is void, reported as void, and reworked. It is not tuned until it passes.

**Record provenance with every artefact.** Which split, which commit, which date, how
many clips per class per pig. The pigs are young and their voices are changing; a result
older than the model it describes is not a current result.

## If you cannot run the evaluation

You will not have the audio, and that is fine. Submit the code with the method described
— which split, which metrics, what you expect to change. Put this in the Evidence
section:

> Evidence to be produced by maintainer — no access to source audio.

The maintainer runs the evaluation and posts the numbers as a PR comment before merging.

## Data and animal welfare

The recordings are made in a family home, and some of them will catch human speech.
Clips containing people stay on the recording machine: they are not committed, not shared
and not trained on. The repository is public, so no addresses, no household detail, no
other voices, and no raw audio in any form — fixtures included.

Recording is passive. Nobody separates the pigs to manufacture separation calls, or
provokes them to collect a chatter sample. If a protocol would make their day worse, the
data does not get collected.

## Language

Say what the model measured. "Classified as a wheek, confidence 0.87" is a claim the
software can support. "She said feed me" is not. Meanings shown to a user are labelled as
an ethogram lookup from published work, and are never presented as the model's output.

This applies to the README, the code, commit messages and the user interface equally. It
is the whole point of the project.
