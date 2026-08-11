# Beszel

Beszel provides a monitoring overview for systems managed by a Beszel instance. The integration fetches system details and historical statistics, then calculates a health score and usage warnings for the widget.

## Files

- `integration.yaml` contains the Dashwise definition, API requests, mappings, health calculations, and widget configuration.
- `beszel.md` documents the integration for contributors and users.

## Configuration

- `URL` — required Beszel URL.
- `PB_USER` and `PB_PASS` — required PocketBase superuser credentials used to retrieve a token.
- `THRESHOLD_*` settings — optional CPU, memory, disk, and load warning thresholds.
- `TOKEN` — managed internally after authentication.

The widget links to the Beszel instance and individual system pages, and reports system health, status, usage, load, and warnings.
