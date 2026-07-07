---
name: fable-promptify
description: Use when the user has a rough prompt, task idea, or half-formed goal they want converted into the best possible prompt for a Claude Fable 5 session. Triggers on "/fable-promptify <prompt>", "promptify this", "make this a fable prompt", "optimise this prompt for fable", or pasting a rough prompt and asking for it to be improved before starting a session. NOT for executing the task itself — the deliverable is the refined prompt.
---

# fable-promptify

## Overview

Fable 5 shifts the operator's job from supervising the work to directing it: the model runs for hours, tests its own work, and self-corrects in loops. That moves all the leverage into the prompt. A Fable prompt earns that autonomy when it has four things: a crystal **aim**, **verification** the model can run without you, **context** behind every constraint (not bare rules), and explicit **scope & ambition**.

This skill converts a rough prompt into that shape by interviewing the user until the aim is crystal, then drafting a structured prompt. It is the "ask Claude to interview me before writing the final spec" pattern, made repeatable.

**Core loop**: ingest → reflect → interview until crystal → draft → deliver + offer to run.

The skill never starts executing the task mid-flow. The deliverable is the prompt.

## Phase 0 — Preflight

1. Read `references/learnings.md` (lessons from prior runs) and `<project-config>/learnings.md` if present.
2. Read `references/fable-prompting-guide.md` — the distilled best-practice checklist used in Phases 2 and 4. Do not draft from memory of Fable guidance; the guide is the source.

## Phase 1 — Ingest

1. Capture the user's rough prompt verbatim. If none was given with the invocation, ask for it.
2. **Investigate before asking.** If the task touches a repo, skim CLAUDE.md, the relevant files, and recent commits first. A question the repo can answer is a wasted interview round.
3. Classify the task shape — it changes which best practices matter (see the guide's per-shape notes):
   - **Quick task** (bounded edit/fix) — lean prompt, skip session mechanics
   - **Feature build** — full four-pillar treatment
   - **Long-horizon autonomous run** (hours, multi-context-window) — add state files, memory, loop design
   - **Research / analysis** — success criteria + source verification + structured notes
   - **Writing / creative** — voice, audience, format examples
4. Alongside the shape, note whether the task is **goal-suited**: it has deterministic exit criteria that Claude's own output can demonstrate (tests pass, a score clears a threshold, a queue empties, a build exits 0). Goal-suited tasks get a `/goal` companion in Phase 4 and a delivery pattern in Phase 5. Subjective or taste-judged outcomes (design quality, prose voice) are not goal-suited — a small evaluator model cannot judge them well; use a verifier subagent instead.

## Phase 2 — Reflect

Score the rough prompt against the four pillars. For each, write down what is missing or ambiguous — this list drives the interview.

| Pillar | Crystal when... |
|---|---|
| **Aim** | "Done" is an observable outcome, not an activity ("users can export a CSV of X", not "work on exports") |
| **Verification** | Fable can check its own work without the user: tests to write or run, a rubric file, checkable criteria, observable behaviour |
| **Context** | Every constraint carries its motivation ("this is an experiment, real chance we delete it in a month" — not "keep it simple") |
| **Scope & ambition** | Boundaries are explicit (what NOT to touch), and the ambition level is stated (minimal fix vs go-beyond-the-basics) |

Also flag if unstated: output format, autonomy level (may it push, deploy, send, spend?), long-horizon needs (state files, memory, fresh-context restart instructions), and convention deviations — any way the user's setup differs from what Fable would assume (branch-to-environment mapping, unusual auth, intentional quirks). Fable's strong priors override unstated reality, so these must surface in the interview and land in the prompt's context section.

## Phase 3 — Interview until crystal

Interview the user with AskUserQuestion, up to 4 questions per round, as many rounds as it takes. Answers often expose new ambiguity — iterate.

Rules:
- Ask only what Phase 2 flagged AND the repo cannot answer.
- Offer concrete options with a recommendation, not open-ended essay questions.
- When the user gives a bare constraint, ask for the why behind it. The motivation is what Fable generalises from; the bare rule is not.
- When the user does not know what they want yet, act as a thought partner: propose 2–3 directions the task could take and let them pick. This is preferable to forcing an answer they don't have.
- Surface ambition explicitly at least once: Fable rewards being asked for more than you'd normally attempt. "Is the minimal version the goal, or should the prompt ask Fable to go beyond the basics?"

**Exit test (the only exit condition):** could a colleague with zero context execute from your current understanding without asking a single question? If no — another round. If yes — stop interviewing; do not pad with extra rounds.

## Phase 4 — Draft the prompt

Build the prompt using the template in `references/fable-prompting-guide.md`. Order: context → aim → constraints-with-why → verification → output expectations → session mechanics (only if long-horizon).

Drafting rules (rationale in the guide):
- Action verbs — "Implement", "Build", "Fix" — never "Can you suggest". Fable follows instructions literally; suggestion language gets suggestions, not changes.
- Tell it what to do, not what to avoid. Replace every "don't X" with the positive instruction where possible.
- Calm prose. No CAPS, no "MUST", no "CRITICAL". Fable overtriggers on aggressive emphasis that older models needed.
- XML tags only when the prompt mixes content types (instructions vs pasted data vs examples). A short single-purpose prompt needs none.
- Bake verification in: end with "Before you finish, verify your work against: [the criteria from the interview]". For long-horizon tasks, prefer a rubric of checkable criteria and a verifier-subagent pass over self-critique.
- Carry the autonomy decision: if the user set boundaries (confirm before irreversible actions, never push, etc.), state them with their why.
- Every line must earn its place. A short crystal prompt beats a long padded one — do not import guide boilerplate the task doesn't need.

### The goal companion (goal-suited tasks only)

For goal-suited tasks, draft a second artifact alongside the prompt: the `/goal` condition. It is a completion contract for an evaluator, not a compressed copy of the prompt. `/goal` wraps a session-scoped stop check: after every turn, a small fast model (Haiku by default) reads the condition plus the transcript and answers met / not met — it cannot run tools or read files, and a "not met" reason is fed back as next-turn steering. Draft the condition accordingly (mechanics and template in the guide's "Loops and /goal" section):

- Each criterion pairs a **measurable end state** with the **check that proves it**, so the evidence lands in the transcript ("all tests in `src/services/__tests__/split` pass — run `npx jest split` and show the summary", not "the split logic is correct").
- **Constraints that matter**: what must not change ("no other test file is modified", "no new dependencies").
- A **turn cap inside the text** ("stop after 20 turns") — there are no flags; the cap is part of the condition.
- Written for a small evaluator model: crisp yes/no statements, no vague quality language.
- Under **4,000 characters** (aim ≤3,900 for margin); report the character count on delivery.
- Context, motivation, and instructions stay in the prompt or spec file; the goal carries only what the evaluator needs to answer yes/no.

## Phase 5 — Deliver

1. Output the final prompt in a fenced code block, copy-paste ready.
2. For goal-suited tasks, deliver the goal companion with the pattern that fits the shape:
   - **Quick verifiable task → goal-only.** One `/goal` invocation carries the task, end state + check, and turn cap. Setting a goal immediately starts a turn, so no separate prompt is needed.
   - **Feature build / long-horizon → spec file + compact goal (the default).** Save the compiled prompt as a handover/plan file; the goal argument opens with "Implement the spec in `<file>` fully" and then the completion contract. One paste, and every turn — including the first — is goal-evaluated.
   - **Already mid-session → prompt-then-goal.** Paste the compiled prompt, then set the compact goal once work is underway. Supported, but in a fresh session the spec-file pattern is better: it needs one paste and no unevaluated first turn.
3. Below it, when relevant, 2–4 lines of session setup advice — pick the loop primitive deliberately: turn-based (default single session), `/goal` (verifiable exit criteria), `/loop` or `/schedule` (recurring or externally-triggered work); plus a Workflow verification pass, progress/state files (`progress.txt`, `tests.json`) for multi-context-window work, verifier subagent over self-critique. Skip entirely for quick tasks.
4. Ask: run it in this session now, or take it to a fresh Fable session? (A fresh session is better when this conversation has accumulated unrelated context.)
5. Append a run-journal entry to `references/run-journal.md`: task shape, rounds of interview, what the user revised in the final prompt (if anything).

## Common mistakes

- **Drafting in the first response.** The whole point is the interview. If your first reply contains a refined prompt, you skipped Phases 2–3.
- **Asking what the repo can answer.** "What test framework do you use?" is a Read, not a question.
- **Translating constraints verbatim.** "Keep it simple" copied into the output is a failure; the interview should have extracted "this is a throwaway experiment" and the prompt should say that.
- **Aggressive emphasis.** CAPS and MUST were crutches for older models; on Fable they cause overtriggering.
- **Boilerplate padding.** Importing every guide section into every prompt. A 10-line bug-fix prompt with state-file instructions is worse than the rough original.
- **Skipping verification criteria.** A prompt without a way for Fable to check itself wastes Fable's main strength: self-correction in a loop.
- **Interviewing past crystal.** The exit test is the colleague test, not a round count. Extra rounds after clarity is annoyance, not rigour.
- **Cramming the compiled prompt into the goal argument.** The 4,000-character cap forces lossy compression of exactly what makes the prompt good (context-with-why), and the evaluator re-reads the condition every turn — noise degrades the stop judgment. Prompt and goal are different artifacts for different readers.
- **Goal criteria the evaluator cannot see.** The stop-check model reads only the transcript — it runs no tools. "The dashboard looks right" or "coverage is adequate" are unjudgeable; every criterion needs the check whose output Fable will surface.
