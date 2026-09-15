# SentryWebsite

The marketing site for [Sentry](https://github.com/malekTheCoder/Sentry), a
native macOS system monitor with iPhone and Apple Watch companions and an
MCP server for coding agents.

Live at **<https://malekswilam.dev/SentryWebsite/>**.

## What's here

```
index.html           the entire page — inline CSS, inline JS, no build step
404.html             the not-found page, same treatment
apple-touch-icon.png 180x180, for iOS home screens
assets/              screenshots (avif + webp + png) and the social card
.nojekyll            stops Pages running the files through Jekyll
```

That's deliberate. There is no bundler, no package.json, no dependency to
update. Open `index.html` in an editor, save, push. The only things fetched
at runtime are two Google Fonts.

To work on it locally:

```bash
python3 -m http.server 8777
```

Then open <http://localhost:8777>. Opening the file directly with `file://`
works too, but the screenshots resolve more predictably over HTTP.

## Deployment

GitHub Pages builds from the `main` branch, root directory. Push to `main`
and it's live in a minute or two.

The custom domain comes from the user-page repo, not this one, which is why
the site sits at `malekswilam.dev/SentryWebsite/` rather than at a root. Two
consequences worth remembering:

- **No `CNAME` file belongs in this repo.** Project pages inherit the apex
  domain from the user site. Adding one here would fight it.
- **A `robots.txt` here would do nothing.** Crawlers only read it at the
  domain root, so it belongs in the user-page repo.

## The email signup

The form is wired for [Buttondown](https://buttondown.com) but switched off
until it has an account to post to. While `BUTTONDOWN_USERNAME` is empty the
field is disabled and the page says the list isn't open — it will not claim
a signup it didn't make.

To turn it on:

1. Create an account at <https://buttondown.com>. The free tier covers the
   first 100 subscribers.
2. Find your username — it's the last part of your newsletter URL,
   `buttondown.com/<username>`, and it's in **Settings → Basics**.
3. In `index.html`, find `BUTTONDOWN_USERNAME` (near the bottom, in the
   release-notes signup block) and put the username between the quotes:

   ```js
   var BUTTONDOWN_USERNAME = "sentry-app";
   ```

4. Push. The form starts posting to
   `https://buttondown.com/api/emails/embed-subscribe/<username>`, which
   opens Buttondown's confirmation page in a new tab.

No API key is involved and nothing secret ends up in the page — the embed
endpoint is public by design, which is why this is safe in a static site.
The API key in your Buttondown settings is for the REST API; don't put it
here.

Two settings worth turning on while you're in there: **double opt-in**, so a
typo'd address can't sit on the list forever, and a **welcome email**, since
someone signing up today is waiting on a release that may be weeks out.

## Keeping the page honest

Most of this page makes checkable claims about the app, and those claims
drifted badly before. When the app changes, these are the things that go
stale — all of them live in `index.html`:

| Claim on the page | Source of truth in the app repo |
|---|---|
| 9 menu bar modules | `SentryKit/Models/MetricID.swift` → `MetricModule` |
| 3 themes, and their swatch hexes | `SentryKit/Settings/Theme.swift` → `builtInPresets` |
| Only System uses behind-window blur | `Theme.swift` → `useMaterialBackground` |
| 14 alert rules | `SentryKit/Services/AlertEngine.swift` → `defaultRules` |
| 2 of those have no editable condition | `Sentry/Settings/Panes/AlertsPane.swift` (read-only rows) |
| 20 MCP tools | `SentryKit/Services/MCPTool.swift` → `MCPToolID` |
| 30-minute cooldown, 6/hour rate cap | `AppSettings.swift` → `defaultAlertCooldownMinutes`, `AlertEngine` → `rateCapPerHour` |
| 48h raw / 90d hourly / daily forever | `SentryKit/Persistence/RollupJob.swift` |
| 13 keep-awake modes, 5 of them conditional | `Sentry/Dropdown/SleepControlCard.swift` → `isConditional` |
| CSV/JSON history export | `SentryKit/Persistence/HistoryExport.swift` |
| History ranges 24h–6mo | `Sentry/Dashboard/TimeRangePicker.swift` |
| macOS 14+, universal binary | `project.yml` → `deploymentTarget`; `ARCHS` is unset, so standard |
| Sensors need Apple Silicon | `SystemMetricsKit/Bridges/` — no `#if arch` anywhere; HID/IOReport are AS-only |
| Nothing runs as root | no `SMAppService.daemon` register, no `SMJobBless`, no setuid |
| `sentryctl`, `SentryMCP` | `project.yml` → `EXECUTABLE_NAME` |
| **iPhone/Watch are not distributed** | `project.yml` — one scheme, and it builds none of them |

Two of these are load-bearing and easy to get wrong:

- **There is no paid tier.** Sentry is free and MIT-licensed; the page must
  never describe a feature as held back, priced, or unlocked by anything. If
  a gate ever reappears in the app, this page is wrong until it is edited.
- **The companion apps ship in nothing.** They are real code built by no
  scheme. The page says so. If they ever get a release channel, the
  companions section needs revisiting.

If you add a theme or an alert rule, the number on the page is wrong the
moment you merge. It's a two-word edit; the cost is only in remembering.

## Screenshots

Every screenshot ships three times — `.avif`, `.webp` and a `.png`
fallback — inside a `<picture>`. **Do not just copy PNGs over the top.**
That would replace the resized, stripped files with full-resolution captures
and leave the `.avif`/`.webp` siblings showing the old image, since browsers
pick those first.

To refresh one, start from the master in `../MacStat/docs/screenshots/`,
resize to roughly 2x its largest displayed size, then re-encode all three:

```bash
cwebp  -q 82 -sharp_yuv shot.png -o assets/shot.webp
avifenc -y 444 -q 64 shot.png assets/shot.avif
pngquant --strip --force --output assets/shot.png -- shot.png
```

`-y 444` matters: chroma subsampling smears the small coloured text in these
UI captures. Strip metadata on all three, then update the `width`/`height`
attributes on the `<img>` to the new intrinsic size, or the page reserves
the wrong box and shifts as it loads.

Current set: `macos-dashboard` (hero), `macos-menubar` (dropdown),
`ios-dashboard`, `ios-alerts`, `watch-overview`. Plus `og-card.png` — the
1200x630 social card, PNG only and never rendered in the page, so scrapers
that reject modern formats still get it.

Two gaps: there is no capture of the desktop widgets, so that section is
text only; and every capture exists in one appearance, so the macOS shots
wash out on the light theme and the iPhone and Watch shots wash out on the
dark one.

## A note on the name

"Sentry" is provisional pending a trademark decision, and it collides with
the well-known error-tracking company. In this page it appears only as
visible text, in `<title>`, and in the meta description — never in a URL, a
class name or an asset filename. A rename is a find-and-replace of the
visible strings plus the GitHub links.
