# Frequently Asked Questions

## Getting started

### How do I sign in?
Open the app's web address and enter your franchise login and password. Each
franchise has its own login; the national office has an administrator login that
can see and manage all branches.

### Do I need to install anything?
No. CE FranchiseOwner runs entirely in your web browser. There's nothing to
download or update on your computer.

### I forgot my password — what do I do?
Ask your national administrator. From **Admin → Logins**, they can set or reset
the password for any franchise login.

## Leads & email

### Where do the property listings come from?
You import PropStream "Active Under Contract" county exports on the **Properties**
screen, or pull them from a live MLS data feed. The app loads new listings and
skips any address it has already seen for that county.

### What is the difference between "Master Agent List Present" and "No-Master" mode?
If your branch has a master list of agents, **Master** mode matches new listing
agents against it so you don't duplicate people. New markets without a master list
use **No-Master** mode, which sorts the raw imported agents into "clean" (ready to
email) and "email missing" so you can make them emailable directly.

### How does the app decide which email each agent gets?
It counts how many times you've already contacted each agent and automatically
picks the matching step in the sequence (1st Contact, 2nd Contact, and so on).
Agents with several active listings get a separate "Multi" set of templates. You
can always override the choice per agent before sending.

### Will the same agent get emailed twice by mistake?
No. The Email Queue tracks every send, skips anyone who unsubscribed or bounced,
and the **New since last email** filter shows only listings loaded since your last
real send. You also get an approval popup with the exact count before anything goes out.

### Can I test an email before sending it for real?
Yes. Use **Send 1 Test** in the Email Queue to email yourself a sample, or the
**Send Utility** in Admin to re-send any past email to an address you choose
(without recording it in your history).

### How do I edit the wording of my emails?
In **Admin → Templates**. Each change is saved as a new version, and you can view
or restore any earlier version. The Opening Line and Subject are unique per
template; the Shared Body is common to all templates of that type.

## Calls & applications

### What does the Call Tracker do?
It turns a spreadsheet of previous clients into a daily call worksheet: it matches
them to your master list, you approve the matches, then you log call outcomes and
follow-up dates with call scripts on hand.

### What is the Gate Check?
An automatic pre-screen for commission-advance applications. The app reads new
application emails, extracts the details, scores them against 12 criteria, and
gives a verdict — Good to Go, Can't Decide, or Don't Waste Time — so you only
underwrite the deals worth your time.

### Is sensitive applicant information safe?
Yes. Personal details like Social Security and bank numbers are stored encrypted
and are never shown on screen.

## Data & branches

### Can one franchise see another franchise's data?
No. Every branch is isolated — you only ever see your own listings, agents, emails,
and applications. Only the national administrator can view across branches, and
even then each branch's private configuration stays its own.

### How does a new franchise get set up?
The national administrator uses **Admin → Onboard Franchise** to create the new
branch's login and copy South Carolina's proven email templates and settings into
it — ready to go in minutes.

### Are my records backed up?
Yes. Use **Admin → Backup & Restore** to run a full backup whenever you like,
download past backups, or restore from one. Restoring overwrites current data and
is double-confirmed so it can't happen by accident.

### Who do I contact for help?
Reach out to your national office. This help site is always available, and the
underlying technical reference is in the Technical and QA sections (intended for
the system administrator).
