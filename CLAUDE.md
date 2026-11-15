# CLAUDE.md

Website of the Fest der Demokratie (Pfaffenhofen a.d. Ilm). See `README.md` for pages, local run and deployment; this file holds what isn't obvious from the code.

## Repo & workflow

- Commit directly on `main`. No feature branches, no PRs.
- `origin` (`fest-der-demokratie/fest-der-demokratie-webseite`) is a GitHub fork of `Open-Project-e-V/fest-der-demokratie`, but it is independent and will never be merged back. Ignore upstream: never open PRs there or fetch from it.
- The organiser is **Fest der Demokratie e.V.** (no longer Open Project e.V.; don't reintroduce `op-paf.de` links or the Open Project name/logo). Contact address: `fest.der.demokratie.paf@gmail.com`.
- Commit messages: short, lowercase, imperative English.

## Site structure

- Static HTML/CSS/JS, no build step. Every page duplicates the nav and footer, so a nav/footer change must be applied to all `*.html` files (incl. `404.html`).
- `index.html`: hero carries the Save the Date for the next festival (Sa, 03.07.2027, Bürgerpark Pfaffenhofen). The date is the most important information on the page; keep it the dominant hero element.
- `archiv-2026.html`: line-up and program of the 2026 festival (04.07.2026). It replaced `programm.html`; nginx 301-redirects `/programm(.html)` there.
- Design tokens are at the top of `css/style.css` (`--ruby` #F51D00 is the CI red, `--grape`, `--sunshine`, `--sky`, `--fuchsia`). Use them instead of hard-coded colors.
- The display font `Komsomol` is referenced in `css/style.css` but `assets/fonts/` is missing, so headings fall back to Arial Black.

## Images

- Hero photo: `assets/hero-bg.jpg`, ~1920px wide, compressed, EXIF/GPS stripped (`sips` + `jpegtran -copy none`).
- Partner/sponsor logos (`assets/logos/partner/`): transparent PNG (or SVG), cropped to the logo without padding, max ~600px. White backgrounds are converted with a color-to-alpha (white → alpha) step, not a hard threshold. No Python imaging libs are installed globally; use a throwaway venv with `pillow` + `numpy`.
- Known: `lichtfabrik.png` is a white logo (invisible on the white page); `lovecareskin.png` and `dsee.png` have large built-in padding.

## Local preview

- `python3 -m http.server 8080` from the repo root. It does not honor `nginx.conf` (no extensionless URLs, no custom 404); use the Docker image for that.
- Screenshots for checking layout: headless Chrome (`"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --screenshot=… --window-size=1440,900 URL`). Headless Chrome won't go narrower than ~500px, so mobile screenshots are unreliable.

## Build & deploy

- `.github/workflows/docker.yml` builds on every push to `main` and pushes `ghcr.io/fest-der-demokratie/fest-der-demokratie-webseite:<N>` and `:latest`, then pushes git tag `v<N>` (N = highest existing `v*` tag + 1).
- Runs on `main` are serialized (`concurrency`), and the git tag is pushed before the image, so a version number can't be used twice.
- Production is Portainer stack `fest-der-demokratie-webseite` (git-based, reads `docker-compose.yml` from `main`, polls every 5 min, re-pulls images). It pins a numbered image tag, never `:latest`. The container joins the external network `nginx-proxy-manager` and publishes no ports.
- After each build the workflow commits the new tag into `docker-compose.yml` (`deploy vN [skip ci]`, by github-actions[bot]) and calls the Portainer webhook. So `main` gets a bot commit after every push: pull before working.
- Portainer stack webhook (verified against Portainer 2.45.3 source, `api/http/handler/stacks/webhook_invoke.go`): **`POST` only, no body, no query parameters** (env vars can't be passed). It answers `204` immediately and redeploys asynchronously; errors only show in Portainer's logs. It redeploys **only if the git commit changed** (`ForceUpdate` is off), so calling it without a new commit does nothing. `409` = deployment already running, or the stack's author user is missing. The URL is the repo secret `PORTAINER_WEBHOOK_URL`; never print or commit it.
- Base image is pinned to a minor version (`nginx:1.31-alpine`); Dependabot (`.github/dependabot.yml`) bumps it and the GitHub Actions weekly.
- The local `gh` token lacks `read:packages`, so GHCR package visibility can't be checked from here.

## Open content issues

- `datenschutz.html` contact section still names the previous responsible person and address.
- `impressum.html` "Registereintrag: Amtsgericht München" belonged to Open Project e.V.; the new association's register entry is still needed.
- `ueber-uns.html` and `spenden.html` still describe the 2026 festival as upcoming.
