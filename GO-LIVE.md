# futkar.com go-live

Same setup as namerohit.com: Google Docs → aem.live → Cloudflare Worker → futkar.com.

## 1. Code Sync (Rohit, 1 min)
github.com/apps/aem-code-sync → Configure → account `futkarj` → add repo **futkar** (skip if it's set to All repositories).

## 2. Register the site (Rohit, 3 min)
tools.aem.live site admin, signed in as the Google account that owns the Drive folder:
- Org `futkarj`, add site **`futkar`**
- Code: `futkarj/futkar`, branch `main`
- Content: Google Drive, `https://drive.google.com/drive/folders/1HMMHhUhk7hMVIfZLKUImTFOzgiPKZIy4`

Done already: folder shared with helix@adobe.com (Editor); `index`, `nav` and `footer` Docs exist.

## 3. Publish (Rohit, 2 min)
Open each of `index`, `nav`, `footer` in Docs → Sidekick → Preview → Publish.
Check https://main--futkar--futkarj.aem.live/

## 4. Move DNS to Cloudflare without breaking email (Rohit + Claude)
futkar.com DNS is still on Google Cloud DNS nameservers (ns-cloud-b1…b4.googledomains.com), managed via Squarespace.
Email for rohit@futkar.com depends on these records. Cloudflare's import should find them; **check every one is
present and set to DNS only (grey cloud) before changing nameservers**:

| Type | Name | Value |
|---|---|---|
| MX 1 | @ | aspmx.l.google.com |
| MX 5 | @ | alt1.aspmx.l.google.com |
| MX 5 | @ | alt2.aspmx.l.google.com |
| MX 10 | @ | alt3.aspmx.l.google.com |
| MX 10 | @ | alt4.aspmx.l.google.com |
| TXT | @ | v=spf1 include:_spf.google.com ~all |
| TXT | google._domainkey | v=DKIM1; k=rsa; p=… (copy the full value from Squarespace) |

Plus any other TXT records you see in Squarespace (e.g. google-site-verification). There's no DMARC record today;
optionally add TXT `_dmarc` = `v=DMARC1; p=none; rua=mailto:rohit@futkar.com`.

Then:
1. Cloudflare → Add site `futkar.com` (Free) → review imported records against the table.
2. Add AAAA `@` → `100::` proxied, and AAAA `www` → `100::` proxied (the Worker answers them).
3. SSL/TLS: Full (strict); Always Use HTTPS on.
4. Squarespace → futkar.com → DNS → switch nameservers to the two Cloudflare gives you.
5. Send Claude the zone ID; Claude fills it into `cdn-worker/wrangler.toml` and runs `npx wrangler deploy`.
6. Test: send yourself an email from another account, and open https://futkar.com.
