# scheiber-prospects

Working prospect tracker for **Scheiber Intelligence**. The single source of truth lives in `prospects.csv`. A daily Claude Code Routine appends new rows; manual edits to existing rows are respected on subsequent runs.

## How it works

A scheduled Claude Code Routine runs every morning at 7:00 AM MT and:

1. Clones this repo
2. Reads `prospects.csv` to dedupe
3. Searches for new landscaping prospects in Ada and Canyon Counties, Idaho
4. Appends new rows
5. Commits and pushes to `main`
6. Emails a briefing to workscheiber@gmail.com

The routine is configured at [claude.ai/code/routines](https://claude.ai/code/routines).

## CSV schema

| Column | Notes |
|---|---|
| `date_added` | YYYY-MM-DD when first surfaced |
| `last_updated` | YYYY-MM-DD of most recent edit |
| `business_name` | Legal name preferred, otherwise DBA |
| `owner_name` | If findable from public sources |
| `industry` | "Landscaping" for now |
| `city` | City within Ada or Canyon County |
| `county` | "Ada" or "Canyon" |
| `state` | "ID" |
| `phone` | Primary phone |
| `email` | Public email only, never guessed |
| `website_url` | URL or "none" |
| `website_status` | none / facebook-only / poor / decent / polished |
| `signal_source` | SOS / hiring / news / no-website / maps |
| `signal_detail` | One-line description of what surfaced them |
| `fit_note` | One-line "why they're a fit for SI" |
| `status` | new / drafted / contacted / qualified / not-fit / dead |
| `notes` | Anything else worth keeping |
| `outreach_method` | email / phone / facebook / none |
| `last_contacted` | YYYY-MM-DD of last outreach actually sent (blank until sent) |
| `outreach_count` | Number of outreach touches sent so far |
| `next_followup` | YYYY-MM-DD to follow up, if scheduled |

## Working a prospect

When you contact, qualify, or disqualify a prospect, edit the row directly:

1. Open `prospects.csv` on GitHub
2. Click the pencil icon
3. Update `status` and `last_updated`, add anything relevant to `notes`
4. Commit

The routine only appends new rows. It does not overwrite your manual edits.

## Outreach workflow

Cold-email outreach is draft-first: a personalized email is created as a Gmail **draft** for each prospect that has an email address, and the row is set to `status = drafted`. Nothing is sent automatically.

When you send a draft:

1. Set `status` to `contacted`
2. Set `last_contacted` to today and bump `outreach_count`
3. Set `next_followup` if you want a reminder
4. Update `last_updated` and commit

Prospects without an email (`phone` / `facebook-only` only) can't be emailed yet; reach them by phone or find an email first.

## Outreach drafting routine

A second scheduled Routine runs each morning right after the prospecting routine and prepares a review-and-send queue. It **creates Gmail drafts only and never sends**. Each draft is personalized to that prospect's signal. Mark checks Drafts each morning and sends the ones he wants.

It is idempotent and safe to re-run: it drafts at most once per prospect (only `status = new` rows with an email), checks Gmail first so it never duplicates an existing draft, and skips rows flagged for manual review.

Configure it at [claude.ai/code/routines](https://claude.ai/code/routines) on a schedule shortly after the 7:00 AM prospecting run (e.g. 7:30 AM MT), with the Gmail integration and GitHub repo access enabled. Routine prompt:

```
You are drafting cold-outreach emails for Scheiber Intelligence (SI), which builds
modern websites and AI phone agents / chatbots for landscaping companies in Ada and
Canyon County, Idaho. This routine runs each morning after the prospecting routine
appends new leads. It creates Gmail DRAFTS ONLY and NEVER sends. Mark reviews the
drafts and sends them by hand.

Repository: markscheiber0/scheiber-prospects (branch: main). Source of truth is
prospects.csv.

STEPS

1. Pull the latest main branch and read prospects.csv.

2. Ensure these outreach columns exist; if any are missing, add them to the header
   and backfill empty values for existing rows without disturbing other data:
   outreach_method, last_contacted, outreach_count, next_followup.
   Full column order:
   date_added,last_updated,business_name,owner_name,industry,city,county,state,
   phone,email,website_url,website_status,signal_source,signal_detail,fit_note,
   status,notes,outreach_method,last_contacted,outreach_count,next_followup

3. Build the eligible list: rows where ALL are true:
   - email is non-empty
   - status is "new" or empty
   - NOT flagged for manual review. Treat a row as flagged if notes or signal_detail
     contains any of: "flag for manual review", "confirm before outreach",
     "license inactive", "inactive license", "dispute", "small claims".
     Skip flagged rows, leave them status "new", and list them in the run summary.

4. For each eligible prospect, search Gmail BEFORE drafting to avoid duplicates
   (query to:THEIR_EMAIL):
   - If a draft to that address already exists, do not create another; just set the
     CSV status to "drafted".
   - If a SENT message to that address exists, set status to "contacted",
     last_contacted to the sent date, outreach_count to 1, and do not draft.
   - Otherwise, draft as below.

5. Write a short personalized cold email and create it as a Gmail draft to the
   prospect's email. Rules (follow exactly):
   - NEVER use em dashes or en dashes. Use periods or commas, or restructure.
   - Lead with one concrete observation from that row's signal_detail / website_status
     / fit_note (broken site, http-only, Facebook-only, template site, GreenPal
     dependence, hiring/expanding, etc.). Make it specific to them.
   - Frame SI's help concretely: a modern website and/or an AI phone agent or chatbot
     that books work while they are on the job. Avoid generic "AI" buzzwords.
   - Include near the end: "Happy to talk through other ways AI could save you time and
     money."
   - End with a low-friction reply CTA, e.g. 'Worth a quick reply? Even a "tell me more"
     or "not right now" tells me whether to follow up.'
   - Greet by owner_name first name if present, otherwise "Hi there,".
   - Do NOT include any opt-out, unsubscribe, or "no thanks" line.
   - Sign off:
       Thanks,
       Mark Scheiber
       Scheiber Intelligence
       mark.scheiber@scheiberintelligence.com
   - Subject: short and specific. With a website: "Quick idea for {business}'s website".
     Without a real website: "Idea for {business}'s online presence".
   - Keep it to about 5-7 short lines. One email only.

6. Update that CSV row:
   - status = "drafted" (or "contacted" if a sent message already existed)
   - outreach_method = "email"
   - outreach_count = 0 for drafted (1 if already sent)
   - last_updated = today (M/D/YYYY)
   - append to notes (semicolon-separated, do not overwrite):
     "Cold email draft created {today}; pending review and send."

7. Commit the updated prospects.csv and push to main with a message like
   "Draft outreach for N new prospects" and include in the commit body the business
   names drafted and any rows skipped for manual review.

GUARDRAILS
- Create drafts only. Never send outreach to a prospect.
- One email per prospect, ever. Never draft for a prospect whose status is already
  drafted, contacted, qualified, not-fit, or dead.
- Skip prospects with no email.
- Do not create a separate summary email (keep the Drafts folder focused on outreach).
- Do not modify rows except to set the outreach fields above.
```

## Adding industries or geography

Edit the routine prompt at [claude.ai/code/routines](https://claude.ai/code/routines). The full setup doc lives outside this repo.

## Future

When the prospect list outgrows a CSV, this same data syncs to HubSpot via a HubSpot connector added to the routine. The CSV remains the source of truth; HubSpot becomes the working surface.
