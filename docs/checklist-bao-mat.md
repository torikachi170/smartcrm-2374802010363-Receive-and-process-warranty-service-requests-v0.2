# Security Self-Assessment — Smart CRM L2 (Track SE)

This checklist is a prototype-level self-assessment, not a penetration test.
Each status must be kept truthful and supported with current command output,
test results, or screenshots under `docs/screenshots/`. Never put `.env`,
passwords, tokens, or real personal data in a screenshot.

Last full-suite run: 16 November 2026, `python -m pytest test -q` reported
**39 passed, 0 failed, 0 skipped** with one deprecation warning. The
PostgreSQL integration tests also passed: **13 passed**. On 15 November, a
clean clone had healthy API and PostgreSQL services; `/health` returned
`status=ok` and `/login` returned HTTP 200. Screenshots for container health
and these HTTP checks are saved as `C1-DOCKER-HEALTH-2026-11-15.png` and
`C1-HTTP-CHECK-2026-11-15.png`. Screenshots for the full suite and the direct
PostgreSQL query are still pending.

| # | Required check | Status | Evidence / remaining work |
|---|---|---|---|
| 1 | No secrets in the repository or Git history | **PARTIAL — no `.env` file found, full secret scan pending** | `.env` and `.env.local` are ignored, `.env.example` is allowed, `.env` is not tracked, and `git log --all -- .env` returned no commits. Source pattern scans found no shell execution, raw HTML rendering markers, or f-string/concatenated SQL construction in `src`. Still review all tracked files, Docker build context, and final screenshots for accidental credentials before release. |
| 2 | Input is validated in the application, not only by the database | **DONE for implemented endpoints; retain evidence** | Pydantic request models and `CustomerService` validate fields and bounds. `test/test_classify_api.py` exercises invalid descriptions and 10/2000/2001-character boundaries; customer tests cover duplicate phone and delete guards. The targeted source scan found no f-string/concatenated SQL construction patterns in `src`; retain current test output and review manually. |
| 3 | Access control limits users to data they are authorized to access | **NOT IMPLEMENTED** | The prototype has one environment-configured demo account and bearer-token checks, but no real employee accounts, roles, or per-owner authorization. Do not claim owner-based authorization. This limitation is also stated in README. |
| 4 | Passwords and sensitive stored data are hashed or encrypted | **NOT IMPLEMENTED for demo credential storage** | No user-password table exists. The demo password is supplied as a plaintext environment value and compared by the prototype; `.env` must remain local and ignored. This is not a production password-storage design. |
| 5 | Error responses do not expose stack traces, table names, or connection strings | **PARTIAL** | The API maps validation, authentication, and database exceptions to structured responses; database errors return a generic message. Add current negative-response evidence and verify an unavailable-database response before marking complete. |
| 6 | Logs do not record secrets or unnecessary personal data | **PARTIAL — code reviewed, runtime evidence pending** | `src/presentation/logging_config.py` records request ID, method, path, status, duration, and message; request bodies and authorization headers are not included by the request middleware. Capture a real request and inspect `docker compose logs` before marking complete. |
| 7 | Dependencies are pinned and basic vulnerability checks are recorded | **DONE for the 15/11/2026 audit snapshot** | `requirements.txt` uses exact `==` pins and `requirements.lock` is committed. The initial audit found pytest 8.4.2 (PYSEC-2026-1845; fixed in 9.0.3) and opentelemetry-api 1.45.1 (CVE-2026-126360; fixed in 1.45.2). Updated both in commit `dc205a6`; `pip check` passed and `pip-audit -r requirements.txt` plus `pip-audit -r requirements.lock` both reported no known vulnerabilities on 15/11/2026. Repeat the audit before release because its database changes over time. |

## Track SE checks to complete

- Confirm the JWT key is supplied through environment configuration and meets
  the minimum length enforced by `AuthService`; never print its value.
- Confirm token verification checks the signature and explicitly allows HS256;
  the implementation uses `jwt.decode(..., algorithms=["HS256"])`.
- Keep token, password, and key values out of URL query strings.
- Do not add a permissive CORS configuration. If browser cross-origin access
  is introduced, use an environment-based origin allowlist; CORS does not
  replace authentication.
- The prototype runs locally without HTTPS. For real deployment, place it
  behind a TLS-terminating proxy and document forwarded-protocol handling.
- Run the three source searches recommended by the course material (SQL
  concatenation, raw HTML injection, and shell-command execution), review every
  match, and attach their outputs.

## Evidence to attach before submission

1. Sanitized output of `.env` ignore/history checks and the source scan.
2. Current test results for invalid input, unauthorized access, and data
   integrity cases.
3. Sanitized API error responses and application logs.
4. `pip check` and the 15/11/2026 package vulnerability-audit result; repeat before release.
5. Docker health and HTTP evidence are saved in `docs/screenshots/`; capture
   the full-suite and direct PostgreSQL query output before marking the
   evidence complete.

Mark every item as **DONE**, **NOT DONE**, or **NOT APPLICABLE** only after
checking the evidence. An honest **NOT DONE** with a short explanation is
preferable to an unsupported **DONE**.
