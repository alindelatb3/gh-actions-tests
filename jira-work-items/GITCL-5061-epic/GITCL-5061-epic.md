# Risky Work Items Widget

## Problem Statement

Dev leads and PMs lack a fast, default way to identify in-progress work that is stuck or at risk without configuring custom Signals views or filters.


## Intended Outcome

- Dev leads can immediately see which tickets and PRs need attention when they open the Delivery Dashboard.
- Risky work is always prioritized meaningfully, without configuration.
- The widget acts as a triage surface, not a comprehensive risk audit.
- Advanced users can drill into a full Signals view that mirrors the widget.
- Standard users understand what additional insight Signals Advanced would unlock, in direct relation to the risks shown.

## Proposed Scope

- Add a Risky Work Items widget to the Delivery Dashboard.
- Display Jira tickets and PRs that have one or more active Signals risk flags.
- Include only non-closed work items.
- Automatically sort results by risk severity, descending.
- Refresh data on page load and via a manual refresh action.
- Require no configuration to provide value.
- Clearly differentiate Jira tickets and PRs visually.
- Support entitlement-aware behavior:
  - Advanced users navigate to a matching Signals view.
  - Standard users see an upgrade-oriented modal explaining Signals Advanced.
- Include lightweight, inline user feedback collection.
- Include a short explanatory tooltip describing the widget's purpose.
- Row: on long hover, show ticket information card (see Figma) similar to Signals row hovers

## Out of Scope

- Custom filtering or configuration within the widget.
- Historical risk trends or analytics.
- Risk rule creation, editing, or tuning UI.
- Alerts, notifications, or automation.
- Widget placement or dashboard layout decisions.
- Pricing, packaging, or licensing changes.
- Analytics dashboards or reporting tied to widget performance.

## Acceptance Criteria

- The widget displays a list of Jira tickets and/or PRs with one or more active risk flags.
- Only non-closed items are shown.
- Items are sorted by risk severity in descending order.
- Each row displays a primary identifier before the summary:
  - Jira tickets show the issue key, linking to the ticket in a new tab.
  - PRs show the repository name and PR number, linking to the PR in the git provider.
- The summary or PR title is shown after the identifier and truncated as needed.
- One or more abbreviated risk rule tags are displayed.
- Hovering over a risk tag shows additional information about that risk.
- The avatar of the assigned user is shown.
- Jira tickets and PRs are visually distinguishable.
- The user can select how many rows are shown:
  - Default: 5
  - Options: 5, 10, 25, 50
- The widget refreshes automatically on page load.
- A manual refresh control is available.
- Advanced users see a "See full view in Signals" button that opens a matching Signals view showing non-closed items with risks, sorted by severity.
- Standard users see a "See full view in Signals" button that opens an upgrade-oriented modal with:
  - A short explainer video
  - Bullet points explaining the value of Signals Advanced in the context of risky work
- The widget header includes a ? icon.
- Hovering over the ? icon shows a tooltip explaining that the widget surfaces in-progress tickets and PRs with Signals risk flags, sorted by severity.
- The widget includes feedback controls:
  - Prompt: "Did you find this widget helpful?"
  - After selection, a single-field form asking: "Would you be willing to share why?"
- Long-Hovering (1 sec) should show an issue/PR details card

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

- Increased interaction with risky work surfaced via the Delivery Dashboard.
- Increased entry into Signals risk views by Advanced users.
- Engagement with the upgrade modal by Standard users.
- Qualitative feedback indicating earlier identification of blocked or risky work.
