<style>
  /* ── Hero ──────────────────────────────────────────────────────── */
  .home-hero {
    margin: 1rem 0 2.5rem;
  }
  .home-eyebrow {
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: rgba(0, 0, 0, 0.55);
    margin-bottom: 0.5rem;
  }
  [data-md-color-scheme="slate"] .home-eyebrow {
    color: rgba(255, 255, 255, 0.65);
  }
  .home-headline {
    font-size: clamp(28px, 4vw, 40px);
    font-weight: 700;
    line-height: 1.15;
    margin: 0 0 0.75rem;
    color: inherit;
  }
  .home-subhead {
    font-size: 15px;
    line-height: 1.55;
    max-width: 720px;
    color: inherit;
    opacity: 0.85;
    margin: 0;
  }

  /* ── Product card grid ─────────────────────────────────────────── */
  .home-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
    margin: 2rem 0;
  }

  .home-card {
    background: #ffffff;
    color: #1f2937;
    border: 1px solid rgba(0, 0, 0, 0.1);
    border-radius: 12px;
    padding: 1.25rem;
    cursor: pointer;
    transition: border-color 180ms ease,
                box-shadow 180ms ease;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI",
                 Roboto, "Helvetica Neue", Arial, sans-serif;
  }
  [data-md-color-scheme="slate"] .home-card {
    background: rgba(255, 255, 255, 0.04);
    border-color: rgba(255, 255, 255, 0.15);
    color: rgba(255, 255, 255, 0.95);
  }
  .home-card:hover {
    border-color: var(--accent);
  }
  .home-card.open {
    border-color: var(--accent);
    box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.04);
  }
  [data-md-color-scheme="slate"] .home-card.open {
    box-shadow: 0 0 0 2px rgba(255, 255, 255, 0.06);
  }

  .home-card-head {
    display: flex;
    align-items: center;
    gap: 0.65rem;
    margin-bottom: 0.75rem;
  }
  .home-card-icon {
    color: var(--accent);
    display: inline-flex;
  }
  .home-tag {
    background: var(--accent-bg);
    color: var(--accent);
    padding: 0.18rem 0.6rem;
    border-radius: 999px;
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.02em;
  }
  .home-chevron {
    margin-left: auto;
    color: rgba(0, 0, 0, 0.45);
    transition: transform 200ms ease;
    display: inline-flex;
  }
  [data-md-color-scheme="slate"] .home-chevron {
    color: rgba(255, 255, 255, 0.5);
  }
  .home-card.open .home-chevron {
    transform: rotate(180deg);
    color: var(--accent);
  }

  .home-card-title {
    font-size: 18px;
    font-weight: 700;
    margin: 0 0 0.4rem;
    line-height: 1.25;
    color: inherit;
  }
  .home-card-summary {
    font-size: 14px;
    line-height: 1.5;
    margin: 0;
    opacity: 0.85;
  }

  .home-card-brief {
    display: none;
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid rgba(0, 0, 0, 0.08);
  }
  [data-md-color-scheme="slate"] .home-card-brief {
    border-top-color: rgba(255, 255, 255, 0.1);
  }
  .home-card.open .home-card-brief {
    display: block;
  }
  .home-card-brief p {
    font-size: 14px;
    line-height: 1.55;
    margin: 0 0 0.75rem;
    opacity: 0.9;
  }
  .home-card-brief ul {
    margin: 0 0 1rem;
    padding-left: 1.25rem;
  }
  .home-card-brief li {
    font-size: 13px;
    line-height: 1.55;
    margin-bottom: 0.3rem;
  }
  .home-card-link {
    color: var(--accent) !important;
    font-size: 14px;
    font-weight: 600;
    text-decoration: none;
  }
  .home-card-link:hover {
    text-decoration: underline;
  }

  /* ── Closing strip ─────────────────────────────────────────────── */
  .home-closing {
    background: #f5f6f8;
    border-radius: 12px;
    padding: 1.4rem 1.6rem;
    margin: 2.5rem 0 2rem;
    text-align: center;
  }
  [data-md-color-scheme="slate"] .home-closing {
    background: rgba(255, 255, 255, 0.05);
  }
  .home-closing p {
    margin: 0;
    font-size: 14px;
    line-height: 1.6;
    opacity: 0.85;
  }

  /* ── Bottom nav ────────────────────────────────────────────────── */
  .home-nav {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 0.75rem;
    margin: 1.5rem 0 2rem;
  }
  .home-nav-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: transparent;
    border: 1px solid rgba(0, 0, 0, 0.12);
    border-radius: 10px;
    padding: 0.9rem 1.1rem;
    font-size: 14px;
    font-weight: 600;
    color: inherit !important;
    text-decoration: none !important;
    transition: border-color 180ms ease,
                background 180ms ease;
  }
  [data-md-color-scheme="slate"] .home-nav-card {
    border-color: rgba(255, 255, 255, 0.15);
  }
  .home-nav-card:hover {
    border-color: rgba(0, 0, 0, 0.35);
    background: rgba(0, 0, 0, 0.02);
  }
  [data-md-color-scheme="slate"] .home-nav-card:hover {
    border-color: rgba(255, 255, 255, 0.4);
    background: rgba(255, 255, 255, 0.04);
  }
  .home-nav-card span.arrow {
    opacity: 0.5;
  }
</style>

<div class="home-hero">
  <div class="home-eyebrow">CE FranchiseOwner</div>
  <h1 class="home-headline">Automate. Digitize. Stay in Control.</h1>
  <p class="home-subhead">Tools built for Commission Express franchise owners to automate day-to-day activities, digitize your client relationships, and run a controlled CRM &mdash; without the complexity.</p>
</div>

<div class="home-grid">

  <div class="home-card" data-toggle="card" role="button" tabindex="0" aria-expanded="false" style="--accent: #534AB7; --accent-bg: #EEEDFE;">
    <div class="home-card-head">
      <span class="home-card-icon" aria-hidden="true">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/></svg>
      </span>
      <span class="home-tag">Automate</span>
      <span class="home-chevron" aria-hidden="true">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"/></svg>
      </span>
    </div>
    <h2 class="home-card-title">Lead Engine</h2>
    <p class="home-card-summary">Automates agent outreach. You control who gets emails, what they say, and when they are sent.</p>
    <div class="home-card-brief">
      <p>Every day the app finds agents whose deals just went Active Under Contract in SC. It matches them to licensed agents and sends a personalised email sequence &mdash; the right message, at the right time, automatically.</p>
      <ul>
        <li>Choose which agents receive emails</li>
        <li>Set the email sequence and frequency</li>
        <li>Track opens, sends and responses</li>
      </ul>
      <a class="home-card-link" href="marketing/feature-guides/#lead-engine">View full guide &rarr;</a>
    </div>
  </div>

  <div class="home-card" data-toggle="card" role="button" tabindex="0" aria-expanded="false" style="--accent: #0F6E56; --accent-bg: #E1F5EE;">
    <div class="home-card-head">
      <span class="home-card-icon" aria-hidden="true">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/></svg>
      </span>
      <span class="home-tag">Digitize</span>
      <span class="home-chevron" aria-hidden="true">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"/></svg>
      </span>
    </div>
    <h2 class="home-card-title">Call Tracker</h2>
    <p class="home-card-summary">Digitizes your call history. Know who to call today and never miss a follow-up.</p>
    <div class="home-card-brief">
      <p>A daily call worksheet for previous and inactive clients. Log every call outcome and schedule follow-ups with one click &mdash; nothing falls through the cracks.</p>
      <ul>
        <li>Daily call list generated automatically</li>
        <li>Log outcomes: Interested, Voicemail, Converted</li>
        <li>Set follow-up dates per agent</li>
      </ul>
      <a class="home-card-link" href="marketing/feature-guides/#call-tracker">View full guide &rarr;</a>
    </div>
  </div>

  <div class="home-card" data-toggle="card" role="button" tabindex="0" aria-expanded="false" style="--accent: #854F0B; --accent-bg: #FAEEDA;">
    <div class="home-card-head">
      <span class="home-card-icon" aria-hidden="true">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
      </span>
      <span class="home-tag">Save time</span>
      <span class="home-chevron" aria-hidden="true">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"/></svg>
      </span>
    </div>
    <h2 class="home-card-title">Gate Check</h2>
    <p class="home-card-summary">Saves 30 minutes per application. Get a clear verdict before you start underwriting.</p>
    <div class="home-card-brief">
      <p>Every application email is automatically scored against 12 criteria. You get a traffic-light verdict &mdash; Worth It, Not Worth It, or Cannot Decide &mdash; before opening a single document.</p>
      <ul>
        <li>Reads incoming application emails automatically</li>
        <li>Scores 12 key criteria per deal</li>
        <li>Traffic-light result: green / amber / red</li>
      </ul>
      <a class="home-card-link" href="marketing/feature-guides/#gate-check">View full guide &rarr;</a>
    </div>
  </div>

</div>

<div class="home-closing">
  <p>More revenue in your pipeline. More clients converted from calls. 30 minutes saved on every single application.</p>
</div>

<div class="home-nav">
  <a class="home-nav-card" href="marketing/feature-guides/">Feature guides<span class="arrow">&rarr;</span></a>
  <a class="home-nav-card" href="technical/architecture/">Technical docs<span class="arrow">&rarr;</span></a>
  <a class="home-nav-card" href="qa/test-suite/">QA &amp; testing<span class="arrow">&rarr;</span></a>
  <a class="home-nav-card" href="sprint/log/">Release notes<span class="arrow">&rarr;</span></a>
</div>

<script>
(function () {
  // Expand/collapse each product card. Clicking anywhere on the card
  // toggles its 'open' class except when the click target is the
  // "View full guide" link, which should navigate without toggling.
  var cards = document.querySelectorAll('.home-card[data-toggle="card"]');
  for (var i = 0; i < cards.length; i++) {
    (function (card) {
      function toggle(e) {
        if (e && e.target && e.target.closest('.home-card-link')) {
          return; // let the link navigate
        }
        var open = card.classList.toggle('open');
        card.setAttribute('aria-expanded', open ? 'true' : 'false');
      }
      card.addEventListener('click', toggle);
      card.addEventListener('keydown', function (e) {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          toggle();
        }
      });
    })(cards[i]);
  }
})();
</script>
