# iLearnCo Signage — display site

The web page your Fire TV sticks show, plus a password-protected editor. Runs
entirely on **GitHub Pages** (free, HTTPS) with **no server**: the signs read
`content.json`; the editor writes `content.json` (and uploads photos) straight to
your repo via the GitHub API. One edit updates every sign within a minute.

```
index.html        ← the sign display (this is the URL the sticks show)
admin.html        ← the editor (staff log in with a GitHub token)
content.json      ← the live content (announcements, events, photos, schedule, ticker)
images/           ← photos uploaded from the editor land here
```

## One-time setup

1. **Create a repo** (e.g. `signage`) on your GitHub account and upload these files
   (drag the folder contents into the repo, or `git push`).
2. **Enable Pages:** repo **Settings ▸ Pages ▸ Build and deployment ▸ Source = Deploy
   from a branch**, branch `main`, folder `/ (root)`. Save.
3. Your site is now live at:
   - Display: `https://<username>.github.io/signage/`
   - Editor:  `https://<username>.github.io/signage/admin.html`
4. **Point the sticks** at the display URL — set it as `DEFAULT_URL` in the Fire TV
   app's `Config.kt`, or push it through the remote-config JSON (`"url": "https://<username>.github.io/signage/"`).

## Editing content

Open `admin.html`, then:

1. Enter your GitHub **username**, **repo name**, and a **fine-grained access token**
   with *Contents: Read and write* on just that repo
   (create at github.com/settings/personal-access-tokens). The token is stored only
   in that browser — it is never committed or shared.
2. Edit announcements, events, photos, schedule, ticker, and school settings.
3. **Save to all signs.** Changes appear on every stick within ~60 seconds.

> The token *is* the password: only someone with a valid write token can change the
> board. Keep the token to the one repo, and revoke it in GitHub if a device is lost.

## Zmanim & Jewish calendar

- **Hebrew date** is computed on the device (works offline).
- **Parsha and zmanim** come from the free [Hebcal](https://www.hebcal.com) API,
  cached on the device and refreshed a few times a day. Set your location in the
  editor via a **geonameid** (find yours at [geonames.org](https://www.geonames.org) —
  search your city and read the ID from the URL). Turn zmanim off with the checkbox
  if you don't want them.
- If Hebcal is briefly unreachable, the sign keeps the last cached values and hides
  the block rather than showing errors.

## Resilience

- If `content.json` can't be fetched, the sign shows a built-in default instead of a
  blank screen.
- The sign re-checks `content.json` every 60s, so edits propagate without touching
  the devices.
- Pairs with the Fire TV app's watchdog: a blank or hung page is auto-recovered.

## Customizing the look

Colors come from `settings.theme` in `content.json` (or edit the CSS variables at the
top of `index.html`). The layout is responsive to any TV resolution. Replace the
school name and logo in the editor.
