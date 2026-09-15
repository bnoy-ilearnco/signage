# iLearnCo Signage

A free, self-hosted **digital signage system** for schools. Repurpose an Amazon Fire TV
Stick into a display that boots straight into a full-screen board — announcements, photos,
events, a live clock, the Hebrew date, the weekly parsha, and zmanim — all editable from a
simple web page. **No server, no monthly cost.**

**Live board:** https://bnoy-ilearnco.github.io/signage/
**Editor:** https://bnoy-ilearnco.github.io/signage/admin.html

---

## How it works

Three pieces work together:

| Piece | What it is |
|---|---|
| **The display site** (this repo) | The page the TV shows. Reads `content.json` and renders the board. |
| **The editor** (`admin.html`) | A password-protected page where staff update the board. Writes `content.json` back to the repo via the GitHub API. |
| **The Fire TV app** (companion) | A custom Android kiosk app that boots into the site, never sleeps, and self-heals. |

```
admin.html  --writes-->  content.json  <--reads--  index.html
(staff edit)             (in this repo)            (the board)
                                                       |
                                                  shown by
                                                       v
                                               Fire TV kiosk app
```

One edit in the editor reaches every stick within about a minute.

---

## Use it yourself (fork & go)

1. Click **Fork** (top-right) to get your own copy.
2. In your fork, open **Settings → Pages** → **Source: Deploy from a branch**, **Branch: `main` / `/root`**, Save.
   Your board goes live at `https://<your-username>.github.io/<your-repo>/`.
3. Create a **fine-grained GitHub token** (see *Editing the board* below), open your `/admin.html`,
   and set your school name, colors, location, and content.

That's the whole website side. For the Fire TV stick, see *The Fire TV stick* below.

> You **can't edit this repo** — only fork it. To run your own, work in your fork.

---

## What's in this repo

```
index.html      The display board (this is the URL the sticks show)
admin.html      The password-protected editor
content.json    The live content: announcements, events, photos, schedule, ticker, settings
images/         Photos uploaded from the editor land here
```

---

## Editing the board

1. Create a token at **github.com/settings/personal-access-tokens** → *Fine-grained token* →
   **Only select repositories: your signage repo** → **Repository permissions → Contents: Read and write**.
2. Open **`/admin.html`**, enter your GitHub username, repo name, and the token, then **Connect**.
   The token is stored **only in that browser** — never committed or shared. It is the password
   that protects the board.
3. Edit announcements, events, photos, schedule, ticker, and settings, then **Save**. All signs
   update within ~60 seconds.

---

## Zmanim & Jewish calendar

- The **Hebrew date** is computed on the device (works offline).
- **Parsha and zmanim** come from the free [Hebcal](https://www.hebcal.com) API, cached on the
  device. Set your location in the editor with a **geonameid** (find yours at
  [geonames.org](https://www.geonames.org)). Zmanim can be toggled off.

---

## The Fire TV stick (companion app)

The display site works in any browser, but for a real signage stick you install the companion
kiosk app (a separate Android project) so the Fire TV boots into the board and stays on. Summary
of the setup — the full, illustrated walkthrough is in **Fire_TV_Signage_Setup_Guide.pdf**.

**Prep the stick:** Settings → My Fire TV → About → click the device name 7× to unlock Developer
options; enable **ADB debugging** and **Apps from Unknown Sources**; note the stick's IP.

**Build & install** (from the app project, on a Mac with Android Studio + JDK 21):

```
export JAVA_HOME="$(/usr/libexec/java_home -v 21)"
./gradlew assembleDebug
ADB=~/Library/Android/sdk/platform-tools/adb
$ADB connect <STICK_IP>:5555
$ADB install -r app/build/outputs/apk/debug/app-debug.apk
$ADB shell cmd package set-home-activity com.ilearnco.signage/.MainActivity
```

**Keep it on 24/7** (Fire OS sleep overrides the app otherwise):

```
$ADB shell settings put global stay_on_while_plugged_in 7
$ADB shell settings put secure screensaver_enabled 0
$ADB shell settings put system screen_off_timeout 2147483647
```

**Stop Fire OS auto-updates:** the on-device disable is blocked on current Fire OS — block
`amzdigitaldownloads.edgesuite.net` and `softwareupdates.amazon.com` at your router instead.

**Power:** run the stick from its **wall adapter**, not the TV's USB port, so it self-boots after
an outage.

---

## Resilience

- If `content.json` can't be fetched, the board shows a built-in default instead of going blank.
- The board re-checks `content.json` every 60s, so edits propagate without touching the devices.
- The kiosk app has a watchdog that recovers a blank or hung page automatically.

---

## Notes

- This repo is **public**, so everything in it is world-readable — keep nothing sensitive here.
  The editor token is **not** stored in the repo.
- Colors, school name, and logo are set in `content.json` (or the editor). The layout is
  responsive to any TV resolution.

Built with [Claude](https://claude.ai) · iLearnCo EdTech Solutions
