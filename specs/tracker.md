# Tracker

**Status:** Draft · **Prefix:** TRK

## Purpose

The small script a site embeds to send pageviews and events to the collect endpoint ([COL](collect.md)).

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| TRK-1 | Served at `/script.js` (operator may rename the path, and the collect path, to dodge blockers); ≤ 6 KB gzipped; `Cache-Control: public, max-age=86400`; no dependencies; works in the last two versions of major browsers. | GET `/script.js` → JS, cache header set |
| TRK-2 | Options are attributes of its own script tag: `data-site` (required), `data-host` (collect origin, default the script's origin), `data-auto-track` (default true), `data-domains` (comma list), `data-respect-dnt` (default false), `data-exclude-search` (default false), `data-exclude-hash` (default false), `data-tag` (≤ 50), `data-distinct-id`, `data-before-send` (name of a global function). | tag without `data-site` → sends nothing |
| TRK-3 | Payload of every send: `site`, `hostname`, `url` (absolute page URL, minus query and/or fragment when excluded), `referrer` (same-origin referrers reduced to their path), `title`, `screen` (`<width>x<height>` of the screen), `language` (browser language), `tag?`, `distinct_id?`. | page `/a?x=1`, exclude-search → `url` without `?x=1` |
| TRK-4 | With auto-track on: one pageview when the document has loaded; then, on every history push or replace that changes the URL, the previous URL becomes `referrer` and a pageview is sent 300 ms later (so the new title is read). Same URL → nothing. | SPA route /a → /b → 2 pageviews, 2nd referrer `/a` |
| TRK-5 | Declarative events: a click on an element (or its descendant) carrying `data-event="<name>"` sends custom event `<name>`; each `data-event-<key>="v"` attribute adds string property `<key>`. For a same-tab link, navigation waits until the send settles (≤ 1 s); new-tab and modifier-key clicks don't wait. | click `<a data-event="signup" data-event-plan="pro">` → event `signup`, `{plan: "pro"}` |
| TRK-6 | Nothing is sent when: no `data-site`; `data-domains` is set and the page host isn't in it (exact match); `data-respect-dnt` is on and the browser's Do Not Track is `1` or `yes`; local storage key `analytics.disabled` is set on this browser; or the server answered `disabled: true` ([SITE-6](sites.md)). | DNT on + respect-dnt → no request |
| TRK-7 | JS API on `window.webAnalytics`: `track()` = pageview; `track(name, data?)` = custom event; `track(object)` sends the object as the payload; `track(fn)` sends `fn(default payload)`; `identify(distinct_id?, data?)` or `identify(object)` sends `type: identify` ([SES-6](sessions.md)). All return a promise and never throw. | `track("buy", {qty: 2})` → event `buy` with `qty` 2 |
| TRK-8 | `data-before-send`: the named function gets `(type, payload)` and returns the payload to send (sync or async); a falsy return cancels the send. | hook returns null → no request |
| TRK-9 | The script sets no cookies and writes nothing to browser storage. It keeps the server's `cache` token ([COL-6](collect.md)) in memory only and returns it in header `X-Analytics-Cache`; `identify` clears it first. | manual: no cookies or storage after visit |
| TRK-10 | Sends are `POST` JSON with `keepalive` so they survive unload, credentials omitted; network errors are swallowed silently. | collect down → page shows no error |

## Out of scope

- Outbound-link and file-download auto events, scroll depth, Core Web Vitals, heatmap and replay recording.

## Decisions

- Generic attribute and global names (`data-site`, `webAnalytics`) instead of the product's.
- Same-tab link clicks wait at most 1 s; the source waited for the request with no limit.
