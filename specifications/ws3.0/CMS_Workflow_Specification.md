# CMS Workflow Specification

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3.0-CMS-WF |
| **Version** | 1.0 |
| **Date** | 2026-01-20 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Sprint** | WS3.0-S2 |
| **Task** | S2.2 |
| **Related Documents** | SS-WS3.0-CLS, SS-WS3.0-CMS-FR |

---

## 1. Executive Summary

This document defines all content management workflows for the Sommelier Spark CMS. It provides detailed step-by-step processes, decision points, error handling, and notifications for each workflow stage from content creation through archival.

**Key Statistics:**
- **6 Core Workflows**: Creation, Review, Publication, Revision, Archival, Emergency
- **8 Workflow Diagrams**: Visual representation of each process
- **12 Notification Types**: Event-driven communication
- **4 SLA Tiers**: Service level commitments

---

## 2. Workflow Overview

### 2.1 Workflow Summary

| Workflow | Purpose | Primary Actors | Typical Duration |
|----------|---------|----------------|------------------|
| Content Creation | Create new content items | Author | 30 min - 2 hours |
| Review/Approval | Validate content quality | Reviewer, Admin | 1-5 days |
| Publication | Make content live | Admin, System | Immediate - Scheduled |
| Revision | Update published content | Author, Reviewer | 1-3 days |
| Archival | Retire content | Admin | 7-day warning + immediate |
| Emergency | Urgent fixes | Senior Admin | 2 hours |

### 2.2 Complete Workflow Lifecycle

```mermaid
flowchart TD
    subgraph Creation["1. Content Creation"]
        C1[Create New] --> C2[Edit Draft]
        C2 --> C3[Save Draft]
        C3 --> C4{Complete?}
        C4 -->|No| C2
        C4 -->|Yes| C5[Submit for Review]
    end

    subgraph Review["2. Review/Approval"]
        R1[Assign Reviewer] --> R2[Review Content]
        R2 --> R3{Decision}
        R3 -->|Changes Needed| R4[Request Changes]
        R3 -->|Approved| R5[Approve]
        R4 --> C2
    end

    subgraph Publication["3. Publication"]
        P1{Publish Type}
        P1 -->|Immediate| P2[Publish Now]
        P1 -->|Scheduled| P3[Schedule]
        P3 --> P4[Auto-Publish at Time]
        P2 --> P5[Content Live]
        P4 --> P5
    end

    subgraph Revision["4. Revision"]
        V1[Create Revision] --> V2[Edit Draft]
        V2 --> V3[Submit for Review]
        V3 --> R1
    end

    subgraph Archival["5. Archival"]
        A1[Request Archive] --> A2{Dependencies?}
        A2 -->|Yes| A3[Resolve Dependencies]
        A2 -->|No| A4[Warning Period]
        A3 --> A4
        A4 --> A5[Archive]
    end

    C5 --> R1
    R5 --> P1
    P5 -->|Update| V1
    P5 -->|Retire| A1
```

---

## 3. Content Creation Workflow

### 3.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Author Initiates]) --> Step1[Click 'Create New']
    Step1 --> Step2[Select Content Type]
    Step2 --> Step3[Enter Required Fields]
    Step3 --> Val1{Validation}
    Val1 -->|Fail| Err1[Show Validation Errors]
    Err1 --> Step3
    Val1 -->|Pass| Step4[Save as Draft]
    Step4 --> Step5{Continue Editing?}
    Step5 -->|Yes| Step6[Edit Additional Fields]
    Step6 --> Step7[Auto-save Every 30s]
    Step7 --> Step5
    Step5 -->|No| Step8{Ready for Review?}
    Step8 -->|No| End1([Save and Exit])
    Step8 -->|Yes| Step9[Submit for Review]
    Step9 --> Val2{Final Validation}
    Val2 -->|Fail| Err2[Show Missing Requirements]
    Err2 --> Step6
    Val2 -->|Pass| End2([Enter Review Queue])
```

### 3.2 Step-by-Step Process

#### Step 1: Initiate Creation

| Attribute | Details |
|-----------|---------|
| **Actor** | Content Author, Content Admin, Org Admin |
| **Action** | Click "Create New" button in CMS |
| **Screen** | Content type selection modal |
| **Options** | Wine, Module, Lesson, Quiz, Question, Scenario |
| **Next** | Step 2 |

#### Step 2: Select Content Type

| Attribute | Details |
|-----------|---------|
| **Actor** | Author |
| **Action** | Select content type from options |
| **Screen** | Type selection with descriptions |
| **Validation** | User has permission for selected type |
| **Error** | "You don't have permission to create [type]" |
| **Next** | Step 3 |

#### Step 3: Enter Required Fields

| Attribute | Details |
|-----------|---------|
| **Actor** | Author |
| **Action** | Fill in required fields for content type |
| **Screen** | Content creation form |
| **Required Fields** | Varies by type (see Content Domain Model) |
| **Real-time Validation** | Field-level validation on blur |
| **Error Display** | Inline error messages below fields |
| **Next** | Step 4 (after validation passes) |

**Required Fields by Content Type:**

| Content Type | Required Fields |
|--------------|-----------------|
| Wine | name, region, country, wineType, grapeVarieties, priceTier |
| Module | title, description, category |
| Lesson | title, content, moduleId |
| Quiz | title, tier, passingScore |
| Question | question, questionType, options (min 2) |
| Scenario | title, description, category, difficulty, customerName |

#### Step 4: Save as Draft

| Attribute | Details |
|-----------|---------|
| **Actor** | System (automatic) or Author (manual) |
| **Action** | Save content with status = DRAFT |
| **System Actions** | - Generate unique ID |
| | - Set createdAt, createdBy |
| | - Set version = 1.0 |
| | - Create audit log entry |
| **Feedback** | "Draft saved successfully" toast |
| **Next** | Step 5 |

#### Step 5: Continue Editing (Optional)

| Attribute | Details |
|-----------|---------|
| **Actor** | Author |
| **Action** | Edit additional/optional fields |
| **Screen** | Full content editor |
| **Auto-save** | Every 30 seconds while editing |
| **Manual Save** | Ctrl+S or "Save" button |
| **Exit Options** | "Save and Close", "Save and Continue" |
| **Next** | Step 6 or Step 7 |

#### Step 6: Submit for Review

| Attribute | Details |
|-----------|---------|
| **Actor** | Author |
| **Action** | Click "Submit for Review" button |
| **Pre-submission Check** | Final validation of all required fields |
| **Validation Errors** | Block submission, show missing items |
| **Success** | Status changes to REVIEW |
| **System Actions** | - Lock content for editing |
| | - Set reviewRequestedAt |
| | - Assign reviewer (auto or manual) |
| | - Send notifications |
| | - Create audit log entry |
| **Next** | Review Workflow |

### 3.3 Error Handling

| Error Scenario | User Message | Recovery Action |
|----------------|--------------|-----------------|
| Required field missing | "Please fill in [field name]" | Highlight field, scroll to first error |
| Invalid format | "[Field] must be [format]" | Show format hint |
| Duplicate detected | "Similar content exists: [link]" | Offer to view existing or continue |
| Network error on save | "Save failed. Retrying..." | Auto-retry 3 times, then manual retry |
| Session expired | "Session expired. Please log in." | Save to local storage, prompt login |
| Permission denied | "You don't have permission" | Redirect to dashboard |

---

## 4. Review/Approval Workflow

### 4.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Content Submitted]) --> Step1[System Assigns Reviewer]
    Step1 --> Notify1[/Notify Reviewer/]
    Notify1 --> Step2[Reviewer Opens Content]
    Step2 --> Step3[Review Content]
    Step3 --> Step4{Quality Check}

    Step4 -->|Issues Found| Path1[Add Comments]
    Path1 --> Dec1{Severity}
    Dec1 -->|Minor| Action1[Request Changes]
    Dec1 -->|Major| Action2[Reject]

    Action1 --> Notify2[/Notify Author/]
    Notify2 --> Return1([Return to Draft])

    Action2 --> Notify3[/Notify Author/]
    Notify3 --> Return2([Return to Draft])

    Step4 -->|Approved| Step5{Workflow Type}

    Step5 -->|Simple| Final[Content Admin Approves]
    Step5 -->|Standard| Expert[Domain Expert Reviews]
    Step5 -->|Extended| QA[QA Reviews First]

    QA --> Expert
    Expert --> Final

    Final --> Notify4[/Notify Author/]
    Notify4 --> End([Enter Publication])
```

### 4.2 Step-by-Step Process

#### Step 1: Reviewer Assignment

| Attribute | Details |
|-----------|---------|
| **Actor** | System (automatic) or Content Admin (manual) |
| **Auto-Assignment Rules** | - Round-robin among available reviewers |
| | - Match reviewer expertise to content category |
| | - Balance workload across reviewers |
| **Manual Override** | Content Admin can reassign any time |
| **SLA** | Assignment within 1 hour |
| **Next** | Notification sent |

#### Step 2: Reviewer Notification

| Attribute | Details |
|-----------|---------|
| **Channels** | Email + In-app notification |
| **Email Subject** | "Review Required: [Content Title]" |
| **Email Content** | Content type, title, author, link to review |
| **In-app** | Badge on review queue, notification bell |
| **Reminder** | After 12 hours if not opened |

#### Step 3: Review Content

| Attribute | Details |
|-----------|---------|
| **Actor** | Domain Expert, QA Reviewer, or Content Admin |
| **Screen** | Side-by-side view: content preview + review panel |
| **Review Panel** | Checklist, comment fields, decision buttons |
| **Preview** | Exactly as learner would see |
| **Time Tracking** | System tracks time spent reviewing |

**Review Checklist by Content Type:**

| Content Type | Checklist Items |
|--------------|-----------------|
| Wine | Accuracy of facts, tasting notes quality, pairing validity |
| Module | Learning objectives clear, lesson flow logical |
| Quiz | Questions accurate, options fair, explanations helpful |
| Scenario | Dialogue realistic, scoring fair, paths complete |

#### Step 4: Make Decision

**Option A: Request Changes**

| Attribute | Details |
|-----------|---------|
| **When** | Minor issues that author can fix |
| **Required** | At least one comment explaining issue |
| **System Actions** | - Status → DRAFT |
| | - Unlock content for editing |
| | - Attach comments |
| | - Notify author |
| **Author Response** | Fix issues, resubmit |

**Option B: Reject**

| Attribute | Details |
|-----------|---------|
| **When** | Fundamental issues, not suitable for platform |
| **Required** | Detailed explanation |
| **System Actions** | - Status → DRAFT |
| | - Flag as rejected |
| | - Notify author |
| **Author Response** | Major revision or abandon |

**Option C: Approve**

| Attribute | Details |
|-----------|---------|
| **When** | Content meets all quality standards |
| **System Actions** | - Move to next approval stage (if any) |
| | - Or move to Publication queue |
| | - Notify author |

### 4.3 Multi-Stage Approval

#### Simple Workflow (Wine, Question)

```mermaid
flowchart LR
    A[Author Submits] --> B{Content Admin}
    B -->|Approve| C[Ready to Publish]
    B -->|Reject| D[Back to Draft]
```

| Stage | Reviewer | Criteria |
|-------|----------|----------|
| 1 | Content Admin | Accuracy, completeness, formatting |

#### Standard Workflow (Module, Lesson, Quiz)

```mermaid
flowchart LR
    A[Author Submits] --> B{Domain Expert}
    B -->|Approve| C{Content Admin}
    B -->|Reject| D[Back to Draft]
    C -->|Approve| E[Ready to Publish]
    C -->|Reject| D
```

| Stage | Reviewer | Criteria |
|-------|----------|----------|
| 1 | Domain Expert | Wine/hospitality accuracy, educational value |
| 2 | Content Admin | Style, formatting, platform standards |

#### Extended Workflow (Scenario)

```mermaid
flowchart LR
    A[Author Submits] --> B{Domain Expert}
    B -->|Approve| C{QA Reviewer}
    B -->|Reject| D[Back to Draft]
    C -->|Approve| E{Content Admin}
    C -->|Reject| D
    E -->|Approve| F[Ready to Publish]
    E -->|Reject| D
```

| Stage | Reviewer | Criteria |
|-------|----------|----------|
| 1 | Domain Expert | Dialogue realism, wine accuracy |
| 2 | QA Reviewer | All paths work, scoring correct, no dead ends |
| 3 | Content Admin | Final approval, style consistency |

### 4.4 Review Feedback Loop

```mermaid
sequenceDiagram
    participant Author
    participant System
    participant Reviewer

    Author->>System: Submit for Review
    System->>Reviewer: Assign & Notify
    Reviewer->>System: Review Content

    alt Changes Requested
        Reviewer->>System: Request Changes (with comments)
        System->>Author: Notify with Feedback
        Author->>System: Revise Content
        Author->>System: Resubmit
        System->>Reviewer: Notify Resubmission
        Note over Reviewer: Review cycle repeats
    else Approved
        Reviewer->>System: Approve
        System->>Author: Notify Approval
        System->>System: Move to Publication
    end
```

---

## 5. Publication Workflow

### 5.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Content Approved]) --> Check1{Publication Type}

    Check1 -->|Immediate| Path1[Publish Now]
    Check1 -->|Scheduled| Path2[Set Schedule]

    Path1 --> Val1[Pre-Publish Validation]
    Path2 --> Sched1[Store Schedule]
    Sched1 --> Wait1[Wait for Scheduled Time]
    Wait1 --> Val1

    Val1 --> Check2{Validation Pass?}
    Check2 -->|No| Err1[Alert Admin]
    Err1 --> Fix1[Fix Issues]
    Fix1 --> Val1

    Check2 -->|Yes| Pub1[Set Status = PUBLISHED]
    Pub1 --> Pub2[Set publishedAt]
    Pub2 --> Pub3[Increment Version]
    Pub3 --> Pub4[Update Search Index]
    Pub4 --> Pub5[Clear Caches]
    Pub5 --> Pub6[Archive Previous Version]
    Pub6 --> Notify1[/Notify Stakeholders/]
    Notify1 --> End([Content Live])
```

### 5.2 Immediate Publication

| Step | Action | Details |
|------|--------|---------|
| 1 | Admin clicks "Publish Now" | Button enabled after final approval |
| 2 | Pre-publish validation | Check all references valid, no broken links |
| 3 | Confirmation dialog | "Publish [Title] now? This will be visible to all users." |
| 4 | Execute publication | Database updates, status change |
| 5 | Post-publish tasks | Index update, cache clear |
| 6 | Notify stakeholders | Author, subscribers |

### 5.3 Scheduled Publication

| Step | Action | Details |
|------|--------|---------|
| 1 | Admin clicks "Schedule" | Opens date/time picker |
| 2 | Select publish date/time | Future date required, timezone shown |
| 3 | Confirm schedule | "Schedule [Title] for [DateTime]?" |
| 4 | System stores schedule | Status remains REVIEW (or new SCHEDULED state) |
| 5 | Scheduled job runs | At specified time, system executes publication |
| 6 | Failure handling | If fails, alert admin, retry once |

### 5.4 Pre-Publication Validation

| Check | Description | Failure Action |
|-------|-------------|----------------|
| Required fields complete | All mandatory fields have values | Block publish, show missing |
| References valid | Linked wines/quizzes/etc. exist and published | Block publish, show broken refs |
| Media accessible | Images/videos load correctly | Warn, allow override |
| No duplicate | No published content with same identifier | Block publish |
| Dependencies met | Prerequisites are published | Block publish |

### 5.5 Publication Actions

| Action | Timing | Details |
|--------|--------|---------|
| Set status = PUBLISHED | Immediate | Database update |
| Set publishedAt | Immediate | Timestamp |
| Set publishedBy | Immediate | Current admin user |
| Increment version | Immediate | If updating existing content |
| Archive previous | Immediate | Old version → ARCHIVED |
| Update search index | Within 30 seconds | Elasticsearch/Algolia update |
| Clear CDN cache | Within 1 minute | Invalidate cached content |
| Generate sitemap | Within 5 minutes | For SEO (if applicable) |

---

## 6. Revision Workflow

### 6.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Published Content]) --> Step1[Click 'Create Revision']
    Step1 --> Step2[System Creates Draft Copy]
    Step2 --> Step3[Author Edits Draft]
    Step3 --> Step4{Version Type}

    Step4 -->|Minor Change| V1[Increment Minor: 1.0 → 1.1]
    Step4 -->|Major Change| V2[Increment Major: 1.0 → 2.0]

    V1 --> Step5[Submit for Review]
    V2 --> Step5

    Step5 --> Step6[Normal Review Process]
    Step6 --> Check1{Approved?}

    Check1 -->|No| Step3
    Check1 -->|Yes| Step7[Publish New Version]

    Step7 --> Step8[Archive Old Version]
    Step8 --> Step9{Active Users?}

    Step9 -->|Quiz in Progress| Handle1[Complete Current Attempt]
    Step9 -->|Module in Progress| Handle2[Map Progress to New Structure]
    Step9 -->|No Active| End([Version Live])

    Handle1 --> End
    Handle2 --> End
```

### 6.2 Step-by-Step Process

#### Step 1: Initiate Revision

| Attribute | Details |
|-----------|---------|
| **Actor** | Content Author, Content Admin |
| **Precondition** | Content is currently PUBLISHED |
| **Precondition** | No pending draft revision exists |
| **Action** | Click "Create Revision" on published content |
| **System** | Creates copy as DRAFT, links to published version |

#### Step 2: Edit Revision

| Attribute | Details |
|-----------|---------|
| **Actor** | Author |
| **Note** | Published version remains live during editing |
| **Screen** | Split view: current published vs draft changes |
| **Tracking** | System tracks all changes for diff |

#### Step 3: Determine Version Increment

| Change Type | Version Impact | Examples |
|-------------|----------------|----------|
| **Minor (1.0 → 1.1)** | Corrections, clarifications | Typo fix, add study tip, fix answer |
| **Major (1.0 → 2.0)** | Structural changes | Reorder lessons, change objectives, new questions |

System suggests based on changes detected:
- Only text changes → Suggest Minor
- Added/removed items → Suggest Major
- Changed structure → Suggest Major

#### Step 4: Submit and Review

Normal review workflow applies (see Section 4).

#### Step 5: Publish New Version

| Action | Details |
|--------|---------|
| New version → PUBLISHED | With new version number |
| Old version → ARCHIVED | Preserved for history |
| Version history | Updated with change summary |

### 6.3 Handling Active Users

#### Quiz In Progress

| Scenario | Action |
|----------|--------|
| User mid-quiz when version changes | Allow completion of current attempt |
| Score calculated | Use version they started with |
| Next attempt | Uses new version |
| Notification | None (seamless transition) |

#### Module In Progress

| Scenario | Action |
|----------|--------|
| Lesson structure unchanged | Progress preserved as-is |
| Lesson added | User sees new lesson as incomplete |
| Lesson removed | Mark as "legacy complete" |
| Lesson reordered | Map progress to new positions |

#### Scenario In Progress

| Scenario | Action |
|----------|--------|
| User mid-scenario | Allow completion |
| Paths changed | Invalidate if structure differs significantly |
| Next attempt | Uses new version |

### 6.4 Rollback Capability

```mermaid
flowchart TD
    Issue([Issue Discovered]) --> Step1{Severity}

    Step1 -->|Critical| Path1[Emergency Unpublish]
    Step1 -->|Non-Critical| Path2[Create New Revision]

    Path1 --> Step2[Content Hidden Immediately]
    Step2 --> Step3[Select Version to Restore]
    Step3 --> Step4[Expedited Review]
    Step4 --> Step5[Republish]

    Path2 --> Step6[Fix in Draft]
    Step6 --> Step7[Normal Review]
    Step7 --> Step5

    Step5 --> End([Corrected Content Live])
```

| Rollback Type | When to Use | Time to Resolve |
|---------------|-------------|-----------------|
| Emergency Unpublish | Critical error, misinformation | Minutes |
| Version Revert | Need previous version exactly | Hours |
| New Revision | Need to fix specific issues | Days |

---

## 7. Archival Workflow

### 7.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Request Archive]) --> Step1[Admin Clicks 'Archive']
    Step1 --> Check1{Check Dependencies}

    Check1 --> Scan1[Scan for References]
    Scan1 --> Check2{References Found?}

    Check2 -->|Yes| Show1[Display Dependencies]
    Show1 --> Dec1{Resolve Dependencies}
    Dec1 -->|Remove References| Action1[Update Referencing Content]
    Dec1 -->|Substitute Content| Action2[Replace with Alternative]
    Dec1 -->|Cancel| End1([Cancel Archive])
    Action1 --> Check2
    Action2 --> Check2

    Check2 -->|No| Step2[Initiate Warning Period]
    Step2 --> Notify1[/Notify Stakeholders/]
    Notify1 --> Wait1[7-Day Warning Period]

    Wait1 --> Check3{Proceed?}
    Check3 -->|Cancel| End2([Archive Cancelled])
    Check3 -->|Confirm| Step3[Execute Archive]

    Step3 --> Action3[Set Status = ARCHIVED]
    Action3 --> Action4[Set archivedAt, archivedBy]
    Action4 --> Action5[Remove from Active Lists]
    Action5 --> Action6[Update Search Index]
    Action6 --> Action7[Preserve Progress Data]
    Action7 --> Notify2[/Notify Org Admins/]
    Notify2 --> End3([Content Archived])
```

### 7.2 Step-by-Step Process

#### Step 1: Request Archive

| Attribute | Details |
|-----------|---------|
| **Actor** | Content Admin, Org Admin (org content only) |
| **Precondition** | Content is PUBLISHED |
| **Action** | Click "Archive" button |
| **Confirmation** | "Are you sure you want to archive [Title]?" |

#### Step 2: Dependency Check

| Reference Type | Example | Action Required |
|----------------|---------|-----------------|
| Wine in Quiz | Quiz question references wine | Remove question or substitute wine |
| Wine in Scenario | Scenario features wine | Update scenario or substitute wine |
| Lesson in Module | Module contains lesson | Remove lesson or move elsewhere |
| Quiz in Module | Module links to quiz | Remove link or substitute quiz |

**Dependency Display:**

```
Dependencies Found for "Château Margaux":
─────────────────────────────────────────
• Quiz: "Wine Fundamentals Bronze" (2 questions)
  - Question: "Which wine is from Bordeaux?"
  - Question: "Match the wine to its region"

• Scenario: "Anniversary Dinner" (1 reference)
  - Step 3: Wine recommendation

Options:
[ ] Remove all references (will require re-review)
[ ] Substitute with: [Select Wine ▼]
[Cancel] [Continue with Removal]
```

#### Step 3: Warning Period

| Phase | Duration | Actions |
|-------|----------|---------|
| Initiation | Day 0 | Admin confirms archive intent |
| Warning | Days 1-6 | Banner on content, notifications sent |
| Final Notice | Day 7 | Last chance notification |
| Execution | Day 7+ | Archive executed if not cancelled |

**Warning Notifications:**

| Recipient | Channel | Timing |
|-----------|---------|--------|
| Content Author | Email | Day 0, Day 5 |
| Org Admins | Email | Day 0 |
| Active Learners | In-app | Day 5 |

#### Step 4: Execute Archive

| Action | Details |
|--------|---------|
| Status → ARCHIVED | Database update |
| archivedAt = now | Timestamp |
| archivedBy = user | Who archived |
| Remove from lists | Not shown in active content |
| Update search | Remove from search index |
| Preserve progress | User progress data retained |
| Audit log | Full record of action |

### 7.3 Restore Process

```mermaid
flowchart TD
    Start([Archived Content]) --> Step1[Admin Clicks 'Restore']
    Step1 --> Check1{Name Conflict?}
    Check1 -->|Yes| Err1[Show Conflict]
    Err1 --> Dec1{Resolution}
    Dec1 -->|Rename| Fix1[Change Title]
    Dec1 -->|Archive Other| Fix2[Archive Conflicting]
    Dec1 -->|Cancel| End1([Cancel Restore])
    Fix1 --> Check1
    Fix2 --> Check1
    Check1 -->|No| Step2[Restore as Draft]
    Step2 --> Step3[Content Requires Re-Review]
    Step3 --> Notify1[/Notify Original Author/]
    Notify1 --> End2([Ready for Review])
```

| Attribute | Details |
|-----------|---------|
| **Actor** | Content Admin only |
| **Result** | Content restored as DRAFT |
| **Review** | Must go through review again |
| **History** | Archive/restore recorded in audit |

### 7.4 Archive vs Delete

| Action | Reversible | Data Retained | When to Use |
|--------|------------|---------------|-------------|
| **Archive** | Yes | All content + progress | Standard retirement |
| **Soft Delete** | Yes (30 days) | Content only | Draft cleanup |
| **Permanent Delete** | No | Audit log only | Legal requirement |

**Permanent Delete Safeguards:**
1. Requires Content Admin + second approval
2. 7-day cooling-off period
3. Cannot delete if user progress exists
4. Full audit trail maintained

---

## 8. Emergency/Expedited Workflow

### 8.1 Workflow Diagram

```mermaid
flowchart TD
    Start([Critical Issue Discovered]) --> Step1[Flag as Emergency]
    Step1 --> Check1{Who Flagged?}

    Check1 -->|Content Admin| Path1[Self-Approve Option]
    Check1 -->|Other User| Path2[Escalate to Senior Admin]

    Path1 --> Dec1{Nature of Fix}
    Dec1 -->|Typo/Minor| Action1[Self-Approve & Publish]
    Dec1 -->|Content Change| Path2

    Path2 --> Notify1[/Alert Senior Admin/]
    Notify1 --> Step2[Senior Admin Reviews]
    Step2 --> Dec2{Approve?}

    Dec2 -->|No| End1([Return to Normal Workflow])
    Dec2 -->|Yes| Step3[Emergency Publish]

    Action1 --> Step3

    Step3 --> Action2[Immediate Publication]
    Action2 --> Action3[Flag for Post-Review]
    Action3 --> Notify2[/Notify All Stakeholders/]
    Notify2 --> Step4[Schedule Post-Publish Review]
    Step4 --> End2([Fix Live, Review Pending])
```

### 8.2 When to Use Emergency Workflow

| Scenario | Priority | Example |
|----------|----------|---------|
| Incorrect wine information | Critical | Wrong region, false health claim |
| Broken scenario (uncompletable) | Critical | Dead-end path, scoring error |
| Offensive content | Critical | Inappropriate language |
| Outdated information | High | Changed regulations |
| Typo in prominent location | Medium | Title spelling error |

### 8.3 Step-by-Step Process

#### Step 1: Flag as Emergency

| Attribute | Details |
|-----------|---------|
| **Actor** | Any CMS user (report) or Content Admin (flag) |
| **Action** | Click "Report Issue" or "Emergency Edit" |
| **Required** | Description of issue, severity selection |
| **System** | Creates emergency ticket, alerts admins |

#### Step 2: Senior Admin Review

| Attribute | Details |
|-----------|---------|
| **Actor** | Senior Content Admin (designated) |
| **SLA** | 2 hours response time |
| **Action** | Review issue, decide on action |
| **Options** | Approve emergency fix, request normal workflow |

#### Step 3: Emergency Publication

| Attribute | Details |
|-----------|---------|
| **Bypass** | Skips normal review stages |
| **Approval** | Senior Admin only |
| **Audit** | Enhanced logging (reason, approver, timestamp) |
| **Flag** | Content flagged for post-publish review |

#### Step 4: Post-Publish Review

| Attribute | Details |
|-----------|---------|
| **Timing** | Within 24-48 hours of emergency publish |
| **Actor** | Domain Expert (if bypassed) |
| **Purpose** | Verify fix is complete and accurate |
| **Outcome** | Clear flag or request further changes |

### 8.4 Audit Requirements

| Field | Required | Details |
|-------|----------|---------|
| Emergency reason | Yes | Why was emergency workflow used |
| Reporter | Yes | Who identified the issue |
| Approver | Yes | Senior admin who approved |
| Time to resolve | Yes | From report to publication |
| Post-review status | Yes | Completed or pending |
| Post-reviewer | Yes | Who conducted post-review |

---

## 9. Notifications

### 9.1 Notification Matrix

| Event | Recipients | Email | In-App | Push | Timing |
|-------|------------|-------|--------|------|--------|
| Content submitted for review | Assigned reviewers | ✓ | ✓ | — | Immediate |
| Review reminder | Assigned reviewer | ✓ | ✓ | — | After 12h |
| Review overdue | Reviewer + Admin | ✓ | ✓ | — | After 24h |
| Changes requested | Author | ✓ | ✓ | — | Immediate |
| Content rejected | Author | ✓ | ✓ | — | Immediate |
| Content approved | Author | ✓ | ✓ | — | Immediate |
| Content published | Author | ✓ | ✓ | — | Immediate |
| Scheduled publish reminder | Author + Admin | ✓ | — | — | 1h before |
| Content archived (warning) | Author + Admins | ✓ | ✓ | — | Day 0, Day 5 |
| Content archived (final) | Author + Admins | ✓ | ✓ | — | On archive |
| Emergency issue reported | All Admins | ✓ | ✓ | ✓ | Immediate |
| Emergency fix published | All stakeholders | ✓ | ✓ | — | Immediate |

### 9.2 Notification Templates

#### Content Submitted for Review

```
Subject: Review Required: [Content Type] - [Title]

Hi [Reviewer Name],

New content has been submitted for your review:

Type: [Content Type]
Title: [Title]
Author: [Author Name]
Submitted: [DateTime]

[Review Now →]

This review is due by [Due DateTime].

---
Sommelier Spark CMS
```

#### Changes Requested

```
Subject: Changes Requested: [Title]

Hi [Author Name],

Your content "[Title]" requires changes before it can be published.

Reviewer: [Reviewer Name]
Feedback:
"[Reviewer Comments]"

[View Feedback & Edit →]

Please address the feedback and resubmit when ready.

---
Sommelier Spark CMS
```

#### Content Approved

```
Subject: Approved: [Title] is Ready to Publish

Hi [Author Name],

Great news! Your content has been approved:

Title: [Title]
Approved by: [Approver Name]
Status: Ready to Publish

[View Content →]

The content will be published by an administrator shortly.

---
Sommelier Spark CMS
```

#### Emergency Alert

```
Subject: URGENT: Emergency Content Issue Reported

Hi [Admin Name],

An emergency content issue has been reported:

Content: [Title]
Reported by: [Reporter Name]
Severity: [Critical/High]
Issue: [Description]

[Review Issue Now →]

Please review within 2 hours.

---
Sommelier Spark CMS
```

### 9.3 Notification Preferences

Users can configure:

| Setting | Options | Default |
|---------|---------|---------|
| Email notifications | On/Off | On |
| In-app notifications | On/Off | On |
| Digest frequency | Immediate/Daily/Weekly | Immediate |
| Quiet hours | Time range | None |

---

## 10. Service Level Agreements (SLAs)

### 10.1 SLA Definitions

| Stage | Target Time | Measurement | Escalation Trigger |
|-------|-------------|-------------|-------------------|
| **Reviewer assignment** | 1 hour | From submission to assignment | Auto-assign if not done |
| **Review start** | 4 hours | From assignment to first action | Reminder notification |
| **Review completion** | 24 hours | From assignment to decision | Escalate to Content Admin |
| **Emergency review** | 2 hours | From flag to resolution | Escalate to Senior Admin |
| **Scheduled publish** | On time ±1 min | Scheduled time vs actual | Alert if delayed |
| **Archive warning** | 7 days | Warning period duration | N/A (fixed) |

### 10.2 Escalation Matrix

```mermaid
flowchart TD
    subgraph Level1["Level 1: Automated"]
        A1[Auto-assign reviewer]
        A2[Send reminder]
        A3[Update dashboard]
    end

    subgraph Level2["Level 2: Content Admin"]
        B1[Reassign reviewer]
        B2[Adjust priority]
        B3[Direct intervention]
    end

    subgraph Level3["Level 3: Senior Admin"]
        C1[Emergency approval]
        C2[Process override]
        C3[Incident review]
    end

    A1 -->|1 hour exceeded| A2
    A2 -->|4 hours exceeded| B1
    B1 -->|24 hours exceeded| B2
    B2 -->|48 hours exceeded| C1
```

### 10.3 SLA Reporting

| Metric | Target | Measurement Period |
|--------|--------|-------------------|
| Review completion rate | 95% within SLA | Monthly |
| Average review time | < 18 hours | Monthly |
| Emergency response time | < 1.5 hours avg | Monthly |
| Scheduled publish accuracy | 99.9% on time | Monthly |
| Escalation rate | < 10% | Monthly |

---

## 11. Appendix

### 11.1 Workflow Summary

| Workflow | Steps | Diagram |
|----------|-------|---------|
| Content Creation | 6 | Section 3.1 |
| Review/Approval | 4 | Section 4.1 |
| Publication | 6 | Section 5.1 |
| Revision | 5 | Section 6.1 |
| Archival | 4 | Section 7.1 |
| Emergency | 4 | Section 8.1 |

**Total: 6 workflows documented**

### 11.2 Diagram Summary

| Diagram | Section | Type |
|---------|---------|------|
| Complete Workflow Lifecycle | 2.2 | Flowchart |
| Content Creation | 3.1 | Flowchart |
| Review/Approval | 4.1 | Flowchart |
| Multi-Stage Approval (3 diagrams) | 4.3 | Flowcharts |
| Review Feedback Loop | 4.4 | Sequence |
| Publication | 5.1 | Flowchart |
| Revision | 6.1 | Flowchart |
| Rollback | 6.4 | Flowchart |
| Archival | 7.1 | Flowchart |
| Restore | 7.3 | Flowchart |
| Emergency | 8.1 | Flowchart |
| Escalation | 10.2 | Flowchart |

**Total: 13 diagrams created**

### 11.3 Notification Summary

| # | Notification | Trigger |
|---|--------------|---------|
| 1 | Review assigned | Content submitted |
| 2 | Review reminder | 12h no action |
| 3 | Review overdue | 24h exceeded |
| 4 | Changes requested | Reviewer feedback |
| 5 | Content rejected | Review decision |
| 6 | Content approved | Review decision |
| 7 | Content published | Publication complete |
| 8 | Scheduled reminder | 1h before publish |
| 9 | Archive warning (initial) | Archive initiated |
| 10 | Archive warning (final) | Day 5 of warning |
| 11 | Content archived | Archive complete |
| 12 | Emergency alert | Issue reported |
| 13 | Emergency resolved | Fix published |

**Total: 13 notification types defined**

### 11.4 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-20 | Obi Wan | Initial draft |

---

*End of Document*
