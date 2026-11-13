# Deployment Guide — Smart CRM L2 (Track SE)

This guide describes the implemented prototype in this repository. It does not
describe warranty-ticket creation or ticket lifecycle endpoints because those
features are not implemented.

## 1. Requirements

- Windows 10/11 with PowerShell, or a supported macOS/Linux host.
- Git installed and available in the terminal.
- Docker Desktop with the Linux container engine running, or Docker Engine with
  Docker Compose v2.
- Internet access for the first image/dependency downloads.
- Host ports `8000` and `5432` available by default. Change `APP_PORT` or
  `DB_PORT` in `.env` if either port is occupied.
- No host Python or PostgreSQL installation is needed to run the application
  with Docker Compose.

## 2. Setup steps

Run these commands in PowerShell:

```powershell
git clone https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2.git
Set-Location .\smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose up -d --build
```

The two setup commands after cloning are creating `.env` from the sample (only
if it does not already exist) and running `docker compose up -d --build`.
Configuration validation is optional:

```powershell
docker compose config --quiet
```

After the `v1.0` release tag has been created and pushed, a release checkout
can replace the clone command with:

```powershell
git clone --branch v1.0 https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2.git
Set-Location .\smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2
```

Then copy `.env.example` to `.env` and run `docker compose up -d --build`. On
macOS/Linux, use `cp .env.example .env` instead of `Copy-Item`.

The sample `.env.example` values are for local demonstration only. Replace
sample passwords and keys before any non-local deployment. Never commit or
share `.env`. The database container initializes its schema and sample
categories on the first start with a new volume.

To stop the application without removing its data:

```powershell
docker compose down
```

`docker compose down -v` deletes the PostgreSQL volume and all data in it. Use
that command only when deliberately resetting a disposable demo environment.

## 3. Environment variables

Values below are examples from `.env.example`; they are not production
credentials.

| Name | Meaning | Sample value | Required |
|---|---|---|---|
| `DATABASE_URL` | Local application connection string; Compose supplies the API container connection internally | `postgresql+psycopg://smart_crm:change-this-local-password@localhost:5432/smart_crm_l2` | Local development |
| `TEST_DATABASE_URL` | Separate PostgreSQL database used by integration tests | `postgresql+psycopg://smart_crm:change-this-local-password@localhost:5432/smart_crm_test` | Required to run PostgreSQL tests |
| `POSTGRES_DB` | Database created by the PostgreSQL container | `smart_crm_l2` | Yes |
| `POSTGRES_USER` | Database user | `smart_crm` | Yes |
| `POSTGRES_PASSWORD` | Local database password | `change-this-local-password` | Yes |
| `DB_PORT` | Host port mapped to PostgreSQL port 5432 | `5432` | No; defaults to `5432` |
| `APP_PORT` | Host port mapped to API port 8000 | `8000` | No; defaults to `8000` |
| `DEMO_USERNAME` | Demo login name | `demo` | No; defaults to `demo` |
| `DEMO_PASSWORD` | Demo login password | `change-this-demo-password` | Yes |
| `JWT_SECRET_KEY` | Key used to sign access tokens; use at least 32 characters | `replace-with-a-random-secret-at-least-32-characters` | Yes |
| `JWT_ACCESS_TOKEN_MINUTES` | Access-token lifetime in minutes | `30` | No; defaults to `30` |

Do not reuse the sample values on a public or production deployment. Keep real
values in an ignored local `.env` file or the deployment platform's secret
settings.

## 4. Sample data

On the first start of an empty `postgres_data` volume, Compose mounts
`db/schema.sql` and `db/seed.sql` into the PostgreSQL initialization directory.
The seed inserts six issue categories: `MAN_HINH`, `PIN`, `SAC`, `PHAN_MEM`,
`NUOC_VAO`, and `KHAC`. The seed uses `ON CONFLICT DO NOTHING`, so applying it
again does not duplicate those categories.

The seed does not create sample customers, devices, or warranty tickets. The
prototype does not implement ticket APIs. The application uses the configured
demo account for sign-in.

## 5. Verify it works

Check that both Compose services report `healthy`:

```powershell
docker compose ps
```

Expected: `db` and `api` both show `healthy`.

Check the API health endpoint:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

Expected response:

```text
status
------
ok
```

The `/health` endpoint confirms that the API process responds; it does not
query PostgreSQL. Confirm database readiness separately in `docker compose ps`.

Open `http://127.0.0.1:8000/login` for the demo UI and
`http://127.0.0.1:8000/docs` for the interactive API documentation.

For developer tests, install Python 3.14 and the pinned dependencies on the
host, configure `TEST_DATABASE_URL` to a disposable PostgreSQL test database,
then run:

```powershell
python -m pip install -r requirements.lock
python -m pytest test -v
```

Do not point `TEST_DATABASE_URL` at a database containing real or valuable
data. The integration tests create and drop an isolated test schema.

### Troubleshooting

- Docker commands fail to connect: start Docker Desktop and wait for its engine
  to become ready.
- Compose reports a missing variable: confirm `.env` was copied from
  `.env.example` in the repository root.
- A host port is occupied: change `APP_PORT` or `DB_PORT` in `.env` and rerun
  `docker compose up -d`.
- The app starts before PostgreSQL is ready: inspect `docker compose ps` and
  `docker compose logs --tail 100 api db`; Compose is configured to wait for
  the database health check.
- Schema or seed changes do not appear in an existing volume: PostgreSQL
  initialization scripts run only for a new data volume. For disposable demo
  data only, stop with `docker compose down -v` and start again.
