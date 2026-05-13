# Product Overview

CE FranchiseOwner is three tools on one platform: a lead-generation engine for outbound agent outreach, a call tracker for previous-client follow-up, and a pre-underwriting gate check that scores inbound application emails before you spend time on them.

## Lead Engine

**Automated targeted outreach to active real estate agents — on your schedule, under your control.**

Every day the app finds real estate agents whose deals just went Active Under Contract in South Carolina. It matches them to licensed agents, then sends a personalised email sequence on your behalf — the right message, to the right agent, at the right time. You decide who gets emails, what they say, how many times they are sent, and how often.

<details>
<summary>How it works (technical)</summary>

Scrapes PropStream across all South Carolina counties for Active Under Contract listings, fuzzy-matches listing agents against a master roster of licensed SC real-estate agents, then sends a personalised drip-email sequence through Rackspace SMTP. Every email is logged to a single email_log source of truth.

</details>

> Targeted marketing giving you full control of where, how, and when to reach agents — no manual work, no missed opportunities.

## Call Tracker

**A smart call logger that keeps your previous client relationships alive.**

Tracks every call made to previous or inactive clients — who you called, what they said, and when to follow up. The app gives you a daily call list so nothing falls through the cracks. You log outcomes (Interested, Call me back, Left voicemail, Not interested, Do not call, Converted) and set follow-up dates with one click.

<details>
<summary>How it works (technical)</summary>

Imports a Previous Clients Excel file, counts deals per agent, fuzzy-matches against the master list, lets you approve or reject each match, then drives a daily call worksheet with outcome tracking and follow-up scheduling.

</details>

> Targeted call marketing with built-in scripts, tracker, and logger — turning inactive clients into active revenue.

## Pre-Underwriting / Gate Check

**Know if a deal is worth your time before you spend a minute on it.**

Every application email that arrives in your inbox gets automatically scored against 12 key criteria. You get a clear traffic-light verdict — Worth It, Not Worth It, or Cannot Decide — before you open a single document. Stop wasting hours on deals that will never close.

<details>
<summary>How it works (technical)</summary>

Reads incoming application emails via IMAP, parses attached PDFs (purchase contract, MLS sheet, pre-qual letter, etc.), and runs a 12-question gate check. Two questions (Q4 open contingencies, Q8 pre-approval / proof of funds) use AI document analysis. Results are encrypted and stored per application.

</details>

> Fully automated pre-screening that traffic-lights every deal — so you only underwrite the ones worth your time.
