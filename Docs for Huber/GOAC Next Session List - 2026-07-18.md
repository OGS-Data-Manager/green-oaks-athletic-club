# GOAC — Next Session List

*Rolling list of work deferred out of the 2026-07-18 pre-launch session.*

---

## 1. Hosting cutover — Mac Studio → GitHub Pages ⚠️ HIGHEST RISK

`greenoaksathleticclub.com` is currently served through the `macstew-ezpzps`
tunnel from `localhost:8095`. **Proof:** an untracked `.DS_Store` returns 200 on
the live domain but 404 on the Pages origin.

This violates the standing decision that signups must not depend on Dave's Mac
Studio or home internet. Mac sleeps, ISP blips, or a wipe → the club's website
*and* its payment funnel go dark. There is a known failure mode on record where a
wipe resets `pmset` and the Mac silently idle-sleeps.

**Steps:**
1. Add a `CNAME` file to the repo root containing `greenoaksathleticclub.com`
2. Repo → Settings → Pages → set the custom domain
3. Repoint DNS at Pages; consider grey-cloud (DNS-only) so GitHub can issue the cert
4. Remove `greenoaksathleticclub.com` + `www` ingress from the `macstew-ezpzps` tunnel
5. Re-verify with the `.DS_Store` test — it should now 404 on the live domain

**Also:** `.DS_Store` is publicly downloadable today, leaking local filenames. Add
it to `.gitignore` and stop serving it.

---

## 2. Enrollment cap hits 50 — notification + button swap

The cap is set on the Stripe Payment Link, but Stripe's behavior at the limit is
ugly: **the link deactivates and parents land on a generic "this payment link is
no longer active" page.** No explanation, no waitlist pointer.

The site's "Session full? Join the waitlist" text sits *below* the button, so the
graceful path exists — but the button itself dead-ends.

**Wanted:**
- A **notification when the cap is reached** (Stripe webhook, or simplest: a
  Zapier/Make watch on payment count) so somebody knows the moment it happens
- **Swap the "Register & Pay" button for the waitlist button** on the site —
  make this a one-line change so it's trivial under pressure

Worth pre-building the swapped button markup so it's a comment-toggle, not an edit.

---

## 3. Reply-from address for the coaches

Cloudflare Email Routing is **forward-only**. Mail to
`coaches@greenoaksathleticclub.com` reaches both coaches, but when Izenstark
replies the parent sees `DavidIzenstark@gmail.com`.

Accepted for launch. To fix properly: an SMTP relay wired into Gmail's
"Send mail as" so replies come from the club address.

---

## 4. Flyers still say Zelle

`images/GOAC-flyer-1page.html` and `-2page.html` both still show **"$120 via
Zelle"** and carry the Zelle payment QR code. Zelle is retired; payment is now
Stripe, and payment *is* registration.

Nothing has been printed, so no live harm — but **do not post these images
alongside the announcement.** Contact lines were already updated to
`coaches@greenoaksathleticclub.com` on 2026-07-18; the payment section was not.

---

## 5. Cody's roster (deferred by Dave — not needed this session)

Zapier/Make: Stripe *Checkout Session Completed* → append row to a Google Sheet
Cody bookmarks, so he never logs into Stripe. Cody is not involved in this
session. Steps in `Stripe Registration Setup - 2026-07-16.md`.

---

## 6. Chat widget coverage gaps

Grade permutations were broadened on the "who can join" entry 2026-07-18. Not yet
covered: **6th grade / middle school / junior high** — deliberately left out
rather than invent an eligibility answer. If a "we're K–5 only" line is wanted,
add those keywords with an explicit answer.

---

## Watch items

- **Club Gmail appeal** — `greenoaksathleticclub@gmail.com`, flagged bot-created.
  Result expected ~2026-07-20 to 07-22. Being abandoned regardless; the domain
  address is now the club's durable identity.
- **Chase business account** may close (moved to Bank of America). Stripe payout
  account must point at BofA *before* Chase closes, or payouts fail.
