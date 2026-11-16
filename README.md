# Public pages: privacy policy and support

The App Store and Google Play both need a **privacy policy URL** and a **support
URL**. These are the two pages they point at. They are hosted on GitHub Pages
from a **separate repo** (the same way the Vessel app uses `vessel-privacy`),
uploaded by hand. Nothing in this folder is built, bundled or served by the app.

| File | Publishes as | Goes in |
|---|---|---|
| `index.html` | `https://skreweverything.github.io/<repo>/` | App Store Connect → App Information → **Privacy Policy URL**; Play Console → App content → Privacy policy |
| `dark-grid-support.html` | `https://skreweverything.github.io/<repo>/dark-grid-support.html` | App Store Connect → version page → **Support URL**; Play Console → Store settings |

`index.html` is the policy on purpose: the bare Pages URL *is* the policy URL.
The two pages link to each other with relative links, so they must sit side by
side in the same repo folder.

## Publishing (first time)

1. On GitHub create a **public** repo. Suggested name `dark-grid-privacy`; the
   repo name becomes the first path segment of both URLs.
2. **Add file → Upload files**, drag in `index.html` and `dark-grid-support.html`
   (just those two; this README stays here), commit to `main`.
3. **Settings → Pages → Build and deployment**: Source *Deploy from a branch*,
   branch `main`, folder `/ (root)`. Save.
4. After a minute open both URLs and click through (header links, the footer
   links, a couple of FAQs). Then paste them into App Store Connect (see
   `SUBMISSION.md`, G1).

To update later: upload the changed file over the old one (same name) and Pages
redeploys in about a minute. The URLs never change.

## The policy has to move when the app does

`index.html` is a statement of what the app *actually does*. Reviewers compare it
with the App Store privacy label, and users believe it. Edit it **before the
build ships**, bump "Last updated" in the header chips, and redo the App Privacy
answers (`SUBMISSION.md`, G2) whenever any of these change:

- [ ] A new SDK, or a new kind of data sent to Firebase (new event families, user
      properties, anything that identifies a person: see `FIREBASE.md`).
- [ ] Ads (AdMob is **not** in the build, whatever the Monetization section of
      CLAUDE.md still says). Ads also mean App Tracking Transparency.
- [ ] Accounts, Game Center, or the global leaderboard coming back on (File 05, 08).
- [ ] In-app purchases (the page says there are none).
- [ ] Analytics retention changed in the Google Analytics console (File 06).
- [ ] The cloud save provider changed (File 02, 04).
- [ ] A new permission: camera, microphone, location, contacts.

The support page's FAQ is also a description of the game. When a rule in
CLAUDE.md changes (range ordering of the guns, what a day's double pays, the
car's fuse, how the shop refunds a gun…) check the matching answer.

## Open items to settle before publishing

Written down because each is a choice that only the owner can make:

1. **There is no contact email on either page**, exactly like the Vessel pages:
   both point people at the store's developer contact link or a review. If you
   want an email, add it in the two `#contact` sections (policy File 11, support
   Section 06). Apple wants the support URL to offer *some* way to reach you.
2. **Android advertising ID.** `@react-native-firebase/analytics` collects it by
   default on Android (`google_analytics_adid_collection_enabled`, read in the
   library's `build.gradle`). The policy says so under "About advertising
   identifiers" and Play's Data safety form must agree. To stop it, add a
   `"react-native": { "google_analytics_adid_collection_enabled": false }` block
   to `firebase.json` (there is none today), prebuild, and delete the Android
   sentence.
3. **Retention is stated as "at most 14 months".** That is GA4's longest setting,
   so it is true whatever the console says, but if you set it shorter
   (Admin → Data collection and modification → Data retention) say so.
4. **There is no in-app analytics opt-out.** The policy says deleting the app
   stops all collection, which is true. If you target the EU/UK seriously,
   consider a consent prompt or a Settings toggle
   (`setAnalyticsCollectionEnabled`) and update File 08.
5. **No in-app "reset progress".** The support FAQ says so (only the DEBUG
   screen can wipe). If one is added, rewrite that answer.
6. **Android cloud save is off until `extra.playGamesProjectId` is set**
   (CLAUDE.md, Cloud save). The copy says "where your device offers it".

## How the files are built

Each page is **one self-contained file (~115 KB)**, deliberately:

- **No third-party requests, no cookies, no scripts but a ticking clock and a
  FAQ deep-link opener.** The policy tells visitors exactly this about itself,
  so do not add a Google Fonts link, a CDN script or any analytics. Doing so
  makes the privacy policy false.
- **Fonts** are the game's own (`assets/fonts/`, SIL OFL), subset to Latin and
  embedded as woff2 base64 in the first `<style>` block (long lines: don't
  hand-edit). To regenerate, with `pip install fonttools brotli`:
  ```
  python3 -m fontTools.subset assets/fonts/<Face>.ttf \
    --unicodes="U+0020-007E,U+00A0,U+00B7,U+00D7,U+2013,U+2014,U+2018,U+2019,U+201C,U+201D,U+2022,U+2026" \
    --flavor=woff2 --layout-features=kern --no-hinting --desubroutinize --output-file=<Face>.woff2
  ```
  then `base64 -i <Face>.woff2` into the matching `@font-face`. The subset has
  **no arrows or symbols**; draw those with CSS or SVG.
- **The icon** is `store-listing/icon.png` at 192 px as a base64 JPEG
  (`magick store-listing/icon.png -resize 192x192 -strip -quality 84 icon.jpg`),
  plus a 64 px PNG favicon.
- **The look** is the game's: tokens in `:root` are copied from
  `src/ui/theme.ts` (concrete blacks, bone type, rust for structure, sodium
  amber only for live readouts such as the status LEDs, blood for warnings).
  The header is a CCTV frame (REC, a live timestamp, corner brackets) over the
  town plan, and the plaza diamond is the watt mark. The page's own pieces are
  `.plate`, `.check`, `.tablewrap`, `details` (FAQ), `ol.steps`, `.readout`.
- **The CSS between `BEGIN SHARED STYLES` and `END SHARED STYLES` is identical
  in both files.** Edit one, copy it to the other, and check:
  ```
  for f in index.html dark-grid-support.html; do sed -n '/BEGIN SHARED/,/END SHARED/p' $f | md5; done
  ```
  The two sums must match.

**Preview** by opening the files in a browser. Phone width matters (the tables
stack into labelled cards under 720 px). Headless Chrome enforces a minimum
window width, so to test a 390 px layout put the page in a 390 px `<iframe>`
rather than trusting `--window-size=390`.
