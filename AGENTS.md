# AGENTS.md

## What this project is

A single self-contained static HTML page: the extensionless file **`S`** at the repo
root (SSS.log — a tripleS fandom hub/wiki). No backend, no database, no build step,
no package manager, and no environment variables or secrets. All data (members,
discography, glossary, fansites, sub-units) is inline in a `<script>` at the bottom
of `S`, and navigation is client-side via the `switchPage()` function — there are no
URL routes.

## Running it

```bash
docker compose -f docker-compose.base44.yml up -d
```

Served at <http://localhost:3000/>. Only the entry file `S` and the nginx config are
bind-mounted, for two reasons:

1. `S` has no file extension. nginx cannot infer a MIME type from it, so
   `docker/nginx.base44.conf` forces `default_type text/html`. Without that the
   browser downloads the file instead of rendering it (this also breaks `python -m
   http.server`, which sends `application/octet-stream`).
2. The repo directory is mode `0700` on the sandbox host, which the unprivileged
   nginx worker user cannot traverse — mounting the whole directory gives 403.

If you add more static files or assets, add each one as its own volume entry in
`docker-compose.base44.yml` (or mount directory by directory), and keep the
`default_type text/html` in mind for any other extensionless file.

## Live editing

There is no dev server and no hot reload. nginx re-reads the file on every request
(caching is disabled), so an edit shows up as soon as the page is reloaded.

## External dependencies

The page pulls Tailwind (CDN script), FontAwesome, Google Fonts and member photos
from Wikimedia at runtime, so **the browser needs outbound internet access** to look
right. These are client-side requests only; nothing is fetched by the container.

### Known content gap: the member/album photos are dead links

Every `photo`/`cover` URL in `S` points at a Wikimedia Commons thumbnail that does
**not exist** — e.g.
`commons/thumb/c/c5/240513_tripleS_Yoon_Seo-yeon.jpg/800px-...`. The Commons API
reports `"missing"` for these titles, so they fail for every visitor regardless of
environment (verified with `curl` and in the browser, while the Tailwind /
FontAwesome / Google Fonts CDNs load fine). Member cards fall back to the empty
dark box from their `bg-black` class, so the layout is intact but the roster,
spotlight and discography grids show no imagery. Fixing this means replacing the
URLs with real image sources (or local placeholder art) — a content change, not a
setup problem.

## Verifying it works

```bash
curl -sI http://localhost:3000/            # expect 200 + Content-Type: text/html
curl -s http://localhost:3000/ | grep -c tripleS
docker compose -f docker-compose.base44.yml ps   # web should be (healthy)
```
