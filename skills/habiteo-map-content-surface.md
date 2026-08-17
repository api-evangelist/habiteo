---
name: habiteo-map-content-surface
description: >-
  Discover exactly what is callable on www.habiteo.com before calling anything — read the live
  WordPress route index, enumerate the registered content types and taxonomies, and pull the
  product pages and portfolio entries. Use this first; it is also the safest way to confirm that
  Habiteo still publishes no product API.
generated: '2026-08-17'
method: generated
source: >-
  openapi/habiteo-discovery-api-openapi.yml, openapi/habiteo-pages-api-openapi.yml,
  openapi/habiteo-portfolio-api-openapi.yml, openapi/habiteo-media-api-openapi.yml
api: habiteo:discovery-api
base_url: https://www.habiteo.com/wp-json
operations:
  - getRouteIndex
  - getWpV2Index
  - getWpV2Types
  - getWpV2TypesType
  - getWpV2Taxonomies
  - getWpV2Pages
  - getWpV2Portfolio
  - getWpV2Media
---

# Map the Habiteo content surface

Habiteo publishes no OpenAPI and no developer documentation. The route index the site itself serves
**is** the contract, and it is the thing to read first — the derived OpenAPI documents in this
repository are a snapshot of it, not a substitute for it.

## Steps

### 1. Read the live route index — `getRouteIndex`

```
GET /wp-json/
```

Returns the site name, the registered `namespaces` and a `routes` map. As of 2026-08-17 the
namespaces are `wp/v2`, `contact-form-7/v1`, `yoast/v1`, `pum/v1` and `oembed/1.0`, across 52
routes. `authentication` is an empty array — the install advertises no auth provider.

Each route entry declares its `methods` and, per endpoint, an `args` object giving every parameter's
type, default, enum and description. That is where the parameters in this repo's OpenAPI documents
came from. If the index and the OpenAPI disagree, **the index wins**.

### 2. Enumerate content types — `getWpV2Types`, `getWpV2TypesType`

```
GET /wp/v2/types
GET /wp/v2/types/{type}
```

Five types are registered: `post` (Articles), `page` (Pages), `attachment` (Fichier média),
`portfolio` (Portfolio) and `uncodeblock` (Content Block, a theme construct). Each declares its
`rest_base` — the collection path — and its `taxonomies`.

### 3. Enumerate taxonomies — `getWpV2Taxonomies`

```
GET /wp/v2/taxonomies
```

`category` and `post_tag` apply to `post`; `portfolio_category` applies to `portfolio`. `category`
and `portfolio_category` are hierarchical, so their terms have a `parent`.

### 4. Pull the product pages — `getWpV2Pages`

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title
```

These are the localized product pages — Modélisation, myHabiteo, myHabiteo Studio, Configurateur,
Habiteo Truck, and the vertical pages for résidentiel, tertiaire, constructeur de maison and agence
de communication. Fetching them is how you read what Habiteo says it sells, at source.

### 5. Pull the showcase — `getWpV2Portfolio`, `getWpV2Media`

```
GET /wp/v2/portfolio?per_page=100
GET /wp/v2/media?per_page=100&media_type=image
```

`portfolio` holds client and project showcases. `media` holds the render and screenshot library,
including the commercial PDFs served under `/media/documents/`.

## What you will NOT find, and should stop looking for

Confirmed by probe on 2026-08-17:

- No OpenAPI, Swagger, GraphQL SDL, AsyncAPI, Postman collection or JSON Schema on any Habiteo host.
- No `/.well-known/` document of any kind — security.txt, openid-configuration,
  oauth-authorization-server, api-catalog and ai-plugin.json all 404.
- No MCP server and no A2A agent card.
- No webhooks, no events, no streaming.
- No SDK on npm, PyPI, RubyGems, Maven Central, NuGet, crates.io, Packagist or the Go proxy.

## The wildcard trap — read this before probing any Habiteo host

`*.habiteo.com` resolves to the marketing site. `api.habiteo.com`, `developer.habiteo.com`,
`app.habiteo.com`, `widgets.habiteo.com` and `funnel.habiteo.com` all return **HTTP 200 with the
www.habiteo.com homepage for every path**, including `/openapi.json` and
`/.well-known/agent-card.json`. `megawidget.habiteo.com` returns the same 1,294-byte SPA shell for
every path.

Never treat a 200 on those hosts as a hit. Always probe a nonsense path first
(`/nonexistent-garbage-xyz`) and compare: if it returns the same body, every 200 on that host is
meaningless. Parse the body and confirm it is the document you asked for before you believe it.
