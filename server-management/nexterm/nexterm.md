# Nexterm

Nexterm provides server-management shortcuts and monitoring information for servers managed through Nexterm, including SSH, VNC, and RDP connections.

## Files

- `integration.yaml` contains authentication, recent connection and monitoring requests, health calculations, and shortcut configuration.
- `nexterm.md` documents the integration for contributors and users.

## Configuration

- `URL` — required base URL of the Nexterm instance.
- `NT_USER` and `NT_PASS` — required Nexterm credentials used to retrieve a token.
- `SHOW_RECENT_COUNT` — optional number of recent connections to show.
- `TOKEN` — managed internally after authentication.

Shortcuts link directly to Nexterm connection targets and include connection type and server status information where available.
