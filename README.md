# com-etzhayyim-app-omikuji

omikuji — Omikuji Fortune Platform. Fortune draw and shrine management with
BPMN-driven workflows. It is the Cloudflare Worker serving
`omikuji.etzhayyim.com` and `9t84azyt.etzhayyim.com`.

## Frontend migrated to ClojureScript (2026-09-07)

The `appview/omikuji-mcp-component/svelte/` directory (SvelteKit) is gone.
The frontend is now ClojureScript — reagent + re-frame + `jp-go-dds`
（デジタル庁デザインシステム） — at
[`appview/omikuji-mcp-component/cljs/`](appview/omikuji-mcp-component/cljs).
This was a **frontend-only** migration; the backend Worker/XRPC logic was
moved, not rewritten:

| Then | Now |
|---|---|
| `svelte/src/routes/+page.svelte` (the status page) | [`cljs/src/omikuji/app.cljs`](appview/omikuji-mcp-component/cljs/src/omikuji/app.cljs) — same fields, faithfully ported |
| `svelte/src/routes/xrpc/[...path]/+server.ts` (the file that actually deployed, per `wrangler.jsonc`'s old `main`) | [`src/xrpc-dispatcher.ts`](appview/omikuji-mcp-component/src/xrpc-dispatcher.ts) — moved byte-for-byte, only a provenance header comment added |
| `wrangler.jsonc` `main: svelte/.svelte-kit/cloudflare/_worker.js` | `main` dropped entirely |
| `wrangler.jsonc` `assets.directory: ./svelte/.svelte-kit/cloudflare/client` | `assets.directory: ./cljs/public` |

Neither `src/app.ts` nor the moved `src/xrpc-dispatcher.ts` calls
`env.ASSETS.fetch`, so `main` was dropped rather than repointed at either —
putting either Worker in front of the static assets with no
`env.ASSETS.fetch` call would mean nothing serves the frontend. Both backend
files are orphaned source, not currently wired to any deploy target. See
`appview/omikuji-mcp-component/wrangler.jsonc`'s header comment and
`src/xrpc-dispatcher.ts`'s header comment for the full reasoning.

**This is unverified**: `wrangler deploy` / `wrangler dev` were not run
against this change.

Three fields in `cljs/src/omikuji/app.cljs`'s `default-db` were also
corrected, not merely ported, against what the old Svelte constant held —
see that namespace's docstring for detail:

- `:app/route-count` / `:app/routes` now report the two route patterns
  `wrangler.jsonc` actually declares, not the stale `0` / `[]` the Svelte
  constant carried.
- `:app/vars` now reports the 8 keys `wrangler.jsonc`'s `vars` map actually
  declares (sorted), not the stale `[]` the Svelte constant carried.
- `:app/xrpc?` is now `false`, because `main` no longer deploys the XRPC
  handler (it is preserved, unwired, at
  `appview/omikuji-mcp-component/src/xrpc-dispatcher.ts`).

## Build and test

```bash
cd appview/omikuji-mcp-component/cljs
npm install
npm run build   # shadow-cljs compile app -> public/js/app.js
npm test        # shadow-cljs compile test && node out/tests.js
```
