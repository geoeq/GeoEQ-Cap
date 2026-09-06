# Update channel

`latest.json` is the only file the application reads from the outside
world. It is fetched once at startup (quietly; offline machines notice
nothing) and again from **Help → Check for updates**.

## Fields

| Field | Required | Meaning |
|---|---|---|
| `version` | yes | Newest released build. Any older build shows an update notice with a **Download** button. |
| `minimum_version` | no | Builds older than this show a red *Update required* notice that cannot be dismissed, plus a dialog at startup. Use it only for a release that fixes a wrong result. |
| `download_url` | yes | Where the Download button goes — normally the Releases page. |
| `notes` | no | One line saying what the new version brings. Shown next to the version in the notice bar and in the update dialog. |
| `announcements` | no | List of messages for the notice bar (see below). |
| `promo` | no | Legacy single message `{id, text, url, menu_text}` — the GeoEQ SPT Logs format. Still accepted. |

## Announcements

Each entry in `announcements` is shown in the notice bar above the model
viewport. Several entries scroll one after another; a message that fits
is shown without scrolling. The user can close the bar, and the `id` of
every message shown is remembered on that machine, so nobody sees the
same message twice.

```json
{
  "id": "unique-slug",
  "text": "What you want the user to read.",
  "url": "https://geoeq.dev/…",
  "link_text": "Learn more",
  "from": "2026-09-05",
  "until": "2026-12-31",
  "min_version": "1.0.0",
  "max_version": "1.9.9"
}
```

| Key | Meaning |
|---|---|
| `id` | Change it and the message is shown again, even to users who closed the earlier one. Keep it and an edited text stays hidden for them. |
| `text` | Plain text, one line. Keep it short — it is a strip, not a page. |
| `url`, `link_text` | Optional. With a URL the message is clickable and `link_text` is appended in brackets. |
| `from`, `until` | Optional inclusive dates (`YYYY-MM-DD`). Outside the window the message is not shown, so a campaign can be scheduled ahead and expires by itself. |
| `min_version`, `max_version` | Optional. Target only some builds — e.g. tell 1.x users about something without repeating it to 2.x. |

## Releasing a new version

1. Bump the version in the application, build the installer, and publish
   it on the [Releases page](https://github.com/geoeq/GeoEQ-Cap/releases)
   with the tag `v<version>`.
2. Add the entry to `CHANGELOG.md`.
3. Set `version` (and `notes`) here. Push. Every running copy learns
   about it at its next start.

Never set `minimum_version` above the version that is actually
downloadable.
