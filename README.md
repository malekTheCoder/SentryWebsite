# SentryWebsite

The marketing site for [Sentry](https://github.com/malekTheCoder/Sentry), a
native macOS system monitor with iPhone and Apple Watch companions and an
MCP server for coding agents.

Live at **<https://malekswilam.dev/SentryWebsite/>**.

## What's here

```
index.html    the entire page — inline CSS, inline JS, no build step
404.html      the not-found page, same treatment
assets/       screenshots, copied from the app repo
.nojekyll     stops Pages running the files through Jekyll
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
| 6 themes | `SentryKit/Settings/Theme.swift` → `builtInPresets` |
| 14 alert rules | `SentryKit/Services/AlertEngine.swift` → `defaultRules` |
| 20 MCP tools | `SentryKit/Services/MCPTool.swift` → `MCPToolID` |
| 13 keep-awake modes | `Sentry/Dropdown/SleepControlCard.swift` |
| Pro feature list | `SentryKit/Pro/ProEntitlement.swift` → `ProFeature` |
| History ranges 24h–6mo | `Sentry/Dashboard/TimeRangePicker.swift` |
| macOS 14+, universal | `project.yml` deployment target |
| `sentryctl`, `SentryMCP` | `project.yml` → `EXECUTABLE_NAME` |
| $14.99 / $19.99, 3 Macs | the Pro plan card, and pricing decisions |

If you add a theme or an alert rule, the number on the page is wrong the
moment you merge. It's a two-word edit; the cost is only in remembering.

## Screenshots

`assets/` is copied from the app repo. To refresh them:

```bash
cp ../MacStat/docs/screenshots/*.png assets/
```

`docs/screenshots/` in the app repo is the master set.

Keep the filenames identical and no HTML changes are needed. If the pixel
dimensions change, update the matching `width`/`height` attributes on the
`<img>` so the page doesn't reflow while they load.

Current set: `macos-dashboard.png` (hero, also the og:image),
`macos-menubar.png` (dropdown), `ios-dashboard.png`, `ios-alerts.png`,
`watch-overview.png`.

There is no capture of the desktop widgets yet — that section is text only
until there is one.

## A note on the name

"Sentry" is provisional pending a trademark decision, and it collides with
the well-known error-tracking company. In this page it appears only as
visible text, in `<title>`, and in the meta description — never in a URL, a
class name or an asset filename. A rename is a find-and-replace of the
visible strings plus the GitHub links.
