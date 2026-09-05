---
name: 32-biosciences-read-newsroom
description: Read 32 Biosciences news releases and feature posts from the company's WordPress REST API, resolve their categories, authors and featured images, and page through the whole newsroom.
api: 32-biosciences:32-biosciences-posts-api
base_url: https://32biosciences.com/wp-json/wp/v2
operations:
  - getPosts
  - getPostsById
  - getCategories
  - getCategoriesById
  - getUsers
  - getMediaById
auth: none
generated: '2026-09-05'
method: generated
source: openapi/_ae-authored/32-biosciences-content-openapi.yml
---

# Read the 32 Biosciences newsroom

32 Biosciences serves its newsroom as JSON from the WordPress REST API. Reads are anonymous — no
key, no sign-up, no header. There are 12 posts as of 2026-09-05, so the whole newsroom fits in one
or two requests.

## 1. List posts (`getPosts`)

```
GET https://32biosciences.com/wp-json/wp/v2/posts?per_page=100&_fields=id,date,modified,slug,link,title,excerpt,author,featured_media,categories
```

- `per_page` maxes out at **100**. Read `X-WP-Total` and `X-WP-TotalPages` from the response headers
  to know whether to page; follow `Link: <...>; rel="next"` rather than incrementing `page` blind.
- `_fields` trims the payload. Without it every post carries fully rendered HTML content.
- Filter with `categories=11` (News Releases) or `categories=7` (Features), `search=<string>`,
  `after=`/`before=` (ISO 8601), `orderby=date|modified|title`, `order=asc|desc`.

**Watch the `link` field.** Some posts are syndicated press releases whose `link` points at an
external wire service (BusinessWire), not at 32biosciences.com. Do not assume `link` is on-domain.

## 2. Resolve the graph in one request

```
GET https://32biosciences.com/wp-json/wp/v2/posts?per_page=100&_embed=true
```

`_embed` inlines author (`_embedded.author`), featured image (`_embedded['wp:featuredmedia']`) and
terms (`_embedded['wp:term']`). Use it instead of calling `getUsers`, `getMediaById` and
`getCategoriesById` per post — it is one round trip rather than 4N.

If you would rather resolve by hand: `author` is a `getUsers` id, `featured_media` is a
`getMediaById` id, and `categories`/`tags` are `getCategoriesById` / `getTagsById` ids.

## 3. Fetch one post (`getPostsById`)

```
GET https://32biosciences.com/wp-json/wp/v2/posts/4612
```

`content.rendered` and `excerpt.rendered` are HTML strings with entities encoded (`&amp;`,
`&#8211;`). Decode before using them as text.

## Rules

- **Errors** use the WordPress envelope `{"code","message","data":{"status"}}` — not
  `application/problem+json`. `rest_post_invalid_id` (404) means the id does not exist or is not
  published. See `errors/32-biosciences-problem-types.yml`.
- **Rate limits** are undocumented and no `X-RateLimit-*` or `Retry-After` header is returned. There
  is no runtime backoff signal, so self-pace: sequential requests, no parallel fan-out.
- **Do not write.** Every POST/PUT/PATCH/DELETE on these routes requires a WordPress Application
  Password held by a 32 Biosciences administrator. There is no public credential and no
  idempotency mechanism — a retried POST creates a duplicate. See
  `conventions/32-biosciences-conventions.yml`.
- **Scope.** This is a CMS content API. Nothing about the GI Discovery Platform, the CS Therapeutic
  Platform, CS-0003 or any pipeline or clinical data is available programmatically — only the prose
  of the pages that describe them.
