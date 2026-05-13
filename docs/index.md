# CE FranchiseOwner Documentation

Welcome to the documentation site for **CE FranchiseOwner**, the
Windows desktop application used by Commission Express of South
Carolina to identify Active Under Contract real-estate listings,
match listing agents against a master agent roster, and send
personalised drip-email campaigns to those agents.

## Documentation sections

<div class="grid cards" markdown>

-   :material-account:{ .lg .middle } **[User Guide](marketing/product-overview.md)**

    ---

    What CE FranchiseOwner does, every feature, and how to use them.
    Written for the franchise owner who's running the system day to
    day.

-   :material-cog:{ .lg .middle } **[Technical Docs](technical/architecture.md)**

    ---

    Architecture, database schema, integrations, and deployment.
    For engineers maintaining or extending the codebase.

-   :material-test-tube:{ .lg .middle } **[QA & Testing](qa/test-suite.md)**

    ---

    Test suite reference and quality-assurance procedures.

-   :material-calendar-clock:{ .lg .middle } **[Release Notes](sprint/log.md)**

    ---

    Chronological log of sprint changes and feature releases.

</div>

## How this site is updated

This site is automatically published from the `main` branch of the
[`CE-FranchiseOwner-Docs`](https://github.com/adssconsulting/CE-FranchiseOwner-Docs)
repository on every push. The content itself is regenerated from
the source code by running the **Docs Engine** tab inside the app's
Admin window — that workflow copies a prompt to the clipboard which
the user pastes into Claude Code; Claude reads the current state of
the codebase and updates the markdown files in this repository.

!!! note
    Every page below is a placeholder until the Docs Engine workflow
    runs for the first time. Run the Docs Engine in the Admin tab to
    populate every section with content generated from the latest
    source.
