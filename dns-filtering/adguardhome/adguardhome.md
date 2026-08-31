# AdGuard Home

AdGuard Home provides DNS query and filtering statistics from an AdGuard Home instance, including total queries, blocked queries, blocked percentage, and known clients.

## Files

- `integration.yaml` contains the AdGuard Home API requests, response mappings, computed statistics, and overview widget.
- `adguardhome.md` documents the integration for contributors and users.

## Configuration

- `URL` — required base URL of the AdGuard Home instance, without a trailing slash (for example, `http://adguardhome:3000`).
- `BASIC_AUTH` — required Base64-encoded `username:password` credentials for AdGuard Home HTTP Basic Authentication.

The integration queries `/control/stats` and `/control/clients`. The overview widget refreshes every 30 seconds and links to the configured AdGuard Home URL.
