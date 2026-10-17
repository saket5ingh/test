The PostgreSQL reproduction and 13/13 regression results look good.

Proceed with the final production-readiness stage for the admission-capacity fix.

1. Apply the corrected implementation to the actual repository on a dedicated feature branch. Do not use the outdated proposed_fix.patch.
2. Verify migration 010 and its compatibility with migrations 007–009, including locking behavior when creating the index on an active PostgreSQL table.
3. Run the complete Linux CI pipeline: backend, migration, deployment, frontend tests, typecheck, lint and build.
4. Confirm the 13 admission regression tests remain green.
5. Review the change for transaction races, deadlocks, task duplication, checkpoint loss and worker recovery after interruption.
6. Document the new admitted-work limit versus total queued database rows, and identify safe monitoring thresholds.
7. Prepare a controlled deployment and rollback plan that preserves the currently running October scan, its staged references, leases and checkpoints.
8. Provide the final git diff, test results, unresolved risks and exact deployment procedure.

Do not increase concurrency or change the 3-second global request spacing.

Do not touch production, restart workers, commit, push, merge or deploy without my explicit approval.

At the end, tell me whether this is genuinely ready for a production release and what downtime or temporary interruption the deployment would require.
