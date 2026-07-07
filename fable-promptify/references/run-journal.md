# fable-promptify — run journal

Dated entry per run: task shape, interview rounds, what the user revised in the final prompt, health counts. Evidence trail for retrospectives — not read at run start.

## 2026-07-07 — skill improvement: /goal integration + loops taxonomy (not a promptify run)

- **Trigger**: operator asked to integrate `/goal` properly (4,000-char argument cap) and fold in "Designing Loops for Claude Code Agents" (@delba_oliveira, July 2026).
- **Research**: official `/goal` docs fetched (code.claude.com/docs/en/goal). Key mechanics that shaped the design: goal = session-scoped prompt-based Stop hook; evaluator is the configured small fast model (Haiku default), sees condition + full transcript, runs NO tools; 4,000-char cap on the condition only; no flags — turn caps go inside the text; setting a goal immediately fires a turn; headless via `claude -p "/goal ..."`.
- **Design decision (operator-approved)**: shape-keyed delivery, not one rule — goal-only for quick verifiable tasks; **spec file + compact goal as the default** for feature builds/long-horizon (one paste, every turn evaluated, file survives context refreshes); prompt-then-goal kept as the mid-session variant. Explicitly rejected: compressing the compiled prompt into the goal argument.
- **Changes**: SKILL.md Phase 1 goal-suitability flag, Phase 4 goal-companion drafting rules, Phase 5 three delivery patterns, two new common mistakes. Guide: "Loops and /goal" section (four-loop taxonomy, /goal mechanics, patterns, condition template, token-budget notes), sources updated. New pressure test `tests/goal-companion.md` (GREEN passed 2026-07-07: 953-char transcript-verifiable goal, correct pattern, count reported).
- **Shipped**: mirrored to public repo yungbose/fable-promptify with README updates.

## 2026-07-02 — TAX-1322 For-review IA hardening (TaxHeaven)

- **Task shape**: feature build (single-session, but merge-autonomous → long-horizon-adjacent; included goal discipline + checkpoint commits, skipped state files).
- **Interview**: 1 round, 4 questions (count model, autonomy, browser verification, ambition). User diverged from two recommendations: chose *Fable decides the count model against the fixture* (over locking the ticket's preferred model) and *full merge autonomy* (over stop-at-PR). Accepted browser-verify and ticket-plus ambition.
- **Pre-interview investigation** did the heavy lifting: worktree/branch created first (user asked), Linear ticket fetched (unusually crystal — acceptance criteria + jest block already in the ticket), and an Explore-agent map of ImportDataPage/UnifiedBankReviewQueue meant zero repo-answerable questions reached the user. Interview stayed to the four genuinely user-owned knobs.
- **Prompt notes**: fixture-first ordering (build the messy production fixture, then make the delegated product choices against it) let the delegated decisions stay verifiable rather than vibes. Merge autonomy paired with a frontend-only tripwire (stop if EF/migration needed) since merges auto-deploy to production.
- **Health**: prompt delivered; no revisions requested at draft time. Run location left with the user (fresh session recommended).

## 2026-06-11 — MTD e2e quarterly + year-end submissions (TaxHeaven)

- **Task shape**: long-horizon autonomous run (multi-context-window feature build + live API verification).
- **Interview**: 1 round, 4 questions (merge/deploy autonomy, live-sandbox autonomy, scope boundary, run location). All recommended options accepted: full merge+deploy, live sandbox autonomously with fallback, full TAX-969 journey, run in-session.
- **Pre-interview investigation** answered most pillars from repo + Linear (TAX-969/977/1034/580 + Explore agent scan of TaxFilingPage data flow), so the interview stayed to 4 genuinely user-owned decisions.
- **Post-draft revision**: user added (mid-flow, after draft delivered): use the do-ticket-style PR review fan-out per PR, and an explicit "tests for everything so the end goal is measurable" bar. Folded into the Verification section. Lesson: ask about review-cycle depth explicitly for long-horizon builds — it's an autonomy-adjacent knob users care about.
- **Health**: prompt delivered + executed in-session; no other revisions requested at draft time.

## 2026-07-02 — MTD quarterly submission audit handover
- Task shape: long-horizon autonomous run (multi-context-window audit + fix loop)
- Interview rounds: 1 (4 questions: fix autonomy, triage line, test-account strategy, ambition)
- Notable answers: full PR+auto-merge loop; fix everything found (no triage); seed fresh test accounts but model seed data on a read-only sample of real users; full design ambition (standing "present options first" preference explicitly waived for this run)
- Repo answered before interview: design spec location (2026-06-24 quarterly revamp + TAX-1213 docs), MTD service guide path, Playwright/e2e setup, seeding guides
- User revisions to final prompt: none yet (delivered this session)

## 2026-07-04 — TAX-1365 handover
- Shape: long-horizon feature build (multi-workstream, merge-authorised)
- Interview: 1 round, 4 questions (session scope, merge autonomy, Gmail-draft cascade, copy approval). User diverged on cascade: update Customer-QA.md only, leave Gmail drafts alone.
- Draft changes: de-CAPSed, prohibitions → positive instructions with motivation, added verifier checklist + progress.txt/checkpoint mechanics.
- User revisions to final prompt: none yet at delivery.

## 2026-07-06 — TAX-1379 + TAX-1380 dev-loop seed-set
- Shape: quick-task pair via dev-loop seed-set; tickets unusually crystal (file/line/fix/test all in ticket), so prompt was mostly assembly.
- Interview: 1 round, 2 questions (ambition, merge behaviour). Both recommendations accepted: ticket scope + cheap sweep for other un-threaded aggregation call sites; dev-loop default auto-merge with panel-authorise on the financial ticket.
- Session constraints from invocation: Codex out of quota → all implementation/review/investigation to opus high (skip codex-implement entirely, no retry); parallel work → fresh worktrees from origin/main on Linear branch names.
- User revisions to final prompt: none yet at delivery.

## 2026-07-06 — TAX-1368 build session (feature build / long-horizon)
- Shape: feature build with autonomous dev-loop execution; ticket already richly specified, so interview focused on latitude, checkpoint placement, deploy autonomy, ambition.
- Interview: 1 round (4 questions). User picked: genuine re-examination of the ticket's design, single checkpoint after proposal, unattended Edge Function deploys, "go beyond" on the UI workstream.
- Mid-flight revision: user added a requirement mid-draft — build in a fresh git worktree + new branch (stale-main gotcha folded in from memory).
