# GitHub Repository

GitHub Repository provides a compact repository overview with open issues, pull requests, recent activity, and stars.

## Files

- `integration.yaml` contains GitHub API requests, response mappings, caching, rate limiting, and widget configuration.
- `github.md` documents the integration for contributors and users.

## Configuration

- `REPOSITORY` — required repository slug, for example `owner/project`.
- `GITHUB_TOKEN` — optional token for higher GitHub API limits.

The integration uses GitHub’s REST API and caches responses for five minutes.
