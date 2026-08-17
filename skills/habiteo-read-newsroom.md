---
name: habiteo-read-newsroom
description: >-
  Read Habiteo's published news and blog articles from the WordPress content API behind
  www.habiteo.com — list them, page through them, filter by category or date, and fetch one in
  full. Anonymous, no credentials. This is CMS content, not a Habiteo product API.
generated: '2026-08-17'
method: generated
source: openapi/habiteo-posts-api-openapi.yml, openapi/habiteo-taxonomy-api-openapi.yml
api: habiteo:posts-api
base_url: https://www.habiteo.com/wp-json
operations:
  - getWpV2Posts
  - getWpV2PostsId
  - getWpV2Categories
  - getWpV2Tags
---

# Read the Habiteo newsroom

Habiteo publishes 384 articles in French, Spanish and English on habiteo.com. They are readable
anonymously through the WordPress REST API on the same host. There is no Habiteo product API — this
is the only readable content surface, and it belongs to the CMS.

## Before you start

- Base URL: `https://www.habiteo.com/wp-json`
- No credential is required for any step below. Do not send an Authorization header.
- Errors come back as `{"code","message","data":{"status"}}`. The `message` is **French**. Branch on
  `code`, never on message text. See `errors/habiteo-problem-types.yml`.
- There are no rate-limit headers. Be conservative: request what you need, respect `per_page`.

## Steps

### 1. List recent articles — `getWpV2Posts`

```
GET /wp/v2/posts?per_page=20&page=1&orderby=date&order=desc
```

Read the paging state from the response headers, not from the body:

- `X-WP-Total` — total articles in the collection (384 at time of writing)
- `X-WP-TotalPages` — number of pages at the current `per_page`
- `Link: <...page=2>; rel="next"` — follow this rather than incrementing `page` yourself

`per_page` is capped at **100**. Asking for more returns HTTP 400 `rest_invalid_param`, with
`data.params.per_page` explaining the bound.

### 2. Narrow the set

All of these are query parameters on the same operation:

- `search=<text>` — free-text match
- `categories=<id>` / `tags=<id>` — integer ids, not slugs
- `after=<ISO 8601>` / `before=<ISO 8601>` — publication window
- `slug=<slug>` — resolve a known URL slug directly

Do **not** send `context=edit`. Anonymously it returns HTTP 401 `rest_forbidden_context`. The
default `context=view` is what you want.

### 3. Resolve category and tag ids — `getWpV2Categories`, `getWpV2Tags`

```
GET /wp/v2/categories?per_page=100
GET /wp/v2/tags?per_page=100
```

Nine categories exist, including language-partitioned editorial buckets (`actu` for French,
`actu-es` for Spanish). Filtering by category is how you get one language's articles.

### 4. Fetch one article — `getWpV2PostsId`

```
GET /wp/v2/posts/{id}
```

The body carries `title.rendered`, `content.rendered`, `excerpt.rendered`, `date`, `modified`,
`link`, `author` (an integer id), `featured_media` (an integer id), `categories[]` and `tags[]`.
`content.rendered` is HTML, not markdown or plain text — strip it before handing it to a model.

### 5. Follow relationships through `_links`, not by guessing URLs

Every resource carries a HAL-like `_links` object. Use it:

- `_links["wp:featuredmedia"]` → the media item for the hero image
- `_links["wp:term"]` → the category and tag terms actually attached
- `_links.author` → `/wp/v2/users/{id}`
- `_links.replies` → `/wp/v2/comments?post={id}`

## Cautions

- `_links.author` resolves to a named Habiteo employee. That is personal data under GDPR. Fetch it
  only if you have a reason to, and do not republish it.
- A 404 with code `rest_no_route` means the path is not in the route index. Re-read
  `https://www.habiteo.com/wp-json/` — the live index is the contract.
- Nothing on this surface tells you about Habiteo programmes, lots, 3D assets, buyers or
  reservations. Those live inside myHabiteo behind a login and have no public API.
