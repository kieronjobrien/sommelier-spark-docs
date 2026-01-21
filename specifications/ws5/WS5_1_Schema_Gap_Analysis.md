# WS5.1 Schema Gap Analysis — Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS5.1-GAP |
| **Version** | 1.0 |
| **Date** | 2026-01-21 |
| **Author** | Claude Code |
| **Status** | DRAFT |
| **Related Documents** | SS-WS3-DATA, prisma/schema.prisma |

---

## 1. Executive Summary

This document provides a comprehensive gap analysis between the existing Prisma schema (`~/Projects/sommelier_spark/prisma/schema.prisma`, 596 lines) and the target specification (`specifications/ws3/Data_Model.md`).

### Key Findings

| Category | Gap Count | Risk Level |
|----------|-----------|------------|
| Missing Enums | 16 | LOW |
| Missing Tables | 15 | MEDIUM |
| Missing Fields on Existing Tables | 45+ | MEDIUM |
| Role Mapping Required | 5 roles | LOW |
| Multi-Tenancy Difference | Structural | DECISION REQUIRED |

### Recommendation

The existing schema uses a 3-level tenancy hierarchy (Organisation → Brand → Location) while the target spec uses single-level (Organisation only). **Recommendation: KEEP the 3-level hierarchy** as it provides greater flexibility and is already implemented. CMS features should be added additively.

---

## 2. Multi-Tenancy Structure Comparison

### 2.1 Existing Schema (3-Level)

```
Organisation
    └── Brand (many)
        └── Location (many)
            └── User (many)
```

**Current Implementation:**
- `Organisation` - top-level tenant
- `Brand` - sub-tenant within organisation
- `Location` - venue within brand
- Content (Wine, Module, Quiz, Scenario) can be scoped to any level:
  - `organisationId`, `brandId`, `locationId` all nullable
  - `null` values = global content accessible to all

### 2.2 Target Schema (Single-Level)

```
Organisation
    └── User (many)
```

**Specification:**
- `Organisation` - single tenant level
- Users belong directly to organisation
- Content scoped only to organisation level

### 2.3 Decision: KEEP 3-Level Hierarchy

| Aspect | Existing (3-Level) | Target (1-Level) |
|--------|-------------------|------------------|
| Flexibility | HIGH - venue chains, hotel groups | MEDIUM - single org |
| Complexity | Higher | Lower |
| Already Implemented | YES | NO |
| Migration Effort | None | HIGH (destructive) |
| Business Value | Supports hospitality groups | Limited |

**Verdict:** Retain 3-level hierarchy. Add CMS features additively without changing tenancy model.

---

## 3. Enum Gap Analysis

### 3.1 Existing Enums (schema.prisma)

| Enum | Values | Status |
|------|--------|--------|
| `UserRole` | super_admin, org_admin, brand_admin, location_admin, staff | DIFFERENT |
| `CertificationLevel` | none, bronze, silver, gold | MATCH (case differs) |
| `WineType` | red, white, rose, sparkling, dessert, fortified | MATCH |
| `PriceTier` | budget, moderate, premium, luxury | MATCH |
| `ModuleCategory` | wine_fundamentals, grape_varieties, regions_appellations, tasting_techniques, food_pairing, service_skills, wine_knowledge | EXTRA (not in spec) |
| `QuestionType` | multiple_choice, multiple_select, true_false | MATCH (spec adds `matching`) |
| `ScenarioCategory` | WINE_PAIRING, CUSTOMER_SERVICE, UPSELLING, DIETARY_ALLERGIES, WINE_FAULTS, SPECIAL_OCCASIONS | MATCH |
| `DifficultyLevel` | BRONZE, SILVER, GOLD | MATCH (maps to Tier) |
| `StepType` | DIALOGUE, ACTION, OBSERVATION, CONCLUSION | MATCH |
| `ProgressStatus` | not_started, in_progress, completed | EXTRA (not in spec) |

### 3.2 Missing Enums (from Data_Model.md)

| Enum | Values | Purpose | Action |
|------|--------|---------|--------|
| `OrganisationType` | RESTAURANT, HOTEL, WINE_BAR, WINE_RETAIL, HOSPITALITY_GROUP, EDUCATION, OTHER | Classify business type | **ADD** |
| `OrganisationStatus` | ACTIVE, TRIAL, SUSPENDED, CANCELLED | Account status | **ADD** |
| `SubscriptionTier` | STARTER, PROFESSIONAL, ENTERPRISE | Billing tier | **ADD** |
| `SubscriptionStatus` | ACTIVE, TRIALING, PAST_DUE, CANCELLED, EXPIRED | Payment status | **ADD** |
| `BillingCycle` | MONTHLY, ANNUAL | Billing frequency | **ADD** |
| `UserStatus` | ACTIVE, INVITED, SUSPENDED, DEACTIVATED | User account status | **ADD** |
| `ContentStatus` | DRAFT, REVIEW, APPROVED, PUBLISHED, ARCHIVED | Content lifecycle | **ADD** |
| `Tier` | bronze, silver, gold | Difficulty tier | **ADD** (alias DifficultyLevel) |
| `Difficulty` | easy, medium, hard | Question difficulty | **ADD** |
| `ReviewStatus` | PENDING, IN_PROGRESS, COMPLETED | Review workflow | **ADD** |
| `ReviewDecision` | APPROVED, REJECTED, NEEDS_CHANGES | Review outcome | **ADD** |
| `ImportStatus` | UPLOADED, VALIDATING, VALIDATION_FAILED, AWAITING_CONFIRMATION, PROCESSING, COMPLETED, COMPLETED_WITH_ERRORS, FAILED, CANCELLED | Import job state | **ADD** |
| `ImportRowStatus` | PENDING, VALID, INVALID, DUPLICATE, IMPORTED, SKIPPED, FAILED | Import row state | **ADD** |
| `ExportStatus` | PENDING, PROCESSING, COMPLETED, FAILED | Export job state | **ADD** |
| `NotificationType` | REVIEW_REQUESTED, REVIEW_APPROVED, REVIEW_REJECTED, CONTENT_PUBLISHED, USER_INVITED, USER_CERTIFIED, DEADLINE_REMINDER, IMPORT_COMPLETED, EXPORT_READY, SYSTEM_ALERT | Notification categories | **ADD** |
| `JobType` | CURRICULUM_GENERATION, QUIZ_GENERATION, SCENARIO_GENERATION, IMPORT_PROCESSING, REPORT_EXPORT, EMAIL_SEND, CACHE_REFRESH, CLEANUP | Background job types | **ADD** |
| `JobStatus` | PENDING, PROCESSING, COMPLETED, FAILED, CANCELLED | Job state | **ADD** |

### 3.3 Role Mapping Strategy

The existing role system supports 3-level tenancy. We need to map to spec roles:

| Existing Role | Maps To | Notes |
|--------------|---------|-------|
| `super_admin` | SYSTEM_ADMIN | Platform-level admin |
| `org_admin` | OWNER | Organisation owner |
| `brand_admin` | ADMIN | Brand-level admin (KEEP) |
| `location_admin` | MANAGER | Location manager |
| `staff` | LEARNER | End user/learner |

**Decision:** Keep existing roles as they support 3-level tenancy. Add `UserStatus` enum separately for account state.

---

## 4. Table Gap Analysis

### 4.1 Summary Matrix

| Table | Exists | Missing Fields | Action |
|-------|--------|----------------|--------|
| Organisation | YES | type, status, subscriptionTier, settings, branding, billingEmail, trialEndsAt, cancelledAt | ADD FIELDS |
| Brand | YES | (not in spec) | KEEP |
| Location | YES | (not in spec) | KEEP |
| User | YES | firstName, lastName, status, jobTitle, department, preferences, invitedAt, invitedBy, activatedAt | ADD FIELDS |
| Wine | YES | price, tastingNotes, foodPairings, servingTemperature, status, version, createdBy, updatedBy, publishedAt | ADD FIELDS |
| Module | YES | tier, status, version, createdBy, updatedBy, publishedAt | ADD FIELDS |
| Lesson | YES | deletedAt | ADD FIELD |
| Quiz | YES | tier, status, version, createdBy, updatedBy | ADD FIELDS |
| Question | YES | difficulty, relatedWineIds | ADD FIELDS |
| Option | YES | — | OK |
| Scenario | YES | status, version, createdBy, updatedBy | ADD FIELDS |
| ScenarioStep | YES | — | OK |
| ScenarioChoice | YES | — | OK |
| ScenarioWine | YES | sortOrder | ADD FIELD |
| Certification | YES | certificateNumber, certificateUrl, expiresAt | ADD FIELDS |
| ModuleProgress | YES | timeSpentSeconds | ADD FIELD |
| LessonProgress | YES | — | OK |
| QuizAttempt | YES | timeSpentSeconds, expiresAt | ADD FIELDS |
| ScenarioAttempt | YES | — | OK |
| WineStudyHistory | YES | — | OK |
| **Subscription** | NO | — | **ADD TABLE** |
| **UserSession** | NO | — | **ADD TABLE** |
| **PasswordResetToken** | NO | — | **ADD TABLE** |
| **ContentVersion** | NO | — | **ADD TABLE** |
| **ContentReview** | NO | — | **ADD TABLE** |
| **ReviewComment** | NO | — | **ADD TABLE** |
| **UserProgress** | NO | (generic progress) | **ADD TABLE** |
| **QuizAnswer** | NO | — | **ADD TABLE** |
| **ScenarioAttemptChoice** | NO | — | **ADD TABLE** |
| **Import** | NO | — | **ADD TABLE** |
| **ImportRow** | NO | — | **ADD TABLE** |
| **Export** | NO | — | **ADD TABLE** |
| **AuditLog** | NO | — | **ADD TABLE** |
| **Notification** | NO | — | **ADD TABLE** |
| **NotificationTemplate** | NO | — | **ADD TABLE** |
| **Job** | NO | — | **ADD TABLE** |

### 4.2 New Tables Required (15 total)

#### 4.2.1 Subscription Management

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `Subscription` | Billing relationship | organisationId, tier, status, billingCycle, pricePerMonth, maxUsers, renewalDate, stripeCustomerId |

#### 4.2.2 Authentication

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `UserSession` | Active sessions | userId, refreshTokenHash, userAgent, ipAddress, expiresAt |
| `PasswordResetToken` | Password reset | userId, tokenHash, expiresAt, usedAt |

#### 4.2.3 Content Lifecycle

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `ContentVersion` | Version history | contentType, contentId, version, snapshot (JSON), changeSummary, createdBy |
| `ContentReview` | Review workflow | contentType, contentId, version, reviewerId, status, decision, feedback |
| `ReviewComment` | Review feedback | reviewId, userId, comment, fieldReference, resolved |

#### 4.2.4 Progress Tracking

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `UserProgress` | Generic progress | userId, contentType, contentId, completionPercent, completed, timeSpentSeconds |
| `QuizAnswer` | Individual answers | attemptId, questionId, selectedOptionIds[], isCorrect |
| `ScenarioAttemptChoice` | Step choices | attemptId, stepId, choiceId, pointsAwarded |

#### 4.2.5 Import/Export

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `Import` | Bulk import jobs | organisationId, importType, fileName, status, totalRows, validRows, errorRows |
| `ImportRow` | Row-level results | importId, rowNumber, rowData (JSON), status, errors (JSON) |
| `Export` | Report exports | organisationId, exportType, format, parameters (JSON), status, fileUrl |

#### 4.2.6 System

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `AuditLog` | Audit trail | timestamp, userId, organisationId, action, resourceType, resourceId, previousState, newState |
| `Notification` | User notifications | userId, type, title, message, data (JSON), read, actionUrl |
| `NotificationTemplate` | Email templates | code, name, type, subject, bodyText, bodyHtml, variables[] |
| `Job` | Background jobs | type, status, priority, payload (JSON), result (JSON), attempts, scheduledAt |

---

## 5. Field Gap Analysis by Table

### 5.1 Organisation

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| name | String | YES | — |
| slug | String | YES | — |
| logo | String? | YES | — |
| **type** | OrganisationType | NO | **ADD** |
| **status** | OrganisationStatus | NO | **ADD** (replace isActive) |
| **subscriptionTier** | SubscriptionTier | NO | **ADD** |
| **settings** | Json | NO | **ADD** |
| **branding** | Json? | NO | **ADD** |
| **billingEmail** | String | NO | **ADD** |
| **billingAddress** | Json? | NO | **ADD** |
| **trialEndsAt** | DateTime? | NO | **ADD** |
| isActive | Boolean | YES | DEPRECATE (use status) |
| suspendedAt | DateTime? | YES | — |
| **cancelledAt** | DateTime? | NO | **ADD** |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.2 User

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| email | String | YES | — |
| name | String | YES | — |
| passwordHash | String | YES | — |
| role | UserRole | YES | — |
| certificationLevel | CertificationLevel | YES | — |
| organisationId | String? | YES | — |
| brandId | String? | YES | — |
| locationId | String? | YES | — |
| **firstName** | String? | NO | **ADD** |
| **lastName** | String? | NO | **ADD** |
| **status** | UserStatus | NO | **ADD** |
| **jobTitle** | String? | NO | **ADD** |
| **department** | String? | NO | **ADD** |
| **hireDate** | Date? | NO | **ADD** |
| **avatarUrl** | String? | NO | **ADD** |
| **preferences** | Json? | NO | **ADD** |
| **lastLoginAt** | DateTime? | NO | **ADD** |
| **invitedAt** | DateTime? | NO | **ADD** |
| **invitedBy** | UUID? | NO | **ADD** |
| **activatedAt** | DateTime? | NO | **ADD** |
| suspendedAt | DateTime? | YES | — |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.3 Wine (Content Table)

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| name | String | YES | — |
| producer | String? | YES | — |
| vintage | Int? | YES | — |
| region | String | YES | — |
| country | String | YES | — |
| wineType | WineType | YES | — |
| grapeVarieties | String[] | YES | — |
| priceTier | PriceTier | YES | — |
| **price** | Decimal? | NO | **ADD** |
| **tastingNotes** | Text? | NO | **ADD** |
| **foodPairings** | String[]? | NO | **ADD** |
| **servingTemperature** | String? | NO | **ADD** |
| **decantingTime** | String? | NO | **ADD** |
| quickFacts | Json | YES | — |
| detailedProfile | Json? | YES | — |
| expertInsights | Json? | YES | — |
| **imageUrl** | String? | NO | **ADD** |
| **status** | ContentStatus | NO | **ADD** (default PUBLISHED for existing) |
| version | Int | YES | — |
| **createdBy** | UUID? | NO | **ADD** |
| **updatedBy** | UUID? | NO | **ADD** |
| **publishedAt** | DateTime? | NO | **ADD** |
| **publishedBy** | UUID? | NO | **ADD** |
| **archivedAt** | DateTime? | NO | **ADD** |
| **archivedBy** | UUID? | NO | **ADD** |
| **reviewRequestedAt** | DateTime? | NO | **ADD** |
| **reviewRequestedBy** | UUID? | NO | **ADD** |
| organisationId | String? | YES | — |
| brandId | String? | YES | — |
| locationId | String? | YES | — |
| suspendedAt | DateTime? | YES | — |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.4 Module (Content Table)

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| title | String | YES | — |
| description | String | YES | — |
| category | ModuleCategory | YES | — |
| sortOrder | Int | YES | — |
| estimatedMinutes | Int | YES | — |
| imageUrl | String? | YES | — |
| **tier** | Tier | NO | **ADD** |
| **status** | ContentStatus | NO | **ADD** |
| **version** | Int | NO | **ADD** (default 1) |
| **createdBy** | UUID? | NO | **ADD** |
| **updatedBy** | UUID? | NO | **ADD** |
| **publishedAt** | DateTime? | NO | **ADD** |
| **publishedBy** | UUID? | NO | **ADD** |
| **archivedAt** | DateTime? | NO | **ADD** |
| **archivedBy** | UUID? | NO | **ADD** |
| organisationId | String? | YES | — |
| brandId | String? | YES | — |
| locationId | String? | YES | — |
| suspendedAt | DateTime? | YES | — |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.5 Quiz (Content Table)

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| moduleId | String? | YES | — |
| title | String | YES | — |
| description | String? | YES | — |
| passingScore | Int | YES | — |
| timeLimit | Int? | YES | — |
| category | String? | YES | — |
| **tier** | Tier | NO | **ADD** |
| **status** | ContentStatus | NO | **ADD** |
| **version** | Int | NO | **ADD** (default 1) |
| **createdBy** | UUID? | NO | **ADD** |
| **updatedBy** | UUID? | NO | **ADD** |
| **publishedAt** | DateTime? | NO | **ADD** |
| **publishedBy** | UUID? | NO | **ADD** |
| **archivedAt** | DateTime? | NO | **ADD** |
| **archivedBy** | UUID? | NO | **ADD** |
| organisationId | String? | YES | — |
| brandId | String? | YES | — |
| locationId | String? | YES | — |
| suspendedAt | DateTime? | YES | — |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.6 Question

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| quizId | String | YES | — |
| question | String | YES | — |
| questionType | QuestionType | YES | — |
| explanation | String? | YES | — |
| sortOrder | Int | YES | — |
| **difficulty** | Difficulty | NO | **ADD** |
| **relatedWineIds** | UUID[]? | NO | **ADD** |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |

### 5.7 Scenario (Content Table)

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| title | String | YES | — |
| description | String | YES | — |
| category | ScenarioCategory | YES | — |
| difficulty | DifficultyLevel | YES | — |
| estimatedTime | Int | YES | — |
| customerName | String | YES | — |
| customerPersona | String | YES | — |
| situation | String | YES | — |
| **status** | ContentStatus | NO | **ADD** |
| **version** | Int | NO | **ADD** (default 1) |
| **createdBy** | UUID? | NO | **ADD** |
| **updatedBy** | UUID? | NO | **ADD** |
| **publishedAt** | DateTime? | NO | **ADD** |
| **publishedBy** | UUID? | NO | **ADD** |
| **archivedAt** | DateTime? | NO | **ADD** |
| **archivedBy** | UUID? | NO | **ADD** |
| organisationId | String? | YES | — |
| brandId | String? | YES | — |
| locationId | String? | YES | — |
| isActive | Boolean | YES | — |
| suspendedAt | DateTime? | YES | — |
| createdAt | DateTime | YES | — |
| updatedAt | DateTime | YES | — |
| deletedAt | DateTime? | YES | — |

### 5.8 Certification

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| userId | String | YES | — |
| level | CertificationLevel | YES | — |
| awardedAt | DateTime | YES | — |
| **certificateNumber** | String | NO | **ADD** (unique) |
| **certificateUrl** | String? | NO | **ADD** |
| **expiresAt** | DateTime? | NO | **ADD** |

### 5.9 QuizAttempt

| Field | Type | Exists | Action |
|-------|------|--------|--------|
| id | UUID | YES | — |
| userId | String | YES | — |
| quizId | String | YES | — |
| score | Int | YES | — |
| passed | Boolean | YES | — |
| answers | Json | YES | MIGRATE to QuizAnswer table |
| attemptedAt | DateTime | YES | RENAME to startedAt |
| **timeSpentSeconds** | Int? | NO | **ADD** |
| **completedAt** | DateTime? | NO | **ADD** |
| **expiresAt** | DateTime? | NO | **ADD** |

---

## 6. Index Gap Analysis

### 6.1 Missing Indexes (High Priority)

| Table | Index | Columns | Purpose |
|-------|-------|---------|---------|
| wines | idx_wines_status | status | Filter published content |
| modules | idx_modules_status | status | Filter published content |
| quizzes | idx_quizzes_status | status | Filter published content |
| scenarios | idx_scenarios_status | status | Filter published content |
| content_versions | idx_content_versions_content | contentType, contentId | Version lookup |
| content_reviews | idx_content_reviews_status | status | Pending reviews |
| audit_logs | idx_audit_logs_timestamp | timestamp | Time-based queries |
| audit_logs | idx_audit_logs_org_time | organisationId, timestamp DESC | Org audit trail |
| notifications | idx_notifications_user_read | userId, read | Unread count |
| jobs | idx_jobs_pending | status, scheduledAt WHERE status='PENDING' | Job queue |

### 6.2 Full-Text Search Index (Future)

```sql
-- Wine search (to be added when PostgreSQL FTS confirmed)
CREATE INDEX idx_wines_search ON wines USING gin(
    to_tsvector('english',
        COALESCE(name, '') || ' ' ||
        COALESCE(producer, '') || ' ' ||
        COALESCE(region, '')
    )
);
```

---

## 7. Data Migration Considerations

### 7.1 Existing Data Defaults

When adding new fields, existing records need sensible defaults:

| Table | Field | Default for Existing Data |
|-------|-------|---------------------------|
| Wine | status | `PUBLISHED` |
| Module | status | `PUBLISHED` |
| Quiz | status | `PUBLISHED` |
| Scenario | status | `PUBLISHED` |
| All content | version | `1` |
| All content | publishedAt | `createdAt` |
| Organisation | type | `OTHER` |
| Organisation | status | `ACTIVE` |
| User | status | `ACTIVE` or `SUSPENDED` (based on suspendedAt) |

### 7.2 QuizAttempt.answers Migration

Current schema stores answers as JSON. Need to:
1. Create `QuizAnswer` table
2. Migrate JSON data to normalized rows
3. Keep JSON field for backward compatibility initially

### 7.3 ScenarioAttempt.choicesMade Migration

Current schema stores choices as JSON array. Need to:
1. Create `ScenarioAttemptChoice` table
2. Migrate JSON data to normalized rows
3. Keep JSON field for backward compatibility initially

---

## 8. Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Breaking existing queries | MEDIUM | Add fields with defaults, don't remove |
| Data loss | LOW | Backup before migration |
| Downtime | LOW | Schema changes are additive |
| Role confusion | LOW | Document role mapping clearly |
| Index creation on large tables | MEDIUM | Create indexes CONCURRENTLY |

---

## 9. Action Summary

### 9.1 Phase 1: Enums (Non-Breaking)

Add 16 new enums, update 2 existing enums.

### 9.2 Phase 2: New Tables (Non-Breaking)

Create 15 new tables with proper indexes.

### 9.3 Phase 3: Add Fields (Non-Breaking with Defaults)

Add ~45 fields to existing tables with appropriate defaults.

### 9.4 Phase 4: Indexes (Non-Breaking)

Add ~15 new indexes using CONCURRENTLY where possible.

### 9.5 Phase 5: Data Backfill (Non-Breaking)

Set status=PUBLISHED, version=1 for existing content.

---

## Appendix A: Field Count Summary

| Table | Existing Fields | New Fields | Total |
|-------|----------------|------------|-------|
| Organisation | 12 | 8 | 20 |
| User | 14 | 11 | 25 |
| Wine | 18 | 12 | 30 |
| Module | 13 | 9 | 22 |
| Quiz | 13 | 9 | 22 |
| Question | 7 | 2 | 9 |
| Scenario | 16 | 8 | 24 |
| Certification | 4 | 3 | 7 |
| QuizAttempt | 7 | 3 | 10 |
| **NEW TABLES** | 0 | ~120 | ~120 |

---

*Document generated: 2026-01-21*
