# Changelog

## v1.1.0 — 2026-06-09

### Added

- **option10 / option11** — Product-level options for the Portal Terms of Service URL and
  Portal Privacy Policy URL. Both are PATCHed onto the new org immediately after `Create()`
  succeeds (non-fatal — provisioning is not rolled back on PATCH failure).
- **`$buttons` property** — `'Resend Welcome Email' => 'resendWelcome'` and
  `'Resync Status' => 'resyncStatus'` buttons appear on the HostBill admin service page.
- **`resendWelcome(): bool`** — Mints a fresh SSO token, validates the `https://` scheme,
  and resends the welcome email via the HostBill `Emails` component (falls back to `@mail()`).
- **`resyncStatus(): bool`** — Calls `orgSummary()` and surfaces zone/sub-client counts as an
  admin info message.
- **`ssoLogin(): void`** — Mints an SSO token and performs a `302` redirect to the validated
  `https://` login URL. Called by HostBill when the client clicks the SSO link.
- **`getUsage(): array`** — Returns `['disk' => active_zones, 'bandwidth' => sub_clients]`
  for HostBill's usage graph. Non-fatal — returns zeros on API failure.
- **`getServiceDetails(): string`** — Returns a minimal HTML table (Org ID, status, plan name,
  zones, sub-clients, API calls, module version) for the HostBill admin service detail panel.
  All values escaped with `htmlspecialchars()`.
- **`clientArea(): string`** — Returns a self-contained HTML block with an SSO login button
  and live usage summary (zones, sub-clients). Non-fatal if `orgSummary()` fails; shows just
  the button. Suspended orgs receive a plain-text note.
- **`driftSync(): array`** — Compares PanelDNS org statuses against a caller-supplied
  `['org_id' => int, 'status' => string]` map and returns mismatched pairs. Intended for
  HostBill Task Scheduler integration.
- Version bumped to `1.1.0`.

## v1.0.0 — 2026-06-08

Initial release.

### Added

- `class.paneldns.php` — HostBill `HostingModule` extending the PanelDNS Platform API.
  - `connect()` — initialises `PanelDnsApiHb` from App config; includes SSRF pre-flight.
  - `testConnection()` — validates Base URL reachability and API key via `/ping` + `/plans`.
  - `Create()` — provisions a new PanelDNS reseller org; sends welcome email with SSO link.
  - `Suspend()` / `Unsuspend()` — delegate to `/orgs/{id}/suspend|unsuspend`.
  - `Terminate()` — deletes org immediately or suspends with a configurable grace period.
  - `ChangePackage()` — switches the org's plan via `/orgs/{id}/plan`.
  - `getPlans()` — loadable field helper for the Plan ID product option.
- `shared/PanelDnsApi.php` — cURL HTTP client for PanelDNS Platform + Reseller APIs.
  - IPv4-only SSRF guard (`CURLOPT_IPRESOLVE_V4` + response-IP check).
  - TLS verification on by default; per-server override flag.
  - Pluggable logger callback — no hard dependency on HostBill's activity log.
- `shared/LicenceCheck.php` — file-based licence cache (no WHMCS dependency).
  - 7-day grace period for `past_due` subscriptions.
  - 2-day stale hard-lock if PanelDNS is unreachable.
- `.github/workflows/release.yml` — GitHub Actions workflow that builds a HostBill-ready
  ZIP on tag push and attaches it to a GitHub Release.
- `README.md` — installation guide, App config reference, product option reference,
  security notes, and local build instructions.
