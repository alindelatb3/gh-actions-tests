# GITCL-5061 Epic: Risky Work Items Widget - Implementation Tasks

## Epic Summary
Build a triage-focused widget for the Delivery Dashboard that surfaces Jira tickets and PRs with active Signals risk flags, sorted by severity, with entitlement-aware behavior for Advanced and Standard users.

---

## Task Breakdown

### Phase 1: Backend API & Data Layer

#### Task 1.1: Design Risky Work Items Data Model
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/main/java/com/bigbrassband/gitforjiracloud/riskywork/`
**Estimated Effort:** 3-5 days

**Description:**
Design the data model for representing risky work items with risk flags and severity information.

**Implementation Details:**
- Create Java classes:
  - `RiskyWorkItem.java` - Base class for risky work representation
    - Fields: `id`, `type` (ISSUE/PR), `key/identifier`, `summary`, `assigneeAvatarUrl`, `riskFlags[]`, `severityScore`, `lastUpdated`
  - `RiskFlag.java` - Individual risk flag representation
    - Fields: `riskRuleId`, `riskRuleName`, `abbreviation`, `description`, `severity`, `detectedAt`
  - `RiskSeverity.java` - Enum for severity levels (HIGH, MEDIUM, LOW)
  - `RiskyWorkItemType.java` - Enum (JIRA_ISSUE, PULL_REQUEST)
- Define severity calculation logic:
  - Multiple HIGH = Critical
  - One HIGH or multiple MEDIUM = High
  - One MEDIUM or multiple LOW = Medium
  - One LOW = Low
- Add JavaDoc documentation

**Acceptance Criteria:**
- Data model classes created with all required fields
- Severity calculation logic implemented
- Unit tests for severity calculation
- Classes use proper null safety annotations (@Nonnull, @Nullable)
- JavaDoc documentation complete

**Dependencies:** None

---

#### Task 1.2: Create Risky Work Items Query Service
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/main/java/com/bigbrassband/gitforjiracloud/riskywork/`
**Estimated Effort:** 7-10 days

**Description:**
Implement service layer to query and aggregate risky work items from existing Signals data sources.

**Implementation Details:**
- Create `RiskyWorkItemService.java`:
  - Method: `List<RiskyWorkItem> getRiskyWorkItems(String installId, @Nullable String projectKey, int limit)`
  - Query Signals risk data (from Team Insights integration or local storage)
  - Filter for non-closed items only
  - Aggregate risk flags per work item
  - Calculate severity scores
  - Sort by severity descending
  - Apply limit
- Create `RiskyWorkItemQueryBuilder.java` for flexible querying:
  - Support filtering by project
  - Support filtering by risk types
  - Support pagination
- Integration with Team Insights API:
  - Call TIJ endpoints to get risk data
  - Map TIJ risk responses to GIJ data model
  - Handle TIJ unavailability gracefully
- Add caching layer:
  - Cache results for 5 minutes
  - Invalidate on manual refresh
  - Use Guava Cache or similar
- Error handling and logging

**Acceptance Criteria:**
- Service returns risky work items correctly
- Items are sorted by severity
- Only non-closed items returned
- Project filtering works
- Caching improves performance
- Graceful degradation if TIJ unavailable
- Unit tests with mocked dependencies
- Integration tests with test data

**Dependencies:** Task 1.1

---

#### Task 1.3: Create REST API Endpoint for Risky Work Items
**Repository:** `jiragitcloud`
**Location:** `docs/api/v1/` (YAML), `appserver/gitforjiracloud/` (Java)
**Estimated Effort:** 5-7 days

**Description:**
Create REST API endpoint to expose risky work items data to the frontend.

**Implementation Details:**
- Create YAML API definition `docs/api/v1/apiV1GetRiskyWorkItems.yaml`:
```yaml
url: /api/v1/risky-work-items
methods: [GET]
parameters:
  - {field: installId, type: string, source: jwt}
  - {field: projectKey, type: string, source: query, required: false}
  - {field: limit, type: integer, source: query, required: false, default: 5}
tasks:
  - getRiskyWorkItems: {}
```
- Create `GetRiskyWorkItemsTask.java` in `appserver/apiendpoint/tasks/`:
  - Call `RiskyWorkItemService`
  - Convert to JSON response
  - Handle errors gracefully
  - Add OpenTelemetry tracing
  - Add Ticker logging
- Response format:
```json
{
  "items": [
    {
      "id": "PROJ-123",
      "type": "JIRA_ISSUE",
      "key": "PROJ-123",
      "summary": "Fix authentication bug",
      "assigneeAvatarUrl": "https://...",
      "riskFlags": [
        {
          "riskRuleId": "pr-aging",
          "name": "Aging Pull Request",
          "abbreviation": "Aging PR",
          "description": "Pull request has been open for >7 days",
          "severity": "HIGH"
        }
      ],
      "severityScore": 3,
      "lastUpdated": "2026-01-29T10:00:00Z"
    }
  ],
  "totalCount": 15,
  "hasMore": true
}
```
- Add rate limiting if needed
- Add authorization checks (user must have access to project)

**Acceptance Criteria:**
- API endpoint accessible via defined URL
- Returns correctly formatted JSON
- Respects query parameters (projectKey, limit)
- Authorization checks work correctly
- Error responses are proper HTTP codes
- OpenTelemetry spans created
- API documentation updated
- Postman/curl examples provided

**Dependencies:** Task 1.2

---

#### Task 1.4: Create Signals Risk Data Integration
**Repository:** `jiragitcloud` and potentially `insights-for-jira-cloud`
**Location:** Various
**Estimated Effort:** 10-14 days

**Description:**
Integrate with Team Insights for Jira (Signals) to retrieve risk flag data, or implement local risk detection if TIJ integration not ready.

**Implementation Details:**
- **Option A: TIJ Integration (preferred)**
  - Call TIJ REST API endpoints for risk data
  - Create `TijRiskApiClient.java` client class
  - Map TIJ risk data to GIJ data model
  - Handle authentication between apps
  - Cache risk data appropriately

- **Option B: Local Risk Detection (fallback)**
  - Implement basic risk rules locally in GIJ:
    - Aging PR (PR open > 7 days)
    - Large PR (files changed > 50)
    - Stale branch (no commits > 14 days)
    - Multiple reviewers needed
  - Store risk calculations in PostgreSQL
  - Run risk detection during indexing
  - Create `LocalRiskDetectionService.java`

- Common tasks:
  - Feature flag to switch between TIJ and local
  - Fallback mechanism if TIJ unavailable
  - Error handling and retry logic
  - Comprehensive logging

**Acceptance Criteria:**
- Risk data successfully retrieved from chosen source
- Data mapping is accurate
- Fallback mechanism works
- Feature flag controls integration method
- Performance is acceptable (<500ms p95)
- Error handling is robust
- Integration tests validate data flow

**Dependencies:** Task 1.1, Task 1.2

---

### Phase 2: Frontend Widget Component

#### Task 2.1: Create Risky Work Widget Container
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 5-7 days

**Description:**
Build the main widget container component with header, controls, and layout.

**Implementation Details:**
- Create `RiskyWorkWidget.tsx`:
  - Widget header with title and info icon
  - Row count selector (5, 10, 25, 50)
  - Refresh button
  - "See full view in Signals" button
  - Loading state
  - Error state
  - Empty state
  - Widget body container for rows
- Create `RiskyWorkWidget.module.scss` for styling:
  - Match Figma designs
  - Responsive layout
  - Maximum height with internal scrolling
- Create `RiskyWorkWidgetContext.tsx` for state management:
  - Selected row count
  - Refresh trigger
  - Loading state
  - Error state
- Add tooltip for help icon:
  - "This widget surfaces in-progress tickets and PRs with Signals risk flags, sorted by severity."

**Acceptance Criteria:**
- Widget container renders correctly
- Header controls work (row selector, refresh)
- Layout matches Figma designs
- Loading/error/empty states display properly
- Context provides state to child components
- Responsive design works
- Accessibility compliant

**Dependencies:** None (can start in parallel with backend)

---

#### Task 2.2: Create Risky Work Item Row Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/components/`
**Estimated Effort:** 7-10 days

**Description:**
Build the row component that displays individual risky work items.

**Implementation Details:**
- Create `RiskyWorkItemRow.tsx`:
  - Primary identifier (issue key or repo name + PR #)
    - For Jira: Clickable link to Jira issue (new tab)
    - For PR: Clickable link to git provider (new tab)
  - Summary/PR title (truncated with ellipsis)
  - Risk flag tags (abbreviated)
  - Assignee avatar
  - Visual distinction between Jira issues and PRs (icon or color)
- Create `RiskFlagTag.tsx` component:
  - Abbreviated risk rule name
  - Color coding by severity (red=high, orange=medium, yellow=low)
  - Hover tooltip with full risk description
- Create `risky-work-item-row.module.scss`:
  - Grid layout for columns
  - Hover effect
  - Truncation with ellipsis
  - Responsive adjustments
- Handle click events:
  - Open links in new tab
  - Track telemetry on clicks
- Visual indicators:
  - Jira icon for issues
  - Git branch/PR icon for pull requests

**Acceptance Criteria:**
- Row displays all required information
- Links work correctly (new tab)
- Jira issues and PRs are visually distinct
- Risk tags display with proper colors
- Hover tooltips show risk details
- Summary text truncates properly
- Avatar displays correctly
- Responsive design works
- Accessible markup

**Dependencies:** Task 2.1

---

#### Task 2.3: Implement Hover Card for Work Items
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/components/`
**Estimated Effort:** 7-10 days

**Description:**
Create information card that appears on long hover (1 second) showing detailed ticket/PR information.

**Implementation Details:**
- Create `RiskyWorkItemHoverCard.tsx`:
  - Trigger on 1-second hover over row
  - Display detailed information:
    - Full summary/title
    - Description preview
    - Status
    - Assignee name
    - All risk flags with descriptions
    - Last updated timestamp
    - Project name
  - Position card near cursor or row
  - Smooth fade-in animation
  - Close on mouse leave or click outside
- Create `useHoverCard.ts` hook:
  - Manage hover timer (1 second delay)
  - Handle mouse enter/leave events
  - Cancel timer on mouse leave before 1 second
  - Position calculation logic
- Fetch additional data if needed:
  - Call API for full issue/PR details
  - Cache fetched data
  - Handle loading state
- Create `risky-work-item-hover-card.module.scss`:
  - Card styling (shadow, border, padding)
  - Animation
  - Responsive sizing

**Acceptance Criteria:**
- Card appears after 1-second hover
- Card shows detailed information
- Card disappears on mouse leave
- Hover timer cancels properly
- Card positioning works correctly
- Smooth animations
- Data fetching works
- Accessible (keyboard navigation shows card on focus)
- Works on touch devices (tap to show)

**Dependencies:** Task 2.2

---

#### Task 2.4: Implement Row Count Selector
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/components/`
**Estimated Effort:** 2-3 days

**Description:**
Create dropdown/selector for choosing how many rows to display (5, 10, 25, 50).

**Implementation Details:**
- Create `RowCountSelector.tsx`:
  - Dropdown component using Atlassian Design System
  - Options: 5, 10, 25, 50
  - Default: 5
  - Persist selection in localStorage
  - Update widget context on change
- Create `useRowCountSelector.ts` hook:
  - Manage selected count state
  - Persist to localStorage
  - Provide to widget context
- Add telemetry:
  - Track row count changes
  - Track default vs custom selections
- Handle responsiveness:
  - On narrow viewports, may show fewer rows even if higher count selected
  - Show message if limiting due to viewport

**Acceptance Criteria:**
- Selector displays all options
- Selection updates widget display
- Selection persists across page loads
- Telemetry tracks changes
- Responsive behavior works
- Accessible

**Dependencies:** Task 2.1

---

#### Task 2.5: Implement Widget Refresh Functionality
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 3-4 days

**Description:**
Implement manual and automatic refresh capabilities for the widget.

**Implementation Details:**
- Create refresh button in widget header:
  - Icon button (circular arrow)
  - Click to trigger refresh
  - Disabled state during loading
  - Spin animation during refresh
- Implement refresh logic:
  - Invalidate cache
  - Re-fetch data from API
  - Update widget state
  - Show loading indicator
- Auto-refresh on page load:
  - Fetch data when widget mounts
  - Respect cached data (TTL 5 minutes)
- Error handling:
  - Show error state if refresh fails
  - Retry mechanism (with exponential backoff)
  - User-friendly error messages
- Add telemetry:
  - Track manual refresh clicks
  - Track auto-refresh triggers
  - Track refresh errors

**Acceptance Criteria:**
- Refresh button works
- Data updates on refresh
- Auto-refresh on page load
- Loading state during refresh
- Error handling works
- Telemetry tracks refreshes
- Button disabled during loading

**Dependencies:** Task 2.1

---

### Phase 3: Entitlement & Integration Features

#### Task 3.1: Implement Entitlement-Aware CTA Button
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 5-7 days

**Description:**
Create "See full view in Signals" button with behavior based on user entitlement (Advanced vs Standard).

**Implementation Details:**
- Create `SignalsViewButton.tsx`:
  - Button text: "See full view in Signals"
  - Different behavior for Advanced vs Standard users
  - Use existing `useIsSignalsEnabled` and entitlement hooks
- **Advanced Users Behavior:**
  - Click opens Signals view in new route/page
  - Navigate to Signals with filters pre-applied:
    - Show only non-closed items
    - Show only items with risks
    - Sort by severity descending
    - Preserve selected project filter
  - Construct URL with query parameters for filters
- **Standard Users Behavior:**
  - Click opens upgrade modal (see Task 3.2)
  - Button may have visual indicator (e.g., "Upgrade" badge)
- Add telemetry:
  - Track button clicks (separate events for Advanced/Standard)
  - Track navigation success
  - Track modal opens

**Acceptance Criteria:**
- Button renders for both user types
- Advanced users navigate to Signals correctly
- Filters are applied in Signals view
- Standard users see upgrade modal
- Telemetry tracks all interactions
- Visual design matches Figma

**Dependencies:** Task 2.1

---

#### Task 3.2: Create Upgrade Modal for Standard Users
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/components/`
**Estimated Effort:** 7-10 days

**Description:**
Build compelling upgrade-oriented modal for Standard users explaining Signals Advanced value.

**Implementation Details:**
- Create `SignalsUpgradeModal.tsx`:
  - Modal overlay with close button
  - Header: "Unlock Deeper Insights with Signals Advanced"
  - Embedded explainer video (YouTube or similar):
    - Demo of Signals Advanced features
    - PMM-provided content
    - Lazy load video
  - Bullet points explaining value:
    - "Identify risky work before it becomes a problem"
    - "Deep dive into all risk factors and trends"
    - "Custom risk rules and alerts"
    - "Historical analytics and reporting"
  - CTA buttons:
    - Primary: "Upgrade to Advanced" (link to marketplace or sales)
    - Secondary: "Learn More" (link to product page)
    - Tertiary: "Close" or X button
  - Option to "Don't show again" (persist in localStorage)
- Create `signals-upgrade-modal.module.scss`:
  - Modal styling
  - Responsive design
  - Video embed styling
  - CTA button styling
- Add telemetry:
  - Track modal opens
  - Track video plays
  - Track CTA clicks (upgrade, learn more)
  - Track "don't show again" selections
  - Track dismissals without action

**Acceptance Criteria:**
- Modal displays correctly
- Video loads and plays
- All CTAs work correctly
- "Don't show again" persists
- Responsive design works
- Accessible (keyboard navigation, focus trap)
- Telemetry tracks all interactions
- Visual design matches Figma

**Dependencies:** Task 3.1

---

#### Task 3.3: Integrate Widget with Delivery Dashboard
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/`
**Estimated Effort:** 3-5 days

**Description:**
Integrate the Risky Work Widget into the Delivery Dashboard section from GITCL-4879.

**Implementation Details:**
- Update `DeliveryDashboard.tsx`:
  - Import `RiskyWorkWidget` component
  - Add widget to dashboard layout
  - Pass dashboard context (selected project) to widget
  - Handle widget visibility based on feature flag
- Update `DeliveryDashboardContext.tsx`:
  - Add risky work widget state if needed
  - Provide selected project to widget
- Widget positioning:
  - Determine placement in dashboard grid
  - Ensure responsive layout
  - Handle multiple widgets layout (future-proofing)
- Feature flag:
  - Add `riskyWorkWidget.enabled` flag
  - Control widget visibility
  - Allow gradual rollout
- Update dashboard empty state:
  - Show appropriate message if widget has no data

**Acceptance Criteria:**
- Widget renders in dashboard
- Widget receives selected project from context
- Feature flag controls visibility
- Layout looks correct
- Responsive design works
- No conflicts with other dashboard components

**Dependencies:** GITCL-4879 Task 4.1 (Delivery Dashboard Container), Task 2.1

---

#### Task 3.4: Implement Project Filter Integration
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 3-5 days

**Description:**
Connect widget to dashboard project selector to filter risky work by selected project.

**Implementation Details:**
- Update `RiskyWorkWidget.tsx`:
  - Subscribe to dashboard context for selected project
  - Pass project key to API query
  - Re-fetch data when project changes
  - Show loading state during project change
  - Handle "All Projects" selection (no project filter)
- Create `useRiskyWorkData.ts` hook:
  - Fetch data from API endpoint
  - Accept project key parameter
  - Handle loading, error, data states
  - Implement SWR or React Query for caching
  - Auto-refetch on project change
- Add telemetry:
  - Track project filter changes
  - Track data fetch by project
- Update empty state:
  - Show project-specific empty message
  - "No risky work found for [Project Name]"

**Acceptance Criteria:**
- Widget data filters by selected project
- Data refetches on project change
- Loading state during refetch
- "All Projects" shows unfiltered data
- Empty state is project-aware
- Telemetry tracks project filters
- Performance is good (caching works)

**Dependencies:** Task 2.1, GITCL-4879 Task 4.2 (Project Selector)

---

### Phase 4: Polish & User Experience

#### Task 4.1: Implement User Feedback Collection
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/components/`
**Estimated Effort:** 5-7 days

**Description:**
Add lightweight inline feedback collection to gather user input on widget usefulness.

**Implementation Details:**
- Create `WidgetFeedback.tsx` component:
  - Display in widget footer or bottom
  - Initial prompt: "Did you find this widget helpful?"
  - Thumbs up/down buttons (or Yes/No)
  - On selection, expand to show follow-up question
  - Follow-up: "Would you be willing to share why?" with text input
  - Submit button
  - Thank you message after submission
  - Collapse after submission or dismissal
- Create `useWidgetFeedback.ts` hook:
  - Manage feedback state
  - Submit feedback to backend API
  - Track submission status
  - Persist "already answered" state in localStorage
- Create backend endpoint for feedback:
  - API: `POST /api/v1/widget-feedback`
  - Store in database or send to analytics service
  - Include widget type, helpful (yes/no), comment, user info
- Add telemetry:
  - Track feedback prompts shown
  - Track responses (helpful/not helpful)
  - Track text submissions
  - Track dismissals
- Design considerations:
  - Non-intrusive placement
  - Show after user has interacted with widget
  - Don't show again if user already responded
  - Option to dismiss permanently

**Acceptance Criteria:**
- Feedback component renders in widget
- Responses submit successfully
- Follow-up question appears on initial response
- Thank you message displays
- "Already answered" state persists
- Telemetry tracks all interactions
- Backend stores feedback data
- Non-intrusive UX

**Dependencies:** Task 2.1

---

#### Task 4.2: Implement Responsive Behavior
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 5-7 days

**Description:**
Ensure widget adapts gracefully to different viewport sizes without horizontal scrolling.

**Implementation Details:**
- Test widget at breakpoints:
  - Mobile: < 768px
  - Tablet: 768px - 1024px
  - Desktop: > 1024px
  - Wide: > 1440px
- Responsive adjustments:
  - **Mobile (<768px):**
    - Stack widget header controls vertically
    - Reduce visible row count to 3-5 even if higher selected
    - Truncate summaries more aggressively
    - Hide less critical metadata (e.g., some risk tags)
    - Maintain primary identifier and assignee avatar
  - **Tablet (768px-1024px):**
    - Moderate truncation
    - Show most metadata
    - Adjust grid columns
  - **Desktop (>1024px):**
    - Full layout
    - All metadata visible
- Implement maximum height:
  - Widget scrolls internally
  - Prevent infinite page expansion
  - Max height ~600px, then scroll
- Update `risky-work-widget.module.scss`:
  - Media queries for each breakpoint
  - Flexible grid layouts
  - Responsive typography
  - Appropriate truncation
- Test on real devices:
  - iOS Safari
  - Android Chrome
  - Desktop browsers

**Acceptance Criteria:**
- No horizontal scrolling at any width
- Widget remains functional at all breakpoints
- Primary information visible at all widths
- Internal scrolling works
- Touch interactions work on mobile
- Visual design maintains quality
- Matches Figma responsive specs

**Dependencies:** Task 2.1, Task 2.2

---

#### Task 4.3: Add Loading Skeletons and States
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/components/`
**Estimated Effort:** 3-4 days

**Description:**
Create polished loading states with skeleton screens for better perceived performance.

**Implementation Details:**
- Create `RiskyWorkItemSkeleton.tsx`:
  - Skeleton row matching real row layout
  - Animated shimmer effect
  - Show multiple skeleton rows (matching selected count)
- Create `WidgetErrorState.tsx`:
  - Error icon
  - User-friendly error message
  - Retry button
  - Support for different error types:
    - Network error
    - Authorization error
    - No data available
    - Signals unavailable
- Create `WidgetEmptyState.tsx`:
  - Positive messaging
  - "No risky work found" message
  - Illustration or icon
  - Suggestion to configure Signals or connect repos
- Update widget logic:
  - Show skeleton during initial load
  - Show skeleton during refresh (optional, or show stale data)
  - Show error state on failures
  - Show empty state when no data
- Add retry logic:
  - Retry button in error state
  - Exponential backoff for automatic retries
  - Track retry attempts

**Acceptance Criteria:**
- Skeleton loading state displays correctly
- Error state shows appropriate messages
- Empty state is encouraging and helpful
- Retry functionality works
- Transitions between states are smooth
- All states match visual design

**Dependencies:** Task 2.1

---

### Phase 5: Testing & Documentation

#### Task 5.1: Write Unit Tests for Widget Components
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/**/*.test.tsx`
**Estimated Effort:** 7-10 days

**Description:**
Create comprehensive unit test suite for all widget components and logic.

**Implementation Details:**
- Test coverage for:
  - `RiskyWorkWidget.tsx` - Container logic
  - `RiskyWorkItemRow.tsx` - Row rendering and interactions
  - `RiskyWorkItemHoverCard.tsx` - Hover behavior
  - `RiskFlagTag.tsx` - Tag rendering
  - `RowCountSelector.tsx` - Selection logic
  - `SignalsViewButton.tsx` - Entitlement-aware behavior
  - `SignalsUpgradeModal.tsx` - Modal interactions
  - `WidgetFeedback.tsx` - Feedback submission
  - All hooks (`useRiskyWorkData`, `useHoverCard`, `useRowCountSelector`, etc.)
- Use React Testing Library
- Mock dependencies:
  - API calls
  - Dashboard context
  - Entitlement hooks
  - localStorage
  - Telemetry
- Test scenarios:
  - Data loading and display
  - Error handling
  - Empty states
  - User interactions (clicks, hovers, selections)
  - Entitlement differences
  - Responsive behavior (viewport resizing)
- Aim for >85% code coverage

**Acceptance Criteria:**
- All components have unit tests
- All hooks have unit tests
- Code coverage >85%
- Tests pass in CI/CD
- Edge cases covered
- Mocks are appropriate and maintainable

**Dependencies:** All Phase 2, 3, 4 implementation tasks

---

#### Task 5.2: Write Integration Tests
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/**/*.integration.test.tsx`
**Estimated Effort:** 5-7 days

**Description:**
Create integration tests for widget workflows with dashboard and API.

**Implementation Details:**
- Test scenarios:
  - Full widget lifecycle (mount → load → display → interact)
  - Integration with delivery dashboard
  - Project selector filter integration
  - API data fetching and caching
  - Entitlement-based rendering (Advanced vs Standard users)
  - Error recovery flows
  - Feedback submission flow
- Use MSW (Mock Service Worker) for API mocking
- Test with different data scenarios:
  - Empty results
  - Single item
  - Multiple items
  - Mixed Jira issues and PRs
  - Various risk severities
- Test state management across components
- Verify telemetry events fire correctly

**Acceptance Criteria:**
- All major workflows tested
- API integration tested with mocks
- Dashboard integration verified
- Entitlement scenarios covered
- Tests are reliable and not flaky
- All tests pass in CI/CD

**Dependencies:** Task 5.1

---

#### Task 5.3: Write E2E Tests with Playwright
**Repository:** `jiragitcloud`
**Location:** `ui/code/tests/e2e/risky-work-widget/`
**Estimated Effort:** 5-7 days

**Description:**
Create end-to-end tests for complete user journeys with the widget.

**Implementation Details:**
- Create test files:
  - `risky-work-widget-display.spec.ts` - Widget rendering and data display
  - `risky-work-widget-interactions.spec.ts` - User interactions
  - `risky-work-widget-entitlement.spec.ts` - Entitlement behaviors
  - `risky-work-widget-responsive.spec.ts` - Responsive design
- Test scenarios:
  - Load home page and see widget in dashboard
  - Widget displays risky work items
  - Click on issue/PR link (opens in new tab)
  - Hover over risk tag (tooltip appears)
  - Long hover on row (info card appears)
  - Change row count selector
  - Click refresh button
  - Select different project (data updates)
  - Advanced user clicks "See full view" (navigates to Signals)
  - Standard user clicks "See full view" (modal opens)
  - Submit feedback
- Use Playwright fixtures for test data
- Test across browsers (Chromium, Firefox, WebKit)
- Add visual regression tests (screenshots)

**Acceptance Criteria:**
- All critical user paths tested
- Tests run in CI/CD
- Tests pass across browsers
- Visual regression tests included
- Tests are not flaky
- Documentation for running tests

**Dependencies:** Task 5.2

---

#### Task 5.4: Backend Unit and Integration Tests
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/test/java/`
**Estimated Effort:** 7-10 days

**Description:**
Create comprehensive backend tests for risky work API and services.

**Implementation Details:**
- Unit tests for:
  - `RiskyWorkItem.java` - Data model
  - `RiskSeverity.java` - Severity calculation
  - `RiskyWorkItemService.java` - Service logic
  - `GetRiskyWorkItemsTask.java` - API task
  - `TijRiskApiClient.java` - TIJ integration
- Integration tests:
  - Full API endpoint tests with mocked TIJ
  - Database integration if applicable
  - Caching behavior
  - Error handling and retry logic
- Use JUnit 5, Mockito
- Use Testcontainers if database needed
- Test scenarios:
  - Query with no results
  - Query with multiple results
  - Filtering by project
  - Pagination/limiting
  - Severity sorting
  - TIJ unavailable (fallback)
  - Authorization failures
  - Invalid parameters
- Aim for >80% code coverage

**Acceptance Criteria:**
- All Java classes have tests
- Code coverage >80%
- Integration tests use Testcontainers
- All edge cases covered
- Tests pass in CI/CD
- Tests follow naming conventions

**Dependencies:** All Phase 1 backend tasks

---

#### Task 5.5: Create Developer Documentation
**Repository:** `jiragitcloud`
**Location:** `dev-docs/widgets/risky-work-widget.md`
**Estimated Effort:** 2-3 days

**Description:**
Document the widget architecture, API, and development guide.

**Implementation Details:**
- Create comprehensive documentation:
  - Widget overview and purpose
  - Architecture diagram
  - Component hierarchy
  - Data flow (API → Service → Widget)
  - State management
  - Entitlement handling
  - Adding/modifying risk rules
  - Troubleshooting guide
- Code examples for:
  - Adding new risk flags
  - Modifying severity calculation
  - Customizing widget layout
  - Testing the widget
- API documentation:
  - Endpoint details
  - Request/response formats
  - Error codes
  - Rate limiting
- Link to Figma designs
- Link to related epics (GITCL-4879)

**Acceptance Criteria:**
- Clear, comprehensive documentation
- Code examples are accurate
- Architecture diagrams included
- Easy to follow for new developers
- Linked from main docs

**Dependencies:** All implementation tasks

---

### Phase 6: Deployment & Monitoring

#### Task 6.1: Implement Feature Flags and Configuration
**Repository:** `jiragitcloud`
**Location:** `docs/conf/` and `ui/code/src/features/`
**Estimated Effort:** 2-3 days

**Description:**
Set up feature flags to control widget visibility and enable gradual rollout.

**Implementation Details:**
- Backend feature flags in `docs/conf/feature-flags.yaml`:
  - `riskyWorkWidget.enabled: false` (default)
  - `riskyWorkWidget.apiEnabled: false` (API separate control)
- Frontend feature flag checks:
  - Add flag to `useFeatureFlags.ts` hook
  - Conditionally render widget based on flag
  - Admin override in localStorage for testing
- Configuration options:
  - Default row count
  - Refresh interval
  - Cache TTL
  - Risk severity thresholds
- Document flag rollout plan

**Acceptance Criteria:**
- Feature flags control widget visibility
- API can be enabled independently
- Configuration values are customizable
- Admin override works
- Documentation complete

**Dependencies:** Task 3.3

---

#### Task 6.2: Add Comprehensive Telemetry
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/risky-work/`
**Estimated Effort:** 3-4 days

**Description:**
Implement detailed telemetry tracking for widget usage and performance.

**Implementation Details:**
- Create `risky-work-widget-telemetry.ts`:
  - Centralized event definitions
  - Type-safe event tracking
  - Consistent event naming
- Track events:
  - Widget rendered
  - Widget data loaded (success/failure)
  - Row count changed
  - Refresh clicked
  - Project filter changed
  - Issue/PR link clicked
  - Risk tag hovered
  - Info card displayed
  - "See full view" clicked (Advanced/Standard)
  - Upgrade modal opened
  - Upgrade modal CTA clicked
  - Feedback submitted
  - Error occurred
- Add performance metrics:
  - API response time
  - Widget render time
  - Time to first data display
- Include contextual properties:
  - User edition type
  - Selected project
  - Row count
  - Number of items displayed
  - Has risky work (yes/no)

**Acceptance Criteria:**
- All interactions tracked
- Performance metrics captured
- Event naming is consistent
- Type-safe event tracking
- Events visible in dev console
- Documentation for all events

**Dependencies:** All implementation tasks

---

#### Task 6.3: Performance Testing and Optimization
**Repository:** `jiragitcloud`
**Location:** Various
**Estimated Effort:** 3-5 days

**Description:**
Test widget performance and optimize any bottlenecks.

**Implementation Details:**
- Measure performance:
  - Widget mount time
  - Data fetch time
  - Render time with various data sizes
  - Hover card interaction latency
  - Memory usage
- Optimization targets:
  - API response < 500ms (p95)
  - Widget render < 100ms
  - Hover card appears within 1 second
  - No memory leaks
  - Smooth scrolling with 50 rows
- Optimizations:
  - Memoization of expensive computations
  - Virtual scrolling for large row counts (if needed)
  - Debounce hover card triggers
  - Optimize re-renders
  - Code splitting if bundle size impact
- Test with large datasets:
  - 100+ risky items
  - Items with many risk flags
  - Rapid user interactions

**Acceptance Criteria:**
- API performance targets met
- Render performance targets met
- No memory leaks detected
- Smooth interactions
- Bundle size impact < 30KB gzipped
- Performance regression tests added

**Dependencies:** All implementation tasks

---

#### Task 6.4: Create Rollout Plan and Runbook
**Repository:** `jiragitcloud`
**Location:** `dev-docs/deployment/risky-work-widget-rollout.md`
**Estimated Effort:** 2-3 days

**Description:**
Document staged rollout plan and operational runbook.

**Implementation Details:**
- Rollout stages:
  - Stage 1: Internal testing (GitKraken org only)
  - Stage 2: Beta customers (opt-in, ~10 customers)
  - Stage 3: Advanced users only (all Advanced edition users)
  - Stage 4: All users (Standard see upgrade prompt)
  - Stage 5: Full release
- For each stage:
  - Success criteria
  - Monitoring metrics
  - Duration (e.g., 1 week per stage)
  - Go/no-go checklist
- Rollback procedures:
  - Disable feature flag
  - Monitoring to confirm rollback
  - Communication plan
- Monitoring dashboard:
  - Widget usage metrics
  - Error rates
  - Performance metrics
  - User feedback
- Support preparation:
  - FAQ document
  - Known issues
  - Troubleshooting guide
  - Escalation path

**Acceptance Criteria:**
- Clear rollout stages defined
- Success criteria for each stage
- Monitoring dashboard ready
- Rollback plan documented
- Support team prepared
- Stakeholder approval

**Dependencies:** All testing and documentation tasks

---

#### Task 6.5: Production Deployment and Monitoring
**Repository:** `jiragitcloud`
**Location:** N/A (operational)
**Estimated Effort:** 2-4 weeks (staged)

**Description:**
Execute staged rollout with continuous monitoring.

**Implementation Details:**
- Deploy code to production (feature flag off)
- Enable for Stage 1 (GitKraken org)
  - Monitor for 3-5 days
  - Collect feedback
  - Fix critical issues
- Enable for Stage 2 (Beta customers)
  - Monitor for 1 week
  - Validate metrics
  - Iterate on feedback
- Enable for Stage 3 (Advanced users)
  - Monitor for 1 week
  - Check Signals navigation increase
  - Verify performance at scale
- Enable for Stage 4 (All users)
  - Monitor upgrade modal engagement
  - Check Standard user experience
  - Measure conversion impact
- Full release
  - Monitor for 2 weeks
  - Document lessons learned
  - Celebrate! 🎉
- Continuous monitoring:
  - Error rates (< 1%)
  - API latency (p95 < 500ms)
  - User engagement (increasing)
  - Feedback sentiment (positive)
  - Support tickets (no increase)

**Acceptance Criteria:**
- Widget rolled out to 100% of users
- Error rates within acceptable limits
- Performance metrics meet targets
- User feedback is positive
- No increase in support burden
- Post-mortem document created

**Dependencies:** Task 6.4

---

## Summary

**Total Tasks:** 33
**Estimated Total Effort:** 140-200 developer days (approximately 7-10 months for 1-2 developers, or 4-5 months for a team of 3-4)

**Critical Path:**
1. Phase 1: Backend API & Data (Tasks 1.1 → 1.2 → 1.3 → 1.4)
2. Phase 2: Frontend Widget (Tasks 2.1 → 2.2 → 2.3)
3. Phase 3: Integration (Tasks 3.1 → 3.2 → 3.3 → 3.4)
4. Phase 4: Polish (Tasks 4.1 → 4.2 → 4.3)
5. Phase 5: Testing (Tasks 5.1 → 5.2 → 5.3 → 5.4)
6. Phase 6: Deployment (Tasks 6.1 → 6.2 → 6.3 → 6.4 → 6.5)

**Parallel Workstreams:**
- Backend (Phase 1) and Frontend (Phase 2) can be developed in parallel after data model is defined (Task 1.1)
- Testing (Phase 5) can begin as components are completed
- Documentation can be written alongside implementation

**Key Dependencies:**
- **GITCL-4879 (Home Page Epic):** Tasks 3.3, 3.4 require Delivery Dashboard container and project selector
- **Team Insights Integration:** Task 1.4 depends on TIJ API availability
- **Signals Feature:** Entitlement logic depends on existing Signals infrastructure

**Key Risks:**
1. **Team Insights Integration Complexity:** If TIJ integration is complex, may need to build local risk detection (Task 1.4 Option B)
2. **Performance at Scale:** Widget must perform well with hundreds of risky items
3. **Entitlement Logic Complexity:** Advanced/Standard differentiation must be solid
4. **Responsive Design:** Ensuring good UX across all viewport sizes is challenging
5. **Signals Navigation:** Deep linking into Signals with filters may require Signals team collaboration

**Success Metrics:**
- Increased interaction with risky work items
- Increased Signals view entry by Advanced users (>20% increase)
- Upgrade modal engagement by Standard users (>5% click-through)
- Positive user feedback (>70% find helpful)
- Early risk identification (measured by time to action on flagged items)
