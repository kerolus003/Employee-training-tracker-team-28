# Limitations of the Floci simulation

AWS is simulated locally with Floci on Docker. Everything here is written from what we know so far; rows marked "to confirm" are filled with real results after `python scripts/dev.py spike` (see `docs/FLOCI_SPIKE.md`). **Everyone may append rows; keep each row short, with a reason and a proposed improvement.**

| Area | What differs from real AWS | Reason | Proposed improvement on real AWS |
| --- | --- | --- | --- |
| IAM | Floci does not enforce IAM policies by default: forbidden calls are not refused | emulator limitation | deploy the same policies from `infra/iam/` to real IAM; test with the IAM policy simulator |
| Database | A plain Postgres container stands in for RDS | simpler, same SQL | RDS in private subnets, Multi-AZ, automated backups, encryption at rest |
| Storage | Floci runs with `FLOCI_STORAGE_MODE=memory`: data is lost when it restarts | keeps local runs clean | run `python scripts/dev.py init` again (idempotent); persistent mode if needed |
| Daily jobs | EventBridge schedule -> HTTP call may not be supported (to confirm); a fallback cron container (`infra/scheduler`) calls the jobs with a service token | scheduled HTTP targets are not verified in the emulator | EventBridge scheduled rule + API destination with OAuth client credentials (what `init.py` tries to create), CloudWatch alarm on failed invocations |
| Cognito | Groups claim, 15-minute access tokens, JWKS signature check and the client-credentials (service) client confirmed working on Floci (`docs/FLOCI_COGNITO_NOTES.md`); MFA: see the spike results below | emulator coverage of MFA unknown | real Cognito user pool with MFA required for managers, 15-minute access tokens |
| CloudWatch | Alarms may be created but not fire (to confirm) | emulator coverage unknown | real alarms with SNS notification |
| TLS | Self-signed certificate on nginx | local only | ACM certificate on CloudFront / load balancer |
| Secrets | The SendGrid secret holds a placeholder locally, so e-mails are skipped | the real key must never be committed | add the real key by hand to Secrets Manager |

### Logging and monitoring (added with the CloudWatch logging branch)

| Area | What differs from real AWS | Reason | Proposed improvement on real AWS |
| --- | --- | --- | --- |
| CloudWatch alarm | `node scripts/setup_log_alarms.js` creates the metric filter on each log group and the alarm `ett-repeated-auth-failures` (5 or more 401/403 in 5 minutes), but the emulator does not evaluate alarms, so its state stays `OK` even after `scripts/generate_suspicious_activity.js` | emulators store alarms without evaluating metrics | on AWS the alarm moves to `ALARM` and notifies the team through an SNS topic |
| Security log queries | The evidence is read back with `node scripts/security_log_queries.js` (FilterLogEvents), counted per service, path and minute | CloudWatch Logs Insights queries are not emulated | use Logs Insights saved queries and a CloudWatch dashboard |
| Log shipping | Each service sends its JSON logs from inside the process (`services/common/cloudwatch.js`). If CloudWatch is down, that batch is dropped after one warning, and the lines stay in `docker logs` | keeps the services independent of the emulator | on AWS use the awslogs log driver or the CloudWatch agent, with a retention policy per log group |

### Floci spike results (added with the floci-cognito-spike branch)

Real results from `python scripts/dev.py spike` (see `docs/FLOCI_SPIKE.md`).

| Area | What differs from real AWS | Reason | Proposed improvement on real AWS |
| --- | --- | --- | --- |
| S3 pre-signed URL expiry | A link signed for 5 s still downloads after 7 s (HTTP 200): Floci does not enforce `X-Amz-Expires`. Our links are signed for 300 s and the S2 tests check that value, but the real expiry cannot be shown locally | emulator does not check the signature time | on AWS the link stops working after 300 s; show it in the demo with a short expiry |
| S3 anonymous read | An object can be read with a plain GET and no signature (HTTP 200) | Floci does not enforce bucket access rules | Block Public Access ON plus a bucket policy that denies non-TLS; the app only hands out S2's signed links anyway |
| S3 versioning, public access block, default encryption | `init.py` sets them, but reading them back (`GetBucketVersioning`, `GetBucketEncryption`) returns HTTP 500 | not implemented in the emulator | check them with `get-bucket-versioning`, `get-public-access-block`, `get-bucket-encryption` on real AWS |
| EventBridge Scheduler | Fixed: `rate(1 days)` was rejected (AWS wants `rate(1 day)`); with the fix `create_schedule` is accepted. Scheduler targets are AWS APIs, not HTTP endpoints | AWS design | EventBridge rule + API destination (OAuth client credentials); locally the cron container |
| Cognito MFA (TOTP) for managers | Only the pool-level TOTP setting is confirmed on Floci; enrolment, the sign-in challenge and rejecting a wrong code are not verified yet (`cognito_spike.js` checks them) | emulator coverage unknown; spike not run yet | MFA Required on the pool (Cognito cannot require it for one group), or a separate user pool for managers |
| Cognito GlobalSignOut | Not verified yet on Floci. On any platform, services check JWTs offline, so a signed-out access token keeps working until it expires | JWTs are self-contained | keep the 15-minute access token; GlobalSignOut revokes the refresh token on AWS |

### Service 1 (added with the kerolus/limitations-s1 branch)

| Area | What differs from real AWS | Reason | Proposed improvement on real AWS |
| --- | --- | --- | --- |
| Cognito issuer | Tokens carry `iss = http://floci:4566/<pool>`, a name that only resolves inside Docker; checks from the host use `localhost:4566` | Floci uses its Docker host name | real issuer `https://cognito-idp.<region>.amazonaws.com/<pool>` reachable everywhere |
| Floci restart (S1) | Floci starts empty, so the Cognito users and the secret `ett/db/s1` disappear; Service 1 then refuses to start (fail closed) until `put-db-secret.js` and `cognito_init.js` are run again | `FLOCI_STORAGE_MODE=memory` | managed Cognito and Secrets Manager keep their data; secret rotation instead of a script |
| Database TLS | Services talk to the Postgres container without TLS locally (`DB_SSL=false`) | local Docker network only | RDS with `DB_SSL=true` (certificate verified), already supported by `services/common/db.js` |
| IAM pattern (S1) | `infra/iam/s1-policy.json` allows `secret:ett/db*`, which also matches the admin secret `ett/db` | written before per-service secrets existed | narrow to `ett/db/s1*` (and the same per service) |
| Forbidden log lines | A 403 caused by a wrong role is logged without the caller's `userSub`/`roles` (the role check rejects before the user is recorded) | order of checks in `services/common/auth.js` | record `req.user` before the role check, so investigations show who tried |
| Training edits | Editing a training (PUT, creator only) keeps no change history | kept simple for the coursework | an audit table for training changes, like `audit_log` for certificates |
