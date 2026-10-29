# Course Sales Automation: Stripe → Google Sheets → Brevo → Gmail (n8n)

An n8n workflow I built for a content creator who sells online courses. Every successful Stripe payment now updates her finance sheet, adds the buyer to the right email list and, for some courses, sends a welcome email, with no manual work.

<!-- 🎥 [Watch the 3-minute demo](https://www.loom.com/share/YOUR_VIDEO) -->

![n8n workflow](images/workflow-canvas.jpg)

## Context
A content creator sells several online courses. Each new sale was handled by hand: checking the payment, logging revenue, adding the buyer to email lists and sending a welcome email.

## Problem
- Each sale required 3-4 manual steps across several tools (~6.5 minutes per sale, my estimate).
- At ~40 sales per week, that added up to ~4.3 hours of repetitive work weekly.
- Revenue tracking in Google Sheets was delayed and prone to typos.
- Buyers sometimes joined the email list late, and welcome emails depended on someone remembering to send them.

## Solution
An n8n workflow triggered by each successful Stripe payment (`payment_intent.succeeded`):

1. **Trigger:** Stripe payment event.
2. **Customer lookup:** a Stripe node fetches the customer to get name and email.
3. **Data preparation:** course name (normalized), date (Montevideo time zone), amount (converted from cents) and currency.
4. **Finance tracking:** a new row is appended to the Google Sheets finance spreadsheet.
5. **Routing:** a Switch node reads the course name and sends the buyer to one or more of four course branches (several can match at once).
6. **Email marketing:** the buyer is added to the matching Brevo list via the Brevo API.
7. **Onboarding (one course):** the buyer is registered in a live-sessions sheet and receives a custom HTML welcome email through Gmail.

**Tools:** n8n, Stripe, Google Sheets, Brevo, Gmail

## Result
- ~4.3 hours saved per week (~18 hours per month), based on an estimated 6.5 minutes of manual work per sale and ~40 sales per week.
- Finance sheet updated automatically, with no typing errors.
- Every buyer is added to the correct Brevo list, and welcome emails go out immediately instead of whenever someone remembered.
- Adding a new course takes ~10 minutes (one new branch in the workflow).

## Challenges & What I Learned
- **Native node limitation:** the built-in Brevo node didn't let me add a contact to a specific list, so I called the Brevo API directly with an HTTP Request node (`POST /v3/contacts` with `listIds` and `updateEnabled`).
- **Secrets management:** the Brevo API key is stored as an n8n credential (Header Auth) instead of inside each node, so it never appears in the exported workflow.
- **Reliability:** enabled automatic retries (3 s between attempts) on the critical nodes so a temporary API failure doesn't lose a sale.
- **No duplicate contacts in Brevo:** used `updateEnabled: true`, so an existing contact is updated instead of failing or duplicating.
- **Time zones and amounts:** Stripe sends timestamps in UTC and amounts in cents, so I converted them to local dates and real currency values before writing to the sheet.
- **Multiple courses per purchase:** configured the Switch node to allow several matching outputs.

## Next Steps
- Deduplicate by Stripe payment ID so a retried webhook never creates a second row.
- Error alerts (Slack or email) when a step fails.
- Dashboard (Looker Studio or Power BI) on top of the finance sheet.

## Using the workflow
`workflow.json` is a sanitized export. To use it:
1. Import it in n8n (*Workflows → Import from file*).
2. Reconnect your own Stripe, Google Sheets and Gmail credentials, and create a Header Auth credential named `api-key` with your Brevo API key.
3. Replace the placeholders: `YOUR_SPREADSHEET_ID`, `YOUR_SHEET_GID`, and the links in the email template.
4. Adjust the Switch rules and Brevo list IDs to your own courses.
