---
name: dashwise-integration-authoring
description: Create, extend, or review Dashwise community integrations backed by integration.yaml files. Use when adding an external-service adapter, defining widget or glanceable consumers, mapping endpoint responses, adding computed fields, documenting an integration, or validating integration runtime and page-config behavior.
---

# Dashwise Integration Authoring

Create integrations as small, reusable data adapters.

Keep external API retrieval, response normalization, computed runtime data, and consumer presentation separate. A single integration runtime may serve multiple widgets, glanceables, and shortcuts.

Follow the behavior implemented by the Dashwise integrations kit. Do not introduce YAML shapes merely because they appear intuitive: verify them against an existing working integration or the current resolver implementation.

## Repository structure

Put every integration in a category-specific directory:

```text
<category>/<integration>/
├── integration.yaml
└── <integration>.md
```

Use lowercase directory names.

Place the integration in the most specific existing category. Current examples include:

* `monitoring`: Beszel, Dashdot
* `server-management`: Nexterm
* `dashwise`: Markets, Weather
* `development`: GitHub
* `media`: Jellyfin
* `bookmarks`: Karakeep

Use:

* `integration.yaml` for the machine-readable integration definition.
* `<integration>.md` for human-readable setup and behavior documentation.

When adding a new integration or category, also update the repository catalogue/index if the repository currently requires it.

## Authoring workflow

1. Inspect two or three existing integrations.

   * Prefer one with a similar API/authentication model.
   * Prefer one using the widget or glanceable template you need.
2. Verify the external service API.

   * Endpoint URLs.
   * Authentication method.
   * Response shapes.
   * Pagination.
   * Rate limits.
   * Relevant link targets.
3. Inspect the current Dashwise resolver when YAML behavior is unclear.
4. Choose the correct integration category.
5. Create:

   * `integration.yaml`
   * `<integration>.md`
6. Define the integration in this order:

   * metadata;
   * environment variables;
   * endpoints;
   * response mappings;
   * computed runtime data;
   * lookup tables;
   * widgets;
   * glanceables;
   * shortcuts.
7. Validate every runtime path against the actual resolved object shape.
8. Parse and lint the YAML.
9. Test against representative API responses when possible.
10. Update repository catalogue/index files when required.

## Basic YAML shape

Use this as the starting structure:

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

Only include sections the integration actually uses.

The directory category organizes repository files. `details.categories` is integration metadata exposed by Dashwise.

## Environment variables

Define user-supplied configuration and credentials under:

```yaml
configuration:
  environment_variables:
```

Example:

```yaml
environment_variables:
  URL:
    required: true
    description: "Base URL of the service without a trailing slash"

  API_KEY:
    required: true
    edit: "overwrite-only"
    description: "API key used to authenticate requests"

  DISPLAY_NAME:
    required: false
    description: "Optional display name override"
```

### Rules

Connection values such as `URL` should normally be:

```yaml
URL:
  required: true
```

Secrets should normally use:

```yaml
edit: "overwrite-only"
```

Use:

```yaml
user_hidden: true
```

for values maintained internally rather than directly edited by the user.

Only define defaults when the service has a safe and meaningful default:

```yaml
INTERVAL:
  required: false
  default: 60
```

Use `type: location` when an integration accepts a location object that can be supplied by the user's localization settings.

Do not expose credentials through widget properties, shortcut labels, actions, documentation examples, or computed display fields.

### Template resolution

Environment variables are referenced using:

```text
${URL}
${API_KEY}
```

Lookup is case-insensitive where supported by the integration environment resolver, but integration files should use consistent casing.

Environment values may come from:

* configured integration environment values;
* widget or glanceable input;
* internally persisted environment state;
* endpoint-produced environment variables;
* runtime/computed data where supported.

## Authentication

Use the authentication format required by the service.

Example bearer authentication:

```yaml
auth: "Bearer ${TOKEN}"
```

Example Basic authentication when the environment variable contains only Base64-encoded `username:password`:

```yaml
auth: "Basic ${BASIC_AUTH}"
```

Do not add `Basic ` to both the environment value and YAML.

If the environment variable already contains the complete header value:

```text
Basic abc123...
```

then use:

```yaml
auth: "${AUTH}"
```

Prefer keeping the authentication scheme in the integration YAML when practical, so the user's secret contains only the credential itself.

## Endpoint-produced tokens

When a service requires logging in before subsequent requests, use a token-retrieval endpoint following an existing working integration.

A token endpoint may use:

```yaml
response:
  type: json
  data_path: "token"
  data_set_env: "TOKEN"
  invalidate:
    after: 86400
```

Later requests can then use:

```yaml
auth: "Bearer ${TOKEN}"
```

Do not assume token retrieval or refresh semantics. Copy the closest working pattern and verify it against the current endpoint resolver.

## Endpoints

Define HTTP requests under:

```yaml
configuration:
  endpoints:
```

Example:

```yaml
endpoints:
  fetch-stats:
    method: GET
    url: "${URL}/api/stats"
    auth: "Bearer ${TOKEN}"
    response:
      type: json
```

Supported endpoint features depend on the current resolver and may include:

* `method`
* `url`
* `body`
* `auth`
* `custom_headers`
* `insecure_skip_verify`
* `response`
* `response_mapping`
* iteration
* grouping
* caching
* rate limiting

Use an existing integration or inspect the endpoint resolver before introducing uncommon keys.

## Response mappings

Normalize vendor-specific responses at the endpoint boundary.

Consumers should depend on stable integration names such as:

* `queries`
* `blocked`
* `servers`
* `items`
* `metrics`

rather than repeating vendor-specific field names throughout widgets and computed expressions.

### Mapping scalar fields

Given an API response such as:

```json
{
  "num_dns_queries": 125000,
  "num_blocked_filtering": 6500
}
```

map it to:

```yaml
response_mapping:
  queries: "num_dns_queries"
  blocked: "num_blocked_filtering"
```

The resulting endpoint data can then be referenced as:

```text
this.endpoints.fetch-stats.mappedResponse.queries
this.endpoints.fetch-stats.mappedResponse.blocked
```

### Mapping an array property

Given:

```json
{
  "items": [
    {
      "id": "abc",
      "name": "Example"
    }
  ]
}
```

use:

```yaml
response_mapping:
  entries:
    iterate: "items"
    mappingProperties:
      id: "id"
      name: "name"
```

### Mapping a response that is itself an array

When the API response root is an array, use the working direct-array pattern supported by the resolver, commonly:

```yaml
response_mapping:
  entries:
    iterate: "response"
    mappingProperties:
      id: "id"
      name: "name"
```

Verify direct-array behavior against the current endpoint resolver before shipping.

### Nested mappings

Nested response values can be normalized with nested `mappingProperties` structures when supported by the existing mapping implementation.

Example:

```yaml
response_mapping:
  systems:
    iterate: "items"
    mappingProperties:
      id: "id"
      name: "name"
      info:
        - status: "status"
        - hostname: "hostname"
```

Copy the closest existing integration when using nested mappings.

### `data_path`

Use `response.data_path` when the relevant JSON payload is nested inside a response.

Use `data_path_fallback` only for APIs with known alternate response shapes and only when supported by the current resolver.

### `discard_unmapped`

Use:

```yaml
discard_unmapped: true
```

when the integration intentionally wants only normalized mapped values retained.

Do not enable it automatically. Raw/unmapped response data may be useful to another consumer.

## Referencing endpoint output

Computed definitions generally reference endpoint data through:

```text
this.endpoints.<endpoint-id>.mappedResponse.<path>
```

For example:

```text
this.endpoints.fetch-stats.mappedResponse.queries
```

or:

```text
this.endpoints.fetch-systems.mappedResponse.systems
```

Verify the path against the **mapped response**, not against the original external API response.

## Computed runtime data

Use:

```yaml
configuration:
  computed:
```

to create stable, consumer-friendly runtime data.

Computed values are resolved after endpoints.

They are exposed to widget rendering under:

```text
computed.<computed-key>...
```

### Critical rule: ordinary computed objects do not use `fields:`

For an ordinary computed object, define properties directly:

```yaml
computed:
  metrics:
    hostname: "this.endpoints.fetch-info.mappedResponse.hostname"

    total_queries: "this.endpoints.fetch-stats.mappedResponse.queries"

    blocked_percent:
      operation: expr
      expression: "round((blocked / max(queries, 1)) * 1000) / 10"
      inputs:
        blocked: "this.endpoints.fetch-stats.mappedResponse.blocked"
        queries: "this.endpoints.fetch-stats.mappedResponse.queries"
      fallback: 0
```

This resolves to approximately:

```json
{
  "metrics": {
    "hostname": "server",
    "total_queries": 129675,
    "blocked_percent": 5
  }
}
```

The widget path is therefore:

```text
${computed.metrics.total_queries}
```

### Do not write this for an ordinary object

Do not write:

```yaml
computed:
  metrics:
    fields:
      total_queries: "this.endpoints.fetch-stats.mappedResponse.queries"
```

For an ordinary computed object, `fields` is preserved as a literal object key.

That produces:

```json
{
  "metrics": {
    "fields": {
      "total_queries": 129675
    }
  }
}
```

and would require:

```text
${computed.metrics.fields.total_queries}
```

That is usually not the intended runtime shape.

### When `fields:` is correct

`fields:` has special meaning for computed definitions that explicitly iterate or expand data.

#### `iterate_over`

Example:

```yaml
computed:
  systems:
    iterate_over: "this.endpoints.fetch-systems.mappedResponse.systems"
    fields:
      name: "name"
      status: "status"
```

For iterated computed values, the resolver uses `fields` to define each resolved entry.

Optional iteration features may include:

```yaml
key: "id"
filter: "status == 'up'"
```

Use them only according to current resolver behavior.

#### `operation: expand`

Example:

```yaml
computed:
  systems:
    operation: expand
    on: "this.endpoints.fetch-systems.mappedResponse.systems as system"

    fields:
      cpu_usage: "system.usage.cpu"
      memory_usage: "system.usage.memory"
```

For `expand`, `fields:` describes the extra fields calculated for each source item.

The resolver merges those resolved fields into the source item when the source item is an object.

### Summary

Use:

```yaml
computed:
  summary:
    queries: ...
    blocked: ...
```

for a single object.

Use:

```yaml
computed:
  systems:
    iterate_over: ...
    fields:
      ...
```

or:

```yaml
computed:
  systems:
    operation: expand
    on: ...
    fields:
      ...
```

for per-item computed structures.

## Computed operations

Prefer direct references for simple renaming:

```yaml
queries: "this.endpoints.fetch-stats.mappedResponse.queries"
```

Use operation objects for derived values.

Example:

```yaml
blocked_percent:
  operation: expr
  expression: "round((blocked / max(queries, 1)) * 1000) / 10"
  inputs:
    blocked: "this.endpoints.fetch-stats.mappedResponse.blocked"
    queries: "this.endpoints.fetch-stats.mappedResponse.queries"
  fallback: 0
```

Existing integrations and the current resolver may support operations including:

* `expr`
* `avg`
* `math`
* `if`
* `expand`
* `index_lookup`
* `nth_from_end`
* lookup operations
* aggregation operations

Do not assume an operation exists because it would be convenient. Verify it in `getComputedField` and the resolver operation implementation.

## Computed references to earlier computed values

Computed groups are resolved in definition order.

When a computed definition needs another computed value, make sure the required value is available at the point it is evaluated.

Use explicit runtime paths where appropriate:

```text
computed.some_value
```

Inside iterated/expanded field resolution, previously resolved fields for the current item may also be available through the current computed scope.

Keep these dependencies straightforward. Prefer endpoint-derived inputs where that makes the data flow easier to understand.

## Null safety and fallbacks

Derived values should handle:

* missing fields;
* empty arrays;
* unavailable devices;
* optional API features;
* divide-by-zero;
* null vendor responses.

Example:

```yaml
blocked_percent:
  operation: expr
  expression: "round((blocked / max(queries, 1)) * 1000) / 10"
  inputs:
    blocked: "this.endpoints.fetch-stats.mappedResponse.blocked"
    queries: "this.endpoints.fetch-stats.mappedResponse.queries"
  fallback: 0
```

Avoid fake sample data as runtime fallbacks.

For real runtime values, prefer neutral fallbacks such as:

* `0`
* `0%`
* `Unknown`
* an empty list

depending on the meaning of the field.

## Lookup tables

Use:

```yaml
configuration:
  lookup_tables:
```

for reusable mappings such as:

* status colors;
* weather codes;
* threshold definitions;
* icon mappings;
* connection state mappings.

Example:

```yaml
lookup_tables:
  statuses:
    online:
      color: "#16a34a"
    offline:
      color: "#ef4444"
```

Reference lookup data using the lookup mechanisms supported by the current computed resolver.

## Widget runtime model

When a widget is resolved, Dashwise resolves the integration runtime and produces runtime data containing the endpoint and computed trees.

Conceptually:

```json
{
  "endpoints": {
    "...": {}
  },
  "computed": {
    "...": {}
  }
}
```

The widget property resolver flattens this runtime data into interpolation keys.

For example:

```json
{
  "computed": {
    "summary": {
      "queries": 129675
    }
  }
}
```

becomes available to widget properties as:

```text
${computed.summary.queries}
```

The property path must exactly match the resolved runtime shape.

## `data.source`

Widgets may contain:

```yaml
data:
  source: "computed.summary"
```

Do not assume that `data.source` rebases property interpolation.

For example, this:

```yaml
data:
  source: "computed.summary"
```

does **not** make this valid automatically:

```text
${queries}
```

if the runtime value is actually:

```text
computed.summary.queries
```

Use the complete runtime path:

```text
${computed.summary.queries}
```

Treat `data.source` as consumer metadata/source declaration unless the current resolver explicitly uses it for a particular behavior.

The widget runtime itself resolves the integration endpoints and computed values independently of interpolation rebasing.

## `data.input`

Use:

```yaml
data:
  input:
```

for integration values the consumer needs to carry into runtime resolution.

Example:

```yaml
data:
  input:
    URL: "${URL}"
    API_KEY: "${API_KEY}"
```

Do not put display-only state into shared computed runtime data when it belongs to a particular widget instance.

## Widgets

Use `widgets` for richer dashboard blocks such as:

* columns;
* progress values;
* status cards;
* lists;
* images;
* iframes.

Give every widget a stable integration-specific key:

```yaml
- name: "Example Overview"
  key: "example-overview"
```

Do not reuse a key for incompatible widget definitions.

### Columns widget

Example:

```yaml
widgets:
  - name: "Example Overview"
    key: "example-overview"
    template: "columns"

    depends_on:
      - "computed.summary"

    refresh_interval: 30

    data:
      source: "computed.summary"
      input:
        URL: "${URL}"

    properties:
      header:
        title: "Example"
        icon: "url:/icons/png/example.png"
        titleAction: "url:${URL}"

      columns:
        - label: "Queries"
          primary: "${computed.summary.queries}"
          secondary: "DNS queries"

        - label: "Blocked"
          primary: "${computed.summary.blocked_percent}%"
          secondary: "${computed.summary.blocked_count} blocked"
```

Static columns are defined as an array.

The current resolver resolves values such as:

* `label`
* `primary`
* `primaryAction`
* `secondary`
* `title`
* `titleAction`
* `thumbnail`
* `icon`
* `progress`
* `badge`
* `show_if`

according to template support.

### Iterated columns

For a collection:

```yaml
columns:
  iterate_over: "computed.systems"

  user_customizations:
    - allow_reorder
    - allow_hide

  prototype:
    primary: "${system.name}"
    secondary: "${system.status}"
```

The widget resolver obtains the collection from runtime data.

For plural collection names, it may derive a singular alias such as:

```text
systems → system
```

Do not rely on complicated alias behavior without testing it against the actual runtime.

## `depends_on`

Use `depends_on` to describe the runtime values required by a consumer, following existing working integrations.

Example:

```yaml
depends_on:
  - "computed.summary"
```

Do not assume `depends_on` changes the interpolation namespace. Widget paths must still match the resolved runtime object.

## Property interpolation

Use:

```text
${...}
```

to interpolate environment and flattened runtime values.

Examples:

```yaml
title: "${DISPLAY_NAME}"
primary: "${computed.metrics.cpu_usage}%"
secondary: "${computed.metrics.total_queries} queries"
```

### Fallback syntax

Dashwise supports:

```text
PRIMARY ??? FALLBACK
```

Example:

```yaml
title: "${DISPLAY_NAME} ??? Server"
```

or:

```yaml
primary: "${computed.summary.queries} ??? 0"
```

Use fallbacks for values that may legitimately be unavailable.

Do not hide structural path bugs with fallback values.

If a computed value unexpectedly always displays its fallback:

1. inspect the resolved computed JSON;
2. compare that JSON path with the widget interpolation path;
3. check for accidental nesting such as `fields`;
4. verify the endpoint mapped response;
5. only then investigate formatting or frontend rendering.

## Actions

URL actions typically use:

```yaml
titleAction: "url:${URL}"
```

or:

```yaml
primaryAction: "url:${URL}/item/${id}"
```

Only include actions supported by Dashwise's current action resolver.

Keep authentication secrets out of URLs.

## Glanceables

Use `glanceables` for compact summaries such as:

* text;
* numeric values;
* status;
* icon/value pairs.

Give every glanceable a stable integration-specific key.

Use computed runtime values for:

* status;
* display text;
* direction;
* thresholds;
* derived labels.

Avoid duplicating the same expression independently across multiple glanceable properties.

Follow an existing working glanceable integration for the exact current schema.

## Shortcuts

Use `shortcuts` for searchable/direct actions to records or external resources.

Example pattern:

```yaml
shortcuts:
  - main:
      iterate_over: "this.endpoints.fetch-items.mappedResponse.items"

      mappingProperties:
        id: "${id}"
        name: "${name}"
        icon: "url:/icons/png/example.png"
        secondaryInfo: "Item"
        type: "exampleItem"
        action: "url:${URL}/items/${id}"

        tags:
          - "Example"
          - "Items"
          - "${name}"
```

Shortcut data should use normalized endpoint or computed data where practical.

Never expose credentials in shortcut actions or metadata.

## Refresh, caching, and rate limits

A widget may define a refresh interval such as:

```yaml
refresh_interval: 30
```

Do not assume this alone limits all backend endpoint activity.

For expensive or rate-limited APIs, inspect the current endpoint/runtime cache implementation and use supported cache or rate-limit configuration.

When adding rate-limit configuration, copy an existing working schema rather than inventing keys.

## Insecure/self-hosted endpoints

Self-hosted services frequently use:

* Docker hostnames;
* private IP addresses;
* self-signed TLS certificates.

Use:

```yaml
insecure_skip_verify: true
```

only where appropriate and supported.

Dashwise's backend may additionally restrict insecure/private endpoint behavior through instance configuration. Integration YAML cannot override server-side security policy.

## Documentation requirements

Write `<integration>.md` beside `integration.yaml`.

Document:

* what service the integration connects to;
* what each widget, glanceable, or shortcut provides;
* required environment variables;
* optional environment variables;
* authentication setup;
* credential format;
* refresh behavior;
* API limitations;
* known service-version requirements;
* useful service/API documentation links.

Example credential wording:

```text
BASIC_AUTH is the Base64 encoding of username:password.
Do not include the "Basic " prefix; the integration adds it to the
Authorization header.
```

Never include real credentials in documentation.

Keep the Markdown documentation synchronized with the YAML.

## Debugging integrations

When an integration returns incorrect widget values, debug in this order.

### 1. Verify endpoint output

Confirm the service actually returns the expected response.

### 2. Inspect `mappedResponse`

Make sure `response_mapping` creates the shape you expect.

Example expected result:

```json
{
  "queries": 129675,
  "blocked": 6504
}
```

### 3. Inspect computed output

Check the actual computed JSON.

For:

```yaml
computed:
  summary:
    queries: "this.endpoints.fetch-stats.mappedResponse.queries"
```

expect:

```json
{
  "summary": {
    "queries": 129675
  }
}
```

If instead you see:

```json
{
  "summary": {
    "fields": {
      "queries": 129675
    }
  }
}
```

then the YAML introduced an unwanted `fields:` wrapper.

### 4. Compare widget interpolation paths

If runtime data contains:

```text
summary.queries
```

the full widget runtime path is:

```text
computed.summary.queries
```

and the widget should use:

```yaml
primary: "${computed.summary.queries}"
```

### 5. Check environment interpolation

Verify:

* `URL`
* credentials;
* token prefixes;
* trailing slashes;
* widget input;
* unresolved `${...}` tokens.

### 6. Check rendering last

If endpoints, computed data, and interpolation are correct, then inspect the widget template/frontend renderer.

## Common mistakes

### Wrong computed nesting

Wrong:

```yaml
computed:
  summary:
    fields:
      queries: ...
```

for an ordinary object.

Correct:

```yaml
computed:
  summary:
    queries: ...
```

### Wrong interpolation path

Runtime:

```json
{
  "computed": {
    "summary": {
      "queries": 100
    }
  }
}
```

Wrong:

```text
${queries}
```

Wrong:

```text
${computed.summary.fields.queries}
```

Correct:

```text
${computed.summary.queries}
```

### Assuming `data.source` rebases values

This:

```yaml
data:
  source: "computed.summary"
```

does not mean you should shorten:

```text
${computed.summary.queries}
```

to:

```text
${queries}
```

### Duplicated authentication scheme

If the YAML says:

```yaml
auth: "Basic ${BASIC_AUTH}"
```

the configured secret should be:

```text
BASE64_USERNAME_PASSWORD
```

not:

```text
Basic BASE64_USERNAME_PASSWORD
```

### Trailing base URL slash

Prefer documenting `URL` without a trailing slash when endpoint definitions append paths:

```text
http://service:3000
```

with:

```yaml
url: "${URL}/api/stats"
```

instead of depending on double-slash handling.

## Validation checklist

Before handing off an integration:

* Parse every `integration.yaml` with a YAML parser.
* Verify `details` and `configuration` exist.
* Verify each consumer has the expected template shape.
* Verify required environment variables are defined.
* Verify secrets use appropriate hidden/overwrite-only behavior.
* Verify endpoint methods and URLs.
* Verify authentication headers and schemes.
* Verify request bodies.
* Verify `data_path`.
* Verify `iterate`.
* Verify `mappingProperties`.
* Verify mapped response paths against representative API data.
* Inspect the actual computed output shape.
* Ensure ordinary computed objects do not accidentally contain a `fields` wrapper.
* Ensure `fields:` is only used where the computed iterator/expand resolver expects it.
* Verify every `${computed...}` path against actual runtime JSON.
* Verify every endpoint reference uses the correct endpoint ID and mapped path.
* Verify every `depends_on` target exists.
* Verify widget `data.input` contains required consumer inputs.
* Do not rely on `data.source` to rebase interpolation paths.
* Search for unresolved `${...}` tokens.
* Give optional values sensible fallbacks.
* Avoid fake production values as runtime fallbacks.
* Verify empty arrays and divide-by-zero cases.
* Verify endpoint polling/cache/rate-limit behavior.
* Verify secrets are never rendered.
* Run repository-specific tests where available.
* Run YAML parsing/linting.
* Run `git diff --check`.
* Test the integration against representative real API responses whenever possible.

## Resolver-first rule

Do not modify Dashwise backend resolver behavior as part of a normal community integration addition unless the required behavior genuinely cannot be represented by the current YAML system.

When:

* documentation;
* an old integration;
* a proposed YAML pattern; and
* the current resolver implementation

disagree, treat the currently deployed resolver implementation as authoritative.

Inspect, as relevant:

```text
packages/integrationskit/data/resolveProperties.tsx
packages/integrationskit/data/getComputedField.tsx
packages/integrationskit/data/getEndpointData.tsx
```

Then update integration documentation or examples to match actual runtime behavior.

If an existing repository example only works because of old behavior, document or fix that inconsistency instead of copying it into new integrations.
