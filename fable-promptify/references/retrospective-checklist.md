# fable-promptify — retrospective checklist

Skill-specific yes/no checks for the end-of-session retrospective. Each catches a known failure mode.

1. **Interview-first:** Did the skill ask clarifying questions before producing any draft prompt? (Failure: refined prompt in the first response.)
2. **Repo-answerable questions:** Was every interview question something the repo/files could NOT answer? (Failure: asking "what test framework?" instead of reading config.)
3. **Constraint motivation:** Does every constraint in the final prompt carry its why? (Failure: "keep it simple" copied verbatim from the user.)
4. **Verification baked in:** Does the final prompt contain criteria Fable can check without the user (tests, rubric, self-check instruction)?
5. **Calm prose:** Is the final prompt free of CAPS/MUST/CRITICAL emphasis?
6. **Right-sized:** Did the prompt include only the sections the task shape needed (no long-horizon boilerplate on a quick fix)? Was it shorter or equal in ceremony to what the task warranted?
7. **First-version acceptance:** Did the user accept the prompt without major revision? If revised, what specifically changed — that delta is a candidate learning.
