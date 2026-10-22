# Privacy

**Status:** Draft · **Prefix:** PRIV

## Purpose

Count visitors without cookies and without keeping anything that identifies a person.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PRIV-1 | No cookies, local storage, fingerprinting scripts or cross-request identifiers are used on the tracked site ([TRK-9](tracker.md)); a visitor is recognised only by the server-side hash of PRIV-3. | manual: browser storage empty |
| PRIV-2 | The IP address and the user-agent string are used only in memory while handling one request (hash, location, device fields) and are never written to the database, queues, caches or logs; collect access logs omit both. | grep stored data and logs for test IP → no hits |
| PRIV-3 | `session_id` = first 128 bits of HMAC-SHA256(key = current salt, message = `site_id`, IP, user agent, `distinct_id` or empty, joined by a separator), formatted as a UUID. | same IP+UA, two sites → two different ids |
| PRIV-4 | Salt: ≥ 256 random bits, one per instance, replaced at the start of every rotation period in UTC. `salt_rotation` (operator setting): `day` (default) \| `week` (ISO week, Monday) \| `month`. | default: same visitor before/after 00:00 UTC → 2 sessions; `month` → 1 |
| PRIV-5 | Only the current salt exists. At rotation the old salt is destroyed and never written to backups, so past hashes can't be recomputed or linked to a person; the salt is chosen by receive time, not `timestamp`. | manual: no salt history in storage |
| PRIV-6 | Location: `country` (ISO 3166-1 alpha-2), `region` (ISO 3166-2, `CC-XX`), `city` (English name), from a local IP geolocation database or the operator's trusted CDN geo headers. Loopback, private and invalid IPs → all null. Coordinates and postal codes are never stored. | IP `192.168.1.5` → country null |
| PRIV-7 | Per session only coarse device fields are kept ([SES-4](sessions.md)): browser and OS family without full version, device class, screen size, language. | stored browser `Chrome`, not `Chrome 126.0.1` |
| PRIV-8 | `store_query` = false ([SITE-1](sites.md)): UTM values and click-id kinds ([EVT-6](events.md)) are extracted, then the query string is discarded (page and referrer). Click-id values are never stored. | `?email=a@b.c&utm_source=x`, store off → only `utm_source` kept |
| PRIV-9 | The same person on two sites can't be linked: ids differ per site (PRIV-3) and no query joins sessions across sites. | manual: no cross-site endpoint |
| PRIV-10 | `distinct_id` and identify properties ([SES-6](sessions.md)) are personal data the site chose to send: shown only to site members, never on public shares ([SHR-5](sharing.md)), erasable by `distinct_id` ([RET-5](retention.md)). | share viewer requests session list → no distinct ids |

## Out of scope

- Consent banners (not needed without cookies); server-side honouring of `DNT` or `Sec-GPC` headers.

## Decisions

- Salts are random and destroyed at rotation; the source derived the salt from the period's start date plus the app secret, so anyone with the secret could recompute every hash.
- Default rotation is `day` (owner's decision): a visitor can't be linked across days, at the cost of counting a returning visitor once per day in multi-day ranges. Operators who need exact weekly or monthly counts choose `week` or `month` (the source's period).
- Click-id values (ad platform identifiers) are reduced to which platform sent the visit; the source stored the raw values.
