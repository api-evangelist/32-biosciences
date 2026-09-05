---
name: 32-biosciences-search-and-pages
description: Search 32biosciences.com and read its marketing, platform, pipeline and leadership-profile pages through the WordPress REST API.
api: 32-biosciences:32-biosciences-discovery-api
base_url: https://32biosciences.com/wp-json
operations:
  - getSearch
  - getPages
  - getPagesById
  - getTypes
  - getTaxonomies
  - getStatuses
  - getOembedEmbed
auth: none
generated: '2026-09-05'
method: generated
source: openapi/_ae-authored/32-biosciences-content-openapi.yml
---

# Search and read 32 Biosciences pages

## 1. Search the whole site (`getSearch`)

```
GET https://32biosciences.com/wp-json/wp/v2/search?search=mucosal&per_page=20
```

Returns a light projection — `id`, `title`, `url`, `type`, `subtype`, `_links.self` — across posts
**and** pages. `subtype` tells you which collection the id belongs to (`post` or `page`); follow
`_links.self.href` for the full object rather than guessing the collection.

Narrow with `subtype=page` or `subtype=post`, and `type=post` (the only registered search type).

## 2. Read the pages (`getPages`)

```
GET https://32biosciences.com/wp-json/wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent,menu_order
```

37 pages. They fall into three groups:

- **Platform and story**: `our-story`, `technology`, `therapeutic-platform`, `diagnostic-platform`,
  `pipeline-proof-of-concept`, `team`, `investor-relations`, `contact`, `news-updates`.
- **People**: one page per leader, board member and scientific advisor — `peter-farmakis`,
  `john-alverdy`, `eugene-chang`, `joseph-pierre` and roughly twenty more. There is no `person`
  custom post type on this site; profiles are ordinary pages, so filter by slug, not by type.
- **Policy and residue**: `privacy-policy`, `cookie-policy`, `accessibility-statement`,
  `conflict-of-interest-policy`, plus theme leftovers like `sample-page` and `homepage-demo-1`.
  Skip the leftovers — `homepage-demo-1` and `pipeline-proof-of-concept` carry "Demo" in their
  titles and are not authoritative.

Fetch one with `getPagesById`: `GET /wp/v2/pages/4642`.

## 3. Confirm the shape of the site before assuming it (`getTypes`, `getTaxonomies`, `getStatuses`)

```
GET https://32biosciences.com/wp-json/wp/v2/types
GET https://32biosciences.com/wp-json/wp/v2/taxonomies
```

`types` returns the registered post types and their `rest_base`; `taxonomies` returns `category` and
`post_tag`. Read these first if you are writing anything that generalises across WordPress sites —
this site registers no custom content types beyond its page-builder's internal ones.

## 4. Embed a page (`getOembedEmbed`)

```
GET https://32biosciences.com/wp-json/oembed/1.0/embed?url=https://32biosciences.com/team/
```

Returns oEmbed 1.0 with `provider_name`, `title`, `author_name`, `type: rich` and an embeddable
`html` fragment.

## Rules

- **Two routes in this namespace do not serve you.** `/wp/v2/settings` returns `401 rest_forbidden`
  and the `wp-abilities/v1` registry returns `401 rest_forbidden` anonymously. Do not retry them
  with a guessed credential.
- **Pagination** is `page` + `per_page` (max 100) with `X-WP-Total` / `X-WP-TotalPages` headers and
  an RFC 5988 `Link` header. Use `_fields` to keep responses small.
- **No rate-limit signal exists.** Self-pace; there is no `Retry-After` to obey.
- **HTML entities** appear encoded in every `*.rendered` string. Decode before comparing or
  displaying.
