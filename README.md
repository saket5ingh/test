Good investigation. Before implementing anything, I want a more rigorous safety review of your proposed circuit-breaker changes.

The cooldown escalation arithmetic is confirmed, but `owns_probe` may represent intentional half-open circuit-breaker protection rather than a bug.

Please investigate:

1. Trace exactly what happens if `owns_probe` is removed and a recovery probe fails while the failure streak is below the normal threshold. Examine paused_until, probe_owner, recovery_successes and subsequent claim behavior.

2. Determine whether removing it could increase request pressure during a genuine upstream outage. Preserve fail-safe circuit-breaker behavior.

3. Reproduce the full failure and recovery sequence with mocked RemoteProtocolError exceptions in isolated PostgreSQL tests.

4. Test repeated probe failures, different failing tasks, successful probes, session bootstrap success followed by case-history failure, and persistent upstream outages.

5. Evaluate a safer alternative: keep half-open probe failure protection but avoid unnecessary escalation of cooldown_level for isolated or endpoint-specific failures.

6. Verify whether a repeatedly failing CNR or endpoint causes unrelated healthy tasks to be blocked. Do not assume the same task is selected repeatedly.

7. Review the actual HTTP diagnostics around 20:39–20:48, especially connection reuse, request phase, response status, incomplete responses, timeout and TCP connection evidence.

8. Compare possible fixes by reliability, upstream safety, fairness, expected throughput and implementation complexity.

Do not implement or deploy anything yet. Do not contact the live eCourts service.

Provide a minimal recommended change, exact before/after state transitions, regression-test plan and an explicit explanation of how the fix preserves source protection. Do not recommend increasing concurrency or reducing the three-second pacing.
