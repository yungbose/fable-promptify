# Fable 5 prompting guide (distilled)

Sources: Anthropic prompting best practices (platform.claude.com, Claude 4.6/5 era), Claude Code team launch guidance (Thariq, June 2026), "Tips for Getting the Most Out of Mythos-Class Models" (Lance Martin, MTS @ Anthropic, June 2026). Distilled June 2026 — re-check the docs page if Anthropic releases new Fable guidance.

## The mindset shift

With Fable the operator's job moves from checking the model is *doing the work right* to checking it is *doing the right work*. Direction and setup over supervision. Three behaviours follow:

1. **Thought partner** — involve Fable early, give it the context it needs, let it interview you before you commit to a spec.
2. **Goals + ways to verify them** — a well-designed goal or rubric adds feedback to the environment; Fable runs, collects feedback, self-corrects, proceeds until satisfied.
3. **Be ambitious** — Fable rewards being asked for things you have not tried before. If you assumed LLMs couldn't do it, give it a chance.

## The four pillars (what every Fable prompt needs)

### 1. Aim
- State "done" as an observable outcome, not an activity.
- Be clear and direct; Fable follows instructions literally and precisely.
- If you want above-and-beyond, ask for it explicitly: "Include as many relevant features and interactions as possible. Go beyond the basics."
- Action verbs for action: "Implement X", not "Can you suggest improvements to X" — suggestion language produces suggestions.

### 2. Verification
- Give Fable a way to check its own work without the user: tests to write first, a rubric of checkable criteria, observable behaviours, a self-check instruction ("Before you finish, verify your answer against [criteria]").
- For long or high-stakes runs, prefer a **verifier subagent** (independent context window) over self-critique — models grade their own outputs poorly.
- A rubric file with discrete checkable criteria (e.g. "run a baseline; run 20 experiments") outperforms vague quality language.
- Tests verify correctness; they do not define the solution. If the prompt includes tests, also say: implement the general solution, don't hard-code to the test inputs, and report incorrect tests rather than working around them.

### 3. Context, not constraints
- Every constraint should carry its motivation. "Keep it simple" → "This feature is an experiment; there's a real chance we delete it in a month, so don't build anything painful to throw away." Fable generalises from the why and catches things you didn't think to forbid.
- Same for format rules: "Never use ellipses" → "The output is read by a text-to-speech engine that can't pronounce ellipses."
- Give Fable a role when it focuses behaviour ("You are reviewing this as a security auditor").

### 4. Scope & ambition
- Explicit boundaries: what NOT to touch, what is out of scope.
- Counter overengineering with scope context, not bare prohibition: say whether this is an MVP, an experiment, or a production feature, and what flexibility is actually needed.
- State the autonomy level: what it may do freely (edit, test, commit locally) vs what needs confirmation (push, deploy, delete, send, spend). Frame as reversibility: local reversible actions free; hard-to-reverse or outward-facing actions confirm first.

## Style rules for the drafted prompt

- **Calm prose.** No CAPS, "MUST", "CRITICAL", "IMPORTANT". Fable and the 4.6-era models are highly responsive to the system/user prompt; aggressive emphasis that fixed undertriggering in old models now causes overtriggering. "Use this tool when..." beats "CRITICAL: You MUST use this tool when...".
- **Positive instructions.** Say what to do, not what to avoid: "Write flowing prose paragraphs" beats "Do not use markdown".
- **XML tags when mixing content types.** `<instructions>`, `<context>`, `<input>`, `<example>` — only needed when the prompt combines instructions with pasted data, documents, or examples. Long documents go at the top, the query/instructions at the end (up to ~30% quality gain on long inputs). For long-document tasks, ask for relevant quotes first, grounded in `<quotes>` tags.
- **Examples steer format best.** 3–5 relevant, diverse examples in `<example>` tags when output format/tone matters.
- **General over prescriptive for reasoning.** "Think thoroughly about edge cases" outperforms a hand-written step-by-step reasoning plan; Fable's own reasoning usually exceeds what a human would prescribe.
- **Match prompt style to desired output style.** A markdown-heavy prompt begets markdown-heavy output.
- **Every line earns its place.** Padding dilutes. A crystal 10-line prompt beats a 60-line template instantiation.

## Per-shape additions

### Quick task (bounded fix/edit)
Aim + verification, nothing else. No session mechanics, no state files. Often 3–8 lines.

### Feature build
Full four pillars. Consider: interview-first ("interview me about the implementation before writing the spec") if the user is still unsure; HTML mockups of 2–3 directions if it's visual.

### Long-horizon autonomous run (hours, multi-context-window)
Add to the prompt:
- **Goal + completion discipline**: "Set a goal to implement the spec fully. Continue working systematically until complete; don't stop early due to token budget — save progress and state before the context refreshes."
- **State files**: structured state in JSON (`tests.json` with pass/fail status), freeform progress in `progress.txt`, git commits as checkpoints. "It is unacceptable to remove or edit tests."
- **Fresh-context restart instructions**: "Review progress.txt, tests.json, and the git log. Run the fundamental integration test before implementing anything new."
- **Setup scripts**: encourage an `init.sh` so servers/tests/linters restart gracefully in a new window.
- **Verification loop**: a Workflow or verifier-subagent pass over each part of the plan, producing a report of what was implemented and what differed.
- **Memory progression** (if memory is available): fail → investigate → verify → distill into a general rule → consult the rule instead of re-deriving. Prompt Fable to distill verified learnings into general rules, not just failure notes.

### Research / analysis
- Define what a successful answer looks like (success criteria).
- Ask for multi-source verification of claims.
- For complex research: competing hypotheses, confidence levels in progress notes, regular self-critique, a hypothesis tree or research-notes file for persistence and transparency.

### Writing / creative
- Voice, audience, length, format — with an example of each if the user has one.
- For frontend/visual work: ask for distinctive, creative output explicitly; name the aesthetic direction. Generic prompts produce generic "AI slop" design.

## Session mechanics to recommend (delivery advice, not always in the prompt)

- **`/goal`** — for runs that should continue until a goal is satisfied; Fable hill-climbs against it.
- **Workflows** — for verification fan-out: "use a workflow to verify each part of the plan and prepare a report on what was implemented and if anything differed."
- **Fresh session vs current** — a fresh session when the current conversation has accumulated unrelated context; current when the conversation context IS the context.
- **Verifier subagent over self-critique** — grading in an independent context window outperforms self-grading.
- **Loops over steering** — rather than directly steering Fable step-by-step, design the loop: goal/rubric for feedback, memory for cross-session learning, and let it self-correct.

## Anti-patterns (never put these in a drafted prompt)

- Aggressive emphasis (CAPS/MUST/CRITICAL) — causes overtriggering.
- "If in doubt, use [tool]" / blanket thoroughness nudges — Fable already explores; these cause overtriggering. Targeted "use X when it would enhance understanding" only if needed.
- Bare constraints without motivation.
- Suggestion language when action is wanted.
- Prescriptive step-by-step reasoning plans for problems Fable can reason through itself.
- Hard-coding/test-gaming incentives — pair any test mention with "general solution, not test-specific".
- Boilerplate from this guide that the task shape doesn't need.
