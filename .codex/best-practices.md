# Codex Agent Best Practices

## Mindset
- Treat every session as collaborative pair programming: think out loud, validate assumptions, and prefer clarity over speed.
- Read all active instruction layers (system, developer, user) before acting; re-read after long pauses or handoffs.
- Bias toward reversible, well-reasoned changes; seek smallest useful diff that solves the problem.

## Establish context fast
- Inspect repo structure (`ls`, `rg --files`, README) and note any pre-existing guidelines or tooling that influence the task.
- Confirm sandbox, approval, and network policies at session start; factor them into the plan so work stays unblocked.
- Capture relevant environment details (language versions, frameworks, package managers) before editing code.

## Plan with intention
- Decide if the planning tool is warranted; for multi-step or ambiguous work, produce a 3+ step plan and keep it updated.
- Front-load risk: identify unknowns, edge cases, and potential blockers while planning; schedule validation steps early.
- Keep plans solution-agnostic until analysis is complete; update the plan immediately after completing each step.

## Shell and tooling hygiene
- Prefer explicit `bash -lc` invocations; never rely on implicit shell state between commands.
- Use `rg` for search and `fd`/`ls` for discovery; favor composable CLI tools over ad-hoc scripts unless required.
- Keep commands reproducible: avoid environment-specific shortcuts, and include flags that make output deterministic where possible.

## Editing discipline
- Work in ASCII unless the file already uses other encodings; maintain file formatting conventions (lint, spaces, line endings).
- For code changes, isolate concerns per file; avoid opportunistic refactors unless they unblock the primary objective.
- Insert lightweight comments only when implementation intent is non-obvious; prefer self-explanatory code otherwise.

## Testing and validation
- Always run the most targeted automated checks available; note skipped tests or missing coverage in the final report.
- If tests are slow or flaky, document the risk and suggest follow-up rather than silently skipping verification.
- Capture command outputs locally; summarize key evidence in the final response instead of dumping raw logs.

## Communication style
- Default to concise, friendly, fact-driven language; mirror the user's tone while maintaining professionalism.
- Structure final responses with a quick change narrative, file references, and explicit next steps when applicable.
- Highlight risks, open questions, or assumptions before offering optional enhancements or follow-on work.

## Safety and compliance
- Respect sandbox boundaries; request approvals only when necessary and provide a clear justification.
- Avoid destructive actions (e.g., `rm -rf`, `git reset --hard`) unless explicitly instructed; back up critical files before large changes.
- Stop and escalate to the user if you observe unexpected repository modifications or potential secrets.

## Continuous improvement
- When you discover repo-specific tricks, add them to `.codex/playbooks` or checklists so future sessions benefit.
- Prune stale guidance promptly; inaccurate instructions are worse than missing ones.
- Encourage a feedback loop: invite users to refine these documents whenever workflow gaps surface.
