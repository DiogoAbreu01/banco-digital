# Aurora — Financial Systems Engineering Case Study

**Portfolio case study | Ruby on Rails · PostgreSQL · Redis/Sidekiq · Next.js · React Native/Expo**

Aurora is a *fictional* digital banking simulator designed to test the engineering properties behind reliable financial transactions. It **does not handle real money, credit decisions, bank integrations or actual investments**.

## The engineering challenge

In a concurrent system, the interface may show a transfer as successful while the backend has only applied one side of the transaction. Network retries, terminated database sessions, simultaneous requests and delayed background jobs can produce duplicate or partially applied effects unless transaction boundaries are explicit.

Aurora therefore separates the user-facing experience from a server-authoritative accounting ledger.

```mermaid
flowchart LR
    A["Web (Next.js) / Mobile (Expo)"] --> B["Rails API and authorization"]
    B --> C["Transactional transfer service"]
    C --> D[("PostgreSQL double-entry ledger")]
    B --> E["Redis / Sidekiq scheduler"]
    E --> C
    B --> F["Risk rules, MFA and audit events"]
    D --> G["Balance reconciliation"]
```

## Key engineering choices

| Risk or requirement | Implemented approach |
| --- | --- |
| Partial debit/credit after failures | Atomic double-entry postings in PostgreSQL transactions, with rollback on interruption |
| Duplicate transfer after client retry | Idempotency keys and deterministic handling of already-committed operations |
| Concurrent withdrawals/transfers | PostgreSQL row locks in consistent order, plus constraints and transactional checks |
| Out-of-date sessions during a sensitive operation | Revalidation of session and MFA state inside the financial transaction boundary |
| Background execution and cancellation races | Scheduled work with persistent intent, transactional status and duplicate-job handling |
| Ledger/cache inconsistencies | Reconciliation and audit evidence; privileged corruption detected in a controlled test |
| API session abuse | HttpOnly/SameSite cookies, origin/CSRF validation, refresh rotation and admin MFA |

## Evidence, not just features

The [phase-8 validation record](phase-8-validation.md) documents a local verification with:

- **155 backend RSpec examples, zero failures** against Ruby 3.4.11, Rails 8.1.4 and PostgreSQL 17 in the recorded phase-8 snapshot.
- A controlled connection kill **between debit and credit**, preserving rollback, balances and ledger consistency.
- Real scheduler/Sidekiq tests including a scheduled one-cent transfer; Redis interruption returned HTTP 503 rather than bypassing dependency checks.
- **7 web unit tests**, Next.js build/type checks, and **3 Playwright Chromium flows** using the real containerized API.
- **34 mobile tests** and successful TypeScript checks; Hermes Android/iOS bundles generated with test endpoints.

These counts describe the **recorded phase-8 validation snapshot** rather than an unqualified guarantee about future versions. The newer [combined Aurora CI run](https://github.com/DiogoAbreu01/banco-digital/actions/runs/45097992898) passed **all five jobs** at SHA `3b4cdace61b8839c749e305d4d2bb6a4bb3d06bb`: Rails/RSpec and security scans, web build/tests, browser/API/MFA/XLSX, mobile compatibility/build/tests, and mobile high-severity dependency audit. This does **not** demonstrate physical-device testing or signed app-store release.

The Ruby-security remediation in [PR #25](https://github.com/DiogoAbreu01/banco-digital/pull/25) updated Rails/Rack/Puma and HTML-parsing gems and recorded **155 RSpec examples, zero failures**, zero Brakeman warnings and no vulnerabilities in the Ruby `bundler-audit` snapshot. The [Expo compatibility and npm audit fix in PR #26](https://github.com/DiogoAbreu01/banco-digital/pull/26) was also verified by CI. Use the [current workflow](../.github/workflows/ci.yml) and re-run the suite for future changes.

## Known limitations and security posture

- The application is educational. No real banking, payments, securities trading, scores or personal finance advice.
- The historical phase-8 report recorded **15 high-severity transitive npm advisory findings** in the mobile toolchain. **After** the Expo SDK dependency and lockfile update, the CI high-severity mobile audit passed in [PR #26](https://github.com/DiogoAbreu01/banco-digital/pull/26); these are different dated snapshots. Dependency audits are point-in-time signals, not proof that no risk remains.
- Mobile releases still require signed builds, real device testing, privacy review and security validation. Hermes export output is **not** a signed APK, AAB or IPA.
- This project does not claim a publicly deployed service, app-store releases, physical-device security testing, independent audit, regulatory certification, or production suitability.
- Production operations would require a fresh threat model, backup/restore tests, privacy review, secrets management, distribution signing, monitoring and incident response.

## Reproduction path

Use Docker Desktop/Compose and follow the repository's [README](../README.md) and [Windows guide](guia-windows.md). With Docker installed:

```bash
cp .env.example .env
# Configure only test values; never commit secrets.
docker compose build
docker compose up -d
docker compose run --rm test
```

The actual setup scripts are `scripts/setup.sh` and `scripts/test.sh` (PowerShell counterparts are available). **Do not use personal documents, passwords, bank accounts or real funds as test inputs.**

## Skills illustrated

Transactional backend engineering · ACID data integrity · concurrency and fault injection · secure session design · API contracts · automated tests · web/mobile architecture · explicit failure reporting.

**Engineer:** Diogo Abreu da Silva — Electronic Engineering undergraduate, UNIFEI, Brazil.

**Further reading:** [Architecture and detailed report](relatorio-do-projeto.md) · [Validation](phase-8-validation.md) · [Security](security.md).
