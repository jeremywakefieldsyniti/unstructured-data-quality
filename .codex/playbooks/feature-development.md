# Feature Development Playbook

1. Clarify scope
   - Restate the feature goal and success criteria in your own words; confirm acceptance with the user if gaps appear.
   - Map the impacted domains (frontend, backend, infra) and locate relevant modules with `rg`/`fd`.
2. Baseline current behavior
   - Identify existing entry points, APIs, or components to extend; read tests and docs to understand expectations.
   - Capture screenshots, logs, or sample payloads when behavior comparisons will help validate changes.
3. Design collaboratively
   - Propose lightweight architecture or data-flow updates; keep options open until constraints are clear.
   - Note risks (backwards compatibility, performance, security) and mitigation strategies before coding.
4. Implement iteratively
   - Code in small increments, validating compilation or targeted tests after each logical chunk.
   - Keep commits cohesive; factor reusable helpers when duplication becomes evident.
5. Validate thoroughly
   - Run unit and integration tests touching the new surface; add coverage for key paths when missing.
   - Manual sanity-checks (CLI, UI) should mimic real-user flows; record what you tried.
6. Enforce logic gates
   - Capture the feature's acceptance gates up front (required automated suites, manual verifications, documentation updates).
   - Run the gate suite locally (`bash backend/scripts/test.sh`, `npx playwright test`, domain-specific smoke checks) before handoff.
   - When any gate fails, pause implementation, remediate, and rerun until every gate passes without manual tweaks.
   - Log the final gate status in the handoff so reviewers know which evidence exists.
7. Document outcomes
   - Update READMEs, changelogs, or API docs if behavior or usage instructions changed.
   - Summarize remaining follow-up work and guardrails for rollout (flags, migrations, monitoring).
