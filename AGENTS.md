# AGENTS.md

## What this project is

A static website made of self-contained HTML pages at the repo root — no backend,
no database, no build step, no package manager, no environment variables and no
secrets. All data (members, sub-units, glossary, fansites, discography, lore) is
inline in a `<script>` at the bottom of each file, and navigation is client-side
via the `switchTab()` / `switchPage()` functions — there are no URL routes.

Two revisions of the page exist:

- **`SS`** (1582 lines) — the current revision, served at `/`. Newest commit on the
  branch ("Create SS").
- **`S`** (587 lines) — the previous revision, kept reachable at `/S`. It still
  hotlinks its member photos from Wikimedia Commons, which do **not** load here
  (Commons rejects those thumbnail widths and rate-limits this environment); `SS`
  was rewritten without those images, so nothing is missing at runtime.

Neither file has a file extension, which is why the setup below exists.

## Running it

```bash
docker compose -f docker-compose.base44.yml up -d
```

Served at <http://localhost:3000/>. Two non-obvious things the compose/nginx
config handles, both of which will bite again if the config is "simplified":

1. `SS` and `S` have no file extension. nginx cannot infer a MIME type from that,
   so `docker/nginx.base44.conf` forces `default_type text/html` for those two
   locations. Without it the browser downloads the page instead of rendering it
   (this also breaks `python -m http.server`, which sends
   `application/octet-stream`).
2. The repo directory is mode `0700` on the sandbox host, which the unprivileged
   nginx worker user cannot traverse — mounting the repo root as the web root
   returns **403**. Mounting each file individually (as done in the compose file)
   works fine.

If you add more static files or assets, add each one (or its directory) as its own
volume entry in `docker-compose.base44.yml`, and keep the `default_type text/html`
rule in mind for any other extensionless file. After changing volumes, run
`docker compose -f docker-compose.base44.yml up -d` to recreate the container.

## Live editing

There is no dev server, so there is no hot reload: nginx re-reads files on every
request (caching is disabled) and an edit appears as soon as the page is reloaded.
That also means nothing needs `reload_preview` beyond a normal page refresh, and
there is nothing to install — edits to `SS` or `S` are live immediately.

## External dependencies

At runtime the browser fetches Tailwind (CDN script), Lucide icons, Google Fonts,
and an Unsplash background image for the `SS` hero, so **outbound internet access
is needed for styling, icons and that hero image**. The page itself renders fine
without them (plain unstyled HTML). Nothing is fetched by the container, and no
credentials are required.

## Verifying it works

```bash
curl -sI http://localhost:3000/    # 200 + Content-Type: text/html
curl -sI http://localhost:3000/S   # 200 + Content-Type: text/html
docker compose -f docker-compose.base44.yml ps   # web should be (healthy)
```
