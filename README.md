# Deep Root-Cause Investigation — eCourts Source Failures and Circuit-Breaker Recovery

Perform an independent, comprehensive investigation of the recurring `RemoteProtocolError` and global source circuit-breaker cooldowns in our eCourts scraper.

The recent admission-capacity fix has been deployed successfully. However, the scraper continues to experience repeated network failures and increasingly long global cooldowns, severely reducing case collection.

## 1. Observed production evidence

All times are IST, 17 October 2026.

- 20:39:20 — Task retry and source cooldown
- 20:41:10 — Task retry and source cooldown
- 20:43:14 — `caseHistoryWebService.php: network failure (RemoteProtocolError)`
- 20:43:14 — Circuit opened at level 3, failure streak 3, paused until 20:47:14
- 20:47:14 — Case task started, attempt 5
- 20:48:24 — Same endpoint reported `RemoteProtocolError`
- 20:48:24 — Circuit reopened at level 4, failure streak 4, recovery successes 0, paused until 20:56:24
- Source retry alert: unavailable after 5 attempts; task retained for delayed retry

The source transition shows recovery probe ownership, but recovery repeatedly fails.

Other observations:
- Global request spacing: 3 seconds
- Worker concurrency: 1
- Source failures previously reached 20.9% in a rolling hour
- Global cooldowns repeatedly interrupt all source requests
- Saved case count at the latest snapshot: 6,153
- Staged child references: 26,788
- Failed tasks: 72

## 2. Investigate the real HTTP failure

Trace the entire request path for `caseHistoryWebService.php`, including:

- HTTP client construction and connection pooling
- Persistent connections and keep-alive reuse
- TCP connection establishment and reuse
- Request/response timeout configuration
- Session lifecycle, cookies and token refresh
- Redirects, response parsing and exception handling
- Retry timing and global pacing interaction

Determine why `RemoteProtocolError` may occur and what evidence would distinguish:

1. Stale keep-alive connection reuse
2. Upstream server or reverse-proxy disconnections
3. Request timeout or connection termination
4. Session expiration or invalid authentication state
5. Throttling or protective controls
6. HTTP client implementation or lifecycle bugs

Do not assume upstream throttling or a stale connection without evidence. Inspect existing request traces and determine whether the connection had actually been established.

## 3. Deep investigation of SourceGate and circuit breaker

Trace the complete state machine from failure to recovery.

Specifically determine:

- Why each recovery attempt increments the failure streak and cooldown level
- Whether recovery probes repeatedly select the same failed case task
- Whether one unhealthy case-history endpoint can block otherwise healthy court/search requests
- How probe ownership, leases, success counters and cooldown expiration interact
- Whether any successful request properly advances recovery
- Whether the circuit is opening too aggressively for isolated failures
- Whether exponential cooldown behavior is correct
- Whether worker restarts or persisted state can affect recovery
- Whether a task retained for repeated retries can cause an indefinite recovery loop

Reconstruct the 20:39–20:48 sequence from the implementation and available logs.

**Most important question: Is a repeatedly failing individual case request causing the global circuit breaker to pause the entire nationwide scan unnecessarily?**

Do not assume the repeated errors involve the same CNR or task without verifying task IDs.

## 4. Production evidence and measurements

Identify exactly what additional evidence is needed.

If read-only production access is available, inspect existing database records and application logs without modifying anything or contacting eCourts.

Otherwise provide precise read-only SQL queries or commands for me to execute.

Prioritize:
- Failed endpoint distribution
- Error kinds and root exception types
- Task IDs associated with consecutive failed probes
- Source circuit transitions
- Retry attempts and cooldown durations
- Recovery probe selection
- Successful requests between failures
- HTTP connection and timeout traces

Explain what the current evidence proves and what remains unknown.

## 5. Evaluate safe solutions

For each confirmed issue, recommend a minimal reliable fix.

Consider whether improvements are needed in:

- HTTP connection management
- Session refresh and validation
- Recovery-probe task selection
- Endpoint-specific versus global circuit protection
- Retry and cooldown behavior
- Failure classification
- Recovery success accounting

Do not weaken genuine upstream protection or bypass CAPTCHA, authentication, rate limits or access restrictions.

Preserve durable retries, task leases, existing case data, checkpoints and the admission-capacity fix.

## 6. Testing and expected output

Provide:

1. Executive summary of the actual problem
2. Confirmed findings with file names, functions and line references
3. Timeline explaining the recovery failures
4. Root causes ranked by confidence and impact
5. Reproduction tests using mocked HTTP failures and isolated PostgreSQL
6. Minimal proposed code changes, if supported by evidence
7. Regression tests covering repeated failed probes, successful recovery, and unaffected work
8. Risks, monitoring recommendations and a safe rollout plan
9. Clear final recommendation identifying the highest-priority fix

Distinguish confirmed findings, hypotheses and unverified possibilities.

## Strict constraints

**Investigation first. Do not modify the code, production database, worker configuration or running scraper.**

Do not make live eCourts requests, restart workers, alter circuit state, change the 3-second pacing, increase concurrency, commit, push or deploy.

If a code fix is appropriate, describe it and its tests first. Wait for my explicit approval before implementing.

Our goal is to restore reliable, sustained case collection without increasing request pressure or compromising source protection.
