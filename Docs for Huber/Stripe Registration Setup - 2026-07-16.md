# GOAC — Stripe Registration Setup Guide
*2026-07-16 · the steps that live outside the website*

The website is done and committed. It now has a **Register & Pay** button, a
waitlist link, and a waiver page — all pointing at **placeholder URLs**. This guide
covers the three things *you* set up (Stripe, the waitlist form, the Google Sheet)
and the one find-and-replace that makes it all live.

**Do not deploy the site until Step 4 is done** — until then the buttons point at
`REPLACE_ME` and won't work.

---

## Step 1 — Stripe account (one-time, ~15 min)

1. Go to **stripe.com** → sign up under the **GOAC LLC** (legal name, EIN).
2. Connect the **business bank account** — this is where session money lands.
3. Stripe fee is **~2.9% + 30¢** per payment → about **$3.78 on $120**, so ~**$116.22**
   nets to the bank. Normal cost of doing business.

## Step 2 — Create the Payment Link for the K–2 session

In Stripe: **Product catalog → Payment Links → New**.

- **Product:** "K–2 Fall Session #1" · **Price:** $120 · one-time.
- **Custom fields** (add these so you capture registration info at checkout):
  - Child's full name — *text, required*
  - Child's grade — *text or dropdown (K, 1, 2), required*
  - Allergies / medical notes — *text, optional*
- **Also collect:** turn on **phone number** (email + name are collected by default).
- **Cap the spots:** in the link's options, **limit the number of completed payments to 50.**
  When 50 is hit, Stripe automatically stops accepting payments — that's your enrollment cap.
- **Waiver checkbox:** enable **"Require customers to accept terms of service"** and point
  it at your waiver page URL:
  `https://<your-github-pages-domain>/waiver.html`
  *(⚠️ Have Dave I / Cody — or ideally an attorney — read `waiver.html` first. I wrote
  standard youth-sports release language, but it's a real legal document for a real LLC.)*
- **Save**, then **copy the link** (looks like `https://buy.stripe.com/xxxxxxxx`).

## Step 3 — Waitlist form + the Google Sheet Cody reads

**Waitlist (tiny):** Make a new **Google Form** with just two questions —
*Parent name* and *Parent email*. Title it "GOAC K–2 Waitlist." Grab its short
`https://forms.gle/...` share link. This only shows on the site when the session is full.

**The roster Cody opens — Stripe → Google Sheet:**
1. Sign up for **Zapier** or **Make.com** (free tier is plenty for this volume).
2. Build one automation: **Trigger** = Stripe "Checkout Session Completed"
   (or "New Payment") → **Action** = "Add row to Google Sheet."
3. Map the columns: *Child Name · Grade · Parent Email · Parent Phone · Paid Date · Amount*
   (the custom fields from Step 2 show up as fields you can map).
4. Share that Google Sheet with Cody (view access) and have him **bookmark it**.
   That's his whole job — open the bookmark, see who's registered. He never touches Stripe.

## Step 4 — Drop the real URLs into the site (I can do this)

The site has exactly **4 placeholders** to swap. Once you have the two real URLs,
either paste them to me and I'll do the find-and-replace + deploy, or do it yourself:

| Placeholder in `index.html` | Replace with |
|---|---|
| `https://buy.stripe.com/REPLACE_ME` (2 spots — line ~434 and ~477) | your Stripe link |
| `https://forms.gle/REPLACE_ME_WAITLIST` (2 spots — line ~437 and ~478) | your waitlist form link |

Then deploy (git push to the Pages repo). **After that it's live and self-running.**

---

## The finished picture

- **Parents:** click **Register & Pay — $120** → Stripe checkout → done in one step.
- **Cody:** opens a bookmarked Google Sheet, sees the roster. Never logs into Stripe.
- **Dave I:** owns the Stripe dashboard for refunds / details when needed.
- **You:** host nothing — GitHub Pages + Stripe's cloud do all the work. Your Mac
  and home internet are out of the loop entirely.

**No more:** Google Form as system-of-record, Zelle reconciliation, choice-eliminator
hack, or "who paid?" chasing — you can't register without paying.
