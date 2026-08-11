# Dashdot

Dashdot provides a live overview of a host’s CPU, memory, storage, network, and GPU activity. The integration combines Dashdot’s information and load endpoints into one metrics model for a compact Dashwise widget.

## Files

- `integration.yaml` contains the Dashwise definition, endpoint mappings, computed metrics, widget, and shortcut configuration.
- `dashdot.md` documents the integration for contributors and users.

## Configuration

- `URL` — required base URL of the Dashdot instance.
- `DISPLAY_NAME` — optional name shown in the widget header; the hostname is used when it is omitted.

The widget refreshes every eight seconds, shows storage when no GPU is reported, and links back to the configured Dashdot instance.
