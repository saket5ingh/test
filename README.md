Your queue-admission diagnosis is promising, but I want the deadlock claim verified before implementation.

Reproduce the exact scenario using local synthetic PostgreSQL tests: 10,000 pending tasks, expanding checkpoints, no admitted case tasks and thousands of staged case references.

Measure checkpoint selection frequency, actual batch size, case admission and eventual backlog draining.

Compare safe solutions: drain prioritization, queue-pressure hysteresis, and separate capacity accounting for expanding tasks. Preserve the hard queue bound, task fairness, retry safety and idempotency.

Also verify the role of source failures and whether the five Act-discovery failures are quarantined. Do not assume their error classification.

Provide a minimal proposed fix with regression tests and benchmark results, but do not change production, pacing, concurrency or the running October scan. No deployment or live eCourts requests.
