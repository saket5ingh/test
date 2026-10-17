Proceed with fixing test isolation first.

Use independent PostgreSQL databases/schemas for each test, verify counter consistency, and rerun all 10 regression tests together.

Then:
1. Confirm the admission-starvation reproduction and proposed fix remain valid.
2. Verify no task loss, duplicates, starvation or queue-bound violations.
3. Assess growth of parked checkpoint rows and whether a separate hard row limit is required.
4. Review migration 010, database performance and compatibility with existing reliability fixes.
5. Run the complete backend and deployment test suites.
6. Provide the exact patch, measured results, remaining risks and merge-readiness verdict.

Do not increase concurrency or change 3-second pacing.
