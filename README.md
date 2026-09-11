# GrowTheory — Landing Page

Single-page marketing site for **GrowTheory**, a digital marketing agency in Cleveland, Ohio.

Built as a **static site with zero dependencies** — no npm packages, no build step, no server, no database. That is a deliberate security decision, explained below.

---

## Run it locally

No install needed. From this folder:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

> Open `index.html` directly via `file://` and the page still renders, but the
> Content-Security-Policy behaves differently and fonts may not load. Always
> test over `http://`.

---

## Project structure

```
index.html              all markup and page copy
assets/css/styles.css   all styling
assets/js/main.js       sticky header, mobile nav, scroll reveal
assets/img/favicon.svg  logo mark
_headers                security headers for Netlify / Cloudflare Pages
```

---

## Things you'll want to edit

| What | Where |
|---|---|
| Contact email | `index.html` — the two `mailto:` links in `<section class="contact">` |
| Services | `index.html`, `<section class="services">` |
| Story / positioning copy | `index.html`, `<section class="manifesto">` |
| Colours | `assets/css/styles.css`, the `:root` block at the top |

There is deliberately **no contact form** and **no performance statistics** on
this page. Both were removed on purpose:

- **No stats.** The agency is new, so there are no results to cite yet. Invented
  numbers are an FTC problem and are trivially called out. The manifesto band
  ("Every great brand starts somewhere") turns being new into the story instead.
  Add real figures once you have them.
- **No form.** Contact runs through `mailto:` links. A form with no backend is
  pure liability — spam surface with nothing to show for it. See below when you
  want to add one properly.

---

## Adding a contact form later

When you're ready, use a hosted form service (Formspree, Web3Forms, Netlify
Forms) rather than writing your own backend — they handle spam filtering and
delivery, and you keep zero servers to patch.

Two rules when you do:

1. **Add the service's origin to `connect-src`** in *both* `index.html` and
   `_headers`, e.g. `connect-src 'self' https://formspree.io;`. If you skip
   this the browser silently blocks the request — that's the CSP working, not a
   bug.
2. **Write user input to the page with `textContent`, never `innerHTML`.** That
   single habit is what makes cross-site scripting impossible. Also strip `\r`
   and `\n` from anything that lands in an email subject line, or someone can
   inject extra email headers through your name field.

## Security

You said you don't know security, so here is what's already handled and what is
still on you.

### Handled in the code

**No attack surface by design.** There is no server, no database, no login, no
user accounts and no third-party JavaScript. The overwhelming majority of
website hacks target exactly those things. A static site can't have SQL
injection because there is no SQL.

**No supply chain risk.** Zero npm packages. Nothing in this repo can be
compromised by someone hijacking a dependency you've never heard of — which is
how a large share of modern web compromises actually happen.

**Content-Security-Policy.** Set in `_headers` (real header) and `index.html`
(fallback). Even if someone found a way to inject a `<script>` tag into the
page, the browser would refuse to run it, because scripts are only permitted
from this site's own origin. `object-src 'none'` and `base-uri 'self'` close
two common bypasses.

**Clickjacking blocked.** `frame-ancestors 'none'` + `X-Frame-Options: DENY`
stop anyone embedding your site invisibly inside theirs to trick your visitors
into clicking things.

**No user input anywhere.** The page collects nothing — no form, no search, no
comments. Cross-site scripting needs attacker-controlled text to reach the
page, and there is no path for any. The JavaScript makes no network calls and
reads nothing from the URL.

**Safe external links.** The Instagram link uses `rel="noopener noreferrer"`,
which stops the opened page from being able to manipulate yours.

**HSTS.** `Strict-Transport-Security` forces browsers to use HTTPS, preventing
downgrade attacks on public Wi-Fi.

**MIME sniffing off.** `X-Content-Type-Options: nosniff` stops browsers from
guessing a file is a script when it isn't.

**Permissions-Policy.** Camera, microphone, geolocation and payment APIs are
explicitly denied. The page has no use for them.

**Secrets kept out of git.** `.gitignore` covers `.env`, `*.key` and `*.pem`.

### Still on you

1. **Serve over HTTPS.** Netlify, Cloudflare Pages, Vercel and GitHub Pages all
   give you a free certificate automatically. Never deploy this over plain
   HTTP — the HSTS header assumes HTTPS.
2. **Never paste an API key into `assets/js/main.js`.** Anything in that file is
   public to every visitor. Client-side code has no secrets. If a service gives
   you a "secret key", it does not belong in this repo.
3. **Turn on 2FA** for your domain registrar, your host and your GitHub account.
   In practice, account takeover is a far likelier way to lose this site than
   anyone attacking the code.
4. **Verify the headers after deploying.** Paste your URL into
   <https://securityheaders.com>. You should score an A or A+. If you don't,
   your host isn't reading `_headers` — check its docs for the right filename.

### Headers on other hosts

`_headers` is for Netlify and Cloudflare Pages. Elsewhere:

- **Vercel** — add a `headers` array to `vercel.json`
- **nginx** — `add_header` directives in your `server` block
- **Apache** — `Header set` directives in `.htaccess`
- **GitHub Pages** — cannot set custom headers at all; put Cloudflare in front
  of it, or use one of the hosts above

Copy the same header names and values from `_headers`.

---

## Accessibility & performance notes

- Skip link, visible focus rings, `aria-expanded` on the mobile menu,
  semantic landmarks throughout.
- Honours `prefers-reduced-motion` — all animation is disabled for visitors who
  ask for that.
- Responsive from 320px up; no horizontal scroll.
- Google Fonts is the only external request. To drop it entirely, download the
  two font families into `assets/fonts/`, `@font-face` them locally, and remove
  `fonts.googleapis.com` / `fonts.gstatic.com` from both CSP declarations.
