# Aurora Ruby dependency security maintenance — November 2026

**Scope:** Rails backend only. This note does **not** certify that Aurora is safe for real banking or real-money use. The app is a financial simulator.

## Why this change exists

The GitHub Actions `bundler-audit` job identified advisories in the previous lockfile (Rails 8.1.4, Rack 3.2.7, Puma 8.0.2 and several HTML/XML parsing libraries). This is a **dependency-security remediation**, not an attempt to hide or suppress advisories.

A one-time GitHub Actions job used the repository's **Bundler 2.6.9** and **Ruby 3.4.11** to re-resolve the patched Ruby dependency graph and commit the resulting lockfile. The temporary resolver workflow was removed from its own branch in the same commit.

## Version changes selected by the resolver

| Package | Previous | Resolved version |
| --- | --- | --- |
| Rails and Rails components | 8.1.4 | 8.1.4.1 |
| Rack | 3.2.7 | 3.2.9 |
| Puma | 8.0.2 | 8.1.2 |
| rails-html-sanitizer | 1.7.1 | 1.7.2 |
| Loofah | 2.25.2 | 2.25.3 |
| Nokogiri | 1.19.4 | 1.19.7 |
| Crass | 1.0.7 | 1.0.8 |

The actual `Gemfile.lock` is authoritative for the version resolved on each listed platform. The `Gemfile` raises the permitted minimum Rails/Puma patch versions. The solver selected newer compatible Puma/Nokogiri patch/minor releases than the original proposed minimum.

## Required validation and risks

Run **all** Rails-related checks in the normal GitHub Actions workflow for the exact final commit:

1. Frozen `bundle install` with the committed lockfile (no unexpected re-resolution).
2. Rails/RSpec tests against disposable PostgreSQL and Redis CI services, including financial ledger, idempotency and concurrency cases.
3. `brakeman` static security scanning and `bundler-audit check --update` (zero reported advisories for the Ruby dependency snapshot).
4. Rebuild Docker, then verify runtime behavior before any production deployment.

**Mobile blocking advisories are independent.** Existing Expo/React Native tooling reports high-severity transitive npm vulnerabilities; the mobile security job must remain blocking and must not be suppressed to obtain a green pipeline. The mobile package compatibility job also needs separate remediation.

A scan is a point-in-time check. It does not replace a penetration test, code review, production threat model, or regulatory/security certification. Do not present the overall pipeline as passing unless every required job passes.
