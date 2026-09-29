# JQL cheatsheet for service desks

Replace `HELP` with your project key.

## Backlog

```sql
-- Everything still open this month (status-based: consistent across tools)
project = HELP AND statusCategory != Done AND created >= startOfMonth()

-- Open and actually waiting on IT (exclude approval states)
project = HELP AND statusCategory != Done
  AND status NOT IN ("Waiting for approval", "Waiting for security approval")

-- Only the approval backlog
project = HELP AND status IN ("Waiting for approval", "Waiting for security approval")

-- Unassigned and older than 1 day
project = HELP AND assignee IS EMPTY AND statusCategory != Done AND created <= -1d
```

> 💡 Pick **one** definition of "open" and use it everywhere: dashboards, bots, reports. `resolution = Unresolved` often over-counts compared with status-based filters.

## SLA (Jira Service Management)

```sql
-- Breached, still open
project = HELP AND "Time to resolution" = breached() AND statusCategory != Done

-- Will breach within 4 hours
project = HELP AND "Time to resolution" < remaining("4h") AND statusCategory != Done

-- Resolved within SLA this month
project = HELP AND "Time to resolution" = everBreached() = false AND resolved >= startOfMonth()
```

## Counting per day

```sql
-- Tickets that LEFT a status on a given day (e.g. approvals actioned)
project = HELP AND status CHANGED FROM "Waiting for security approval"
  DURING ("2026-09-01 00:00", "2026-09-02 00:00")
```

> ⚠️ Use explicit `00:00` boundaries. Date-only `DURING ("2026-09-01","2026-09-02")` can double-count the shared boundary day when you loop over days.

## Hygiene

```sql
-- Stale: no update for 5 days
project = HELP AND statusCategory != Done AND updated <= -5d ORDER BY updated ASC

-- Reopened this week
project = HELP AND status CHANGED FROM Resolved AFTER startOfWeek()
```
