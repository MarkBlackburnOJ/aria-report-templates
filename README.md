# ARIA report templates

The six HTML templates the **ARIA Report Engine** renders its reports from. They are fetched by **raw URL at
render time**, on every single run — so a change here is live on the very next report, with no deploy and no
restart, and a broken one is live just as fast.

| Template | Report |
|---|---|
| `template.html` | Store report V1 — the detailed desktop view |
| `templateV2.html` | Store report V2 — the mobile action punch-list |
| `am-report.html` | Account-Manager portfolio + per-client reports (all four channels) |
| `engagement.html` | PNP Store Reports Engagement — open-tracking roll-up |
| `daily-action.html` | Action Effectiveness |
| `weekly-rep.html` | Weekly Rep Report |

Each contains a literal `__PAYLOAD__` token that the engine replaces with the report's JSON. They therefore
cannot be opened standalone — `const DATA = __PAYLOAD__;` is not valid JavaScript until it is rendered.

## Why this repo exists

**This repo is public on purpose, and must stay that way**: the engine fetches these files with an
unauthenticated HTTP request, so making it private breaks every report immediately.

It was split out of `MarkBlackburnOJ/StoreReports` on **2026-08-05**. That repo previously held both the
templates *and* every published report. Report hosting moved to Cloudflare R2 behind Entra SSO on 14 July
2026, but the historical report files remained readable in the public repo's git history — including copies
predating the recipient-redaction change, which carried real rep and staff email addresses. Deleting a file
from a public repo does not remove the blob, so the only real fix was to make that repo private, which in
turn required the templates to live somewhere else.

**Nothing but templates belongs here.** No rendered reports, no payloads, no recipient lists. This repo is
public, permanently, and anything committed to it is public permanently — including after deletion.

## Editing

A commit to `main` is a production deployment. There is no staging step and no review gate.

For anything larger than a one-line change, work on a branch and point the engine at it for a single run
using a process environment variable — never a committed file — pinning the **commit SHA** rather than the
branch, because `raw.githubusercontent.com` caches branch URLs for several minutes and will serve you the
previous version:

```powershell
$env:ARIA_Aria__Feedback__EngagementTemplateUrl =
  'https://raw.githubusercontent.com/MarkBlackburnOJ/aria-report-templates/<SHA>/engagement.html'
dotnet run --project src/Aria.ReportEngine -- feedback-rollup-once --dry-run
```

Then merge to `main` to promote, or revert to roll back. Both take effect on the next run.

Full context, including the payload contract each template consumes, is in `HANDOVER.md` in the
`aria-report-engine` repo.
