# Coherence

A paced breathing timer. Two tones — a rising D5 for inhale, a falling C4 for exhale —
at a default of 5.5 breaths per minute, with a 10-second settle before the first tone.
Sessions round to whole breaths and end on a completed exhale.

Static files only. No build step, no dependencies, no network calls except Google Fonts
(which are cached after first load, and fall back to system faces if unavailable).

## Running it

**Locally.** Service workers need an origin, so open it with a server rather than
double-clicking:

    python3 -m http.server 8000

Then visit http://localhost:8000. Opening `index.html` directly still works — you just
lose offline caching.

**GitHub Pages.** A workflow is included at `.github/workflows/deploy.yml`. Push the repo,
then in the repo go to Settings → Pages → Build and deployment → Source and choose
**GitHub Actions** (not "Deploy from a branch"). That's the only setting to change.

    git init -b main
    git add -A
    git commit -m "Coherence"
    git remote add origin git@github.com:USERNAME/coherence.git
    git push -u origin main

The workflow runs on every push to `main` and publishes the repo root as-is. There is no
build step — `path: .` uploads the directory verbatim.

Served at `https://USERNAME.github.io/coherence/`. Every path in the app is relative, so
the subdirectory works with no changes. Name the repo `USERNAME.github.io` instead if you
want it at the domain root.

`.nojekyll` disables Jekyll processing, which would otherwise ignore any file or folder
starting with an underscore and slow every deploy down for no reason.

**Cloudflare Pages.**

    npx wrangler pages deploy . --project-name coherence

Or drag the folder into the Cloudflare dashboard under Workers & Pages → Create → Pages →
Upload assets.

**Anywhere else.** Netlify, S3 + CloudFront, an nginx directory. It needs HTTPS for the
service worker and the screen wake lock; localhost is exempt.

## Installing to a phone

- **iOS:** open in Safari, Share → Add to Home Screen. Launches full screen, no browser
  chrome. Audio starts on the Begin tap, which satisfies Safari's gesture requirement.
- **Android:** Chrome offers an install prompt, or use ⋮ → Add to Home screen.

## Files

    index.html              the whole app — markup, styles, audio engine, scheduler
    manifest.webmanifest    name, icons, standalone display
    sw.js                   offline cache of the shell plus runtime font caching
    icon-*.png              app icons, including a maskable variant
    .github/workflows/      GitHub Pages deploy workflow
    .nojekyll               tells Pages to serve the files untouched

## Changing things

Pace and duration options are the `RATES` and `DURATIONS` arrays near the top of the
script. Tone character lives in `inhaleTone` / `exhaleTone` — `f` is the fundamental,
`glide` the pitch bend over the note, `dur` its length. The settle period is `LEADIN`.

Tones are scheduled against the Web Audio clock with a 12-second lookahead, so timing
doesn't drift over a long session. Bump `CACHE` in `sw.js` when you change any file, or
installed copies keep serving the old version.
