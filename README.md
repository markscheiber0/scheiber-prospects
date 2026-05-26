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

## Adding industries or geography

Edit the routine prompt at [claude.ai/code/routines](https://claude.ai/code/routines). The full setup doc lives outside this repo.

## Future

When the prospect list outgrows a CSV, this same data syncs to HubSpot via a HubSpot connector added to the routine. The CSV remains the source of truth; HubSpot becomes the working surface.
