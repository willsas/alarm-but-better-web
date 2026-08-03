# alarmbutbetter.com

Marketing and legal site for the **Alarm but better** iPhone app. Plain static
HTML and CSS — no build step, no dependencies, no trackers.

```
index.html      Landing page          → App Store "Marketing URL"
assets/screens/ App screenshots (real captures, 620px wide)
support.html    Help + FAQ + contact  → App Store "Support URL"  (required)
privacy.html    Privacy Policy        → App Store "Privacy Policy URL" (required)
terms.html      Terms of Use / EULA   → linked from the in-app paywall (required)
404.html        Not-found page
styles.css      All styling
favicon.svg     Icon
CNAME           Custom domain for GitHub Pages
.nojekyll       Tells GitHub Pages to serve files as-is
robots.txt / sitemap.xml
```

## Preview locally

```bash
python3 -m http.server 4321
```

Then open <http://localhost:4321>.

---

# Deploying: Cloudflare domain → GitHub Pages

Do these in order. Steps 1–3 take minutes; step 5 (DNS) can take up to an hour
to propagate, and step 6 (HTTPS certificate) can take another hour.

## 1. Buy the domain at Cloudflare

1. Go to <https://dash.cloudflare.com> → **Domain Registration → Register Domain**.
2. Search `alarmbutbetter.com` and buy it.

Cloudflare Registrar sells at cost and includes WHOIS privacy. Buying it here
means the domain and DNS are already in the same place — no nameserver change
needed.

## 2. Put this folder on GitHub

```bash
cd alarm-but-better-web
git init -b main
git add .
git commit -m "Add marketing and legal site"
gh repo create alarm-but-better-web --public --source=. --push
```

No `gh`? Create an empty repo on github.com, then:

```bash
git remote add origin https://github.com/<you>/alarm-but-better-web.git
git push -u origin main
```

> Keep the repo **public** — GitHub Pages on free accounts only serves public
> repos.

## 3. Turn on GitHub Pages

Repo → **Settings → Pages**:

- **Source:** Deploy from a branch
- **Branch:** `main`, folder `/ (root)` → **Save**

Wait a minute, then check `https://<you>.github.io/alarm-but-better-web/`. The
CSS may look broken there — that's expected, because links are absolute (`/styles.css`)
and this URL serves from a subfolder. It resolves once the custom domain is live.

## 4. Add the custom domain in GitHub

Repo → **Settings → Pages → Custom domain** → enter `alarmbutbetter.com` → **Save**.

GitHub will report a DNS check failure. That's expected until step 5.

The `CNAME` file in this repo already contains the domain, so GitHub may pick it
up automatically.

## 5. Point DNS at GitHub, in Cloudflare

Cloudflare dashboard → your domain → **DNS → Records**. Add **five** records.

Four `A` records for the apex (all with name `@`):

| Type | Name | IPv4 address    | Proxy status         |
|------|------|-----------------|----------------------|
| A    | @    | 185.199.108.153 | **DNS only** (grey)  |
| A    | @    | 185.199.109.153 | **DNS only** (grey)  |
| A    | @    | 185.199.110.153 | **DNS only** (grey)  |
| A    | @    | 185.199.111.153 | **DNS only** (grey)  |

And one `CNAME` for `www`:

| Type  | Name | Target                | Proxy status        |
|-------|------|-----------------------|---------------------|
| CNAME | www  | `<you>.github.io`     | **DNS only** (grey) |

> **Two things people get wrong here.**
>
> 1. **Set the proxy to "DNS only" (grey cloud), not proxied (orange).** With the
>    orange cloud on, GitHub cannot complete the HTTP validation it needs to
>    issue your HTTPS certificate, and "Enforce HTTPS" stays greyed out forever.
>    You can switch the proxy back on later — but only after step 6 succeeds, and
>    if you do, set Cloudflare **SSL/TLS mode to "Full (strict)"**. Anything else
>    causes a redirect loop.
> 2. **Verify the four IPs against
>    <https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site>**
>    before relying on them. They've been stable for years, but GitHub can change
>    them and these are copied from memory.

## 6. Enable HTTPS

Back in **Settings → Pages**, wait for the DNS check to go green (usually
minutes, up to an hour), then tick **Enforce HTTPS**.

If it's greyed out, GitHub hasn't issued the certificate yet. Wait, then remove
and re-add the custom domain to retry. The most common cause is the Cloudflare
proxy being left on — see above.

## 7. Verify

- <https://alarmbutbetter.com> loads over HTTPS with a valid padlock
- <https://www.alarmbutbetter.com> redirects to the apex
- `/privacy.html`, `/terms.html`, `/support.html` all load
- A made-up path shows the 404 page

## 8. Set up email

The site and App Store Connect both reference `hello@alarmbutbetter.com`, and
**App Review will email that address**, so it must work.

Cloudflare **Email Routing** (free) forwards it to your personal inbox:

1. Cloudflare dashboard → your domain → **Email → Email Routing**.
2. Enable it and accept the MX records it offers to add.
3. Add a custom address `hello@alarmbutbetter.com` → forward to your real inbox.
4. Verify the destination address from the confirmation email.

Prefer a different address? Change it in `privacy.html`, `terms.html` and
`support.html` — it appears in all three.

---

## Where each URL goes in App Store Connect

| Field | URL |
|---|---|
| Privacy Policy URL (App Information) | `https://alarmbutbetter.com/privacy.html` |
| Support URL (version page) | `https://alarmbutbetter.com/support.html` |
| Marketing URL (optional) | `https://alarmbutbetter.com` |

The iOS app's paywall already links to the terms and privacy pages via
`LegalLinks` in `PaywallView.swift`.

## Two placeholders to replace at launch

1. **App Store link.** The badge in `index.html` links to `#`. Swap it for your
   product URL once the app is live — search for `TODO` in that file.
2. **The badge artwork.** The badge is a hand-built approximation so the page
   looks right today. Apple requires the *official* badge asset, with its
   specified clear space and minimum size. Download it from
   <https://developer.apple.com/app-store/marketing/guidelines/> and drop it in
   before you promote the site.

To refresh the screenshots, capture from the Simulator and resize:

```bash
sips -Z 620 raw.png --out assets/screens/alarms.png
```

## Updating the site

Edit, commit, push. GitHub Pages redeploys in under a minute.

```bash
git add . && git commit -m "Update privacy policy" && git push
```

---

## A note on the legal text

The privacy policy is accurate to what the app actually does — it was written
after auditing the source, and the "no data collected, no networking code"
claim was verified by grep. Keep it that way: **if you ever add analytics, a
crash reporter, or any network call, this page has to change first.**

The terms are a solid, conventional starting point, but they are not legal
advice. If you operate as a registered business, in a regulated market, or want
certainty, have a lawyer review them.
