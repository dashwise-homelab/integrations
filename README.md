# Dashwise integrations

This repository contains community integration definitions for Dashwise.

## Directory layout

Each integration lives in a category and has the same two-file structure:

```text
<category>/<integration>/
├── integration.yaml   # Dashwise integration definition
└── <integration>.md    # Human-readable integration documentation
```

The YAML file is the machine-readable source used by Dashwise. The Markdown file explains what the integration provides, which settings are required, and how its data is presented.

## Categories

| Category | Integrations |
| --- | --- |
| `monitoring` | Beszel, Dashdot |
| `server-management` | Nexterm |
| `dashwise` | Markets, Weather |
| `development` | GitHub |
| `media` | Jellyfin |
| `bookmarks` | Karakeep |

## Adding an integration

Place the integration in the most specific existing category, copy the two-file structure above, and keep the directory name lowercase. Add the integration to this index when introducing a new category or integration.
