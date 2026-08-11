---
name: dashwise-integration-authoring
description: Create, extend, or review Dashwise community integrations backed by integration.yaml files. Use when adding an external-service adapter, defining widget or glanceable consumers, mapping endpoint responses, adding computed fields, documenting an integration, or validating integration runtime and page-config behavior.
---

# Dashwise Integration Authoring

Create integrations as small, reusable data adapters. Keep endpoint retrieval, response normalization, computed runtime data, and consumer presentation separate so the same runtime data can safely serve multiple page-config consumers.

## Repository structure

Put every integration in a category-specific directory:

```text
<category>/<integration>/
├── integration.yaml
└── <integration>.md
```

Use lowercase directory names. Place the YAML in the most specific existing category. Current categories include:

- `monitoring`: Beszel, Dashdot
- `server-management`: Nexterm
- `dashwise`: Markets, Weather
- `development`: GitHub
- `media`: Jellyfin
- `bookmarks`: Karakeep

Use `integration.yaml` as the machine-readable definition and the companion Markdown file for human-readable setup and behavior documentation.

## Authoring workflow

1. Inspect two or three existing integrations, especially one with a similar API shape and one with the desired consumer type.
2. Confirm the external service’s API response shapes, authentication method, polling limits, and link targets. Do not invent response paths.
3. Choose the category and create the two-file integration directory.
4. Define metadata, environment variables, endpoints, mappings, computed fields, and consumers in that order.
5. Add the companion Markdown document.
6. Parse and lint the YAML, inspect every response path and template reference, and test with representative API data when possible.
7. Update the repository integration index when adding a new integration or category.

## YAML shape

Start from this shape, then copy the closest existing pattern rather than introducing unsupported keys:

```yaml
# Dashwise Integration YAML
details:
  name: Example Service
  description: Short description of the data exposed by the integration.
  author: Author or organization
  version: 1.0.0
  repository: "https://github.com/example/service"
  icon: "/icons/png/example.png"
  links:
    website: "https://example.com"
    documentation: "https://example.com/docs"
    support: "https://example.com/support"
  categories:
    - Monitoring

configuration:
  environment_variables: {}
  endpoints: {}
  computed: {}
  lookup_tables: {}
  widgets: []
  glanceables: []
  shortcuts: []
```

Only include sections the integration uses. Keep metadata categories aligned with the directory category where practical; the metadata category is what Dashwise exposes to users, while the directory organizes repository files.

## Environment variables

Define all user configuration and stateful credentials under `configuration.environment_variables`.

- Mark connection inputs such as `URL` as `required: true`.
- Give user-facing variables a precise `description` and a safe `default` only when one exists.
- Mark secrets with `edit: "overwrite-only"`; use `user_hidden: true` for values managed internally.
- Keep authentication tokens out of visible widget properties, shortcuts, and documentation examples.
- Use `type: location` for location objects when the user’s localization preferences can supply the value.
- Use a token-retrieval endpoint with `response.data_path`, `data_set_env`, and `invalidate` when the service requires a session token. Pass the token to later endpoints with `auth` or the service’s required header.

Environment values are applied before runtime resolution. Template lookup is case-insensitive, preferring the exact name, then uppercase, then lowercase. Values can come from integration environment state, consumer input, user-injected values, or computed fields.

## Endpoints and mappings

Define HTTP requests under `configuration.endpoints`. An endpoint normally includes `method`, `url`, optional `body`, `auth`, `custom_headers`, request security settings, a JSON `response`, and `response_mapping`.

Normalize external responses at the endpoint boundary. Use stable application names such as `items`, `servers`, `current`, or `metrics` rather than making widgets depend on vendor field names.

For a response containing an array under a property, iterate that property:

```yaml
response_mapping:
  entries:
    iterate: "items"
    mappingProperties:
      id: "id"
      name: "name"
```

For a response that is itself an array, iterate `response`:

```yaml
response_mapping:
  entries:
    iterate: "response"
    mappingProperties:
      id: "entryId"
      name: "name"
```

Use nested `mappingProperties` for nested objects and indexed paths when an API returns parallel arrays. Use `data_path` when the useful JSON is nested and `data_path_fallback` when the service has known response variants. Use `discard_unmapped: true` only when dropping the rest of the response is intentional.

Reference endpoint output in later stages through:

```text
this.endpoints.<endpoint-id>.mappedResponse.<path>
```

When endpoints are expensive or rate-limited, configure supported cache and rate-limit settings using an existing integration as the schema reference. Do not assume that a frontend polling interval alone prevents backend requests.

## Computed runtime data

Use `configuration.computed` to produce a stable, consumer-friendly runtime model. Prefer simple references for renaming fields and operation objects for derived values such as averages, rounding, expressions, lookups, aggregation, expansion, and index correlation.

Use `lookup_tables` for reusable mappings such as status colors, weather codes, connection icons, or threshold definitions. Reference them through the integration lookup-table path or the established lookup operation pattern.

For collections, use `expand` or an equivalent existing pattern to derive one object per source item. Keep health, status, warning, and display fields in computed data rather than recalculating them independently in every widget.

Example computed field references:

```yaml
computed:
  metrics:
    fields:
      name: "this.endpoints.fetch-info.mappedResponse.hostname"
      used_percent:
        operation: expr
        expression: "round((used / total) * 100)"
        inputs:
          used: "this.endpoints.fetch-usage.mappedResponse.used"
          total: "this.endpoints.fetch-info.mappedResponse.total"
```

Keep computed output deterministic and null-safe. Define fallbacks for optional vendor fields, empty arrays, missing GPUs, unavailable servers, and divide-by-zero cases.

## Consumers: widgets, glanceables, and shortcuts

Choose the consumer that matches the product surface:

- Use `widgets` for columns, progress, stats, and richer page blocks.
- Use `glanceables` for compact text/icon/value summaries.
- Use `shortcuts` for direct links to external resources or records.

Give every consumer a stable, integration-specific `key`. A page config identifies a consumer by type, key, and instance properties/input. Do not reuse a key for incompatible shapes.

Widget definitions typically include `name`, `key`, `template`, `depends_on`, optional refresh/cache settings, `data`, and `properties`. Use `data.source` to point at computed runtime data and `data.input` for values needed by that source. Render normalized fields through `properties`.

Glanceables should expose a compact `key`, display properties, icon, text, color, and polling behavior following the existing glanceable examples. Use computed fields for direction, status, and display text instead of duplicating expressions across properties.

Use the fallback syntax supported by the integration template engine:

```yaml
title: "${name} ??? Untitled"
icon: "${icon} ??? /icons/png/default.png"
action: "url:${URL}/items/${id}"
value: "${computed.metrics.value}% ??? 0%"
```

`${VAR}` resolves environment, runtime input, or computed values. `???` uses the right-hand value when the primary value is empty or still contains unresolved template tokens. If the final interpolated string starts with `{` or `[`, the runtime parses it as JSON.

Remember that page-config resolution extracts widget and glanceable consumers, resolves each consumer’s effective environment and input, and returns a blueprint containing `consumer`, `key`, `input`, `env`, `data`, `blueprint`, and cache metadata. Runtime data can be reused for consumers sharing the same integration and resolved environment, so avoid putting instance-specific presentation into shared computed fields.

## Documentation requirements

Write `<integration>.md` beside `integration.yaml`. Include:

- what service the integration connects to;
- what the widget, glanceable, or shortcut displays;
- required and optional environment variables;
- authentication/token behavior without exposing secrets;
- refresh, cache, or API limitations that affect users;
- links or setup notes that help a contributor verify the integration.

Keep the documentation aligned with the YAML. If a setting, endpoint, consumer, or output field changes materially, update both files.

## Validation checklist

Before handing off an integration:

- Parse every `integration.yaml` with a YAML parser.
- Check that `details`, `configuration`, and each consumer have the expected shape.
- Verify every endpoint URL, body, header, auth value, `data_path`, `iterate`, and mapped response path against the service API.
- Verify that each `depends_on` and `data.source` points to an existing endpoint or computed value.
- Search for unresolved `${...}` tokens that are not intentional templates and ensure optional values have fallbacks.
- Check direct-array responses use `iterate: "response"`.
- Check secrets are hidden or overwrite-only and never rendered into properties or shortcuts.
- Check endpoint polling, cache TTLs, and rate limits are safe for the external service.
- Run repository-specific tests or integration validation if available, then review `git diff --check`.

Do not modify backend resolver behavior as part of a normal integration addition unless the YAML capability is genuinely missing. When runtime behavior and an existing YAML example disagree, inspect the resolver and kit implementation named in the integration runtime documentation and follow the implementation that is actually deployed.
