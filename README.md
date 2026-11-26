# AI Quota Page Deployment Guide

How to deploy the AI quota page (Token Quota) on a new host. The page is
a single static file plus two hero assets — no backend, no database, no
containers. Deployment is straightforward; the main work is replacing the
hardcoded URL and quota cards with the new host's.

Chinese version: [README-zh.md](README-zh.md)

## 1. What's in this directory

| File | What it is | How to use it |
|---|---|---|
| `README.md` | This guide | Follow it |
| `README-zh.md` | This guide (Chinese) | Same content as `README.md` |
| `token-quota.html.template` | Quota page template | Fill in the placeholders, drop the `.template` suffix, and place it on the new host as `token-quota.html` |
| `images/hero-video.mp4` | Hero video | Place it on the new host together with the template (in an `images/` directory) |
| `images/forest-poster.jpg` | Hero poster | Same as above; also the `.hero` element's own background layer |

Every `__PLACEHOLDER__` in the template must be filled in with the new
host's real values (see Section 3). The file cannot be used as-is until
then. The template contains no real IPs, emails, or domains, so it is
safe to keep as it is.

Note: `token-quota.html.template` is a snapshot of the old host's
`token-quota.html` at a point in time. If the page is changed later,
regenerate this template (or copy the new `token-quota.html` and change
its values using the table in Section 3).

## 2. How the page works

- Single static file: HTML + CSS + JS all in one file, depending only on
  the two assets under `images/`. Put it on any web server that can serve
  static files and it runs.
- Quota cards: one line per card in the `CARDS` array,
  `{ software, ai, start, cycle, expires? }`. `start` is any single reset
  moment you know (Beijing time), `cycle` is the reset period (`7d` /
  `24h` / `30d`), `expires` is optional. The page derives the next reset
  from the current Beijing time automatically, so you never need to update
  the dates by hand.
- Clock: server time (from the Date header of a HEAD request, UTC+8),
  re-synced every 60 seconds, ticking every second.
- Hero video: still behind frosted glass on load; hover or tap to play,
  tap again or move the mouse away and it must STOP (pause + back to
  frame 0), not pause.
- Theme: **no toggle button** — it only follows the system
  `prefers-color-scheme`, like Bing's "System default"; when the system
  changes, it changes.
- The timer refreshes every 60 seconds; in the final 10 minutes it
  switches to a 31 ms millisecond countdown; at zero the chip crossfades
  to 🎉 for 10 seconds, then the cycle advances and the cards re-sort;
  expired cards turn grey (⌛) and sink to the bottom.

## 3. Placeholder table

Use this table when filling in the template. The third column shows the
old host's (OL8) current values, for reference only; fill in the new
host's values per the fourth column. Sensitive values are redacted, not
written out.

| Placeholder | What it is | Old host (OL8) value (example) | What to fill in on the new host |
|---|---|---|---|
| `__PAGE_URL__` | Page URL | `https://example.com/token-quota.html` | Full URL of this page on the new host (used for canonical and og:url) |
| `__META_DESCRIPTION__` | Page description | (see template) | One sentence describing the page (shared by description / og:description / twitter:description) |
| `__SOFTWARE_1__` | Card 1 software name | (private, redacted) | e.g. `MuseAI`, `ChatGPT` |
| `__AI_1__` | Card 1 AI name | (private, redacted) | e.g. `Yang`, `Work` |
| `__START_1__` | Card 1 start moment | (private, redacted) | Any single reset moment you know, Beijing time, format `YYYY-MM-DD HH:mm:ss`, e.g. `2027-01-01 00:00:00` |
| `__CYCLE_1__` | Card 1 cycle | `7d` | `7d` / `24h` / `30d` |
| `__SOFTWARE_2__` | Card 2 software name | (private, redacted) | Same as above |
| `__AI_2__` | Card 2 AI name | (private, redacted) | Same as above |
| `__START_2__` | Card 2 start moment | (private, redacted) | Same as above |
| `__CYCLE_2__` | Card 2 cycle | `7d` | Same as above |
| `__SOFTWARE_3__` | Card 3 software name | (private, redacted) | Same as above |
| `__AI_3__` | Card 3 AI name | (private, redacted) | Same as above |
| `__START_3__` | Card 3 start moment | (private, redacted) | Same as above |
| `__CYCLE_3__` | Card 3 cycle | `7d` | Same as above |
| `__EXPIRES_3__` | Card 3 expiry moment | (private, redacted) | Optional; if there is no expiry, delete the whole `, expires: "__EXPIRES_3__"` part |
| `__FOOTER__` | Footer | (private, redacted) | e.g. `© 2026 Your Name` |

To add a card, copy one line in `CARDS` and change its values per the
table above; to remove a card, delete its line. `start` only needs to be
any single reset moment you know — the page rolls it forward
automatically, so you do not need to work out the next reset date.

Hero assets: `images/hero-video.mp4` and `images/forest-poster.jpg`
ship with the template. You can replace them with your own — either
overwrite the files keeping the same names, or change the paths in the
template (`images/hero-video.mp4?v=5` / `images/forest-poster.jpg?v=5`;
`?v=5` is a cache-buster, bump the number when you swap assets so
visitors' browsers do not keep showing the old ones).

## 4. Deployment steps (in order)

1. Copy `token-quota.html.template` and `images/` from this directory to
   the new host, fill in the placeholders from Section 3, drop the
   `.template` suffix, and place it as `token-quota.html` in the site's
   root directory (with `images/` next to it).
2. Confirm `images/hero-video.mp4` and `images/forest-poster.jpg` are
   reachable (opening those URLs directly shows something).
3. Open the new URL in a browser and confirm three things: the clock is
   running (and the time is right), the quota cards are counting down,
   and tapping the hero video plays it and tapping again stops it.
4. Switch the system between light and dark and confirm the page follows
   and the frosted glass stays.

## 5. Notes

- The page is purely static and needs no backend; but the clock reads
  server time from the Date header, so it must be accessed over HTTP(S)
  — opening the file directly from disk leaves the clock without a time.
- `?v=5` is the assets' cache-buster; bump the number after swapping
  the video or poster, or visitors' browsers may keep showing the old
  ones.
- Templates are snapshots: after `token-quota.html` changes, regenerate
  the template (or copy the new version and change the placeholders); do
  not apply an old template to a new page.
- Anti-translation: the page carries
  `<meta name="google" content="notranslate">` + `translate="no"`;
  do not remove them.
