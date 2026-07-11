# Green Oaks Athletic Club - Site & Network Restore Guide

*Pre-wipe capture of the site, its serving chain, and how to bring it back. Prepared 2026-07-08.*

---

> [!tip] Resolved 2026-07-08 — the site is safely on GitHub
> The live 463 KB `index.html` had **never been committed**; it existed only on this Mac's disk. It is now committed and pushed. `origin/main` is at **`994b5ec`**, and `index.html` on GitHub is 463,365 bytes — byte-identical to what Caddy serves.
>
> **The machine is safe to wipe.**

For the record, what was rescued from `~/Claude Projects - MacStew/OGS Projects/Green Oaks Athletic Club/Green Oaks Athletic Club Website/` (note: **not** the `~/WebProjects` repo):

- the live 463 KB `index.html` (the "Join the interest list" page)
- `index.session1-backup.html`
- `images/G.O.A.C. original flyer.png`
- `images/original flyer issues annotated.jpg`
- `images/original flyer issues.jpg`
- the deletion of the superseded `images/GOAC-flyer.html`

That checkout was a commit behind and had **diverged** from `origin/main` — both sides had edited `index.html`. Because the live page (2026-07-06) postdates origin's archive commit `e3a4a5b` (2026-05-07), the merge kept the live page and took `index.archive.html` from origin. History now reads `e3a4a5b → 6901d7c → 994b5ec (merge)`.

> [!note] Two side effects of that push
> **GitHub Pages content changed.** `ogs-data-manager.github.io/green-oaks-athletic-club/` now serves the interest-list page, not the 2026 Session 1 archive page. This was intended.
>
> **`index.archive.html` and `index.session1-backup.html` are byte-identical duplicates.** Harmless, but worth collapsing to one someday.

The nightly backup (`~/bin/macstudio-backup.sh`, 2:00 AM, rsync of `$HOME` → `/Volumes/MAA Thunderbay Dropbox/MacStudio-Backup`) also covers this folder, but only if that volume was mounted at 2 AM. GitHub is now the source of truth.

---

## What the site is

A one-page marketing site for the **Green Oaks Athletic Club** — an after-school multi-sport program for grades K–5 at Oak Grove School District 68.

**Content:** hero ("Play, learn, and have a blast"), a "new sport every week" pitch, age-right grouping, coach bios (Coach Izenstark, Coach Callahan), a three-step signup flow, and an FAQ.

| Call to action | Points to |
|---|---|
| Interest list | Google Form `1FAIpQLSfBR3N5cbLwnxUvVKYba4ofbSHJWWrLhMGEutZh_xMuMURwWg` |
| Contact (K–2) | `izenstark@ogschool.org` |
| Contact (3–5) | `callahan@ogschool.org` |

Two QR code PNGs (registration, Zelle payment) are inlined as base64 data URIs, which is why the file is 463 KB. It is a single static HTML file — no build step, no framework, no server-side code, no database.

---

## The two-copy problem

> [!important] Two checkouts of one repo on one machine
> This is the single most confusing thing about the setup, and it's how the live page sat uncommitted for two months.

| | Cloudflare copy (live public site) | WebProjects copy |
|---|---|---|
| Path | `~/Claude Projects - MacStew/…/Green Oaks Athletic Club Website/` | `~/WebProjects/green-oaks-athletic-club/` |
| Serves | `greenoaksathleticclub.com` | *(nothing locally)* |
| Git HEAD | `994b5ec` — **in sync with `origin/main`** | `e3a4a5b` — now behind, needs a `git pull` |
| Working tree | clean (only untracked `_site/`) | clean |
| `index.html` | 463 KB — "Join the interest list" | 25 KB until pulled |

Both point at the same remote: `https://github.com/OGS-Data-Manager/green-oaks-athletic-club.git` (public repo, created 2026-03-23, default branch `main`).

As of the 2026-07-08 push, both the custom domain and GitHub Pages publish the same interest-list page. **After the wipe, restore only one checkout** (the Cloudflare-served path below) and this whole problem goes away.

---

## Network setup

### Serving chain

```
Browser
  → greenoaksathleticclub.com  (Cloudflare DNS, zone in EZPeasyPS account)
  → Cloudflare Tunnel "macstew-ezpeasyps"  (id fadf3ea2…)
  → cloudflared on this Mac (root, system LaunchDaemon)
  → http://localhost:8095
  → Caddy file-server (launchd job com.goac.web)
  → …/Green Oaks Athletic Club Website/_site/
  → index.html  (symlink → ../index.html)
```

### The pieces

**Cloudflare zone** — `greenoaksathleticclub.com`, active, in the **EZPeasyPS** account (`a100a2a86cf0abbf92b3c99ed8fd6a67`). Both apex and `www` route into the tunnel. All server-side; **survives the wipe untouched.**

Tunnel ingress (already in Cloudflare, nothing to restore):

```
greenoaksathleticclub.com      -> http://localhost:8095
www.greenoaksathleticclub.com  -> http://localhost:8095
```

> [!note] No Access policy on this hostname — the site is intentionally public
> Contrast `wiki.ezpzps.com`, `banker.*`, `clipboard.*`, which are all gated to Dave. Don't accidentally add a policy during rebuild.

**cloudflared** — runs as **root** via `/Library/LaunchDaemons/com.cloudflare.cloudflared.ezpeasyps.plist`, started with `tunnel run --token <TUNNEL_TOKEN>`. Note `~/Library/LaunchAgents/com.cloudflare.cloudflared.plist` is **`.disabled`** — the user-level agent is not the one running. Credentials also live at `~/.cloudflared/` (`cert.pem`, two `<uuid>.json` credential files, `config.yml`).

> [!warning] Rotate, don't restore
> Those credentials are secrets and this is a security incident. Delete and recreate the tunnel token (or the whole tunnel) in the Cloudflare dashboard rather than restoring the old credentials from backup.

**Caddy** — Homebrew `caddy` v2.11.2 at `/opt/homebrew/bin/caddy`, run as a plain static file server. Not via `brew services` — via launchd (`brew services list` shows `caddy: none`).

**launchd job** — `~/Library/LaunchAgents/com.goac.web.plist`:

```xml
<key>Label</key><string>com.goac.web</string>
<key>ProgramArguments</key>
<array>
  <string>/opt/homebrew/bin/caddy</string>
  <string>file-server</string>
  <string>--root</string><string>/Users/hubiemacstewbie/Claude Projects - MacStew/OGS Projects/Green Oaks Athletic Club/Green Oaks Athletic Club Website/_site</string>
  <string>--listen</string><string>127.0.0.1:8095</string>
</array>
<key>RunAtLoad</key><true/>
<key>KeepAlive</key><true/>
<key>StandardOutPath</key><string>/tmp/goac-web.log</string>
<key>StandardErrorPath</key><string>/tmp/goac-web.err</string>
```

> [!tip] The `_site/` trick
> `_site/` is untracked and holds exactly one entry: `index.html`, a **relative symlink to `../index.html`**. It exists so Caddy serves only the page, not the whole repo (`.git`, flyers, source images). Recreating `_site/` is a required restore step — git will not do it for you.

### GitHub Pages (the second, independent path)

Repo config: source `main` branch, `/` root, `https_enforced: true`, **no CNAME** (`cname: null`), so it publishes at `https://ogs-data-manager.github.io/green-oaks-athletic-club/`. The `.nojekyll` file disables Jekyll processing. This path is **entirely server-side and unaffected by the wipe** — as long as the repo is pushed, this URL keeps working.

### GitHub auth

Two `gh` accounts are in the keyring. The repo lives under the **`OGS-Data-Manager`** org, and that account (not the currently-active `EZPeasyDave`) holds `admin:org`, `repo`, `workflow`. After the wipe you'll need to `gh auth login` for **both**, and you need `OGS-Data-Manager` to push to this repo.

---

## Restore procedure (after the wipe)

**Prerequisite — already satisfied.** `origin/main` is at `994b5ec` and `index.html` on GitHub is 463,365 bytes. Verify with:

```bash
gh api repos/OGS-Data-Manager/green-oaks-athletic-club/contents/index.html --jq '.size'
# expect: 463365
```

**1. Install tooling**

```bash
brew install caddy cloudflared gh
```

**2. Authenticate GitHub**

```bash
gh auth login   # do this for OGS-Data-Manager (needed to push) and EZPeasyDave
```

**3. Clone into the path the launchd job expects**

> [!warning] The path is hardcoded in the plist, spaces and all
> Match it exactly, or edit the plist.

```bash
mkdir -p "$HOME/Claude Projects - MacStew/OGS Projects/Green Oaks Athletic Club"
cd "$HOME/Claude Projects - MacStew/OGS Projects/Green Oaks Athletic Club"
git clone https://github.com/OGS-Data-Manager/green-oaks-athletic-club.git "Green Oaks Athletic Club Website"
```

**4. Rebuild `_site/`** (untracked — will not come from git)

```bash
cd "Green Oaks Athletic Club Website"
mkdir -p _site
ln -sf ../index.html _site/index.html
```

**5. Restore the Caddy launchd job**

Recreate `~/Library/LaunchAgents/com.goac.web.plist` with the XML above, then:

```bash
launchctl load ~/Library/LaunchAgents/com.goac.web.plist
curl -s -o /dev/null -w "%{http_code} %{size_download}\n" http://127.0.0.1:8095/
# expect: 200 463365
```

**6. Reconnect the tunnel**

Rotate first: Cloudflare dashboard → Zero Trust → Networks → Tunnels → `macstew-ezpeasyps` → refresh token. Then:

```bash
sudo cloudflared service install <NEW_TUNNEL_TOKEN>
```

The tunnel's ingress rules live in Cloudflare, not on disk — the `greenoaksathleticclub.com → localhost:8095` mapping comes back on its own once `cloudflared` reconnects.

> [!important] This tunnel carries everything else too
> Restoring it also restores every other `ezpzps.com` / `ezpeasyps.com` service riding it — wiki, clipboard, banker, easydata, dashboard, sessions, ogsef, wedding. Make sure those local services are up, or their subdomains will 502.

**7. Verify end to end**

```bash
curl -s -o /dev/null -w "public  %{http_code} %{size_download}\n" https://greenoaksathleticclub.com/
curl -s -o /dev/null -w "www     %{http_code} %{size_download}\n" https://www.greenoaksathleticclub.com/
curl -s -o /dev/null -w "ghpages %{http_code} %{size_download}\n" https://ogs-data-manager.github.io/green-oaks-athletic-club/
```

Expect `200` on all three. Public should be ~463 KB; GitHub Pages will match whatever is on `main`.

---

## Survives the wipe on its own

- Cloudflare zone, DNS records, tunnel definition, ingress rules, Access policies
- The GitHub repo and GitHub Pages hosting
- **The site content itself** — `index.html`, the flyer images, the archive page (all pushed 2026-07-08)
- The Google Form (interest list) and its responses

## Only exists on this machine

- `_site/` and its symlink (trivial to recreate — step 4 below)
- `com.goac.web.plist` (captured above)
- `~/.cloudflared/` credentials (rotate rather than restore)

Nothing on that list is irreplaceable. **The machine is safe to wipe.**

---

## Loose ends worth deciding, post-restore

1. **Restore only one checkout.** Clone to the Cloudflare-served path and stop there. Don't recreate `~/WebProjects/green-oaks-athletic-club` — a second copy of the same repo is what caused this.
2. **`index.archive.html` and `index.session1-backup.html` are byte-identical.** Pick one name and delete the other.
3. **GitHub Pages now serves the interest-list page.** If the archive page should be the public `github.io` face instead, that's a content decision to make deliberately, not a restore step.
