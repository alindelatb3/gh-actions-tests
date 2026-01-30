# GITCL-5062 Epic: Work in Progress Widget - Implementation Tasks

## Epic Summary
Build a momentum-focused widget for the Delivery Dashboard that surfaces Jira tickets with recent commit activity, sorted by recency, helping teams quickly understand what work is actively moving.

---

## Task Breakdown

### Phase 1: Backend API & Data Layer

#### Task 1.1: Design Work in Progress Data Model
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/main/java/com/bigbrassband/gitforjiracloud/workinprogress/`
**Estimated Effort:** 2-3 days

**Description:**
Design the data model for representing active work items with recent commit activity.

**Implementation Details:**
- Create Java classes:
  - `WorkInProgressItem.java` - Active work representation
    - Fields: `issueKey`, `summary`, `assigneeAvatarUrl`, `assigneeName`, `lastCommitTimestamp`, `timeSinceLastCommit`, `commitCount`, `repositoryName`, `projectKey`
  - `TimeRange.java` - Enum for time display (HOURS, DAYS, WEEKS)
  - `CommitActivity.java` - Recent commit summary
    - Fields: `commitHash`, `message`, `authorName`, `timestamp`
- Define recency calculation logic:
  - Calculate time since last commit
  - Format as "1h", "3h", "2d", "1w", etc.
  - Handle edge cases (seconds ago, months ago)
- Add JavaDoc documentation
- Use proper null safety annotations

**Acceptance Criteria:**
- Data model classes created with all required fields
- Time calculation logic implemented and accurate
- Unit tests for time formatting
- Classes use @Nonnull and @Nullable annotations
- JavaDoc documentation complete

**Dependencies:** None

---

#### Task 1.2: Create Work in Progress Query Service
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/main/java/com/bigbrassband/gitforjiracloud/workinprogress/`
**Estimated Effort:** 10-14 days

**Description:**
Implement service to query active Jira tickets with recent commit activity from Git indexes.

**Implementation Details:**
- Create `WorkInProgressService.java`:
  - Method: `List<WorkInProgressItem> getWorkInProgress(String installId, @Nullable String projectKey, int limit, int maxAgeDays)`
  - Query Git commit indexes for recent commits
  - Extract Jira issue keys from commit messages
  - Filter for non-closed Jira issues
  - Aggregate commits per issue
  - Sort by most recent commit timestamp descending
  - Apply limit
- Create `CommitIndexQuerier.java`:
  - Interface with existing Git index data structures
  - Extract commit metadata (hash, message, timestamp, author)
  - Filter by date range (e.g., last 30 days)
  - Efficient querying of large indexes
- Integration with existing Git indexing:
  - Access indexed commit data from `InstallFolderMonster` structure
  - Read from `installs/{installId}/repos/{repoId}/` indexes
  - Parse commit messages for Jira keys (reuse existing regex patterns)
  - Cross-reference with Jira issue status
- Jira issue status verification:
  - Call Jira API to check issue status
  - Filter out closed/resolved issues
  - Cache issue status (5-minute TTL)
  - Handle Jira API errors gracefully
- Add caching layer:
  - Cache results for 5 minutes per project
  - Invalidate on manual refresh
  - Use Guava Cache
- Performance optimization:
  - Index scanning optimization
  - Limit date range for queries
  - Parallel processing for multiple repos

**Acceptance Criteria:**
- Service returns active work items correctly
- Items sorted by most recent commit
- Only non-closed issues returned
- Project filtering works
- Performance acceptable (<1 second for typical dataset)
- Caching improves performance
- Handles large repositories efficiently
- Unit tests with mocked dependencies
- Integration tests with test data

**Dependencies:** Task 1.1

---

#### Task 1.3: Create REST API Endpoint for Work in Progress
**Repository:** `jiragitcloud`
**Location:** `docs/api/v1/` (YAML), `appserver/gitforjiracloud/` (Java)
**Estimated Effort:** 5-7 days

**Description:**
Create REST API endpoint to expose work in progress data to the frontend.

**Implementation Details:**
- Create YAML API definition `docs/api/v1/apiV1GetWorkInProgress.yaml`:
```yaml
url: /api/v1/work-in-progress
methods: [GET]
parameters:
  - {field: installId, type: string, source: jwt}
  - {field: projectKey, type: string, source: query, required: false}
  - {field: limit, type: integer, source: query, required: false, default: 5}
  - {field: maxAgeDays, type: integer, source: query, required: false, default: 30}
tasks:
  - getWorkInProgress: {}
```
- Create `GetWorkInProgressTask.java` in `appserver/apiendpoint/tasks/`:
  - Call `WorkInProgressService`
  - Convert to JSON response
  - Handle errors gracefully
  - Add OpenTelemetry tracing
  - Add Ticker logging for debugging
- Response format:
```json
{
  "items": [
    {
      "issueKey": "PROJ-123",
      "summary": "Implement user authentication",
      "assigneeAvatarUrl": "https://...",
      "assigneeName": "John Doe",
      "lastCommitTimestamp": "2026-01-30T14:30:00Z",
      "timeSinceLastCommit": "2h",
      "commitCount": 5,
      "repositoryName": "backend-api",
      "projectKey": "PROJ"
    }
  ],
  "totalCount": 23,
  "hasMore": true
}
```
- Add authorization checks:
  - User must have access to project
  - Validate installId from JWT
- Add rate limiting if needed
- Handle edge cases:
  - No recent commits
  - Invalid project key
  - Jira API unavailable

**Acceptance Criteria:**
- API endpoint accessible via defined URL
- Returns correctly formatted JSON
- Respects query parameters
- Authorization checks work
- Error responses use proper HTTP codes
- OpenTelemetry spans created
- API documentation updated
- Postman/curl examples provided

**Dependencies:** Task 1.2

---

#### Task 1.4: Optimize Commit Index Querying
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/main/java/com/bigbrassband/gitforjiracloud/workinprogress/`
**Estimated Effort:** 5-7 days

**Description:**
Optimize the querying of Git indexes for performance with large repositories and many commits.

**Implementation Details:**
- Analyze existing commit index structure:
  - Understand how commits are stored
  - Identify bottlenecks in current access patterns
  - Document index format
- Optimization strategies:
  - Create time-based index for recent commits
  - Build in-memory cache of recent commit metadata
  - Implement incremental index reading (don't read full index)
  - Use binary search for timestamp-based queries
  - Parallel processing for multi-repo queries
- Create `CommitIndexCache.java`:
  - LRU cache of recent commits per repository
  - TTL of 10 minutes
  - Automatic invalidation on repo reindex
  - Memory-efficient storage
- Benchmark performance:
  - Test with repositories of various sizes (1K, 10K, 100K commits)
  - Measure query time before and after optimization
  - Target: <200ms for typical query
- Add metrics:
  - Cache hit/miss rates
  - Query duration
  - Index scan size

**Acceptance Criteria:**
- Query performance meets targets (<200ms typical)
- Handles large repositories efficiently
- Cache reduces repeated query time
- Memory usage is reasonable
- Benchmarks documented
- Metrics tracked

**Dependencies:** Task 1.2

---

### Phase 2: Frontend Widget Component

#### Task 2.1: Create Work in Progress Widget Container
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
**Estimated Effort:** 5-7 days

**Description:**
Build the main widget container component with header, controls, and layout.

**Implementation Details:**
- Create `WorkInProgressWidget.tsx`:
  - Widget header with title and info icon
  - Row count selector (5, 10, 25, 50)
  - Refresh button
  - "See full view in Signals" button
  - Loading state
  - Error state
  - Empty state
  - Widget body container for rows
- Create `WorkInProgressWidget.module.scss`:
  - Match Figma designs
  - Responsive layout
  - Maximum height with internal scrolling
  - Grid layout for rows
- Create `WorkInProgressWidgetContext.tsx`:
  - Selected row count
  - Refresh trigger
  - Loading state
  - Error state
  - Selected project (from dashboard context)
- Add tooltip for help icon:
  - "This widget highlights tickets and PRs with recent activity to show current momentum."
- Widget header styling:
  - Title: "Work in Progress"
  - Consistent with dashboard theme

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

#### Task 2.2: Create Work in Progress Item Row Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 5-7 days

**Description:**
Build the row component that displays individual work in progress items.

**Implementation Details:**
- Create `WorkInProgressItemRow.tsx`:
  - Issue key (clickable link to Jira issue, new tab)
  - Summary text (truncated with ellipsis)
  - Time since last commit (e.g., "2h", "3d", "1w")
  - Assignee avatar
  - Repository name (small, secondary text)
  - Commit count badge (optional, e.g., "5 commits")
- Create `TimeSinceCommit.tsx` component:
  - Display formatted time
  - Tooltip with exact timestamp
  - Color coding (optional):
    - Green: < 24h
    - Yellow: 1-7 days
    - Gray: > 7 days
- Create `work-in-progress-item-row.module.scss`:
  - Grid layout for columns
  - Hover effect (subtle background change)
  - Truncation with ellipsis for summary
  - Responsive adjustments
  - Avatar sizing
- Handle click events:
  - Open Jira issue in new tab
  - Track telemetry on clicks
- Display logic:
  - Show all information at desktop width
  - Hide less critical info on mobile (e.g., repo name)

**Acceptance Criteria:**
- Row displays all required information
- Issue key link works (new tab)
- Time since commit displays correctly
- Tooltip shows exact timestamp
- Summary text truncates properly
- Avatar displays correctly
- Hover effect works
- Responsive design works
- Accessible markup

**Dependencies:** Task 2.1

---

#### Task 2.3: Implement Hover Card for Work Items
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 7-10 days

**Description:**
Create information card that appears on long hover (1 second) showing detailed ticket information.

**Implementation Details:**
- Create `WorkInProgressItemHoverCard.tsx`:
  - Trigger on 1-second hover over row
  - Display detailed information:
    - Full summary
    - Description preview (first 200 chars)
    - Status
    - Assignee name
    - Last commit details:
      - Commit message (truncated)
      - Author name
      - Exact timestamp
      - Repository name
    - Recent commit activity (last 3-5 commits)
    - Project name
  - Position card near cursor or row
  - Smooth fade-in animation (200ms)
  - Close on mouse leave or click outside
- Reuse hover card hook from Risky Work widget:
  - Share `useHoverCard.ts` hook between widgets
  - Consistent hover behavior
  - 1-second delay
  - Cancel on mouse leave
- Fetch additional data if needed:
  - Call API for full issue details (if not in initial response)
  - Call API for recent commit list
  - Cache fetched data
  - Handle loading state within card
- Create `work-in-progress-item-hover-card.module.scss`:
  - Card styling (shadow, border, padding)
  - Animation
  - Responsive sizing
  - Proper z-index
- Commit list display:
  - Show 3-5 most recent commits
  - Commit hash (short, 7 chars)
  - Commit message (truncated)
  - Relative time

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
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 2-3 days

**Description:**
Create dropdown/selector for choosing how many rows to display (5, 10, 25, 50).

**Implementation Details:**
- Reuse `RowCountSelector.tsx` from Risky Work widget if possible:
  - Generic component that works for both widgets
  - Move to shared `widgets/components/` directory
  - Accept widget-specific localStorage key as prop
- If not reusable, create widget-specific version:
  - Same functionality as Risky Work widget
  - Different localStorage key: `wip-widget-row-count`
- Dropdown component using Atlassian Design System
- Options: 5, 10, 25, 50
- Default: 5
- Persist selection in localStorage
- Update widget context on change
- Add telemetry for row count changes
- Responsive behavior:
  - May show fewer rows on narrow viewports even if higher selected

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
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
**Estimated Effort:** 3-4 days

**Description:**
Implement manual and automatic refresh capabilities for the widget.

**Implementation Details:**
- Reuse refresh patterns from Risky Work widget:
  - Similar refresh button component
  - Similar loading states
  - Similar error handling
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
  - Retry mechanism (exponential backoff)
  - User-friendly error messages
  - Fallback to stale data if available
- Add telemetry:
  - Track manual refresh clicks
  - Track auto-refresh triggers
  - Track refresh errors
  - Track refresh duration

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
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
**Estimated Effort:** 5-7 days

**Description:**
Create "See full view in Signals" button with behavior based on user entitlement (Advanced vs Standard).

**Implementation Details:**
- Reuse `SignalsViewButton.tsx` from Risky Work widget if possible:
  - Make component generic and reusable
  - Accept widget-specific filter parameters as props
  - Move to shared `widgets/components/` directory
- If not reusable, create widget-specific version:
  - Button text: "See full view in Signals"
  - Different behavior for Advanced vs Standard users
  - Use existing `useIsSignalsEnabled` and entitlement hooks
- **Advanced Users Behavior:**
  - Click opens Signals view in new route/page
  - Navigate to Signals with filters pre-applied:
    - Show tickets with recent commits
    - Sort by most recent commit descending
    - Preserve selected project filter
  - Construct URL with query parameters for filters
  - Example: `/insights/?filter=recent-activity&sort=recency&project={projectKey}`
- **Standard Users Behavior:**
  - Click opens upgrade modal (reuse from Risky Work widget)
  - Contextual messaging about active work visibility
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

#### Task 3.2: Reuse or Adapt Upgrade Modal for Standard Users
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/components/`
**Estimated Effort:** 3-5 days

**Description:**
Reuse the upgrade modal from Risky Work widget or adapt it with Work in Progress-specific messaging.

**Implementation Details:**
- Evaluate reusability of `SignalsUpgradeModal.tsx` from GITCL-5061:
  - If fully reusable, use as-is
  - If needs customization, make it accept content props
- **If making generic:**
  - Move to shared location
  - Accept props for customization:
    - Header text
    - Bullet points
    - Video URL
    - CTA button text
  - Create content configuration files for each widget
- **If creating WIP-specific version:**
  - Header: "Unlock Deeper Activity Insights with Signals Advanced"
  - Bullet points explaining value:
    - "Track all development activity across your team"
    - "Identify stale work and blockers early"
    - "Deep analytics on commit patterns and velocity"
    - "Custom filters and saved views"
  - Same modal structure as Risky Work widget
  - Explainer video (PMM-provided, may be same as Risky Work)
  - CTA buttons (Upgrade, Learn More, Close)
  - "Don't show again" option
- Add telemetry:
  - Track modal opens (from WIP widget)
  - Track video plays
  - Track CTA clicks
  - Track dismissals

**Acceptance Criteria:**
- Modal displays correctly
- Content is relevant to Work in Progress context
- All CTAs work
- "Don't show again" persists
- Telemetry tracks interactions
- Visual design consistent with other modals

**Dependencies:** Task 3.1, GITCL-5061 Task 3.2 (if reusing)

---

#### Task 3.3: Integrate Widget with Delivery Dashboard
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/`
**Estimated Effort:** 3-5 days

**Description:**
Integrate the Work in Progress Widget into the Delivery Dashboard section from GITCL-4879.

**Implementation Details:**
- Update `DeliveryDashboard.tsx`:
  - Import `WorkInProgressWidget` component
  - Add widget to dashboard layout
  - Position relative to Risky Work widget (if both enabled)
  - Pass dashboard context (selected project) to widget
  - Handle widget visibility based on feature flag
- Dashboard layout with multiple widgets:
  - Grid layout for widgets
  - Responsive: Stack vertically on mobile
  - Horizontal on desktop (2-column or 1-column based on screen size)
  - Consistent spacing between widgets
- Update `DeliveryDashboardContext.tsx`:
  - Add WIP widget state if needed
  - Provide selected project to widget
- Feature flag:
  - Add `workInProgressWidget.enabled` flag
  - Control widget visibility
  - Allow independent enabling from Risky Work widget
  - Support enabling one or both widgets
- Update dashboard empty state:
  - Show appropriate message if no widgets have data
  - "No active work or risks to display"

**Acceptance Criteria:**
- Widget renders in dashboard
- Widget receives selected project from context
- Feature flag controls visibility
- Layout looks correct with multiple widgets
- Responsive design works
- No conflicts with other dashboard components

**Dependencies:** GITCL-4879 Task 4.1 (Delivery Dashboard Container), Task 2.1

---

#### Task 3.4: Implement Project Filter Integration
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
**Estimated Effort:** 3-5 days

**Description:**
Connect widget to dashboard project selector to filter work items by selected project.

**Implementation Details:**
- Update `WorkInProgressWidget.tsx`:
  - Subscribe to dashboard context for selected project
  - Pass project key to API query
  - Re-fetch data when project changes
  - Show loading state during project change
  - Handle "All Projects" selection (no project filter)
- Create `useWorkInProgressData.ts` hook:
  - Fetch data from API endpoint
  - Accept project key parameter
  - Handle loading, error, data states
  - Implement SWR or React Query for caching
  - Auto-refetch on project change
  - Debounce rapid project changes
- Add telemetry:
  - Track project filter changes
  - Track data fetch by project
  - Track filter performance (time to refetch)
- Update empty state:
  - Show project-specific empty message
  - "No recent activity for [Project Name]"
  - Suggestion: "Commits will appear here when developers push code"

**Acceptance Criteria:**
- Widget data filters by selected project
- Data refetches on project change
- Loading state during refetch
- "All Projects" shows unfiltered data
- Empty state is project-aware
- Telemetry tracks project filters
- Performance is good (caching works)
- Debouncing prevents excessive API calls

**Dependencies:** Task 2.1, GITCL-4879 Task 4.2 (Project Selector)

---

### Phase 4: Polish & User Experience

#### Task 4.1: Implement User Feedback Collection
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 3-5 days

**Description:**
Add lightweight inline feedback collection to gather user input on widget usefulness.

**Implementation Details:**
- Reuse `WidgetFeedback.tsx` from Risky Work widget:
  - Make component generic and reusable
  - Move to shared `widgets/components/` directory
  - Accept widget type as prop
  - Share backend endpoint for feedback submission
- If not reusable, create widget-specific version:
  - Same prompt: "Did you find this widget helpful?"
  - Thumbs up/down or Yes/No
  - Follow-up: "Would you be willing to share why?"
  - Text input for comments
  - Submit button
  - Thank you message after submission
  - Collapse after submission or dismissal
- Backend integration:
  - Use same API endpoint as Risky Work widget
  - Include `widgetType: "work-in-progress"` in payload
  - Store in database or send to analytics
- Add telemetry:
  - Track feedback prompts shown
  - Track responses (helpful/not helpful)
  - Track text submissions
  - Track dismissals
- Display logic:
  - Show after user has interacted with widget
  - Don't show again if user already responded
  - Option to dismiss permanently

**Acceptance Criteria:**
- Feedback component renders in widget
- Responses submit successfully
- Follow-up question appears
- Thank you message displays
- "Already answered" state persists
- Telemetry tracks interactions
- Backend stores feedback
- Non-intrusive UX

**Dependencies:** Task 2.1, GITCL-5061 Task 4.1 (if reusing)

---

#### Task 4.2: Implement Responsive Behavior
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
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
    - Hide repository name
    - Hide commit count
    - Maintain issue key, summary, time, and avatar
  - **Tablet (768px-1024px):**
    - Moderate truncation
    - Show most metadata
    - Adjust grid columns
    - May hide commit count
  - **Desktop (>1024px):**
    - Full layout
    - All metadata visible
- Implement maximum height:
  - Widget scrolls internally
  - Prevent infinite page expansion
  - Max height ~600px, then scroll
- Update `work-in-progress-widget.module.scss`:
  - Media queries for each breakpoint
  - Flexible grid layouts
  - Responsive typography
  - Appropriate truncation
  - Touch-friendly hit targets (44x44px minimum)
- Test on real devices:
  - iOS Safari
  - Android Chrome
  - Desktop browsers
- Multi-widget responsive behavior:
  - When both WIP and Risky Work widgets present
  - Stack vertically on mobile
  - Side-by-side on desktop (if space allows)

**Acceptance Criteria:**
- No horizontal scrolling at any width
- Widget remains functional at all breakpoints
- Primary information visible at all widths
- Internal scrolling works
- Touch interactions work on mobile
- Visual design maintains quality
- Matches Figma responsive specs
- Works well with other widgets

**Dependencies:** Task 2.1, Task 2.2

---

#### Task 4.3: Add Loading Skeletons and States
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 3-4 days

**Description:**
Create polished loading states with skeleton screens for better perceived performance.

**Implementation Details:**
- Create `WorkInProgressItemSkeleton.tsx`:
  - Skeleton row matching real row layout
  - Animated shimmer effect
  - Show multiple skeleton rows (matching selected count)
  - Skeleton for issue key, summary, time, avatar
- Reuse error and empty state components if possible:
  - Share `WidgetErrorState.tsx` from Risky Work widget
  - Share `WidgetEmptyState.tsx` with custom messaging
  - Move to shared `widgets/components/` if reusing
- If creating widget-specific states:
  - Error state with retry button
  - Different error messages:
    - Network error
    - Authorization error
    - No recent activity
    - Git indexing in progress
  - Empty state:
    - Positive messaging
    - "No recent commit activity"
    - Illustration or icon
    - Suggestion: "Activity will appear when commits are pushed"
- Update widget logic:
  - Show skeleton during initial load
  - Show skeleton during refresh (or show stale data with loading indicator)
  - Show error state on failures
  - Show empty state when no data
- Add retry logic:
  - Retry button in error state
  - Exponential backoff for automatic retries
  - Track retry attempts
- Progressive loading:
  - Show skeleton immediately
  - Fade in real data when loaded
  - Smooth transitions

**Acceptance Criteria:**
- Skeleton loading state displays correctly
- Error state shows appropriate messages
- Empty state is encouraging and helpful
- Retry functionality works
- Transitions between states are smooth
- All states match visual design
- Skeleton matches row layout

**Dependencies:** Task 2.1

---

#### Task 4.4: Implement Time Formatting and Updates
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/components/`
**Estimated Effort:** 2-3 days

**Description:**
Implement smart time formatting and optional auto-update of "time since commit" displays.

**Implementation Details:**
- Create `useRelativeTime.ts` hook:
  - Accept timestamp
  - Return formatted relative time string
  - Format: "1h", "3h", "2d", "1w"
  - Optionally auto-update every minute
  - Stop auto-update when widget unmounted or hidden
- Time formatting logic:
  - < 1 hour: "Xm" (minutes)
  - 1-23 hours: "Xh" (hours)
  - 1-6 days: "Xd" (days)
  - 7-29 days: "Xw" (weeks)
  - >= 30 days: "Xmo" (months)
- Color coding (optional):
  - Green: < 1 day (very recent)
  - Yellow: 1-7 days (recent)
  - Gray: > 7 days (older)
- Tooltip with exact timestamp:
  - On hover, show full date/time
  - Format: "Jan 30, 2026 at 2:30 PM"
  - Use user's locale and timezone
- Auto-update consideration:
  - Pros: Always accurate, feels live
  - Cons: Re-renders, battery on mobile
  - Decision: Optional, default off for performance
  - If enabled, update every 60 seconds
- Performance optimization:
  - Memoize formatted strings
  - Batch updates for multiple rows
  - Use requestAnimationFrame for updates

**Acceptance Criteria:**
- Time displays in compact format
- Tooltip shows exact timestamp
- Format is clear and intuitive
- Optional auto-update works
- Performance is good (no excessive re-renders)
- Color coding enhances readability (if implemented)
- Accessible (tooltip accessible to screen readers)

**Dependencies:** Task 2.2

---

### Phase 5: Testing & Documentation

#### Task 5.1: Write Unit Tests for Widget Components
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/**/*.test.tsx`
**Estimated Effort:** 7-10 days

**Description:**
Create comprehensive unit test suite for all widget components and logic.

**Implementation Details:**
- Test coverage for:
  - `WorkInProgressWidget.tsx` - Container logic
  - `WorkInProgressItemRow.tsx` - Row rendering and interactions
  - `WorkInProgressItemHoverCard.tsx` - Hover behavior
  - `TimeSinceCommit.tsx` - Time formatting
  - `RowCountSelector.tsx` - Selection logic (if widget-specific)
  - `SignalsViewButton.tsx` - Entitlement-aware behavior (if widget-specific)
  - `WidgetFeedback.tsx` - Feedback submission (if widget-specific)
  - All hooks (`useWorkInProgressData`, `useRelativeTime`, etc.)
- Use React Testing Library
- Mock dependencies:
  - API calls
  - Dashboard context
  - Entitlement hooks
  - localStorage
  - Telemetry
  - Current time (for time formatting tests)
- Test scenarios:
  - Data loading and display
  - Error handling
  - Empty states
  - User interactions (clicks, hovers, selections)
  - Entitlement differences
  - Responsive behavior
  - Time formatting (various durations)
  - Project filtering
- Aim for >85% code coverage

**Acceptance Criteria:**
- All components have unit tests
- All hooks have unit tests
- Code coverage >85%
- Tests pass in CI/CD
- Edge cases covered
- Time-dependent tests use mocked time
- Mocks are appropriate and maintainable

**Dependencies:** All Phase 2, 3, 4 implementation tasks

---

#### Task 5.2: Write Integration Tests
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/**/*.integration.test.tsx`
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
  - Multi-widget integration (with Risky Work widget)
- Use MSW (Mock Service Worker) for API mocking
- Test with different data scenarios:
  - Empty results (no recent activity)
  - Single item
  - Multiple items (5, 10, 50)
  - Items with various time ranges (hours, days, weeks)
  - Mixed assignees
  - Long summaries (truncation)
- Test state management across components
- Verify telemetry events fire correctly
- Test caching behavior (SWR/React Query)

**Acceptance Criteria:**
- All major workflows tested
- API integration tested with mocks
- Dashboard integration verified
- Entitlement scenarios covered
- Multi-widget scenarios tested
- Tests are reliable and not flaky
- All tests pass in CI/CD

**Dependencies:** Task 5.1

---

#### Task 5.3: Write E2E Tests with Playwright
**Repository:** `jiragitcloud`
**Location:** `ui/code/tests/e2e/work-in-progress-widget/`
**Estimated Effort:** 5-7 days

**Description:**
Create end-to-end tests for complete user journeys with the widget.

**Implementation Details:**
- Create test files:
  - `work-in-progress-widget-display.spec.ts` - Widget rendering and data display
  - `work-in-progress-widget-interactions.spec.ts` - User interactions
  - `work-in-progress-widget-entitlement.spec.ts` - Entitlement behaviors
  - `work-in-progress-widget-responsive.spec.ts` - Responsive design
  - `multiple-widgets.spec.ts` - Both WIP and Risky Work widgets
- Test scenarios:
  - Load home page and see widget in dashboard
  - Widget displays work in progress items
  - Click on issue key (opens in new tab)
  - Hover over time display (tooltip appears)
  - Long hover on row (info card appears)
  - Change row count selector
  - Click refresh button
  - Select different project (data updates)
  - Advanced user clicks "See full view" (navigates to Signals)
  - Standard user clicks "See full view" (modal opens)
  - Submit feedback
  - Multiple widgets display together correctly
- Use Playwright fixtures for test data
- Test across browsers (Chromium, Firefox, WebKit)
- Add visual regression tests (screenshots)
- Test time display updates (if auto-update enabled)

**Acceptance Criteria:**
- All critical user paths tested
- Tests run in CI/CD
- Tests pass across browsers
- Visual regression tests included
- Multi-widget scenarios tested
- Tests are not flaky
- Documentation for running tests

**Dependencies:** Task 5.2

---

#### Task 5.4: Backend Unit and Integration Tests
**Repository:** `jiragitcloud`
**Location:** `appserver/gitforjiracloud/src/test/java/`
**Estimated Effort:** 7-10 days

**Description:**
Create comprehensive backend tests for work in progress API and services.

**Implementation Details:**
- Unit tests for:
  - `WorkInProgressItem.java` - Data model
  - `TimeRange.java` - Time formatting
  - `WorkInProgressService.java` - Service logic
  - `GetWorkInProgressTask.java` - API task
  - `CommitIndexQuerier.java` - Index querying
  - `CommitIndexCache.java` - Caching logic
- Integration tests:
  - Full API endpoint tests
  - Index querying with test data
  - Caching behavior verification
  - Error handling and retry logic
  - Performance tests with large datasets
- Use JUnit 5, Mockito
- Create test fixtures:
  - Mock commit indexes
  - Sample Jira issues
  - Various time ranges
- Test scenarios:
  - Query with no results
  - Query with multiple results
  - Filtering by project
  - Limiting results
  - Time range filtering
  - Recency sorting
  - Jira API unavailable (fallback)
  - Authorization failures
  - Invalid parameters
  - Large dataset performance
- Aim for >80% code coverage
- Benchmark critical paths:
  - Index query performance
  - Cache effectiveness

**Acceptance Criteria:**
- All Java classes have tests
- Code coverage >80%
- Integration tests verify full workflows
- All edge cases covered
- Performance benchmarks documented
- Tests pass in CI/CD
- Tests follow naming conventions

**Dependencies:** All Phase 1 backend tasks

---

#### Task 5.5: Create Developer Documentation
**Repository:** `jiragitcloud`
**Location:** `dev-docs/widgets/work-in-progress-widget.md`
**Estimated Effort:** 2-3 days

**Description:**
Document the widget architecture, API, and development guide.

**Implementation Details:**
- Create comprehensive documentation:
  - Widget overview and purpose
  - Architecture diagram (data flow from Git indexes → API → Widget)
  - Component hierarchy
  - State management
  - Entitlement handling
  - Commit index querying logic
  - Performance considerations
  - Troubleshooting guide
- Code examples for:
  - Querying commit indexes
  - Adding time formatting logic
  - Customizing widget layout
  - Testing the widget
- API documentation:
  - Endpoint details
  - Request/response formats
  - Query parameters
  - Error codes
  - Performance considerations
- Index structure documentation:
  - How commits are indexed
  - Index query optimization
  - Cache invalidation
- Link to Figma designs
- Link to related epics (GITCL-4879, GITCL-5061)
- Comparison with Risky Work widget (similarities/differences)

**Acceptance Criteria:**
- Clear, comprehensive documentation
- Code examples are accurate
- Architecture diagrams included
- Performance considerations documented
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
  - `workInProgressWidget.enabled: false` (default)
  - `workInProgressWidget.apiEnabled: false` (API separate control)
  - `workInProgressWidget.maxAgeDays: 30` (query time range)
- Frontend feature flag checks:
  - Add flag to `useFeatureFlags.ts` hook
  - Conditionally render widget based on flag
  - Admin override in localStorage for testing
- Configuration options:
  - Default row count (default: 5)
  - Refresh interval (default: 5 minutes cache TTL)
  - Cache TTL
  - Max age days for commit query (default: 30)
  - Auto-update time display (default: false)
- Document flag rollout plan:
  - Coordinate with Risky Work widget rollout
  - Can be enabled independently or together

**Acceptance Criteria:**
- Feature flags control widget visibility
- API can be enabled independently
- Configuration values are customizable
- Admin override works
- Documentation complete
- Can be deployed with flag off

**Dependencies:** Task 3.3

---

#### Task 6.2: Add Comprehensive Telemetry
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/widgets/work-in-progress/`
**Estimated Effort:** 3-4 days

**Description:**
Implement detailed telemetry tracking for widget usage and performance.

**Implementation Details:**
- Create `work-in-progress-widget-telemetry.ts`:
  - Centralized event definitions
  - Type-safe event tracking
  - Consistent event naming pattern
- Track events:
  - Widget rendered
  - Widget data loaded (success/failure)
  - Row count changed
  - Refresh clicked
  - Project filter changed
  - Issue link clicked
  - Time display hovered (tooltip)
  - Info card displayed
  - "See full view" clicked (Advanced/Standard)
  - Upgrade modal opened
  - Feedback submitted
  - Error occurred
  - Empty state displayed
- Add performance metrics:
  - API response time
  - Index query duration
  - Widget render time
  - Time to first data display
  - Cache hit/miss rate
- Include contextual properties:
  - User edition type
  - Selected project
  - Row count
  - Number of items displayed
  - Has recent activity (yes/no)
  - Time range of displayed items
  - Whether showing cached data
- Correlation with Risky Work widget:
  - Track usage of both widgets together
  - Compare engagement metrics

**Acceptance Criteria:**
- All interactions tracked
- Performance metrics captured
- Event naming is consistent
- Type-safe event tracking
- Events visible in dev console
- Documentation for all events
- Correlated with other widget metrics

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
  - Data fetch time (API + backend processing)
  - Index query duration
  - Render time with various data sizes
  - Hover card interaction latency
  - Memory usage
  - Time display update overhead (if enabled)
- Optimization targets:
  - API response < 500ms (p95)
  - Index query < 200ms (p95)
  - Widget render < 100ms
  - Hover card appears within 1 second
  - No memory leaks
  - Smooth scrolling with 50 rows
- Backend optimizations:
  - Index query optimization (Task 1.4)
  - Commit metadata caching
  - Parallel repository processing
  - Efficient date range filtering
- Frontend optimizations:
  - Memoization of expensive computations
  - Virtual scrolling for large row counts (if needed)
  - Debounce hover card triggers
  - Optimize re-renders
  - Lazy load hover card content
  - Code splitting if bundle size impact
- Test with realistic scenarios:
  - Large repositories (100K+ commits)
  - Multiple repositories (50+)
  - Large result sets (100+ active items)
  - Rapid user interactions
  - Slow network conditions
- Memory profiling:
  - Check for leaks on mount/unmount
  - Monitor cache size growth
  - Verify cleanup on page leave

**Acceptance Criteria:**
- API performance targets met
- Index query performance targets met
- Render performance targets met
- No memory leaks detected
- Smooth interactions
- Bundle size impact < 30KB gzipped
- Performance regression tests added
- Performs well with large datasets

**Dependencies:** All implementation tasks

---

#### Task 6.4: Create Rollout Plan and Runbook
**Repository:** `jiragitcloud`
**Location:** `dev-docs/deployment/work-in-progress-widget-rollout.md`
**Estimated Effort:** 2-3 days

**Description:**
Document staged rollout plan and operational runbook.

**Implementation Details:**
- Coordinate rollout with Risky Work widget:
  - Option 1: Sequential rollout (Risky Work first, then WIP)
  - Option 2: Parallel rollout (both together in stages)
  - Option 3: Staggered (enable one widget for some users, both for others)
  - Recommended: Parallel for users who get dashboard
- Rollout stages:
  - Stage 1: Internal testing (GitKraken org only)
  - Stage 2: Beta customers (opt-in, ~10 customers)
  - Stage 3: Advanced users (Signals Advanced customers)
  - Stage 4: Percentage rollout (10% → 25% → 50% → 100%)
  - Stage 5: Full release
- For each stage:
  - Success criteria specific to WIP widget:
    - Widget usage (views, interactions)
    - Data quality (accuracy of recent activity)
    - Performance (API latency, index query time)
    - User feedback sentiment
  - Combined success criteria (with Risky Work):
    - Dashboard engagement overall
    - Multiple widget usage patterns
  - Duration (e.g., 1 week per stage)
  - Go/no-go checklist
- Rollback procedures:
  - Disable feature flag
  - Clear cached data
  - Monitoring to confirm rollback
  - Communication plan
- Monitoring dashboard:
  - Widget-specific metrics
  - Combined dashboard metrics
  - Backend performance (index queries)
  - Error rates
  - User feedback
- Support preparation:
  - FAQ document (WIP-specific questions)
  - Known issues
  - Troubleshooting for "no activity showing"
  - Data freshness expectations
  - Escalation path

**Acceptance Criteria:**
- Clear rollout stages defined
- Success criteria for each stage
- Coordination with Risky Work widget documented
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
- Enable for Stage 1 (GitKraken org):
  - Internal team testing
  - Monitor for 3-5 days
  - Collect feedback
  - Fix critical issues
  - Verify index query performance
- Enable for Stage 2 (Beta customers):
  - Select ~10 diverse customers (various repo sizes)
  - Monitor for 1 week
  - Validate data accuracy
  - Check performance across different repo sizes
  - Iterate on feedback
- Enable for Stage 3 (Advanced users):
  - Enable for all Signals Advanced customers
  - Monitor for 1 week
  - Check Signals navigation increase
  - Verify performance at scale
  - Compare usage with Risky Work widget
- Enable for Stage 4 (Percentage rollout):
  - Gradual rollout: 10% → 25% → 50% → 100%
  - Monitor upgrade modal engagement (Standard users)
  - Check multi-widget usage patterns
  - Measure overall dashboard engagement
- Full release:
  - Enable for 100% of users
  - Monitor for 2 weeks
  - Document lessons learned
  - Gather long-term usage data
  - Celebrate! 🎉
- Continuous monitoring:
  - Error rates (< 1%)
  - API latency (p95 < 500ms)
  - Index query latency (p95 < 200ms)
  - User engagement (interactions/day)
  - Feedback sentiment (positive vs negative)
  - Support tickets (no increase expected)
  - Cache hit rate (>80%)
  - Data accuracy (spot checks)
- Performance monitoring:
  - Backend: Index query times, cache efficiency
  - Frontend: Widget render time, API call duration
  - Alert on performance degradation
- Data quality monitoring:
  - Spot check: Are displayed items actually recent?
  - Verify commit timestamps are accurate
  - Check for false positives/negatives

**Acceptance Criteria:**
- Widget rolled out to 100% of users
- Error rates within acceptable limits
- Performance metrics meet targets
- Index queries are efficient
- Data quality is high
- User feedback is positive
- No increase in support burden
- Works well alongside Risky Work widget
- Post-mortem document created

**Dependencies:** Task 6.4

---

## Summary

**Total Tasks:** 33
**Estimated Total Effort:** 130-190 developer days (approximately 6.5-9.5 months for 1-2 developers, or 3.5-5 months for a team of 3-4)

**Critical Path:**
1. Phase 1: Backend API & Data (Tasks 1.1 → 1.2 → 1.3 → 1.4)
2. Phase 2: Frontend Widget (Tasks 2.1 → 2.2 → 2.3)
3. Phase 3: Integration (Tasks 3.1 → 3.2 → 3.3 → 3.4)
4. Phase 4: Polish (Tasks 4.1 → 4.2 → 4.3 → 4.4)
5. Phase 5: Testing (Tasks 5.1 → 5.2 → 5.3 → 5.4)
6. Phase 6: Deployment (Tasks 6.1 → 6.2 → 6.3 → 6.4 → 6.5)

**Parallel Workstreams:**
- Backend (Phase 1) and Frontend (Phase 2) can be developed in parallel after data model is defined (Task 1.1)
- Testing (Phase 5) can begin as components are completed
- Documentation can be written alongside implementation
- Can reuse components from Risky Work widget (GITCL-5061) where applicable

**Code Reuse Opportunities:**
Many components can be shared with the Risky Work widget (GITCL-5061):
- `RowCountSelector.tsx` - Identical functionality
- `WidgetFeedback.tsx` - Same feedback collection
- `SignalsViewButton.tsx` - Same entitlement logic, different filters
- `SignalsUpgradeModal.tsx` - Same or similar modal with different content
- `WidgetErrorState.tsx` and `WidgetEmptyState.tsx` - Similar patterns
- `useHoverCard.ts` - Same hover behavior
- Shared telemetry utilities
- Shared responsive CSS patterns

**Key Dependencies:**
- **GITCL-4879 (Home Page Epic):** Tasks 3.3, 3.4 require Delivery Dashboard container and project selector
- **GITCL-5061 (Risky Work Widget):** Many reusable components if developed first
- **Git Indexing Infrastructure:** Must have efficient access to commit indexes
- **Jira API Integration:** Need to check issue status (open vs closed)

**Key Risks:**
1. **Index Query Performance:** Querying large commit indexes efficiently is challenging (Task 1.4 critical)
2. **Data Accuracy:** Ensuring commit-to-issue mapping is accurate and up-to-date
3. **Jira API Dependency:** Widget depends on Jira API for issue status (must handle API unavailability)
4. **Performance at Scale:** Widget must perform well with large repositories and many active items
5. **Time Display:** Relative time display must be accurate and update appropriately
6. **Multi-Widget Layout:** Must work well alongside Risky Work widget without conflicts

**Success Metrics:**
- Increased Delivery Dashboard usage for day-to-day monitoring
- Increased Signals view entry by Advanced users (>15% increase)
- Upgrade modal engagement by Standard users (>5% click-through)
- Positive user feedback (>70% find helpful)
- Improved visibility into active work (measured by qualitative feedback)
- Fast performance (API <500ms p95, index query <200ms p95)
