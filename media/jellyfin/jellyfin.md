# Jellyfin

Jellyfin provides a media-library overview from a Jellyfin server, including recently added or relevant media items configured through the integration settings.

## Files

- `integration.yaml` contains the Jellyfin API request, item mapping, and widget or shortcut configuration.
- `jellyfin.md` documents the integration for contributors and users.

## Configuration

- `URL` — required base URL of the Jellyfin instance.
- `TOKEN` — optional Jellyfin API token.
- `INCLUDE_ITEM_TYPES` — optional comma-separated list of Jellyfin item types.
- `LIMIT` — optional maximum number of items to fetch.
