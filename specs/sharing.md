# Sharing

**Status:** Draft · **Prefix:** SHR

## Purpose

Let anyone with a link see a site's dashboard read-only, without an account.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SHR-1 | Share: `{id, site_id, slug, name (≤ 200), sections, allow_filters (default true), theme? (light\|dark), created_at, updated_at}`. `slug`: `[A-Za-z0-9]{8,50}`, unique across the instance, random 16 chars if not given; taken → `taken`. A site may have several shares. | slug `abc` → refused (short) |
| SHR-2 | `sections` ⊆ `overview`, `events`, `sessions`, `realtime`; default `overview` + `events`. | create without sections → overview, events |
| SHR-3 | `/share/{slug}` opens the dashboard read-only, without sign-in, limited to its sections; unknown slug → 404. Pages carry `noindex`. | open link logged out → overview shown |
| SHR-4 | Opening a share issues a signed share token `{share_id, site_id, sections}`; analytics endpoints accept it only for that site and for a section it includes, and check the share still exists on every call. Other sites, settings, export, segments editing and erasure → 403. | share token asks another site → 403 |
| SHR-5 | Through a share, `distinct_id`, identify properties and session detail ([SES-9](sessions.md)) are never returned; the `sessions` section shows the session list without them. | `sessions` section → list with no distinct ids |
| SHR-6 | `allow_filters` = false: filter, property-filter and segment parameters are ignored server-side. | filter param on locked share → unfiltered numbers |
| SHR-7 | Deleting a share or changing its slug stops the old link and its tokens at once. Create, edit and delete: admin+ ([SITE-7](sites.md)). | delete share, reload → 404 |

## Out of scope

- Password-protected shares, shared multi-site boards, custom branding.

## Decisions

- Shares never expose visitor identifiers, whatever the sections; the source showed whatever the section included.
- Fewer sections (four) than the source because reports are out of scope.
