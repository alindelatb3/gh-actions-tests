# Update GIJ Home Page UI

## Problem Statement

The GIJ home page underutilizes high-visibility space and lacks a durable, dashboard-oriented structure, reducing its effectiveness as both an onboarding surface and a long-term entry point for delivery insights.


## Intended Outcome

- The GIJ home page serves as a progressively more dashboard-centric landing experience.
- New users are oriented quickly through a dedicated Getting Started section.
- Existing shortcuts remain available but consume less visual space.
- Users can collapse upper sections and use the page primarily as a dashboard if desired.
- A stable Delivery Dashboard container exists to surface Signals insights over time.
- Behavior is consistent across Standard and Advanced plans, with entitlement differences limited to widget interactions:
  - Advanced users navigate into full Signals views.
  - Standard users see a compelling, explanatory upgrade-oriented experience.
- Explanatory empty states are shown when Signals data is unavailable.

## Proposed Scope

- Add three primary sections to the GIJ home page:
  - Getting Started
  - Shortcuts
  - Delivery Dashboard
- Implement collapsible behavior for all sections:
  - Collapsed sections persist across sessions.
  - Collapsed sections remain sticky, allowing the dashboard to dominate the page.
- Build the Delivery Dashboard container to host Signals widgets.
- Add a project selector scoped to the Delivery Dashboard section only.
- Display explanatory empty states when Signals data is unavailable for the selected project.
- Support entitlement-aware interactions at the Delivery Dashboard level:
  - Advanced users link into full Signals views.
  - Standard users see a visually compelling explanation of the value unlocked by upgrading.
- Update the visual layout of existing shortcuts to reduce vertical space without changing functionality.
- Support PMM-driven Getting Started content updates via engineering-delivered changes.
- The Delivery Dashboard section may be conditionally hidden or suppressed until at least one Signals widget is available.
- The section should not render an empty container during the interim period while widgets are still under development.

## Out of Scope

- Implementation of the Risky Work Items widget.
- Implementation of the In-Progress Snapshot widget.
- Widget-level business logic, ranking algorithms, or data sourcing.
- Creation of reusable widget components.
- Widget-level OTel events and instrumentation.
- Changes to left-side navigation or other non-home GIJ surfaces.
- Licensing, entitlement, or pricing changes.
- Definition of event semantics, metric ownership, or analytics dashboards.
- A self-serve CMS, admin UI, or runtime content editing mechanism for PMM or other non-engineering roles.
- Updating Getting Started content without an engineering deploy.
- Displaying an empty or placeholder Delivery Dashboard section before any widgets are implemented.

## Acceptance Criteria

- The GIJ home page contains three clearly labeled sections:
  - Getting Started
  - Shortcuts
  - Delivery Dashboard
- Each section can be collapsed and expanded by the user.
- Collapsed sections persist their state across page loads.
- Collapsed sections remain sticky.
- The Getting Started section supports:
  - Embedded or modal-based YouTube video content.
  - PMM-owned onboarding and educational messaging delivered via engineering changes.
- The Shortcuts section displays existing shortcuts with reduced vertical spacing and unchanged behavior.
- A distinct Delivery Dashboard section exists on the home page.
- The Delivery Dashboard includes a project selector scoped to that section.
- An explanatory empty state is shown when Signals data is unavailable for the selected project.
- Delivery Dashboard interactions respect entitlement:
  - Advanced users navigate into full Signals views.
  - Standard users see an upgrade-oriented explanatory experience.
- The home page layout matches approved Figma designs.
- The Delivery Dashboard section is hidden until Signals widgets are available to render within it.
- Once widgets are implemented, the Delivery Dashboard section becomes visible without requiring additional home page structural changes.

## Success Signal

- Week-over-week increase in interaction with the home page and Delivery Dashboard section.
- Increased discovery and entry into Signals experiences for Advanced users.
- Increased exposure of Signals value for Standard users through dashboard interactions.
- Qualitative feedback indicating improved clarity and usefulness of the GIJ home page as a landing experience.

