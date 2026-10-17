Your bounded recovery proposal looks promising, and I agree that simply removing `owns_probe` is unsafe.

Before implementing, please complete these checks:

1. Verify what `error.session_generation is not None` guarantees. Does it prove successful bootstrap in the same request attempt, or can it reference an earlier session?
2. Do not claim the 12 failed production tasks were distinct unless actual production events confirm their task IDs.
3. Provide read-only commands to extract the real HTTP diagnostics around 20:39–20:48 IST, including connection phases, timeout evidence and failure endpoints.
4. Verify that `probe_holds` is updated atomically, bounded at 3 and reset correctly on successes and genuine escalations.
5. Add regression tests for bootstrap success followed by case failure, total source outage, correlated network failures, throttling, Retry-After and repeated half-open failures.
6. Confirm that `PROBE_HOLD_LIMIT=0` preserves the existing behavior exactly.
7. Run complete Linux CI and provide a migration 011 deployment and rollback plan.

Do not change production, disable the circuit breaker, reduce the 3-second request interval, increase concurrency or deploy anything yet.

Give me a final go/no-go recommendation based on production evidence and test results. Distinguish the simulated 92% improvement from a reliable production forecast.
