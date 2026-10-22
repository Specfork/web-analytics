# Events

**Status:** Draft · **Prefix:** EVT

## Purpose

What one pageview or custom event stores, and how URLs, referrers, campaigns and properties are parsed.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| EVT-1 | Event: `{id, site_id, session_id, visit_id, type: pageview\|custom, created_at, hostname, url_path, url_query?, page_title?, referrer_domain?, referrer_path?, referrer_query?, utm_source?, utm_medium?, utm_campaign?, utm_content?, utm_term?, click_id?, name?, tag?}`. | pageview → `name` null |
| EVT-2 | `type` = `custom` when `name` is present, else `pageview`. An identify request creates no event. | `{url, name: "signup"}` → custom |
| EVT-3 | URL: `url` is resolved against `https://<hostname>` (or `https://localhost`); `url_path` = path plus `#fragment`, percent-decoded where valid; `url_query` = query without `?`. `hostname` = payload `hostname`, else the URL's host. With `strip_trailing_slash` ([SITE-1](sites.md)) a trailing `/` is removed except for the root `/`. | `/a/?x=1#t` → path `/a/#t`, query `x=1` |
| EVT-4 | Lengths: `url_path`, `url_query`, `page_title`, referrer fields ≤ 500; `hostname` ≤ 100; `name`, `tag` ≤ 50; UTM values ≤ 255. Longer values are truncated, not refused. | 600-char path → 500 kept |
| EVT-5 | Referrer: parsed like EVT-3 (a path-only referrer resolves against the event's host); `referrer_domain` lower-cased, punycode, port and leading `www.` removed. A referrer on the event's own domain (also without `www.`) is a self-referral: `referrer_domain` null, path kept. | `www.ex.com` page from `ex.com/x` → domain null |
| EVT-6 | Campaigns: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` read from the page URL's query. `click_id`: the ad platform whose click parameter is present: `google` (`gclid`), `meta` (`fbclid`), `microsoft` (`msclkid`), `tiktok` (`ttclid`), `linkedin` (`li_fat_id`), `x` (`twclid`); the value is dropped ([PRIV-8](privacy.md)). | `?gclid=abc` → `click_id` `google` |
| EVT-7 | Custom event names are case-sensitive, 1–50 chars after truncation; `tag` groups traffic (A/B variants, sections) and applies to pageviews and events. | `Signup` ≠ `signup` |
| EVT-8 | Event properties: `{event_id, key, type: string\|number\|boolean\|date\|array, string_value?, number_value?, date_value?}`, stored only for custom events. | pageview with `data` → no properties |
| EVT-9 | Flattening: nested objects become dotted keys (`a.b`); a string in ISO 8601 date-time form is a `date`; numbers keep 4 decimals; booleans store `true`/`false`; arrays store their JSON text and are dropped if it exceeds 500 chars; strings truncated to 500; keys to 500. More than 50 keys after flattening → 400 `too_many_properties`. | `{a: {b: 1}}` → key `a.b`, number 1 |
| EVT-10 | Event data views per range and filters: event names with counts and visitors; per name, its property keys with counts; per key, top 100 values with counts, and for numbers `sum`, `avg`, `min`, `max`. | event `buy`, key `qty` → sum of quantities |
| EVT-11 | Activity list: the site's events in a range, newest first, paged (default 20, max 100), searchable by path, title, event name or referrer domain (case-insensitive substring). | search `pricing` → only matching events |

## Out of scope

- Revenue and currency fields, link and pixel events, web-vitals events.

## Decisions

- Property count limit (50) is enforced; the source documented it but never checked.
- A self-referral also matches across a `www.` difference, as the source did.
