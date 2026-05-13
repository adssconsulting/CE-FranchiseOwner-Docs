# API & Integrations

!!! info "Placeholder"
    Run the **Docs Engine** tab to regenerate this page from the
    current state of `scraper/`, `email_engine/`, and any external
    HTTP/SMTP usage in the codebase.

## External integrations

### PropStream

Browser-automation scrape of `propstream.com` using Selenium with a
persistent Chrome user-profile. Credentials live in `app_settings`
(base64-encoded password). Login uses human-paced typing on the
sign-in form.

### SMTP (Rackspace)

Outgoing email via `smtplib`. Configuration lives in `app_settings`
keys prefixed `smtp.*`. Signature appended with a `---` separator.

### Future / planned

_To be detailed once any new integrations are introduced._
