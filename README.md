Proceed with implementing the bounded circuit-breaker recovery fix based on your completed safety investigation.

### Objective

Improve the scraper's recovery from repeated `RemoteProtocolError` failures without weakening protection during genuine upstream outages.

Preserve the current scraper architecture, admission-capacity fix, durable retries, request pacing and worker concurrency.

### Implementation requirements

1. Implement the bounded probe-hold mechanism with `PROBE_HOLD_LIMIT=0` as the default and a configurable maximum of 3 when explicitly enabled.

2. Add migration `011` for `pcma.scraper_control.probe_holds`, ensuring it is additive, idempotent and compatible with migrations 007–010.

3. Ensure state updates are atomic and read `probe_holds` only after acquiring the appropriate database row lock.

4. A qualifying failed recovery probe should re-arm the cooldown at its current level, release probe ownership and increment the bounded hold counter.

5. After three qualifying consecutive holds, normal cooldown escalation must resume. Genuine source outages, bootstrap failures, throttling and Retry-After must retain appropriate protection.

6. Reset the counter correctly following successful recovery or actual cooldown escalation.

7. Preserve the existing recovery semantics exactly when `PROBE_HOLD_LIMIT=0`.

8. Validate any session-generation assumptions against the real HTTP client. Do not use session bootstrap success alone as proof that the failing endpoint is healthy.

### Tests

Include regression tests for:
- Three bounded holds followed by escalation
- Total upstream outage
- Successful bootstrap followed by case-history failure
- Throttling and Retry-After
- Repeated recovery probes
- Concurrent source reports
- Recovery success and counter reset
- Worker interruption and persisted circuit state
- Exact compatibility with existing behavior at limit 0
- Existing admission-capacity and reliability regressions

Run every test available locally. Clearly identify anything that requires Linux CI.

### Deliverables

- Minimal code changes on a dedicated local feature branch
- Migration 011
- Regression tests
- pyhton patch file 

Also provide the exact read-only production commands needed to verify whether case-history failures dominate over bootstrap failures.

### Restrictions

Do not change global request pacing, worker concurrency, source-failure thresholds or unrelated scraper logic.

Do not contact live eCourts.

Do not modify production, deploy, merge, commit or push anything without my explicit approval.

After preparing the patch, stop and provide a review-ready summary. We will verify production diagnostics and Linux CI before enabling the new behavior.
