# Course Sales Automation: Stripe → Google Sheets → Brevo → Gmail (n8n)

## Context
A content creator sells online courses and was handling each new sale by hand:
checking the payment, logging revenue, adding buyers to email lists, and
sending welcome emails.

## Problem
- Each sale required 3-4 manual steps across several tools (~6.5 minutes
  per sale, my estimate).
- At ~40 sales per week, that added up to ~4.3 hours of repetitive work weekly.
- Revenue tracking in Google Sheets was delayed and prone to typos.
- Buyers sometimes joined the email list late, and welcome emails depended
  on someone remembering to send them.

## Solution
An n8n workflow triggered by each successful Stripe payment:

1. **Trigger:** Stripe payment event.
2. **Routing:** the workflow reads the course name and branches by course.
3. **Finance tracking:** a new row is added to the Google Sheets finance
   spreadsheet (date, course name, product type, amount).
4. **Email marketing:** the buyer's email is added to the matching Brevo list.
5. **Onboarding:** for selected courses, a custom HTML welcome email is
   sent through Gmail.

**Tools:** n8n, Stripe, Google Sheets, Brevo, Gmail

## Result
- ~4.3 hours saved per week (~18 hours per month), based on an estimated
  6.5 minutes of manual work per sale and ~40 sales per week.
- Finance sheet updated automatically, with no typing errors.
- Every buyer is added to the correct Brevo list, and welcome emails go out
  immediately instead of whenever someone remembered.
- Adding a new course takes ~10 minutes (one new branch in the workflow).

## Challenges & What I Learned
- Avoiding duplicate rows when Stripe retries a webhook.
- Testing safely with Stripe test mode before going live.
- Handling a failed Brevo call without losing the sale record.

## Next Steps
- Error alerts (Slack or email) when a step fails.
- Dashboard (Looker Studio or Power BI) on top of the finance sheet.
