# Intent template

The body of a wheek issue. Five headings and a size. Fill it in as prose — the point is
to think before building, not to complete a form.

This replaces the issue structure in `~/.claude/CLAUDE.md` for this repository.

---

## What is wanted

One or two sentences. The thing that will be true when this is done.

## Why

What is not possible today, or what is currently wrong. If this is a bug, what was
observed and when. If it is a feature, the question it lets us answer about the pigs.

## Constraints

What the solution must respect. Existing schema, the call-type vocabulary, the honesty
rules, the simplest-thing-that-works principle. Also what is out of scope, so the work
does not drift into it.

Ask here: does this change what the project can honestly claim, and on what evidence?

## How success is measured

The command that will demonstrate it, or the metric and the split. For anything touching
a model, name the metric and the held-out sessions **now**, before the work starts —
choosing them afterwards is how the flattering split gets picked.

If success is not measurable, say so plainly and say what will be observed instead.

## Open questions

What is not known. If this section is long, the issue is probably a 13 and wants a spike
before it wants code.

---

**Size:** 1 / 2 / 3 / 5 / 8 / 13

The number is a confidence signal, not just effort. A 13 means it is not understood well
enough to start: run `/understand`, then split it into issues of 8 or less.
