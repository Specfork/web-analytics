---
specfork: 1
name: "@specfork/web-analytics"
summary: Cookieless website analytics with a tracking script, hashed sessions, pageviews, custom events, referrers, UTM, dashboards, filters, realtime, public shares and retention.
license: CC-BY-4.0
tags: [saas, analytics]
requires:
  "@specfork/auth": "^0.3.0"
  "@specfork/teams": "^0.3.0"
derived_from:
  repo: https://github.com/umami-software/umami
  commit: e92fdab99508a8ed05249f6c5f07e140deab79e1
  url: https://umami.is
  license: MIT
---

# Web analytics

A team adds its websites, embeds one small script, and sees who visits, from where, on which
pages and devices, without cookies and without storing IP addresses. Visitors are counted with a
salted hash that rotates daily, so nobody can be followed across periods or sites.
Behaviour derived from Umami's open-source code (MIT), written fresh and stack-free.
Features that exist only in Umami Cloud (plans, usage limits, white-labelling) were not used.
Not affiliated with or endorsed by Umami Software, Inc. The name is used only to credit the source.

| Area | Prefix | Covers |
|---|---|---|
| [sites.md](specs/sites.md) | SITE | Sites, settings, tracking snippet, moving between teams, permissions |
| [tracker.md](specs/tracker.md) | TRK | Tracking script: options, automatic pageviews, click events, JS API, opt-out |
| [collect.md](specs/collect.md) | COL | Collect endpoint, validation, limits, batch, client IP, bot and IP exclusion |
| [privacy.md](specs/privacy.md) | PRIV | What is stored and never stored, session hashing, salt rotation, location |
| [sessions.md](specs/sessions.md) | SES | Sessions, visits, visitors, devices, identify, session properties |
| [events.md](specs/events.md) | EVT | Pageviews, URL and referrer parsing, UTM, custom events and properties |
| [metrics.md](specs/metrics.md) | MET | Stats formulas, comparison, time series, breakdowns, channels, export |
| [query.md](specs/query.md) | QRY | Time ranges, units, time zones, filters, property filters, segments |
| [realtime.md](specs/realtime.md) | LIVE | Active visitors and the last-30-minutes view |
| [sharing.md](specs/sharing.md) | SHR | Public read-only dashboards by link |
| [retention.md](specs/retention.md) | RET | Retention, purge, reset, site deletion, erasure by visitor |

Every site belongs to one team (`@specfork/teams`); all analytics data belongs to exactly one site.
All times are stored in UTC; API names are `snake_case`.
