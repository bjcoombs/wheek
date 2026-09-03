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

The project's value is honesty about accuracy. Therefore:

- **Held-out test set always.** Report per-class precision/recall, not just accuracy.
- **Background-only test is mandatory** for the pig ID model. Feed it empty-cage
  audio; above-chance identification means the model is cheating on room noise or
  proximity, and the result is void.
- **Guard against the proximity cheat**: record ground truth with each pig in
  varied positions; prefer stereo so left/right balance is an explicit, inspectable
  feature rather than a hidden shortcut.
- **Expect and state limits**: three same-age brothers share body size and vocal
  tract dimensions, so ID is the hard case. 70–85% on three siblings is a good
  result. Below ~50% is a coin flip and must be reported as such.
- **Voices drift.** Young pigs change pitch as they grow. Re-record and retrain
  every couple of months; log the training date alongside every model artefact.

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

Audio recorded in a home. Raw clips stay local and out of the repository — only
committed derived data (embeddings, aggregate stats) may be shared, and only
deliberately. `.gitignore` covers `data/` and `models/` by default.

## Reference

- Google YAMNet — AudioSet classifier with a 1024-d embedding for transfer learning
- Individual differences in infant guinea pig isolation whistles — identity lives
  in *sets* of acoustic parameters, not any single one
- Automatic acoustic identification of individuals across recording conditions
  (Stowell et al.) — small training sets viable; check for confounds
- Hierarchical acoustic identification of individuals — call type first, then ID
