# CMS Functional Requirements

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3.0-CMS-FR |
| **Version** | 1.0 |
| **Date** | 2026-01-20 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Sprint** | WS3.0-S2 |
| **Task** | S2.1 |
| **Related Documents** | SS-WS3.0-CDM, SS-WS3.0-CLS, SS-WS3.0-ORG |

---

## 1. Executive Summary

This document defines the comprehensive functional requirements for the Sommelier Spark Content Management System (CMS). The CMS enables content administrators to create, manage, review, and publish educational content including wines, modules, lessons, quizzes, and scenarios.

**Key Statistics:**
- **4 User Personas**: Content Admin, Org Admin, Content Author, Domain Expert
- **8 Requirement Categories**: Wine, Module, Quiz, Scenario, Search, Bulk Ops, Reporting, Audit
- **Priority Distribution**: Must (core), Should (important), Could (nice-to-have)

---

## 2. User Personas

### 2.1 Persona Definitions

| Persona | Role | Scope | Primary Responsibilities |
|---------|------|-------|--------------------------|
| **Content Admin** | Global Administrator | All content | Manage all content, approve publications, maintain global library, configure system settings |
| **Org Admin** | Organization Administrator | Organization content | Manage org-specific wines, custom quizzes, team content, view org reports |
| **Content Author** | Contributor | Own content | Create and edit content drafts, submit for review, respond to feedback |
| **Domain Expert** | Reviewer | Review queue | Review content for accuracy, provide feedback, approve/reject submissions |

### 2.2 Persona Permissions Matrix

| Capability | Content Admin | Org Admin | Content Author | Domain Expert |
|------------|---------------|-----------|----------------|---------------|
| Create global content | ✓ | — | ✓ | — |
| Create org content | ✓ | ✓ | — | — |
| Edit any content | ✓ | Org only | Own only | — |
| Submit for review | ✓ | ✓ | ✓ | — |
| Review content | ✓ | — | — | ✓ |
| Approve/Publish | ✓ | Org only | — | — |
| Archive content | ✓ | Org only | — | — |
| View all content | ✓ | Org only | Own + Published | Review queue |
| Manage users | ✓ | Org only | — | — |
| View reports | ✓ | Org only | Own only | — |
| Configure settings | ✓ | Org only | — | — |

---

## 3. Priority Definitions

| Priority | Code | Definition | Implementation |
|----------|------|------------|----------------|
| **Must** | M | Required for minimum viable product. System cannot launch without this. | Sprint 2-3 |
| **Should** | S | Important functionality that significantly improves usability. | Sprint 4-5 |
| **Could** | C | Nice-to-have features that enhance experience but aren't critical. | Future sprints |

---

## 4. Wine Management Requirements

### 4.1 Core Wine CRUD Operations

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-W-001 | Admin can create a new wine entry with all required attributes (name, region, country, wineType, grapeVarieties, priceTier) | Must | Validate required fields |
| CMS-W-002 | Admin can edit any attribute of an existing wine | Must | Track changes in audit log |
| CMS-W-003 | Admin can view wine details in a read-only preview mode | Must | Match learner view |
| CMS-W-004 | Admin can delete a wine in DRAFT status | Must | Soft delete with 30-day recovery |
| CMS-W-005 | Admin can archive a published wine | Must | 7-day warning period |
| CMS-W-006 | Admin can restore an archived wine to draft status | Should | Requires re-review |

### 4.2 Wine Content Management

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-W-007 | Admin can enter Quick Facts (Level 1) content for a wine | Must | Required for all wines |
| CMS-W-008 | Admin can enter Detailed Profile (Level 2) content for a wine | Must | Optional, unlocks at Silver |
| CMS-W-009 | Admin can enter Expert Insights (Level 3) content for a wine | Must | Optional, unlocks at Gold |
| CMS-W-010 | Admin can upload a wine bottle image (JPG, PNG, WebP, max 5MB) | Should | Auto-resize and optimize |
| CMS-W-011 | Admin can add multiple food pairings to a wine | Must | Tag-based selection |
| CMS-W-012 | Admin can specify serving temperature range | Should | Dropdown with common ranges |

### 4.3 Wine Relationships and References

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-W-013 | Admin can link a wine to one or more quizzes | Must | Bi-directional linking |
| CMS-W-014 | Admin can link a wine to one or more scenarios | Must | With talking points |
| CMS-W-015 | System displays where a wine is referenced (quizzes, scenarios) | Must | Dependency view |
| CMS-W-016 | System prevents archiving wine if referenced by published content | Must | Show dependencies |

### 4.4 Wine Import/Export

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-W-017 | Admin can bulk import wines from CSV file | Must | Template provided |
| CMS-W-018 | System validates CSV data before import and shows errors | Must | Row-by-row validation |
| CMS-W-019 | Admin can export all wines to CSV | Should | Include all attributes |
| CMS-W-020 | Admin can export selected wines to CSV | Should | Checkbox selection |
| CMS-W-021 | Admin can duplicate an existing wine entry | Could | Creates draft copy |

### 4.5 Wine Validation and Quality

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-W-022 | System validates wine data on save (required fields, formats) | Must | Real-time validation |
| CMS-W-023 | System warns if Quick Facts content is below minimum length | Should | Configurable threshold |
| CMS-W-024 | System checks for duplicate wines (name + vintage + producer) | Should | Suggest existing match |

---

## 5. Module Management Requirements

### 5.1 Core Module CRUD Operations

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-M-001 | Admin can create a new module with title, description, and category | Must | Validate required fields |
| CMS-M-002 | Admin can edit module metadata (title, description, category) | Must | Track changes |
| CMS-M-003 | Admin can delete a module in DRAFT status | Must | Cascades to lessons |
| CMS-M-004 | Admin can archive a published module | Must | Lessons also archived |
| CMS-M-005 | Admin can view module as learner would see it | Must | Full preview mode |

### 5.2 Lesson Management Within Modules

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-M-006 | Admin can add a new lesson to a module | Must | Inline or separate page |
| CMS-M-007 | Admin can edit lesson content using rich text editor | Must | HTML output |
| CMS-M-008 | Admin can reorder lessons within a module via drag-and-drop | Must | Update sortOrder |
| CMS-M-009 | Admin can remove a lesson from a module | Must | Confirm if published |
| CMS-M-010 | Admin can move a lesson to a different module | Should | Cross-module move |
| CMS-M-011 | Admin can duplicate a lesson within or across modules | Could | Creates draft copy |

### 5.3 Module Configuration

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-M-012 | Admin can set module tier classification (Bronze/Silver/Gold) | Must | Affects unlock rules |
| CMS-M-013 | Admin can set estimated completion time (auto-calculated from lessons) | Must | Editable override |
| CMS-M-014 | Admin can upload module thumbnail image | Should | Display in catalog |
| CMS-M-015 | Admin can set module prerequisites (other modules that must be completed first) | Should | Dependency chain |
| CMS-M-016 | Admin can associate a quiz with a module | Must | One quiz per module |

---

## 6. Quiz Management Requirements

### 6.1 Core Quiz CRUD Operations

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-Q-001 | Admin can create a new quiz with title, description, and tier | Must | Validate required fields |
| CMS-Q-002 | Admin can edit quiz metadata | Must | Track changes |
| CMS-Q-003 | Admin can delete a quiz in DRAFT status | Must | Cascades to questions |
| CMS-Q-004 | Admin can archive a published quiz | Must | Preserve attempt history |
| CMS-Q-005 | Admin can duplicate a quiz | Should | Creates draft copy |
| CMS-Q-006 | Admin can preview quiz as learner would experience it | Must | Full simulation |

### 6.2 Question Management

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-Q-007 | Admin can add a multiple choice question (single correct answer) | Must | 2-6 options |
| CMS-Q-008 | Admin can add a multiple select question (multiple correct answers) | Must | 2-6 options |
| CMS-Q-009 | Admin can add a true/false question | Must | Two options |
| CMS-Q-010 | Admin can add a matching question | Could | Pair items |
| CMS-Q-011 | Admin can edit question text and options | Must | Track changes |
| CMS-Q-012 | Admin can set which option(s) are correct | Must | Required |
| CMS-Q-013 | Admin can add explanation text shown after answering | Must | Educational feedback |
| CMS-Q-014 | Admin can reorder questions within a quiz via drag-and-drop | Must | Update sortOrder |
| CMS-Q-015 | Admin can remove a question from a quiz | Must | Confirm action |
| CMS-Q-016 | Admin can set question difficulty (Easy/Medium/Hard) | Should | Analytics use |

### 6.3 Quiz Configuration

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-Q-017 | Admin can set pass threshold percentage (default: Bronze 70%, Silver 80%, Gold 90%) | Must | Tier-based defaults |
| CMS-Q-018 | Admin can set time limit in minutes (optional) | Should | Countdown timer |
| CMS-Q-019 | Admin can enable/disable question randomization | Should | Per-quiz setting |
| CMS-Q-020 | Admin can enable/disable answer option randomization | Should | Per-quiz setting |
| CMS-Q-021 | Admin can set quiz category | Must | For filtering/grouping |
| CMS-Q-022 | Admin can set unlock conditions (prerequisite quizzes/modules) | Must | Tier progression |

### 6.4 Question Bank

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-Q-023 | Admin can view all questions in a searchable question bank | Should | Cross-quiz view |
| CMS-Q-024 | Admin can filter question bank by category, tier, type, difficulty | Should | Multi-filter |
| CMS-Q-025 | Admin can add existing question from bank to a quiz | Should | Reuse questions |
| CMS-Q-026 | Admin can import questions from CSV | Should | Bulk creation |
| CMS-Q-027 | Admin can export questions to CSV | Should | Backup/transfer |

### 6.5 Wine References in Questions

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-Q-028 | Admin can link a question to one or more wines | Must | Context for learning |
| CMS-Q-029 | System displays wine details when editing linked question | Should | Quick reference |

---

## 7. Scenario Management Requirements

### 7.1 Core Scenario CRUD Operations

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SC-001 | Admin can create a new scenario with title, description, category, and difficulty | Must | Validate required fields |
| CMS-SC-002 | Admin can edit scenario metadata | Must | Track changes |
| CMS-SC-003 | Admin can delete a scenario in DRAFT status | Must | Cascades to steps |
| CMS-SC-004 | Admin can archive a published scenario | Must | Preserve attempt history |
| CMS-SC-005 | Admin can duplicate a scenario | Should | Creates draft copy |

### 7.2 Customer Persona Configuration

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SC-006 | Admin can set customer name and persona description | Must | Character background |
| CMS-SC-007 | Admin can set initial situation/context | Must | Scene setup |
| CMS-SC-008 | Admin can set estimated completion time | Must | Display to learners |

### 7.3 Decision Tree Building

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SC-009 | Admin can add a new step to a scenario | Must | Sequential or branched |
| CMS-SC-010 | Admin can set step type (Dialogue, Action, Observation, Conclusion) | Must | Affects display |
| CMS-SC-011 | Admin can enter customer dialogue text for a step | Must | What customer says |
| CMS-SC-012 | Admin can set customer mood for a step | Should | Emotional context |
| CMS-SC-013 | Admin can add choices to a step (2-4 options) | Must | Staff responses |
| CMS-SC-014 | Admin can set points for each choice (0-10) | Must | Scoring |
| CMS-SC-015 | Admin can mark optimal choice | Must | Best answer flag |
| CMS-SC-016 | Admin can add feedback text for each choice | Must | Learning explanation |
| CMS-SC-017 | Admin can link choice to next step (branching) or end scenario | Must | Decision tree |
| CMS-SC-018 | Admin can view scenario as visual flowchart | Should | Tree visualization |
| CMS-SC-019 | Admin can reorder steps via drag-and-drop | Should | Adjust flow |

### 7.4 Scenario Testing and Preview

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SC-020 | Admin can preview scenario as learner would experience it | Must | Full playthrough |
| CMS-SC-021 | Admin can test all paths through scenario | Should | Path coverage |
| CMS-SC-022 | System validates all steps are reachable | Must | No orphan steps |
| CMS-SC-023 | System validates at least one path reaches conclusion | Must | Completable |

### 7.5 Scenario Wine References

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SC-024 | Admin can link wines to a scenario | Must | Featured wines |
| CMS-SC-025 | Admin can add talking points for each linked wine | Should | Staff guidance |

---

## 8. Search and Filtering Requirements

### 8.1 Full-Text Search

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SR-001 | Admin can search across all content types with a single search box | Must | Global search |
| CMS-SR-002 | Search indexes wine name, producer, region, country, tasting notes | Must | Full-text |
| CMS-SR-003 | Search indexes module and lesson titles and content | Must | Full-text |
| CMS-SR-004 | Search indexes quiz and question text | Must | Full-text |
| CMS-SR-005 | Search indexes scenario titles and dialogue | Must | Full-text |
| CMS-SR-006 | Search results display content type, status, and relevance | Must | Result cards |

### 8.2 Filtering

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SR-007 | Admin can filter content by type (Wine, Module, Quiz, Scenario) | Must | Type tabs/dropdown |
| CMS-SR-008 | Admin can filter content by status (Draft, Review, Published, Archived) | Must | Status filter |
| CMS-SR-009 | Admin can filter content by tier (Bronze, Silver, Gold) | Must | Tier filter |
| CMS-SR-010 | Admin can filter content by category | Must | Category filter |
| CMS-SR-011 | Admin can filter content by author | Should | Author dropdown |
| CMS-SR-012 | Admin can filter content by date range (created, modified, published) | Should | Date picker |
| CMS-SR-013 | Admin can combine multiple filters (AND logic) | Must | Multi-filter |

### 8.3 Saved Searches

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-SR-014 | Admin can save current search/filter as named preset | Could | Quick access |
| CMS-SR-015 | Admin can load saved search preset | Could | Restore filters |
| CMS-SR-016 | Admin can delete saved search preset | Could | Cleanup |

---

## 9. Bulk Operations Requirements

### 9.1 Bulk Selection

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-BK-001 | Admin can select multiple content items via checkboxes | Must | Multi-select |
| CMS-BK-002 | Admin can select all visible items | Must | Select all |
| CMS-BK-003 | Admin can deselect all items | Must | Clear selection |
| CMS-BK-004 | System displays count of selected items | Must | Selection counter |

### 9.2 Bulk Actions

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-BK-005 | Admin can bulk submit selected items for review | Should | Batch workflow |
| CMS-BK-006 | Admin can bulk publish selected items (if approved) | Should | Batch publish |
| CMS-BK-007 | Admin can bulk archive selected items | Should | Batch archive |
| CMS-BK-008 | Admin can bulk change tier for selected items | Should | Batch update |
| CMS-BK-009 | Admin can bulk change category for selected items | Should | Batch update |
| CMS-BK-010 | Admin can bulk assign selected items to an organization | Should | Org scoping |
| CMS-BK-011 | Admin can bulk export selected items | Should | CSV/JSON export |
| CMS-BK-012 | Admin can bulk delete selected draft items | Should | Batch delete |

### 9.3 Bulk Operation Safeguards

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-BK-013 | System requires confirmation for bulk destructive actions | Must | Confirm dialog |
| CMS-BK-014 | System shows progress during bulk operations | Should | Progress bar |
| CMS-BK-015 | System reports success/failure count after bulk operation | Must | Result summary |

---

## 10. Reporting Requirements

### 10.1 Content Inventory Reports

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-RP-001 | Admin can view total content count by type | Must | Dashboard widget |
| CMS-RP-002 | Admin can view content count by status (Draft, Review, Published, Archived) | Must | Status breakdown |
| CMS-RP-003 | Admin can view content count by tier | Should | Tier breakdown |
| CMS-RP-004 | Admin can view content count by category | Should | Category breakdown |

### 10.2 Content Quality Reports

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-RP-005 | Admin can view list of wines missing Level 2 or Level 3 content | Should | Content gaps |
| CMS-RP-006 | Admin can view quizzes with fewer than 10 questions | Should | Quality check |
| CMS-RP-007 | Admin can view modules with no associated quiz | Should | Completeness check |
| CMS-RP-008 | Admin can view content pending review for more than X days | Must | Review bottlenecks |

### 10.3 Author Activity Reports

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-RP-009 | Admin can view content created by each author | Should | Author productivity |
| CMS-RP-010 | Admin can view content published by each author | Should | Publication stats |
| CMS-RP-011 | Admin can view review activity by each domain expert | Should | Reviewer workload |

### 10.4 Report Export

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-RP-012 | Admin can export any report to CSV | Should | Data export |
| CMS-RP-013 | Admin can export any report to PDF | Could | Printable format |

---

## 11. Audit Logging Requirements

### 11.1 Action Logging

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-AU-001 | System logs all content create actions with user, timestamp, and content ID | Must | Create audit |
| CMS-AU-002 | System logs all content update actions with user, timestamp, and changed fields | Must | Update audit |
| CMS-AU-003 | System logs all content delete actions with user, timestamp, and content snapshot | Must | Delete audit |
| CMS-AU-004 | System logs all status changes (Draft→Review→Published→Archived) | Must | State audit |
| CMS-AU-005 | System logs all review actions (approve, reject, request changes) | Must | Review audit |
| CMS-AU-006 | System logs all bulk operations with affected item count | Must | Bulk audit |

### 11.2 Audit Log Access

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-AU-007 | Admin can view audit log for any content item | Must | Per-item history |
| CMS-AU-008 | Admin can search audit log by user, action type, date range | Should | Log search |
| CMS-AU-009 | Admin can filter audit log by content type | Should | Type filter |
| CMS-AU-010 | Admin can export audit log to CSV | Should | Compliance export |

### 11.3 Audit Retention

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-AU-011 | System retains audit logs for minimum 7 years | Must | Compliance |
| CMS-AU-012 | System archives audit logs older than 2 years to cold storage | Should | Cost optimization |

---

## 12. Non-Functional Requirements

### 12.1 Performance

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-NF-001 | Page load time under 3 seconds for content lists | Must | UX |
| CMS-NF-002 | Search results returned within 2 seconds | Must | UX |
| CMS-NF-003 | Autosave draft every 30 seconds | Should | Data protection |

### 12.2 Usability

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-NF-004 | CMS is fully responsive on tablet devices | Should | Mobile admin |
| CMS-NF-005 | All actions have clear success/error feedback | Must | UX |
| CMS-NF-006 | Unsaved changes prompt user before navigation | Must | Data protection |

### 12.3 Security

| Req ID | Requirement | Priority | Notes |
|--------|-------------|----------|-------|
| CMS-NF-007 | All CMS actions require authenticated user | Must | Security |
| CMS-NF-008 | CMS enforces role-based access control per persona | Must | Authorization |
| CMS-NF-009 | CMS session timeout after 8 hours of inactivity | Must | Security |

---

## 13. Requirements Summary

### 13.1 Requirements by Category

| Category | Code Prefix | Count |
|----------|-------------|-------|
| Wine Management | CMS-W | 24 |
| Module Management | CMS-M | 16 |
| Quiz Management | CMS-Q | 29 |
| Scenario Management | CMS-SC | 25 |
| Search and Filtering | CMS-SR | 16 |
| Bulk Operations | CMS-BK | 15 |
| Reporting | CMS-RP | 13 |
| Audit Logging | CMS-AU | 12 |
| Non-Functional | CMS-NF | 9 |
| **Total** | | **159** |

### 13.2 Requirements by Priority

| Priority | Count | Percentage |
|----------|-------|------------|
| Must | 89 | 56% |
| Should | 57 | 36% |
| Could | 13 | 8% |
| **Total** | **159** | **100%** |

### 13.3 Must-Have Requirements by Category

| Category | Must Count |
|----------|------------|
| Wine Management | 15 |
| Module Management | 10 |
| Quiz Management | 16 |
| Scenario Management | 14 |
| Search and Filtering | 9 |
| Bulk Operations | 5 |
| Reporting | 2 |
| Audit Logging | 8 |
| Non-Functional | 6 |
| **Total Must** | **85** |

---

## 14. Appendix

### 14.1 Glossary

| Term | Definition |
|------|------------|
| CMS | Content Management System |
| CRUD | Create, Read, Update, Delete |
| Tier | Content difficulty level (Bronze, Silver, Gold) |
| Status | Content lifecycle state (Draft, Review, Published, Archived) |
| Bulk Operation | Action applied to multiple items simultaneously |

### 14.2 Related Documents

| Document | ID | Relevance |
|----------|----|-----------|
| Content Domain Model | SS-WS3.0-CDM | Entity definitions |
| Content Lifecycle Specification | SS-WS3.0-CLS | State management |
| Organization Model | SS-WS3.0-ORG | User roles and permissions |

### 14.3 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-20 | Obi Wan | Initial draft |

---

*End of Document*
