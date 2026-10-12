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

Served at <http://localhost:3000/>. The compose file bind-mounts the entry file `S`,
the `assets/` directory, and the nginx config — not the repo root:

1. `S` has no file extension. nginx cannot infer a MIME type from it, so
   `docker/nginx.base44.conf` forces `default_type text/html`. Without that the
   browser downloads the page instead of rendering it (this also breaks `python -m
   http.server`, which sends `application/octet-stream`).
2. The repo directory is mode `0700` on the sandbox host, which the unprivileged
   nginx worker user cannot traverse — mounting the repo root as the web root gives
   403. Mounting a single file or a subdirectory (as done for `S` and `assets/`)
   works fine.

If you add more static files or assets, add each one (or its directory) as a volume
entry in `docker-compose.base44.yml`, and keep `default_type text/html` in mind for
any other extensionless file. After changing volumes, run
`docker compose -f docker-compose.base44.yml up -d` to recreate the container.

## Live editing

There is no dev server and no hot reload. nginx re-reads files on every request
(caching is disabled), so an edit shows up as soon as the page is reloaded.

## External dependencies

At runtime the browser also fetches Tailwind (CDN script), FontAwesome and Google
Fonts, so **outbound internet access is needed for the styling and icons**. All
imagery is local; the container fetches nothing.

### Member / album imagery lives in `assets/`

`TRIPLES_MEMBERS[].photo` and `TRIPLES_DISCOGRAPHY[].cover` point at
`/assets/member-<NN>.jpg`, served by this same nginx service. Album covers reuse the
associated member's photo, as the original page did.

These are the real member photos from Wikimedia Commons (files named
`TripleS S<n> <hangul name>.jpg`). They were **downloaded into the repo** because
hotlinking Commons is unreliable here and previously produced a console full of
`net::ERR_BLOCKED_BY_ORB` errors:

- Commons rejects arbitrary thumbnail widths (`400, Use thumbnail sizes listed…`), so
  the `/thumb/<hash>/<file>/800px-<file>` URLs the page originally used could never
  load. The original filenames were also invented (`240513_tripleS_<Name>.jpg`) and do
  not exist on Commons at all.
- `upload.wikimedia.org` / `thumb.wikimedia.org` aggressively rate-limit this
  environment (`429`), which the browser reports as `ERR_BLOCKED_BY_ORB` because the
  error body is HTML.

To swap a photo, overwrite `assets/member-<NN>.jpg` — no code change is needed. See
`assets/CREDITS.md` for the source file of each image and the licence caveat.

## Verifying it works

```bash
curl -sI http://localhost:3000/                      # 200 + Content-Type: text/html
curl -sI http://localhost:3000/assets/member-01.jpg  # 200 + image/jpeg
docker compose -f docker-compose.base44.yml ps       # web should be (healthy)
```
