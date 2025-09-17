# Bugfix Playbook

1. Reproduce
   - Capture the failing scenario with exact inputs, environment, and observed output; automate with a minimal test when possible.
   - Verify the issue is still present on the current `main`/`develop` branch before investing in the fix.
2. Diagnose
   - Trace the code path with logs, breakpoints, or reasoning; check recent commits or configs for regressions.
   - Inspect assumptions (nullability, concurrency, feature flags) and probe edge cases adjacent to the failure.
3. Contain blast radius
   - Plan a fix that targets the root cause without collateral behavior changes; avoid speculative refactors.
   - Add regression tests that fail prior to the fix and pass afterward.
4. Implement & verify
   - Apply the minimal code change; re-run the new and existing tests to confirm no unintended impact.
   - If full validation is expensive, document what was skipped and why.
5. Communicate clearly
   - Summarize root cause, fix approach, and validation evidence in the final response.
   - Suggest monitoring, alerting, or guard-rail follow-ups if the bug points to deeper systemic issues.
