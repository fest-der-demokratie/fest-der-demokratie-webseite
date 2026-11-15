# Fest der Demokratie — Webseite

Website of the **Fest der Demokratie**, a free open-air festival with live music, political education and workshops in the Bürgerpark Pfaffenhofen a.d. Ilm, organised by the Fest der Demokratie e.V.

**Next festival: Saturday, 3 July 2027 — Bürgerpark Pfaffenhofen**

## Pages

| File | Content |
| --- | --- |
| `index.html` | Home: hero with Save the Date, highlights, stats, partners & sponsors |
| `archiv-2026.html` | Archive of the 2026 festival: line-up and program (`/programm` redirects here) |
| `ueber-uns.html` | Infos: FAQ, awareness, youth protection, sustainability, contact |
| `spenden.html` | Donations and sponsoring |
| `impressum.html`, `datenschutz.html` | Legal notice and privacy policy |
| `404.html` | Custom error page |

Plain static HTML/CSS/JS, no build step:

```
css/style.css     all styles (design tokens at the top)
js/main.js        navigation, animations, counters, carousel, donation form
assets/           images, logos (assets/logos/partner/ for partner & sponsor logos)
nginx.conf        server config used in the Docker image
```

## Run locally

```sh
python3 -m http.server 8080
```

Then open http://127.0.0.1:8080/. Extensionless URLs (`/impressum`) and the custom 404 page only work in the nginx container:

```sh
docker build -t fest-der-demokratie-webseite .
docker run --rm -p 8080:80 fest-der-demokratie-webseite
```

## Deployment

1. Every push to `main` runs `.github/workflows/docker.yml`, which builds the nginx image and pushes it to
   `ghcr.io/fest-der-demokratie/fest-der-demokratie-webseite` with the tags `latest` and an increasing version (`v1`, `v2`, …; a matching git tag is created).
2. The server runs `docker-compose.yml` as a Portainer stack. It pins a fixed image version (e.g. `:24`), never `:latest`. The container joins the external Docker network `nginx-proxy-manager` and publishes no ports.
3. In Nginx Proxy Manager, the proxy host forwards to `http://fest-der-demokratie-webseite:80`.
4. To update, set the image tag in `docker-compose.yml` to the new version (the newest `vN` git tag) and redeploy the stack in Portainer.

If the GHCR package is private, add `ghcr.io` as a registry in Portainer with a token that has `read:packages`.

## Working on the site

- Work happens directly on `main`; there are no feature branches.
- Partner and sponsor logos: prefer PNG or SVG with a transparent background, cropped to the logo without extra padding.
- When a new page is added, also add it to `sitemap.xml`; the Dockerfile copies all `*.html` files automatically.
