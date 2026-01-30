# Work in Progress Widget

## Problem Statement

Team leads and PMs lack a fast, default way to scan what work is actively moving right now without building filters or dashboards.

## Intended Outcome

- Users can quickly understand what work is actively in flight.
- Recent momentum is visible independently of Jira status.
- The widget surfaces activity, not blockers.
- Advanced users can drill into a full Signals view matching the widget.
- Standard users understand what deeper Signals visibility would unlock.
- The widget delivers value immediately with minimal configuration.

## Proposed Scope

- Add a Work in Progress widget to the Delivery Dashboard.
- Display active Jira tickets with recent commits.
- Sort results by most recent commit activity, descending.
- Refresh data on page load and via a manual refresh action.
- Support entitlement-aware behavior:
  - Advanced users navigate to a matching Signals view.
  - Standard users see an upgrade-oriented modal explaining Signals Advanced.
- Include lightweight, inline user feedback collection.
- Include a short explanatory tooltip describing the widget's purpose.
- Row: on long hover, show ticket information card (see Figma) similar to Signals row hovers
- Display time since most recent commit (1h, 3h, 2d, etc.)

## Out of Scope

- Long-term activity history or analytics.
- Alerts, notifications, or automation.
- Complex filtering or saved custom views.
- Real-time polling or live updates.
- Widget placement or dashboard layout decisions.
- Pricing, packaging, or licensing changes.
- Analytics dashboards or reporting tied to widget performance.

## Acceptance Criteria

- The widget displays a list of Jira tickets.
- Items are sorted by most recent commit
- Each row displays:
  - A primary identifier before the summary:
    - Jira tickets show the issue key.
    - PRs show the repository name and PR number.
  - A truncated summary or PR title.
  - Time since the most recent commit, shown in hours, days, or weeks.
  - The avatar of the assigned user.
- The widget refreshes automatically on page load.
- A manual refresh control is available.
- The user can select how many rows are shown:
  - Default: 5
  - Options: 5, 10, 25, 50
- Advanced users see a "See full view in Signals" button that opens a matching Signals view showing tickets with recent commits, sorted by recency.
- Standard users see a "See full view in Signals" button that opens an upgrade-oriented modal with:
  - A short explainer video
  - Bullet points explaining the value of Signals Advanced in the context of active work visibility
- The widget header includes a ? icon.
- Hovering over the ? icon shows a tooltip explaining that the widget highlights tickets and PRs with recent activity to show current momentum.
- The widget includes feedback controls:
  - Prompt: "Did you find this widget helpful?"
  - After selection, a single-field form asking: "Would you be willing to share why?"
- Long-Hovering (1 sec) should show an issue details card

## Responsive Behavior (Guidance)

The following are preferred behaviors intended to guide responsive implementation, not strict requirements:

- Widgets should adapt fluidly to available width without introducing horizontal scrolling.
- On narrower viewports, widgets should stack vertically rather than compress into unreadable layouts.
- As available width decreases:
  - Visible row count may decrease, even if a higher row count is selected.
  - Summaries may truncate more aggressively.
  - Secondary metadata may be visually de-emphasized.
- Primary identifiers (issue key or repository name + PR number), ordering logic (severity or recency), and primary actions should remain visible at all widths.
- Widgets should enforce a reasonable maximum height and scroll internally rather than expanding the full page indefinitely.

## Success Signal

- Increased use of the Delivery Dashboard for day-to-day monitoring.
- Increased entry into Signals activity views by Advanced users.
- Engagement with the upgrade modal by Standard users.
- Qualitative feedback indicating improved visibility into what work is actively progressing.
