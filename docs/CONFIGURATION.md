# Configuration

Almost everything is set in the browser (Settings > Admin settings, admins only). This page explains what
that writes, for backups, for scripting, and for the few things that stay outside the page.

Everything is read from `CONFIG_DIR` (set to `/config` by the Dockerfile). Examples are in
`config.example/`. None of the real files are ever committed.

## Roles

| | Regular user | Admin |
| --- | --- | --- |
| Watch, search, favorites, choose and reorder their own groups | yes | yes |
| Hide channels and put channels in their own order (per group, and for favorites) | yes | yes |
| Refresh a provider's channel list, stop their own streams | yes | yes |
| Change their own password (needs the current one) | yes | yes |
| See or reach Admin settings | no (the server refuses the requests) | yes |
| Providers, people, name cleanup, guide sources, security | no | yes |
| Change which guide a channel uses (applies to everyone) | no | yes |
| Set another person's password | no | yes |

Rules the server enforces: there is always at least one admin; you can't remove your own account or reset
your own password from the admin screen (use *Change password*); a temporary or default password must be
replaced at first sign-in before anything else works; changing a password signs that person out everywhere
else.

## Files

| File | Purpose |
| --- | --- |
| `providers.json` | Provider list. Written by the Providers screen. |
| `users.json` | People: name, role, scrypt hash, state file, `mustChange`. Written by the People screen and the commands below. |
| `settings.json` | Names, guide and security settings. Written by the Names and Security screens and by guide addresses on a provider. Unknown keys are kept. |
| `secret.key` | Key that seals URLs sent to the browser and signs sessions. Created if missing. Keep it when restoring. |
| `epgmap.json` | Stream-to-guide-id overrides: `{ "<provider id>": { "<stream id>": "<guide id>" } }`. Written by the guide screen in Edit mode (admins tap a channel's name) and by `epgimport`. |
| `state-<person>.json` | That person's chosen groups (in their order), favorites, hidden channels, channel order per group, and last channel. |
| `cache/` | Catalog and guide caches. Rebuilt automatically. For M3U providers the cache holds stream addresses, so it is written with owner-only permissions. |

### providers.json

```
{ "providers": [
  { "id": "primary", "name": "Primary", "type": "xtream", "url": "http://host:8080", "username": "...", "password": "...", "format": "m3u8" },
  { "id": "playlist", "name": "My playlist", "type": "m3u", "url": "https://.../list.m3u", "format": "m3u8" },
  { "id": "antenna", "name": "Antenna", "type": "hdhr", "url": "http://192.168.1.50" }
] }
```

- `type` is `xtream` (default, so v1 files work unchanged), `m3u`, `hdhr` (an HDHomeRun tuner, see below) or `tablo` (a Tablo 4th Gen, experimental, see below).
- `format` is `m3u8` (HLS) or `ts`. For an M3U playlist, *Test connection* looks at the stream addresses and
  picks the right one.
- `id` is made from the name when you add a provider and never changes afterwards.
- An M3U channel gets a stable numeric id from its guide id, name and group, so favorites survive a refresh.
  If two entries have identical guide id, name and group they are numbered in file order.
- Groups for an M3U provider come from `group-title` (or `#EXTGRP`). Entries without one appear under
  *Uncategorized*.
- Guide addresses live in `settings.json` under `epg`, keyed by provider id, and are edited on the provider.
  Xtream providers use their own guide as a fallback; M3U providers have none unless an address is added.
- The admin screens never send a password back to the browser. When you edit a provider, leaving a secret
  blank keeps the saved one.

### settings.json

| Key | Meaning | Default |
| --- | --- | --- |
| `trimFromNames` | Text hidden in channel names (comma separated string, or a list). | none |
| `stripSuperscripts` | Remove superscript and subscript characters from channel names. | off |
| `epg` | `{ "<provider id>": ["guide address", ...] }` | none |
| `epgRefreshHours` | How often guides refresh. | 12 |
| `epgNameMatch` | Match guide data by cleaned-up channel name when there is no guide id. | on |
| `trustIpHeader` | Header your reverse proxy sets to the real visitor address. | none |
| `lockUserFails` | Wrong passwords per person and address before lockout. | 5 |
| `lockIpFails` | Wrong passwords from one address in total before lockout. | 20 |
| `lockMinutes` | Lockout length and counting window. | 10 |
| `adminAllow` | List of addresses or ranges (IPv4 or IPv6, CIDR) allowed to open Admin settings. Empty means no limit. | none |

All of these apply immediately after saving. `TRUST_IP_HEADER` in the environment overrides `trustIpHeader`
(and the page shows it as locked).

## Tablo 4th Gen (experimental)

**Experimental.** Sign-in, the channel list and playback have been confirmed on one real Tablo 4th Gen; the guide
and ending streams cleanly have only been tried against a pretend Tablo so far. It was written from community
descriptions of how the Tablo app talks to its cloud and device (the `tablo-api` library and TabloView's API
notes). Treat it as a preview. Tablo publishes no API, so Tablo can change or block this at any time. Only 4th Gen devices
are covered.

Add a provider of type *Tablo 4th Gen*, with your Tablo account email and password. The address is optional: leave
it blank to use the one your account reports, or enter the Tablo's address on your network (for example
`192.168.1.60`) if that doesn't work. Zibbit signs in to the Tablo cloud, lists the antenna channels in one group
called *Antenna*, then asks the device on your network to start each channel you watch. Streaming-app (OTT)
channels are left out.

- **Your Tablo password is stored in `providers.json`** on the server, like any other provider login, because
  Zibbit signs in again as needed. It is never sent to the browser.
- **The server must be able to reach the Tablo** on your network (port 8885, plus the port it streams on).
- **Video is converted while someone watches**, the same way as for an HDHomeRun, so ffmpeg is needed. People on
  the same channel share one stream.
- **Guide:** the program listings come from the Tablo's own guide, with no XMLTV address needed. They are read from
  the Tablo cloud's daily guide grid, or, if that has none of your channels, from the Tablo itself in slow batches
  (it refuses connections for about a minute after a burst). Any XMLTV addresses you add are read after it.
- Stopping a channel tells the Tablo the session is over, so its tuner is freed straight away.
- The two request-signing keys come from the Tablo app (as listed by `tablo-api`). If Tablo changes them, set
  `TABLO_HASH_KEY` and `TABLO_DEVICE_KEY`.
- The number of tuners isn't read from the device, so nothing stops you starting more streams than it has; the
  Tablo then answers "all tuners are busy". `STREAM_LIMIT` can cap it.

## HDHomeRun tuners

An HDHomeRun is added as a provider of type *HDHomeRun tuner (antenna)*; the address is all it needs (for example
`192.168.1.50`). Zibbit reads the device's channel list and puts every channel in one group called *Antenna*,
named with its number (`3.1 WLBT-DT`). Copy-protected cable channels are left out.

- **The server must be able to reach the tuner.** If Zibbit runs on a separate network segment from the tuner,
  allow that traffic (the tuner's web port, and port 5004 for video).
- **Video is converted while someone watches.** Antenna video is MPEG-2 with AC3 audio, which browsers and
  phones can't play, so ffmpeg turns each watched channel into H.264/AAC HLS (720p by default, see
  `TRANSCODE_MAX_HEIGHT`). People on the same channel share one conversion, and it stops as soon as the last
  person stops watching or closes the page. The image includes ffmpeg; running from source needs it installed.
  Expect roughly a third of one processor core per watched channel at 720p, and a few seconds before video
  starts.
- **Tuners are the limit.** The stream limit for a tuner is its tuner count, counted per channel (two people on
  one channel use one tuner). If something else is using the tuners, playback fails with "all tuners are busy".
- **Guide.** If you have SiliconDust's guide subscription, the listings are read from their service automatically (using
  the tuner's own key, so nothing to enter) and channels are matched by number. Zibbit then needs internet access to
  `api.hdhomerun.com`. Without the subscription the tuner has no program data; add an XMLTV guide address on the
  provider instead. Any address you add is read after the tuner's own guide.

## How the guide is matched

Guide sources are XMLTV files listed per provider. A channel uses the first source that has a programme on
right now. Matching goes in this order:

1. A saved override in `epgmap.json`.
2. The provider's own guide id for the channel (`epg_channel_id` or `tvg-id`).
3. The cleaned-up channel name, when `epgNameMatch` is on. When several guide channels share a name, the one
   listed first in the file wins, which can pick the wrong country. Use an override for those.
4. Local-station call signs (for example `WLBT`), preferring the plain `.us` id.

If nothing is on right now in the XMLTV data, an Xtream provider's own guide answers instead. Only channels
somebody has ticked or starred are kept, for a few hours back and a day ahead, so memory stays small.
The server logs one line per guide load (`guide <name>: n/m sources read, x of y channels matched`) and never
logs guide addresses. Per-channel misses are not logged; use the Edit-mode guide sheet to see one channel.

## Environment variables

| Variable | Default | Notes |
| --- | --- | --- |
| `PORT` | 8080 | |
| `CONFIG_DIR` | `./config` (`/config` in the image) | |
| `ADMIN_INITIAL_PASSWORD` | `admin` | Only used when the very first account is created. Must still be changed at first sign-in. |
| `APP_PASSWORD` | | Used the same way as `ADMIN_INITIAL_PASSWORD` if that is not set. |
| `CATALOG_TTL_HOURS` | 6 | How long a provider's channel list is reused. |
| `TRUST_IP_HEADER` | | Overrides `trustIpHeader`. |
| `EPG_REFRESH_HOURS`, `TRIM_FROM_NAMES`, `STRIP_SUPERSCRIPTS` | | Used only when the matching key is absent from `settings.json`. |
| `STREAM_LIMIT` | 0 | A number caps simultaneous streams per provider; `auto` uses the Xtream account's own limit (Xtream only). |
| `LIMIT_CACHE_MS` | 60000 | |
| `TRANSCODE_MAX_HEIGHT` | 720 | HDHomeRun channels are converted to at most this many lines of video (0 keeps the original size, which needs more processor time). |
| `TRANSCODE_DIR` | system temp folder | Where the short-lived video segments are written. Nothing there is kept. |
| `FFMPEG_PATH` | `ffmpeg` | Only needed if ffmpeg is not on the path (the image includes it). |

## Command line

Run against the running container. These cover what the browser can't, mainly recovery.

```
docker exec -it zibbit node server.js users
docker exec -it zibbit node server.js adduser <name> [admin]
docker exec -it zibbit node server.js passwd <name>
docker exec -it zibbit node server.js deluser <name>
docker exec -it zibbit node server.js promote <name>
docker exec -it zibbit node server.js demote <name>
docker exec -it zibbit node server.js clear-ip-rules
docker exec -it zibbit node server.js epgimport <provider id> <playlist file>
```

- `passwd` sets a password without forcing a change (you are at the console).
- `clear-ip-rules` removes the `adminAllow` list if you ever lock yourself out of the Admin page.
- `epgimport` copies a playlist's stream-to-guide-id pairs into `epgmap.json`. A provider playlist contains
  the provider login in its URLs, so never commit one.

## Security notes

- **Admin address rule.** It applies only to Admin settings requests, not to sign-in or playback, so a
  phone or car on a changing mobile address can still watch. Loopback and private home-network addresses are
  always allowed. The page refuses a rule that would block the address you are using, and warns when the
  rules cover only IPv4 or only IPv6. It can't know about your *other* devices, which is why
  `clear-ip-rules` exists.
- **The address the app sees.** Lockouts and the address rule use the visitor address the app is given. Behind
  a reverse proxy, name a header the proxy sets itself and overwrites on every request (for example `cf-connecting-ip` behind Cloudflare). Don't expose the container's port directly if you rely on
  these, because a direct client could send its own header.
- **Default password.** Change it at first sign-in is enforced, but the account exists until then. Set
  `ADMIN_INITIAL_PASSWORD` or keep the app off the internet until you have signed in once.
- **Secrets at rest.** `providers.json`, `users.json`, `settings.json` and `secret.key` are written with
  owner-only permissions. Provider passwords are stored in `providers.json` (the app needs them to stream),
  so protect the config directory and its backups.
