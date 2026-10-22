# Sites

**Status:** Draft · **Prefix:** SITE

## Purpose

The websites a team measures, their settings, and who may do what with them.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SITE-1 | Site: `{id (UUID, also the public tracking id), team_id, name (1–100, trimmed), domain, paused (default false), store_query (default true), strip_trailing_slash (default false), excluded_ips ([COL-10](collect.md)), retention_months? ([RET-1](retention.md)), reset_at?, created_by, created_at, updated_at}`. | create without name → refused |
| SITE-2 | `domain`: a bare host name, lower-cased and punycode-encoded, labels `[a-z0-9-_]` ≤ 63 chars, a TLD of ≥ 2 chars, ≤ 500 chars total; `localhost` with an optional port is allowed. A scheme, path, query or IP address → `bad_domain`. `domain` labels the site; it doesn't restrict collection ([TRK-6](tracker.md) does). | `https://ex.com/a` → `bad_domain`; `Ex.COM` → `ex.com` |
| SITE-3 | After create, the site shows its embed snippet: a script tag loading the tracker from this instance with the site `id` set ([TRK-2](tracker.md)). The `id` never changes. | create → snippet contains site id |
| SITE-4 | A user lists the sites of every team they belong to: search by name or domain (case-insensitive substring), sort by `name` or `created_at`, page size default 20, max 100. Sites of other teams → 404, never 403. | non-member GET site → 404 |
| SITE-5 | Move to another team per `@specfork/teams:TEAM-14`; its data, shares and segments move with it. | admin in source, member in target → 403 |
| SITE-6 | `paused` = true: the collect endpoint stores nothing for the site and answers `{disabled: true}` ([COL-6](collect.md)); dashboards keep working. | pause, send pageview → no new event |
| SITE-7 | Resource rows (`@specfork/teams:TEAM-10`): view dashboards, sessions, realtime, event properties, export, apply segments: viewer+ (`@specfork/teams:TEAM-11`); create site, edit `name` and `domain`, create and edit segments ([QRY-10](query.md)): member+; edit other settings, pause, shares ([SHR](sharing.md)), reset, erasure, delete ([RET](retention.md)): admin+. | member edits `retention_months` → 403; viewer creates segment → 403 |
| SITE-8 | Overview of all sites the user can see: per site, `visitors`, `visits`, `pageviews` ([MET-1](metrics.md)) and a `pageviews` series for the chosen range ([QRY-1](query.md)). | 3 sites → 3 cards with totals |

## Out of scope

- Personal (team-less) sites, per-team site quotas, plans and billing.
- Short links, tracking pixels, heatmaps, session replay, Core Web Vitals, revenue, funnels, journeys, retention cohorts, goals, attribution, boards and annotations: separate specs if wanted.

## Decisions

- Sites always belong to a team (a person uses a one-member team); the source let users own sites directly, which doubled every permission check.
- Destructive actions are admin+; the source let any non-viewer member delete a site.
- The source's view-only role is team `viewer`: dashboards, sessions, realtime and export, no writes.
