# Kerolus: individual proof of work (Employee Training Tracker, Group 28)

This is my personal repository: a fork of the team repository, [karikasaad/Employee-training-tracker-team-28](https://github.com/karikasaad/Employee-training-tracker-team-28).

My branch on the team repo is [`kerolus`](https://github.com/karikasaad/Employee-training-tracker-team-28/tree/kerolus). All my work was done on my own branches here (`kerolus/...`). Each one was merged into the team repo through a reviewed pull request. Every commit is authored by **kerolous** (`kerlos245671@bue.edu.eg`).

**My part:** the shared foundation (database, API contract, authentication, common code) and **Service 1: Employee & Training** (backend, frontend pages, security tests, AWS integration).

## My branches → pull requests in the team repo

| My branch (this repo) | What it contains | Team PR |
|---|---|---|
| [`kerolus/db-schema-seed`](../../tree/kerolus/db-schema-seed) | PostgreSQL schema (9 tables, constraints), least-privilege roles `svc_s1/2/3`, seed data, 11 DB checks | [#2](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/2) |
| [`kerolus/api-contract`](../../tree/kerolus/api-contract) | API contract for all 3 services + ER diagram | [#3](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/3) |
| [`kerolus/cognito-auth-module`](../../tree/kerolus/cognito-auth-module) | Cognito JWT check used by every service (RS256/JWKS, issuer, client, token_use, expiry → 401/403), Cognito provisioning script | [#4](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/4) |
| [`kerolus/common-db-logging`](../../tree/kerolus/common-db-logging) | parameterised-SQL guard, JSON logging with secret redaction, one error format | [#5](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/5) |
| [`kerolus/s1-lookup-teams`](../../tree/kerolus/s1-lookup-teams) | S1: employee lookup + team endpoints (used by S2 and S3) | [#14](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/14) |
| [`kerolus/s1-trainings`](../../tree/kerolus/s1-trainings) | S1: trainings create / list / get / edit | [#17](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/17) |
| [`kerolus/s1-assignments-overdue`](../../tree/kerolus/s1-assignments-overdue) | S1: assignments, Overdue rule (calculated on read, UTC), team view | [#20](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/20) |
| [`kerolus/s1-complete-renewal`](../../tree/kerolus/s1-complete-renewal) | S1: employee completes own assignment + automatic renewal (race-safe) | [#21](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/21) |
| [`kerolus/s1-team-views`](../../tree/kerolus/s1-team-views) | S1: team summary for the manager page | [#23](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/23) |
| [`kerolus/s1-frontend`](../../tree/kerolus/s1-frontend) | S1 pages: My trainings, Team trainings | [#24](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/24) |
| [`kerolus/s1-security-tests`](../../tree/kerolus/s1-security-tests) | 21 security tests (S1-SEC-01..21) + evidence table generated from a real run | [merged (see my PRs)](https://github.com/karikasaad/Employee-training-tracker-team-28/pulls?q=is%3Apr+author%3Akerolus003) |
| [`kerolus/floci-cognito-notes`](../../tree/kerolus/floci-cognito-notes) | real Cognito results on Floci (`docs/FLOCI_COGNITO_NOTES.md`) | [merged (see my PRs)](https://github.com/karikasaad/Employee-training-tracker-team-28/pulls?q=is%3Apr+author%3Akerolus003) |
| [`kerolus/s1-aws-services`](../../tree/kerolus/s1-aws-services) | S1 reads its DB login from **Secrets Manager**, sends logs to **CloudWatch Logs**, Node 22 image | [#33](https://github.com/karikasaad/Employee-training-tracker-team-28/pull/33) |
| [`kerolus/limitations-s1`](../../tree/kerolus/limitations-s1) | `docs/LIMITATIONS.md`: confirmed Cognito results and Service 1 differences from real AWS | [my PRs](https://github.com/karikasaad/Employee-training-tracker-team-28/pulls?q=is%3Apr+author%3Akerolus003) |

## AWS services used by my component (deployed on Floci)

| AWS service | How Service 1 uses it | Evidence |
|---|---|---|
| Amazon Cognito | every request's access token is verified (JWKS signature, issuer, client, expiry); groups `Employee`/`Manager` are the roles | `docs/FLOCI_COGNITO_NOTES.md` |
| AWS Secrets Manager | at start-up S1 reads its database login (`svc_s1`) from `ett/db/s1`; it refuses to start without the secret and refuses an admin login | log line `db_login_loaded … Secrets Manager ett/db/s1` |
| Amazon CloudWatch Logs | all S1 log lines go to `/ett/s1`; 401/403 are `auth_failure` / `forbidden` (warn) with secrets redacted | `aws logs filter-log-events --log-group-name /ett/s1 --filter-pattern forbidden` |
| AWS IAM (design) | `infra/iam/s1-policy.json`: own secret and own log group only | policy file (Floci does not enforce IAM) |

## Tests (from real runs on my machine)

- **Service 1:** `npm test` → **111 / 111 pass** (API tests on real PostgreSQL as `svc_s1`, rule tests, page tests, AWS SDK tests).
- **Security suite:** `npm run test:security` → **21 / 21**, in [`docs/evidence/s1/security_results.md`](https://github.com/karikasaad/Employee-training-tracker-team-28/blob/main/docs/evidence/s1/security_results.md).
- **Shared code:** `services/common` → **46 / 46**.
- **Database checks:** T-DB1..T-DB11 → PASS.

## Where my code lives

| Folder | What |
|---|---|
| `infra/db/` | schema, roles, seed, checks, loader |
| `infra/floci/cognito/` | Cognito provisioning |
| `services/common/` | auth, db, logging, errors |
| `services/employee-training/` | Service 1 |
| `frontend/s1/` | Service 1 pages |
| `docs/API_CONTRACT.md`, `docs/ER_DIAGRAM.md` | contract and data model |

*AI assistance was used as permitted by the module. Every change was reviewed, run and committed by me.*

---

The original team project README follows.

# Employee Training Tracker (Group 28)

A small web app where a company tracks mandatory employee training and certificates. Employees complete trainings and upload certificates; managers assign training, verify certificates and see who is compliant. Cloud Security coursework: the AWS design (Cognito, S3, RDS, Secrets Manager, CloudWatch, EventBridge) is simulated locally with Floci on Docker.

## Quick start (Windows PowerShell, Docker Desktop required)

```powershell
git clone <team repo url>
cd Employee-training-tracker-team-28
python scripts/dev.py up
python scripts/dev.py health
```

Full instructions and troubleshooting: [docs/DEV_SETUP.md](docs/DEV_SETUP.md).

## Folders

| Folder | What | Owner |
| --- | --- | --- |
| `services/employee-training` | Service 1: teams, employees, trainings, assignments (Node.js + Express) | Kerolus |
| `services/certificate-compliance` | Service 2: certificates, validity, flags, reminders (Node.js + Express) | Karim |
| `services/dashboard-reports` | Service 3: compliance dashboard and reports (Node.js + Express) | Yahia |
| `frontend` | plain HTML/CSS/JS served by nginx | everyone (own pages) |
| `infra` | nginx, Floci setup, IAM policies, scheduler | Karim |
| `docs` | setup, limitations, IAM, report | everyone |

`.env` is local only and never committed; see `.env.example`.
