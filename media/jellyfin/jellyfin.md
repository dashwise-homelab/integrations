# Jellyfin

Jellyfin provides a media-library overview from a Jellyfin server, including recently added media items configured through the integration settings. The `/Items` response is read from its `Items` array and mapped for widget and shortcut output.

## Files

- `integration.yaml` contains the Jellyfin API request, item mapping, and widget or shortcut configuration.
- `jellyfin.md` documents the integration for contributors and users.

## Configuration

- `URL` — required base URL of the Jellyfin instance.
- `TOKEN` — required Jellyfin API token, sent in the `X-Emby-Token` header.
- `INCLUDE_ITEM_TYPES` — optional comma-separated list of Jellyfin item types.
- `LIMIT` — optional maximum number of items to fetch.
