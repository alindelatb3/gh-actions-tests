# GITCL-4879 Epic: Update GIJ Home Page UI - Implementation Tasks

## Epic Summary
Transform the GIJ home page from a simple shortcuts page into a progressively dashboard-centric landing experience with collapsible sections (Getting Started, Shortcuts, Delivery Dashboard) and widget support for Signals insights.

---

## Task Breakdown

### Phase 1: Home Page Infrastructure & Layout

#### Task 1.1: Create Collapsible Section Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/components/`
**Estimated Effort:** 3-5 days

**Description:**
Build a reusable `CollapsibleSection` component that will be used for all three sections (Getting Started, Shortcuts, Delivery Dashboard).

**Implementation Details:**
- Create `CollapsibleSection.tsx` component with:
  - Header with title and collapse/expand icon
  - Collapsible content area with smooth animation
  - Sticky behavior when collapsed
  - Local state management for collapsed/expanded
- Create `CollapsibleSection.module.scss` for styling
- Support props: `title`, `defaultCollapsed`, `onToggle`, `sectionKey`, `children`
- Implement accessibility features (ARIA labels, keyboard navigation)

**Acceptance Criteria:**
- Component renders with proper styling matching Figma designs
- Smooth collapse/expand animation (300ms transition)
- Collapsed sections remain sticky at the top
- Keyboard accessible (Enter/Space to toggle)
- Unit tests for component behavior

**Dependencies:** None

---

#### Task 1.2: Implement Section State Persistence
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/hooks/`
**Estimated Effort:** 2-3 days

**Description:**
Create a hook to persist collapsed/expanded state of sections across page loads using browser localStorage.

**Implementation Details:**
- Create `useCollapsibleSectionState.ts` hook
- Store state in localStorage with key pattern: `gij-home-section-{sectionName}-collapsed`
- Return `{ isCollapsed, toggleCollapsed, setCollapsed }` interface
- Handle edge cases (localStorage unavailable, corrupted data)
- Add telemetry events for section collapse/expand actions

**Acceptance Criteria:**
- Section states persist across page reloads
- Graceful degradation if localStorage unavailable
- Telemetry events fire on state changes
- Unit tests for hook behavior

**Dependencies:** None

---

#### Task 1.3: Restructure Home Dashboard Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/dashboard/`
**Estimated Effort:** 5-7 days

**Description:**
Refactor `gij-home-dashboard.tsx` to support three distinct collapsible sections instead of current shortcuts-only layout.

**Implementation Details:**
- Update `GijHomeDashBoard` component structure:
  - Wrap in container with three sections
  - Section 1: Getting Started (initially empty, will be populated in Task 2.1)
  - Section 2: Shortcuts (migrated from current implementation)
  - Section 3: Delivery Dashboard (placeholder container)
- Update `home-dashboard.m.scss` with new layout:
  - Grid/Flexbox layout for sections
  - Responsive design for different viewport sizes
  - Proper spacing between sections
- Integrate `CollapsibleSection` component for each section
- Integrate `useCollapsibleSectionState` for persistence
- Update section keys: `getting-started`, `shortcuts`, `delivery-dashboard`

**Acceptance Criteria:**
- Three distinct sections render correctly
- Each section uses CollapsibleSection component
- Layout matches Figma designs
- Sections collapse/expand independently
- State persists across page loads
- Responsive design works on various screen sizes

**Dependencies:** Task 1.1, Task 1.2

---

### Phase 2: Getting Started Section

#### Task 2.1: Build Getting Started Content Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/getting-started/`
**Estimated Effort:** 5-7 days

**Description:**
Create the Getting Started section with video content and onboarding messaging controlled by PMM through engineering changes.

**Implementation Details:**
- Create `GettingStartedSection.tsx` component:
  - Embedded YouTube video player or modal-based video
  - Configurable content sections (title, description, bullets)
  - Support for inline and modal video display
- Create `getting-started-content.ts` configuration file:
  - Structured content data (PMM can request changes via tickets)
  - Video URL configuration
  - Onboarding text and messaging
- Create `GettingStartedVideo.tsx` for video player:
  - Use react-youtube or similar library
  - Support for modal and embedded display modes
  - Lazy loading for performance
- Add styling in `getting-started.module.scss`
- Add telemetry for video plays, content interactions

**Acceptance Criteria:**
- Video content displays correctly (embedded or modal)
- Content is configurable via static file changes
- Responsive design for various screen sizes
- Video lazy loads when section is visible
- Telemetry tracks user engagement
- Accessibility compliant (captions, ARIA labels)

**Dependencies:** Task 1.3

---

#### Task 2.2: Create PMM Content Update Documentation
**Repository:** `jiragitcloud`
**Location:** `dev-docs/home-page-content/`
**Estimated Effort:** 1 day

**Description:**
Document the process for PMM to request Getting Started content updates.

**Implementation Details:**
- Create `dev-docs/home-page-content/getting-started-updates.md`:
  - Explain content structure in `getting-started-content.ts`
  - Provide examples of content changes
  - Document the process: PMM creates ticket → Engineering updates file → Deploy
  - Include video URL update process
  - Include screenshot examples
- Add JSON schema for content validation

**Acceptance Criteria:**
- Clear documentation for non-engineers
- Examples of common content changes
- Process workflow documented

**Dependencies:** Task 2.1

---

### Phase 3: Shortcuts Section Update

#### Task 3.1: Refactor Shortcuts Display with Reduced Spacing
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/dashboard/`
**Estimated Effort:** 3-4 days

**Description:**
Update the existing shortcuts display to use reduced vertical spacing while maintaining current functionality.

**Implementation Details:**
- Update `home-dashboard.m.scss` shortcuts section:
  - Reduce vertical padding between shortcuts
  - Reduce shortcut card height
  - Adjust grid gap for tighter layout
  - Maintain horizontal spacing for readability
- Update `HomeShortcut` component usage (from `@bigbrassband/cloud-ui-common`):
  - Use compact variant if available
  - Adjust image sizes if needed
- Ensure shortcuts remain clickable and accessible
- Maintain existing telemetry events

**Acceptance Criteria:**
- Shortcuts consume less vertical space (approximately 40% reduction)
- All existing shortcuts remain functional
- Layout matches Figma designs
- No accessibility regressions
- Responsive design maintained

**Dependencies:** Task 1.3

---

### Phase 4: Delivery Dashboard Foundation

#### Task 4.1: Create Delivery Dashboard Container Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/`
**Estimated Effort:** 5-7 days

**Description:**
Build the Delivery Dashboard section container that will host Signals widgets in future tasks.

**Implementation Details:**
- Create `DeliveryDashboard.tsx` component:
  - Container layout for future widgets
  - Conditional rendering (hidden until widgets available)
  - Header with section title and info tooltip
  - Project selector integration
  - Empty state placeholder
- Create `delivery-dashboard.module.scss` for styling
- Create `DeliveryDashboardContext.tsx` for shared state:
  - Selected project state
  - Dashboard visibility state
  - Widget configuration state
- Add feature flag check: `showDeliveryDashboard` (initially false)
- Add telemetry for dashboard interactions

**Implementation Details - Conditional Rendering:**
```typescript
// Only render if feature flag is enabled AND widgets are available
if (!showDeliveryDashboard || widgets.length === 0) {
  return null; // Don't render empty container
}
```

**Acceptance Criteria:**
- Container renders correctly when enabled
- Section hidden by default (no widgets yet)
- Feature flag controls visibility
- Context provides shared state for future widgets
- Layout prepared for widget insertion
- Telemetry tracks dashboard visibility

**Dependencies:** Task 1.3

---

#### Task 4.2: Build Project Selector Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/components/`
**Estimated Effort:** 5-7 days

**Description:**
Create a project selector scoped to the Delivery Dashboard section for filtering widget data.

**Implementation Details:**
- Create `DashboardProjectSelector.tsx` component:
  - Dropdown/Select UI using Atlassian Design System
  - Fetch available Jira projects from existing API
  - Store selected project in `DeliveryDashboardContext`
  - Persist selected project in localStorage
  - Show "All Projects" option
- Add API integration:
  - Use existing project list API or create new one if needed
  - Handle loading, error states
  - Cache project list
- Create `useDashboardProjectSelector.ts` hook:
  - Manage selected project state
  - Provide project data to widgets
  - Handle project change events
- Add telemetry for project selection changes

**Acceptance Criteria:**
- Project selector displays available projects
- Selection persists across page loads
- Selected project updates dashboard context
- Loading and error states handled gracefully
- Telemetry tracks project selection
- Responsive design

**Dependencies:** Task 4.1

---

#### Task 4.3: Create Dashboard Empty State Component
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/components/`
**Estimated Effort:** 2-3 days

**Description:**
Build explanatory empty state component for when Signals data is unavailable for selected project.

**Implementation Details:**
- Create `DeliveryDashboardEmptyState.tsx` component:
  - Informative message about Signals data availability
  - Visual illustration or icon
  - Different messages based on context:
    - No project selected
    - Selected project has no Signals data
    - No repositories connected
  - Optional CTA to connect repositories or configure Signals
- Create `delivery-dashboard-empty-state.module.scss`
- Add telemetry for empty state displays

**Acceptance Criteria:**
- Clear, helpful messaging for each empty state scenario
- Visual design matches Figma
- CTA buttons work correctly
- Accessible markup
- Telemetry tracks empty state impressions

**Dependencies:** Task 4.1

---

#### Task 4.4: Implement Entitlement-Aware Dashboard Behavior
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/delivery-dashboard/`
**Estimated Effort:** 5-7 days

**Description:**
Add entitlement checking to Delivery Dashboard that differentiates behavior for Advanced vs Standard users.

**Implementation Details:**
- Create `useDeliveryDashboardEntitlement.ts` hook:
  - Check user's edition type (Standard, Advanced, Trial)
  - Return entitlement flags: `{ isAdvanced, isStandard, showUpgradePrompt }`
  - Integrate with existing `useIsSignalsEnabled` hook
- Update `DeliveryDashboard.tsx`:
  - Pass entitlement context to child components
  - Conditionally render upgrade prompts for Standard users
- Create `UpgradePromptModal.tsx` for Standard users:
  - Compelling explanation of Signals Advanced value
  - Video or visual demonstrations
  - CTA to upgrade (link to marketplace or contact sales)
  - Close/dismiss functionality
- Add telemetry:
  - Track upgrade prompt displays
  - Track upgrade CTA clicks
  - Track modal dismissals

**Acceptance Criteria:**
- Advanced users see full Signals navigation
- Standard users see upgrade-oriented experience
- Upgrade modal displays correctly for Standard users
- Entitlement checks are accurate
- Telemetry tracks all entitlement interactions
- No impact on existing Signals Advanced features

**Dependencies:** Task 4.1

---

### Phase 5: Integration & Polish

#### Task 5.1: Update Navigation and Routing
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/`
**Estimated Effort:** 2-3 days

**Description:**
Ensure navigation and routing properly support the new three-section home page structure.

**Implementation Details:**
- Update `gij-home-navigation.tsx` if needed:
  - Ensure Home route points to new dashboard
  - Verify navigation state management
- Update `use-gij-home-navigation.tsx`:
  - Add telemetry for new sections
  - Update navigation item tracking
- Test deep linking to home page
- Verify browser back/forward behavior

**Acceptance Criteria:**
- Navigation to home page works correctly
- Deep links work properly
- Browser navigation (back/forward) works
- Telemetry accurately tracks page visits
- No console errors or warnings

**Dependencies:** Task 1.3, Task 2.1, Task 3.1, Task 4.1

---

#### Task 5.2: Responsive Design Implementation
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/`
**Estimated Effort:** 5-7 days

**Description:**
Ensure all new home page sections are fully responsive across different viewport sizes.

**Implementation Details:**
- Test and adjust layouts for breakpoints:
  - Mobile: < 768px
  - Tablet: 768px - 1024px
  - Desktop: > 1024px
  - Wide: > 1440px
- Update CSS with media queries:
  - Stack sections vertically on mobile
  - Adjust grid layouts for tablet
  - Optimize spacing for each breakpoint
- Test shortcuts grid responsiveness
- Test Delivery Dashboard widget container responsiveness
- Ensure collapsible sections work on touch devices
- Test with browser dev tools device emulation

**Acceptance Criteria:**
- All sections display correctly at all breakpoints
- Touch interactions work on mobile devices
- No horizontal scrolling on any viewport size
- Content remains readable at all sizes
- Collapse/expand works on touch devices
- Visual regression tests pass

**Dependencies:** All Phase 1-4 tasks

---

#### Task 5.3: Add Comprehensive Telemetry
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/`
**Estimated Effort:** 3-4 days

**Description:**
Implement comprehensive OpenTelemetry tracking for all home page interactions.

**Implementation Details:**
- Add telemetry events for:
  - Page load and view
  - Section collapse/expand actions
  - Getting Started video plays
  - Shortcuts clicks (already exists, verify)
  - Project selector changes
  - Empty state displays
  - Entitlement-related interactions
  - Upgrade prompt displays and clicks
- Create `home-dashboard-telemetry.ts` helper:
  - Centralized event naming
  - Consistent event properties
  - Event typing for type safety
- Add event documentation in code comments
- Verify events in local development

**Acceptance Criteria:**
- All user interactions tracked
- Event names follow naming conventions
- Event properties are consistent and complete
- Events visible in local telemetry console
- Documentation for all tracked events
- Type-safe event tracking

**Dependencies:** All previous tasks

---

#### Task 5.4: Accessibility Audit & Improvements
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/`
**Estimated Effort:** 3-5 days

**Description:**
Conduct comprehensive accessibility audit and address any issues.

**Implementation Details:**
- Run automated accessibility tests:
  - axe DevTools browser extension
  - Lighthouse accessibility audit
  - WAVE browser extension
- Manual testing:
  - Keyboard navigation (Tab, Enter, Space, Escape)
  - Screen reader testing (NVDA, JAWS, VoiceOver)
  - Focus management
  - Color contrast validation
- Fix identified issues:
  - Add missing ARIA labels
  - Fix focus order
  - Ensure proper heading hierarchy
  - Add skip links if needed
  - Fix color contrast issues
- Document accessibility features

**Acceptance Criteria:**
- Lighthouse accessibility score > 90
- No critical axe issues
- Full keyboard navigation support
- Screen reader compatible
- WCAG 2.1 AA compliance
- Focus indicators visible and clear
- Proper ARIA landmarks

**Dependencies:** All previous tasks

---

#### Task 5.5: Create Feature Flag Management
**Repository:** `jiragitcloud`
**Location:** `docs/conf/` (backend) and `ui/code/src/features/` (frontend)
**Estimated Effort:** 2-3 days

**Description:**
Implement feature flag system to control Delivery Dashboard visibility and enable gradual rollout.

**Implementation Details:**
- Backend (Java):
  - Add feature flag in `docs/conf/feature-flags.yaml`:
    - `deliveryDashboard.enabled: false` (default)
    - `deliveryDashboard.showWithoutWidgets: false`
  - Create API endpoint to expose flags to frontend
  - Add flag to existing configuration system
- Frontend (TypeScript):
  - Create `useFeatureFlags.ts` hook
  - Add `showDeliveryDashboard` flag check
  - Conditionally render dashboard section based on flag
  - Add admin override for testing (localStorage)
- Documentation:
  - Document flag purpose and rollout plan
  - Create runbook for enabling in production

**Acceptance Criteria:**
- Feature flag controls dashboard visibility
- Flag can be toggled without code changes
- Admin override works for testing
- Documentation complete
- No errors when dashboard is disabled

**Dependencies:** Task 4.1

---

### Phase 6: Testing & Documentation

#### Task 6.1: Write Unit Tests
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/**/*.test.tsx`
**Estimated Effort:** 5-7 days

**Description:**
Create comprehensive unit test suite for all new components and hooks.

**Implementation Details:**
- Test coverage for:
  - `CollapsibleSection` component
  - `useCollapsibleSectionState` hook
  - `GettingStartedSection` component
  - `DeliveryDashboard` component
  - `DashboardProjectSelector` component
  - `DeliveryDashboardEmptyState` component
  - Entitlement hooks and logic
  - Refactored `GijHomeDashBoard` component
- Use React Testing Library
- Mock external dependencies (API calls, localStorage)
- Test edge cases and error scenarios
- Aim for >80% code coverage

**Acceptance Criteria:**
- All new components have unit tests
- All new hooks have unit tests
- Code coverage > 80%
- All tests pass in CI/CD
- Edge cases covered
- Mock dependencies properly

**Dependencies:** All implementation tasks

---

#### Task 6.2: Write Integration Tests
**Repository:** `jiragitcloud`
**Location:** `ui/code/src/blocks/gij-home/**/*.integration.test.tsx`
**Estimated Effort:** 3-5 days

**Description:**
Create integration tests for home page workflows.

**Implementation Details:**
- Test scenarios:
  - Full page load and render
  - Section collapse/expand workflows
  - State persistence across page reload
  - Project selector integration
  - Navigation between sections
  - Entitlement-based rendering
- Use React Testing Library with MSW for API mocking
- Test with different user roles (Admin, Standard, Advanced)
- Test with different feature flag states

**Acceptance Criteria:**
- All major user workflows tested
- Tests cover different user roles
- Tests cover different entitlement states
- API mocking works correctly
- All tests pass reliably

**Dependencies:** Task 6.1

---

#### Task 6.3: Write E2E Tests with Playwright
**Repository:** `jiragitcloud`
**Location:** `ui/code/tests/e2e/home-page/`
**Estimated Effort:** 5-7 days

**Description:**
Create end-to-end tests using Playwright for complete user journeys.

**Implementation Details:**
- Create test files:
  - `home-page-layout.spec.ts` - Basic layout and navigation
  - `collapsible-sections.spec.ts` - Section interactions
  - `getting-started.spec.ts` - Getting Started section
  - `delivery-dashboard.spec.ts` - Dashboard interactions
  - `entitlement-behavior.spec.ts` - Different user types
- Test real browser interactions:
  - Click, type, navigate
  - Verify visual state
  - Check persistence
  - Test responsive breakpoints
- Use Playwright fixtures for test data
- Add visual regression tests if applicable
- Document test scenarios

**Acceptance Criteria:**
- All critical user paths covered
- Tests run in CI/CD pipeline
- Tests are reliable (not flaky)
- Visual regression tests included
- Documentation for running tests locally

**Dependencies:** Task 6.2

---

#### Task 6.4: Update Developer Documentation
**Repository:** `jiragitcloud`
**Location:** `dev-docs/home-page/`
**Estimated Effort:** 2-3 days

**Description:**
Create comprehensive developer documentation for the new home page architecture.

**Implementation Details:**
- Create `dev-docs/home-page/architecture.md`:
  - Component hierarchy diagram
  - State management explanation
  - Data flow documentation
  - Feature flag usage
- Create `dev-docs/home-page/adding-widgets.md`:
  - Guide for adding new dashboard widgets
  - Widget interface requirements
  - Integration with project selector
  - Entitlement handling
- Update existing docs if needed
- Add code examples and snippets
- Include troubleshooting section

**Acceptance Criteria:**
- Clear architecture documentation
- Widget development guide complete
- Code examples are accurate
- Easy to follow for new developers
- Linked from main README

**Dependencies:** All implementation tasks

---

#### Task 6.5: Create User-Facing Documentation
**Repository:** `jiragitcloud`
**Location:** Documentation site (external) or in-app help
**Estimated Effort:** 2-3 days

**Description:**
Create user documentation explaining the new home page features.

**Implementation Details:**
- Document sections:
  - Overview of new home page layout
  - Getting Started section usage
  - Collapsible sections feature
  - Delivery Dashboard introduction (when enabled)
  - Project selector usage
- Create screenshots or animated GIFs
- Explain entitlement differences (Standard vs Advanced)
- Create FAQ section
- Coordinate with PMM for messaging

**Acceptance Criteria:**
- Clear, user-friendly documentation
- Screenshots/visuals included
- Covers all new features
- PMM review completed
- Published to appropriate location

**Dependencies:** All implementation tasks

---

### Phase 7: Deployment & Rollout

#### Task 7.1: Staged Rollout Plan
**Repository:** `jiragitcloud`
**Location:** `dev-docs/deployment/`
**Estimated Effort:** 1-2 days

**Description:**
Create detailed rollout plan for gradual feature deployment.

**Implementation Details:**
- Define rollout stages:
  - Stage 1: Internal testing (feature flag enabled for GitKraken org only)
  - Stage 2: Beta customers (opt-in list)
  - Stage 3: Percentage rollout (10% → 25% → 50% → 100%)
  - Stage 4: Full release
- Document rollback procedures
- Define success metrics for each stage
- Create monitoring dashboard queries
- Define go/no-go criteria

**Acceptance Criteria:**
- Clear rollout stages defined
- Rollback plan documented
- Success metrics identified
- Monitoring queries ready
- Stakeholder approval obtained

**Dependencies:** All testing tasks

---

#### Task 7.2: Performance Testing & Optimization
**Repository:** `jiragitcloud`
**Location:** Various
**Estimated Effort:** 3-5 days

**Description:**
Conduct performance testing and optimize any bottlenecks.

**Implementation Details:**
- Measure performance metrics:
  - Time to First Contentful Paint (FCP)
  - Time to Interactive (TTI)
  - Largest Contentful Paint (LCP)
  - Cumulative Layout Shift (CLS)
- Profile with Chrome DevTools:
  - Identify expensive renders
  - Check bundle size impact
  - Analyze network requests
- Optimize as needed:
  - Code splitting for large components
  - Lazy loading for sections
  - Memoization for expensive computations
  - Image optimization
- Run Lighthouse audits (target >90 performance score)

**Acceptance Criteria:**
- Lighthouse performance score > 90
- No layout shifts during load
- TTI < 3 seconds on 3G
- Bundle size increase < 50KB gzipped
- No performance regressions

**Dependencies:** All implementation tasks

---

#### Task 7.3: Production Deployment
**Repository:** `jiragitcloud`
**Location:** Various
**Estimated Effort:** 1-2 days

**Description:**
Deploy the new home page to production with feature flag disabled initially.

**Implementation Details:**
- Merge all PRs to main branch
- Deploy to staging environment
- Run full test suite in staging
- Smoke test all functionality
- Deploy to production
- Verify feature flag is disabled
- Monitor error rates and performance
- Document deployment completion

**Acceptance Criteria:**
- Code deployed to production
- Feature flag disabled by default
- No errors in production logs
- Performance metrics normal
- Rollback plan ready if needed

**Dependencies:** Task 7.1, Task 7.2

---

#### Task 7.4: Internal Beta Testing
**Repository:** `jiragitcloud`
**Location:** N/A (operational task)
**Estimated Effort:** 3-5 days

**Description:**
Enable feature for internal GitKraken organization and conduct beta testing.

**Implementation Details:**
- Enable feature flag for GitKraken org install
- Notify internal team members
- Collect feedback via:
  - Slack channel
  - Feedback form
  - Direct interviews
- Monitor telemetry data:
  - Feature usage
  - Error rates
  - Performance metrics
- Iterate on feedback:
  - Fix critical bugs
  - Address UX issues
  - Update documentation
- Get approval to proceed to next stage

**Acceptance Criteria:**
- Feature enabled for internal users
- Feedback collected from >10 users
- Critical issues resolved
- Telemetry data validates design
- Approval to proceed obtained

**Dependencies:** Task 7.3

---

#### Task 7.5: Gradual Public Rollout
**Repository:** `jiragitcloud`
**Location:** N/A (operational task)
**Estimated Effort:** 2-3 weeks

**Description:**
Gradually enable the feature for increasing percentages of users.

**Implementation Details:**
- Follow rollout plan from Task 7.1
- Enable for beta customers (opt-in list)
- Monitor metrics at each stage:
  - Error rates
  - User engagement
  - Performance
  - Support tickets
- Increase percentage incrementally: 10% → 25% → 50% → 100%
- Pause rollout if issues detected
- Communicate with stakeholders at each stage
- Document lessons learned

**Acceptance Criteria:**
- Feature rolled out to 100% of users
- No increase in error rates
- User engagement metrics positive
- No increase in support tickets
- Post-mortem document created

**Dependencies:** Task 7.4

---

## Summary

**Total Tasks:** 29
**Estimated Total Effort:** 85-120 developer days (approximately 4-6 months for 1-2 developers)

**Critical Path:**
1. Phase 1: Infrastructure (Tasks 1.1 → 1.2 → 1.3)
2. Phase 4: Dashboard Foundation (Tasks 4.1 → 4.2 → 4.3)
3. Phase 5: Integration (Tasks 5.1 → 5.2)
4. Phase 6: Testing (Tasks 6.1 → 6.2 → 6.3)
5. Phase 7: Deployment (Tasks 7.1 → 7.2 → 7.3 → 7.4 → 7.5)

**Parallel Workstreams:**
- Phase 2 (Getting Started) can be developed in parallel with Phase 4 (Dashboard Foundation)
- Phase 3 (Shortcuts) can be developed independently after Task 1.3
- Testing (Phase 6) can begin as soon as components are complete

**Key Risks:**
1. Delivery Dashboard hidden until widgets available - must coordinate with widget implementation epics
2. Responsive design complexity across different screen sizes
3. Entitlement logic integration with existing Signals system
4. Performance impact from additional sections and components

**Success Metrics:**
- Week-over-week increase in home page interaction
- Reduction in bounce rate from home page
- Increased Signals feature discovery (Advanced users)
- Positive user feedback on new layout
