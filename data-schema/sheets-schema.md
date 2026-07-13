# Google Sheets Input Schema

The automation reads from a single Google Sheet with three tabs. Each tab acts as a stand-in for a live data source. Rows are filtered to the last 7 days on each run.

---

## Tab 1 — `Notion_Updates`

Represents project status updates (equivalent to a Notion database export or manual log).

| Column | Header | Type | Description |
|---|---|---|---|
| A | `date` | Date (`YYYY-MM-DD`) | Date of the update — used as the 7-day filter |
| B | `project` | Text | Project or workstream name |
| C | `update` | Text | Status update or note (1–3 sentences) |
| D | `author` | Text | Person who logged the update |

**Example row:**

| date | project | update | author |
|---|---|---|---|
| 2026-07-10 | AI Readiness Assessment | Final draft approved by program lead; stakeholder review scheduled July 14. | R. Son |

---

## Tab 2 — `Slack_Messages`

Represents key messages from the leadership Slack channel.

| Column | Header | Type | Description |
|---|---|---|---|
| A | `timestamp` | Datetime (`YYYY-MM-DD HH:MM`) | Message timestamp — used as the 7-day filter |
| B | `channel` | Text | Slack channel name (e.g., `#leadership`) |
| C | `author` | Text | Slack display name of the sender |
| D | `message` | Text | Full message text |

**Example row:**

| timestamp | channel | author | message |
|---|---|---|---|
| 2026-07-09 14:23 | #leadership | A. Torres | Data engineering is at 40% overallocation through end of July — need resource decision before Friday or pipeline integration slips. |

---

## Tab 3 — `Calendar_Events`

Represents key meetings and calendar events for the week.

| Column | Header | Type | Description |
|---|---|---|---|
| A | `date` | Date (`YYYY-MM-DD`) | Event date — used as the 7-day filter |
| B | `event` | Text | Event or meeting name |
| C | `attendees` | Text | Comma-separated list of attendees |
| D | `notes` | Text | Key outcomes, decisions, or action items from the event |

**Example row:**

| date | event | attendees | notes |
|---|---|---|---|
| 2026-07-10 | Sprint Retrospective | R. Son, A. Torres, M. Chen | Closed 3 backlog items ahead of plan; velocity up 18% vs. prior sprint. |

---

## Notes

- All date/time filtering uses `>=` comparison against `now - 7 days` at run time.
- The 7-day window is computed dynamically: `{{formatDate(addDays(now; -7); "YYYY-MM-DD")}}`.
- Empty rows are handled gracefully — if a tab has no rows in the window, the aggregator returns an empty bundle and the LLM returns an empty array for that section.
- Column range is set to `A1:CZ1` to allow for future column expansion without breaking the filter.

---

## Phase 2: Replace Sheets with Live Sources

| Tab | Live Make.com module |
|---|---|
| `Notion_Updates` | **Notion — Search Objects** → filter by last modified date |
| `Slack_Messages` | **Slack — Get Messages** → specify channel + time range |
| `Calendar_Events` | **Google Calendar — Search Events** → filter by date range |

No changes to the LLM or Gmail modules are required for this upgrade.
