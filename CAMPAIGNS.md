# Akita — Comfort Club win-back & referral scheme

Build spec + message copy for the two GoHighLevel campaigns that close the gap
between "job completed" and "customer on a service plan / sending us referrals".

Everything here points at pages that exist in this repo:

| Page | Purpose | Link format |
|---|---|---|
| `club.html` | Self-serve Comfort Club signup for a completed job | `https://<host>/club.html?ref={{contact.quote_ref}}&name={{contact.first_name}}` |
| `confirm.html` | Quote acceptance + Comfort Club upsell at point of sale | (existing) |

Prices are never written into a message. `club.html` reads them live from
`comfortClubQuote`, priced per indoor unit — so a 1-unit and a 3-unit customer
get the same link and see different, correct numbers.

---

## Campaign 1 — Comfort Club win-back

### Who it targets

Customers whose install is **complete** but who have **no active plan**. Today
that's everyone who hit "No service plan" on `confirm.html`, plus every job
completed before the upsell panel existed.

### The hook

This is not a discount campaign. The lever is the manufacturer warranty: 5-year
cover is conditional on a documented annual service. No service, no cover — and
a compressor or PCB failure is £600–£1,400. The plan costs less than one failure.

Anchor every touch on that. Do not lead with "£14.99 a month".

### GHL setup

**Custom fields** (contact level)

| Field | Type | Notes |
|---|---|---|
| `quote_ref` | Text | Already set at acceptance — the key `club.html` needs |
| `install_complete_date` | Date | Set by the field app on job closeout |
| `comfort_club_status` | Dropdown | `none` / `active` / `declined` / `manual` |
| `indoor_units` | Number | Drives whether they're self-serve or the 5+ "call us" path |
| `warranty_expiry` | Date | `install_complete_date` + 5 years |

**Tags**: `cc-winback-active`, `cc-member`, `cc-declined`, `cc-call-back`

**Trigger**: contact enters when `install_complete_date` is set AND
`comfort_club_status = none`. Start the sequence **14 days after completion** —
soon enough that the install is fresh, late enough that any snagging is done.

**Exit conditions — set these before you turn it on:**
- `comfort_club_status` becomes `active` → exit immediately, add `cc-member`
- Contact replies to any SMS/WhatsApp → exit to a human, add `cc-call-back`
- Contact clicks *Not interested* → exit, add `cc-declined`, suppress 11 months
- Any open support ticket / unresolved snag → hold the sequence, don't sell into a complaint

**5+ units**: route to a task for a callback instead of the sequence.
`club.html` already blocks self-serve checkout for these, so a link-only
sequence would dead-end them.

### The sequence

**Day 14 — Email: "Your warranty has one condition"**

> Subject: One thing to sort before your warranty needs it
> Preview: Takes two minutes, then it's handled for five years
>
> Hi {{contact.first_name}},
>
> Your system's been running a couple of weeks now — hopefully you've barely
> thought about it since. That's the idea.
>
> One thing worth knowing while it's fresh: your 5-year manufacturer warranty
> has a condition attached. It stays valid **only** if the system gets a
> documented annual service. That's not an Akita rule, it's the manufacturer's
> — and it's the first thing they check on a claim.
>
> Miss it and the expensive parts stop being covered. A compressor or control
> board runs £600–£1,400 to replace.
>
> Comfort Club is how we handle it for you: annual service booked and done,
> no call-out fee if anything goes wrong, 15% off any repair, priority
> response, and we keep the paperwork the manufacturer wants.
>
> [See your plan →]
>
> Cancel anytime. If you'd rather just book services ad-hoc, that's genuinely
> fine — reply and I'll make a note so we stop nudging you.
>
> — {{user.first_name}}, Akita

**Day 18 — SMS** (only if the email went unopened or unclicked)

> Hi {{contact.first_name}} — Akita here. Quick one: your 5yr warranty needs a
> documented annual service to stay valid. Comfort Club covers it from
> £14.99/mo, cancel anytime: {{link}}
> Reply STOP to opt out.

**Day 25 — Email: the arithmetic**

> Subject: What an out-of-warranty repair actually costs
>
> Hi {{contact.first_name}},
>
> Short one. Three real numbers from jobs we've attended this year:
>
> - Compressor replacement, out of warranty — £1,180
> - Control board (PCB) — £640
> - Fan motor — £395
>
> All three would have been covered under manufacturer warranty. All three
> weren't, because the annual service hadn't been done and logged.
>
> Comfort Club is the cheapest of those numbers by a wide margin, and it
> includes the service, no call-out fees, and 15% off repairs.
>
> [Set it up in two minutes →]
>
> — {{user.first_name}}

**Day 32 — WhatsApp** (see the WhatsApp note below — needs an approved template)

> Hi {{contact.first_name}}, {{user.first_name}} from Akita 👋
> Your annual service is what keeps the 5yr warranty valid. Want me to get you
> on Comfort Club so it's handled automatically? Takes 2 mins: {{link}}
> Or reply CALL and I'll ring you.

**Day 45 — Email: last touch, low pressure**

> Subject: Last one from me on this
>
> Hi {{contact.first_name}},
>
> I won't keep nudging. If Comfort Club isn't for you, no problem at all — the
> system will run fine either way.
>
> The one thing I'd ask: **diarise your annual service**. Whether you book it
> with us ad-hoc or go elsewhere, get it done and get the paperwork, because
> that's what the manufacturer asks for if you ever claim.
>
> Your first one is due {{contact.install_complete_date + 12 months}}.
>
> [Join Comfort Club] · [Book a one-off service] · [Not interested — stop these]
>
> — {{user.first_name}}

**Day 46 — no action** → tag `cc-declined`, suppress 11 months, then re-enter at
the pre-service-due window (Campaign 1b below).

### Campaign 1b — the annual re-ask

Far higher intent than the win-back, because the deadline is real. Trigger at
`install_complete_date + 11 months` for anyone still `comfort_club_status = none`.

> Subject: Your annual service is due next month
>
> Hi {{contact.first_name}},
>
> Your system hits its first year on {{date}}. To keep the manufacturer
> warranty valid, it needs its annual service logged before then.
>
> Two ways to do it:
> - **One-off service** — book a visit, pay per visit
> - **Comfort Club** — the service plus no call-out fees, 15% off repairs, and
>   priority response, from £14.99/mo
>
> [Book the service →] · [See Comfort Club →]
>
> — {{user.first_name}}

---

## Campaign 2 — Referral scheme

### Structure

Keep it dead simple. Complexity is what kills referral schemes.

**£50 to the referrer, £50 off for the friend**, paid when the referred job's
**deposit clears** — not on introduction. Paying on introduction gets you
tyre-kickers; paying on deposit gets you real jobs.

For a Comfort Club member, make it **£75** and say so explicitly. It gives the
plan a second reason to exist and makes members your best channel.

Referrer reward paid as bank transfer or knocked off their plan — let them pick.
Cap at nothing; if someone sends ten jobs, pay them ten times and take them for
lunch.

### GHL setup

**Custom fields**

| Field | Type | Notes |
|---|---|---|
| `referral_code` | Text | Per-customer, e.g. `AK-SMITH-4821` |
| `referred_by_code` | Text | Set on the new lead |
| `referral_count` | Number | Increments on each deposit-cleared referral |
| `referral_owed` | Currency | Accrues; cleared when paid |

**Tags**: `referrer`, `referred-lead`, `referral-paid`, `advocate` (3+ referrals)

**Pipeline**: add a *Referrals* pipeline with stages
`Introduced → Contacted → Surveyed → Quoted → Deposit paid → Reward paid`.
You need this — without it, referrals get lost inside the main pipeline and
nobody gets paid, which kills the scheme in one cycle.

**Workflows**

1. **Issue the code** — on job completion, generate `referral_code`, send the
   referral email/WhatsApp. Runs alongside Campaign 1, not inside it.
2. **Capture** — inbound lead with `referred_by_code` set → tag `referred-lead`,
   create a Referrals-pipeline opportunity, notify the referrer that their
   friend got in touch. That notification matters more than people expect: it's
   proof the scheme is real, and it's what makes them refer a second time.
3. **Pay out** — on deposit cleared, increment `referral_count`, add to
   `referral_owed`, create a task to pay, notify the referrer.
4. **Advocate** — at `referral_count >= 3`, tag `advocate` and flag for a
   personal thank-you. These are the people worth protecting.

### Copy — the ask (send ~30 days post-install, once the system has proved itself)

> Subject: Know anyone else sweating through the summer?
>
> Hi {{contact.first_name}},
>
> Now you've had the system a month — if it's doing its job, a favour.
>
> Most of our work comes from people like you telling a neighbour, a colleague,
> or family. So we've made it worth your while:
>
> **You get £50. They get £50 off their installation.**
> {{#if cc-member}}As a Comfort Club member, yours is **£75**.{{/if}}
>
> Your code: **{{contact.referral_code}}**
>
> Just pass it on — they quote it when they get in touch, and your reward lands
> when their deposit clears. No limit on how many.
>
> [Send it to someone →]
>
> — {{user.first_name}}, Akita

### Copy — WhatsApp version

> Hi {{contact.first_name}}, {{user.first_name}} from Akita 👋
> Glad the system's treating you well. If you know anyone who could use one:
> your code **{{contact.referral_code}}** gets them £50 off and gets you £50
> when their deposit clears. Just forward this 👍

### Copy — the payout notification (do not skip this one)

> {{contact.first_name}} — {{referred_name}} just paid their deposit, so your
> £{{amount}} is on its way. Thanks for the intro, genuinely.
> That's {{contact.referral_count}} you've sent us now.

---

## WhatsApp — read before building the WhatsApp steps

The WhatsApp touches above will not send as written until this is handled:

- **Business-initiated messages outside the 24-hour customer service window
  must use a Meta-approved template.** Free-form text only works inside 24h of
  the customer's last message. Every touch in these sequences is
  business-initiated, so each needs its own approved template. Submit them
  early — approval is not instant.
- **Opt-in is required and is separate from email/SMS consent.** Under PECR,
  email and SMS to existing customers about a similar service is covered by
  soft opt-in; WhatsApp marketing is not the same thing. Capture WhatsApp
  consent explicitly — the survey form and the acceptance page are the two
  natural places.
- **Route replies to a human fast.** The reply rate on WhatsApp is far higher
  than email, and a bot that stonewalls a real question does more damage than
  no WhatsApp at all. Bot handles: plan pricing, what's included, booking a
  service, referral code lookup. Everything else → engineer or office, tagged
  `cc-call-back`.

Suggested bot scope, in priority order:

1. "What does Comfort Club include / what does it cost?" → answer + `club.html` link
2. "Book my annual service" → calendar link
3. "What's my referral code?" → field lookup
4. "Something's not working" → **straight to a human**, no triage attempt

---

## Sequencing the two campaigns

Don't run them on top of each other. Order:

1. **Day 14–45** — Comfort Club win-back (Campaign 1)
2. **Day 30** — referral ask, *only if* they've gone quiet on Campaign 1 or
   already joined. A referral ask landing mid-warranty-pitch muddies both.
3. **Month 11** — annual re-ask (Campaign 1b)
4. **Post-service, every year** — referral ask again. Right after a good
   service visit is the highest-intent referral moment there is.

## What to measure

Per campaign, weekly:

- Win-back: entered → clicked → checkout started → **plan active** (the only
  number that counts), and cost per conversion
- Which touch converts. If Day 25's arithmetic email carries it, move it earlier
- Referral: codes issued → codes quoted → deposits cleared → rewards paid.
  A gap between "quoted" and "paid" means workflow 3 is broken; fix it that week
- Unsubscribe and STOP rates per touch. If any single touch is above ~0.5%,
  it's too aggressive — rewrite it
