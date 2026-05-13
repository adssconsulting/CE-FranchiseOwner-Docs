# Product Overview

## What it solves

Commission Express franchise owners need a steady inflow of two kinds
of work:

1. **Outbound leads** — real-estate agents whose listings just went
   *Active Under Contract* are the textbook commission-advance
   prospect. They have a deal in flight; they're waiting on a
   60–90-day settlement; cashflow is the leverage. The hard part is
   finding them before they go to someone else, with current
   information, and reaching out fast.
2. **Inbound applications** — agents who do submit an advance
   request arrive via a Gravity Forms application that produces a
   long, multi-page email. Reviewing that by hand — pulling out the
   numbers, opening every attached PDF, deciding whether the deal
   is fundable — takes 20-40 minutes per file. Most of that work
   is mechanical.

CE FranchiseOwner attacks both ends. The **Lead Engine** finds and
emails the outbound side; the **Pre-Underwriting** module
processes the inbound applications and produces a score the
franchise owner can act on in under a minute per file.

## Value proposition

| Without CE FranchiseOwner | With CE FranchiseOwner |
|---|---|
| Manual PropStream pulls, county by county, weekly | Saved searches loop automatically; results land in one screen |
| Agent name vs. MLS name reconciled by eye | Fuzzy match + a human verification queue with a Golden Record workflow |
| Email blasts from a generic SMTP tool, no per-agent context | Personalised drip with the agent's actual listings inserted into the body |
| Inbox manually scanned for "stop", "unsubscribe" | IMAP poller checks twice a day and suppresses automatically |
| Application emails reviewed by hand, PDFs opened one at a time | Email parsed into encrypted fields; PDFs classified and scanned by LLM; 12-question gate check returns a verdict |
| No record of which DB you sent from, or when | Every send and every app start logged with environment + DB name |
| Backups by hand, when remembered | Auto-backup on every production startup, plus manual button |

## Who it's built for

- **A franchise owner running CE in South Carolina.** Familiar with
  Excel, less so with Python or databases. Wants buttons, not
  command lines.
- **A small ops team** (1-3 people) who share one prod machine.
- **An engineer maintaining the system part-time** — code is
  organised so business logic is separable from the tkinter GUI so
  a future Qt or web rewrite doesn't have to touch the DB or email
  layer.

## Out of scope

- Multi-tenant SaaS. The app assumes one franchise, one machine.
- Cloud database. Everything runs against an embedded MariaDB on
  localhost, by design — keeps PII on the franchise owner's hardware.
- Mobile / web app. There is no browser UI; the only web-facing
  surface is this documentation site.
