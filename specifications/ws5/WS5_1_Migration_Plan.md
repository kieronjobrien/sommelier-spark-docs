# WS5.1 Migration Plan — Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS5.1-MIG |
| **Version** | 1.0 |
| **Date** | 2026-01-21 |
| **Author** | Claude Code |
| **Status** | DRAFT |
| **Related Documents** | SS-WS5.1-GAP |

---

## 1. Executive Summary

This document outlines the phased migration plan to extend the existing Sommelier Spark database schema with CMS functionality. The migration is designed to be **non-breaking** and **reversible**.

### Migration Overview

| Phase | Description | Risk | Downtime |
|-------|-------------|------|----------|
| Phase 1 | Add new enums | LOW | NONE |
| Phase 2 | Create new tables | LOW | NONE |
| Phase 3 | Add columns to existing tables | MEDIUM | NONE |
| Phase 4 | Create indexes | LOW | NONE |
| Phase 5 | Backfill existing data | LOW | NONE |
| Phase 6 | Data migration (JSON → tables) | MEDIUM | NONE |

**Total Estimated Migration Time:** 5-10 minutes (development), 15-30 minutes (production with data)

---

## 2. Pre-Migration Checklist

### 2.1 Before Starting

- [ ] Backup database (full dump)
- [ ] Verify Prisma version >= 5.x
- [ ] Review current disk space (new tables ~5% overhead)
- [ ] Schedule maintenance window (recommended, not required)
- [ ] Notify team of migration

### 2.2 Environment Requirements

```bash
# Required
node --version  # >= 18.x
npx prisma --version  # >= 5.x
psql --version  # >= 15.x

# Verify database connection
npx prisma db pull
```

---

## 3. Phase 1: Add New Enums

**Risk Level:** LOW
**Reversible:** YES
**Downtime:** NONE

### 3.1 SQL Commands

```sql
-- ============================================
-- PHASE 1: NEW ENUMS
-- ============================================

-- Organisation enums
CREATE TYPE "OrganisationType" AS ENUM (
    'RESTAURANT',
    'HOTEL',
    'WINE_BAR',
    'WINE_RETAIL',
    'HOSPITALITY_GROUP',
    'EDUCATION',
    'OTHER'
);

CREATE TYPE "OrganisationStatus" AS ENUM (
    'ACTIVE',
    'TRIAL',
    'SUSPENDED',
    'CANCELLED'
);

CREATE TYPE "SubscriptionTier" AS ENUM (
    'STARTER',
    'PROFESSIONAL',
    'ENTERPRISE'
);

CREATE TYPE "SubscriptionStatus" AS ENUM (
    'ACTIVE',
    'TRIALING',
    'PAST_DUE',
    'CANCELLED',
    'EXPIRED'
);

CREATE TYPE "BillingCycle" AS ENUM (
    'MONTHLY',
    'ANNUAL'
);

-- User enums
CREATE TYPE "UserStatus" AS ENUM (
    'ACTIVE',
    'INVITED',
    'SUSPENDED',
    'DEACTIVATED'
);

-- Content lifecycle enums
CREATE TYPE "ContentStatus" AS ENUM (
    'DRAFT',
    'REVIEW',
    'APPROVED',
    'PUBLISHED',
    'ARCHIVED'
);

CREATE TYPE "Tier" AS ENUM (
    'bronze',
    'silver',
    'gold'
);

CREATE TYPE "Difficulty" AS ENUM (
    'easy',
    'medium',
    'hard'
);

-- Review workflow enums
CREATE TYPE "ReviewStatus" AS ENUM (
    'PENDING',
    'IN_PROGRESS',
    'COMPLETED'
);

CREATE TYPE "ReviewDecision" AS ENUM (
    'APPROVED',
    'REJECTED',
    'NEEDS_CHANGES'
);

-- Import/Export enums
CREATE TYPE "ImportStatus" AS ENUM (
    'UPLOADED',
    'VALIDATING',
    'VALIDATION_FAILED',
    'AWAITING_CONFIRMATION',
    'PROCESSING',
    'COMPLETED',
    'COMPLETED_WITH_ERRORS',
    'FAILED',
    'CANCELLED'
);

CREATE TYPE "ImportRowStatus" AS ENUM (
    'PENDING',
    'VALID',
    'INVALID',
    'DUPLICATE',
    'IMPORTED',
    'SKIPPED',
    'FAILED'
);

CREATE TYPE "ExportStatus" AS ENUM (
    'PENDING',
    'PROCESSING',
    'COMPLETED',
    'FAILED'
);

-- System enums
CREATE TYPE "NotificationType" AS ENUM (
    'REVIEW_REQUESTED',
    'REVIEW_APPROVED',
    'REVIEW_REJECTED',
    'CONTENT_PUBLISHED',
    'USER_INVITED',
    'USER_CERTIFIED',
    'DEADLINE_REMINDER',
    'IMPORT_COMPLETED',
    'EXPORT_READY',
    'SYSTEM_ALERT'
);

CREATE TYPE "JobType" AS ENUM (
    'CURRICULUM_GENERATION',
    'QUIZ_GENERATION',
    'SCENARIO_GENERATION',
    'IMPORT_PROCESSING',
    'REPORT_EXPORT',
    'EMAIL_SEND',
    'CACHE_REFRESH',
    'CLEANUP'
);

CREATE TYPE "JobStatus" AS ENUM (
    'PENDING',
    'PROCESSING',
    'COMPLETED',
    'FAILED',
    'CANCELLED'
);

-- Update existing QuestionType enum (add 'matching')
ALTER TYPE "QuestionType" ADD VALUE 'matching';
```

### 3.2 Rollback Commands

```sql
-- ROLLBACK PHASE 1
-- Note: DROP TYPE only works if no tables use the type

DROP TYPE IF EXISTS "OrganisationType";
DROP TYPE IF EXISTS "OrganisationStatus";
DROP TYPE IF EXISTS "SubscriptionTier";
DROP TYPE IF EXISTS "SubscriptionStatus";
DROP TYPE IF EXISTS "BillingCycle";
DROP TYPE IF EXISTS "UserStatus";
DROP TYPE IF EXISTS "ContentStatus";
DROP TYPE IF EXISTS "Tier";
DROP TYPE IF EXISTS "Difficulty";
DROP TYPE IF EXISTS "ReviewStatus";
DROP TYPE IF EXISTS "ReviewDecision";
DROP TYPE IF EXISTS "ImportStatus";
DROP TYPE IF EXISTS "ImportRowStatus";
DROP TYPE IF EXISTS "ExportStatus";
DROP TYPE IF EXISTS "NotificationType";
DROP TYPE IF EXISTS "JobType";
DROP TYPE IF EXISTS "JobStatus";

-- Note: Cannot remove enum value from QuestionType
-- Would need to recreate the type (complex migration)
```

---

## 4. Phase 2: Create New Tables

**Risk Level:** LOW
**Reversible:** YES
**Downtime:** NONE

### 4.1 SQL Commands

```sql
-- ============================================
-- PHASE 2: NEW TABLES
-- ============================================

-- 2.1 Subscription Management
CREATE TABLE "subscriptions" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "organisation_id" UUID NOT NULL UNIQUE REFERENCES "organisations"("id") ON DELETE CASCADE,
    "tier" "SubscriptionTier" NOT NULL DEFAULT 'STARTER',
    "status" "SubscriptionStatus" NOT NULL DEFAULT 'ACTIVE',
    "billing_cycle" "BillingCycle" NOT NULL DEFAULT 'MONTHLY',
    "price_per_month" DECIMAL(10,2) NOT NULL DEFAULT 0,
    "max_users" INTEGER NOT NULL DEFAULT 5,
    "current_users" INTEGER NOT NULL DEFAULT 0,
    "features" JSONB NOT NULL DEFAULT '{}',
    "start_date" DATE NOT NULL DEFAULT CURRENT_DATE,
    "renewal_date" DATE NOT NULL DEFAULT (CURRENT_DATE + INTERVAL '1 month'),
    "trial_ends_at" TIMESTAMP,
    "cancelled_at" TIMESTAMP,
    "stripe_customer_id" VARCHAR(255),
    "stripe_subscription_id" VARCHAR(255),
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 2.2 Authentication Tables
CREATE TABLE "user_sessions" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "user_id" UUID NOT NULL REFERENCES "users"("id") ON DELETE CASCADE,
    "refresh_token_hash" VARCHAR(255) NOT NULL,
    "user_agent" VARCHAR(500),
    "ip_address" INET,
    "expires_at" TIMESTAMP NOT NULL,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "last_used_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE "password_reset_tokens" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "user_id" UUID NOT NULL REFERENCES "users"("id") ON DELETE CASCADE,
    "token_hash" VARCHAR(255) NOT NULL,
    "expires_at" TIMESTAMP NOT NULL,
    "used_at" TIMESTAMP,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 2.3 Content Lifecycle Tables
CREATE TABLE "content_versions" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "content_type" VARCHAR(50) NOT NULL,
    "content_id" UUID NOT NULL,
    "version" INTEGER NOT NULL,
    "snapshot" JSONB NOT NULL,
    "change_summary" TEXT,
    "created_by" UUID NOT NULL REFERENCES "users"("id"),
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE("content_type", "content_id", "version")
);

CREATE TABLE "content_reviews" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "content_type" VARCHAR(50) NOT NULL,
    "content_id" UUID NOT NULL,
    "version" INTEGER NOT NULL,
    "reviewer_id" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    "status" "ReviewStatus" NOT NULL DEFAULT 'PENDING',
    "decision" "ReviewDecision",
    "feedback" TEXT,
    "reviewed_at" TIMESTAMP,
    "assigned_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "due_at" TIMESTAMP
);

CREATE TABLE "review_comments" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "review_id" UUID NOT NULL REFERENCES "content_reviews"("id") ON DELETE CASCADE,
    "user_id" UUID NOT NULL REFERENCES "users"("id") ON DELETE SET NULL,
    "comment" TEXT NOT NULL,
    "field_reference" VARCHAR(100),
    "resolved" BOOLEAN NOT NULL DEFAULT FALSE,
    "resolved_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    "resolved_at" TIMESTAMP,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 2.4 Generic Progress Table
CREATE TABLE "user_progress" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "user_id" UUID NOT NULL REFERENCES "users"("id") ON DELETE CASCADE,
    "content_type" VARCHAR(50) NOT NULL,
    "content_id" UUID NOT NULL,
    "completion_percent" INTEGER NOT NULL DEFAULT 0,
    "completed" BOOLEAN NOT NULL DEFAULT FALSE,
    "time_spent_seconds" INTEGER NOT NULL DEFAULT 0,
    "last_accessed_at" TIMESTAMP,
    "completed_at" TIMESTAMP,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE("user_id", "content_type", "content_id")
);

-- 2.5 Normalized Answer Tables
CREATE TABLE "quiz_answers" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "attempt_id" UUID NOT NULL REFERENCES "quiz_attempts"("id") ON DELETE CASCADE,
    "question_id" UUID NOT NULL REFERENCES "questions"("id") ON DELETE CASCADE,
    "selected_option_ids" UUID[] NOT NULL DEFAULT '{}',
    "is_correct" BOOLEAN NOT NULL,
    "answered_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE("attempt_id", "question_id")
);

CREATE TABLE "scenario_attempt_choices" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "attempt_id" UUID NOT NULL REFERENCES "scenario_attempts"("id") ON DELETE CASCADE,
    "step_id" UUID NOT NULL REFERENCES "scenario_steps"("id") ON DELETE CASCADE,
    "choice_id" UUID NOT NULL REFERENCES "scenario_choices"("id") ON DELETE CASCADE,
    "points_awarded" INTEGER NOT NULL DEFAULT 0,
    "chosen_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 2.6 Import/Export Tables
CREATE TABLE "imports" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "organisation_id" UUID NOT NULL REFERENCES "organisations"("id") ON DELETE CASCADE,
    "created_by" UUID NOT NULL REFERENCES "users"("id") ON DELETE SET NULL,
    "import_type" VARCHAR(50) NOT NULL,
    "file_name" VARCHAR(255) NOT NULL,
    "file_size" INTEGER NOT NULL,
    "file_url" VARCHAR(500) NOT NULL,
    "status" "ImportStatus" NOT NULL DEFAULT 'UPLOADED',
    "total_rows" INTEGER,
    "valid_rows" INTEGER,
    "error_rows" INTEGER,
    "imported_rows" INTEGER,
    "skipped_rows" INTEGER,
    "duplicate_handling" VARCHAR(20) NOT NULL DEFAULT 'skip',
    "validated_at" TIMESTAMP,
    "confirmed_at" TIMESTAMP,
    "completed_at" TIMESTAMP,
    "error_message" TEXT,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE "import_rows" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "import_id" UUID NOT NULL REFERENCES "imports"("id") ON DELETE CASCADE,
    "row_number" INTEGER NOT NULL,
    "row_data" JSONB NOT NULL,
    "status" "ImportRowStatus" NOT NULL DEFAULT 'PENDING',
    "errors" JSONB,
    "created_record_id" UUID
);

CREATE TABLE "exports" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "organisation_id" UUID NOT NULL REFERENCES "organisations"("id") ON DELETE CASCADE,
    "created_by" UUID NOT NULL REFERENCES "users"("id") ON DELETE SET NULL,
    "export_type" VARCHAR(50) NOT NULL,
    "format" VARCHAR(10) NOT NULL DEFAULT 'csv',
    "parameters" JSONB NOT NULL DEFAULT '{}',
    "status" "ExportStatus" NOT NULL DEFAULT 'PENDING',
    "file_url" VARCHAR(500),
    "expires_at" TIMESTAMP,
    "completed_at" TIMESTAMP,
    "error_message" TEXT,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 2.7 System Tables
CREATE TABLE "audit_logs" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "timestamp" TIMESTAMP NOT NULL DEFAULT NOW(),
    "user_id" UUID,
    "user_email" VARCHAR(255),
    "organisation_id" UUID,
    "action" VARCHAR(50) NOT NULL,
    "resource_type" VARCHAR(50) NOT NULL,
    "resource_id" UUID,
    "resource_title" VARCHAR(255),
    "previous_state" JSONB,
    "new_state" JSONB,
    "reason" TEXT,
    "ip_address" INET,
    "user_agent" VARCHAR(500),
    "request_id" UUID
);

CREATE TABLE "notifications" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "user_id" UUID NOT NULL REFERENCES "users"("id") ON DELETE CASCADE,
    "type" "NotificationType" NOT NULL,
    "title" VARCHAR(255) NOT NULL,
    "message" TEXT NOT NULL,
    "data" JSONB,
    "read" BOOLEAN NOT NULL DEFAULT FALSE,
    "read_at" TIMESTAMP,
    "action_url" VARCHAR(500),
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE "notification_templates" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "code" VARCHAR(50) NOT NULL UNIQUE,
    "name" VARCHAR(100) NOT NULL,
    "type" "NotificationType" NOT NULL,
    "subject" VARCHAR(255) NOT NULL,
    "body_text" TEXT NOT NULL,
    "body_html" TEXT,
    "variables" TEXT[] NOT NULL DEFAULT '{}',
    "active" BOOLEAN NOT NULL DEFAULT TRUE,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "updated_at" TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE "jobs" (
    "id" UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "type" "JobType" NOT NULL,
    "status" "JobStatus" NOT NULL DEFAULT 'PENDING',
    "priority" INTEGER NOT NULL DEFAULT 0,
    "payload" JSONB NOT NULL DEFAULT '{}',
    "result" JSONB,
    "error_message" TEXT,
    "attempts" INTEGER NOT NULL DEFAULT 0,
    "max_attempts" INTEGER NOT NULL DEFAULT 3,
    "scheduled_at" TIMESTAMP NOT NULL DEFAULT NOW(),
    "started_at" TIMESTAMP,
    "completed_at" TIMESTAMP,
    "created_at" TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 4.2 Rollback Commands

```sql
-- ROLLBACK PHASE 2
DROP TABLE IF EXISTS "jobs" CASCADE;
DROP TABLE IF EXISTS "notification_templates" CASCADE;
DROP TABLE IF EXISTS "notifications" CASCADE;
DROP TABLE IF EXISTS "audit_logs" CASCADE;
DROP TABLE IF EXISTS "exports" CASCADE;
DROP TABLE IF EXISTS "import_rows" CASCADE;
DROP TABLE IF EXISTS "imports" CASCADE;
DROP TABLE IF EXISTS "scenario_attempt_choices" CASCADE;
DROP TABLE IF EXISTS "quiz_answers" CASCADE;
DROP TABLE IF EXISTS "user_progress" CASCADE;
DROP TABLE IF EXISTS "review_comments" CASCADE;
DROP TABLE IF EXISTS "content_reviews" CASCADE;
DROP TABLE IF EXISTS "content_versions" CASCADE;
DROP TABLE IF EXISTS "password_reset_tokens" CASCADE;
DROP TABLE IF EXISTS "user_sessions" CASCADE;
DROP TABLE IF EXISTS "subscriptions" CASCADE;
```

---

## 5. Phase 3: Add Columns to Existing Tables

**Risk Level:** MEDIUM
**Reversible:** YES
**Downtime:** NONE

### 5.1 SQL Commands

```sql
-- ============================================
-- PHASE 3: ADD COLUMNS TO EXISTING TABLES
-- ============================================

-- 3.1 Organisation columns
ALTER TABLE "organisations"
    ADD COLUMN IF NOT EXISTS "type" "OrganisationType" DEFAULT 'OTHER',
    ADD COLUMN IF NOT EXISTS "org_status" "OrganisationStatus" DEFAULT 'ACTIVE',
    ADD COLUMN IF NOT EXISTS "subscription_tier" "SubscriptionTier" DEFAULT 'STARTER',
    ADD COLUMN IF NOT EXISTS "settings" JSONB DEFAULT '{}',
    ADD COLUMN IF NOT EXISTS "branding" JSONB,
    ADD COLUMN IF NOT EXISTS "billing_email" VARCHAR(255),
    ADD COLUMN IF NOT EXISTS "billing_address" JSONB,
    ADD COLUMN IF NOT EXISTS "trial_ends_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "cancelled_at" TIMESTAMP;

-- 3.2 User columns
ALTER TABLE "users"
    ADD COLUMN IF NOT EXISTS "first_name" VARCHAR(100),
    ADD COLUMN IF NOT EXISTS "last_name" VARCHAR(100),
    ADD COLUMN IF NOT EXISTS "user_status" "UserStatus" DEFAULT 'ACTIVE',
    ADD COLUMN IF NOT EXISTS "job_title" VARCHAR(100),
    ADD COLUMN IF NOT EXISTS "department" VARCHAR(100),
    ADD COLUMN IF NOT EXISTS "hire_date" DATE,
    ADD COLUMN IF NOT EXISTS "avatar_url" VARCHAR(500),
    ADD COLUMN IF NOT EXISTS "preferences" JSONB DEFAULT '{}',
    ADD COLUMN IF NOT EXISTS "last_login_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "invited_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "invited_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "activated_at" TIMESTAMP;

-- 3.3 Wine columns (content lifecycle)
ALTER TABLE "wines"
    ADD COLUMN IF NOT EXISTS "price" DECIMAL(10,2),
    ADD COLUMN IF NOT EXISTS "tasting_notes" TEXT,
    ADD COLUMN IF NOT EXISTS "food_pairings" TEXT[],
    ADD COLUMN IF NOT EXISTS "serving_temperature" VARCHAR(20),
    ADD COLUMN IF NOT EXISTS "decanting_time" VARCHAR(50),
    ADD COLUMN IF NOT EXISTS "image_url" VARCHAR(500),
    ADD COLUMN IF NOT EXISTS "status" "ContentStatus" DEFAULT 'PUBLISHED',
    ADD COLUMN IF NOT EXISTS "created_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "updated_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "published_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "published_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "archived_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "archived_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "review_requested_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "review_requested_by" UUID REFERENCES "users"("id") ON DELETE SET NULL;

-- 3.4 Module columns (content lifecycle)
ALTER TABLE "modules"
    ADD COLUMN IF NOT EXISTS "tier" "Tier" DEFAULT 'bronze',
    ADD COLUMN IF NOT EXISTS "status" "ContentStatus" DEFAULT 'PUBLISHED',
    ADD COLUMN IF NOT EXISTS "version" INTEGER DEFAULT 1,
    ADD COLUMN IF NOT EXISTS "created_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "updated_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "published_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "published_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "archived_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "archived_by" UUID REFERENCES "users"("id") ON DELETE SET NULL;

-- 3.5 Quiz columns (content lifecycle)
ALTER TABLE "quizzes"
    ADD COLUMN IF NOT EXISTS "tier" "Tier" DEFAULT 'bronze',
    ADD COLUMN IF NOT EXISTS "status" "ContentStatus" DEFAULT 'PUBLISHED',
    ADD COLUMN IF NOT EXISTS "version" INTEGER DEFAULT 1,
    ADD COLUMN IF NOT EXISTS "created_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "updated_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "published_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "published_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "archived_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "archived_by" UUID REFERENCES "users"("id") ON DELETE SET NULL;

-- 3.6 Question columns
ALTER TABLE "questions"
    ADD COLUMN IF NOT EXISTS "difficulty" "Difficulty" DEFAULT 'medium',
    ADD COLUMN IF NOT EXISTS "related_wine_ids" UUID[];

-- 3.7 Scenario columns (content lifecycle)
ALTER TABLE "scenarios"
    ADD COLUMN IF NOT EXISTS "status" "ContentStatus" DEFAULT 'PUBLISHED',
    ADD COLUMN IF NOT EXISTS "version" INTEGER DEFAULT 1,
    ADD COLUMN IF NOT EXISTS "created_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "updated_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "published_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "published_by" UUID REFERENCES "users"("id") ON DELETE SET NULL,
    ADD COLUMN IF NOT EXISTS "archived_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "archived_by" UUID REFERENCES "users"("id") ON DELETE SET NULL;

-- 3.8 Lesson columns
ALTER TABLE "lessons"
    ADD COLUMN IF NOT EXISTS "deleted_at" TIMESTAMP;

-- 3.9 ScenarioWine columns
ALTER TABLE "scenario_wines"
    ADD COLUMN IF NOT EXISTS "sort_order" INTEGER DEFAULT 0;

-- 3.10 Certification columns
ALTER TABLE "certifications"
    ADD COLUMN IF NOT EXISTS "certificate_number" VARCHAR(50),
    ADD COLUMN IF NOT EXISTS "certificate_url" VARCHAR(500),
    ADD COLUMN IF NOT EXISTS "expires_at" TIMESTAMP;

-- 3.11 QuizAttempt columns
ALTER TABLE "quiz_attempts"
    ADD COLUMN IF NOT EXISTS "time_spent_seconds" INTEGER,
    ADD COLUMN IF NOT EXISTS "completed_at" TIMESTAMP,
    ADD COLUMN IF NOT EXISTS "expires_at" TIMESTAMP;

-- Rename attemptedAt to startedAt if exists
-- (Handle this carefully - may already be named differently)
-- ALTER TABLE "quiz_attempts" RENAME COLUMN "attempted_at" TO "started_at";
```

### 5.2 Rollback Commands

```sql
-- ROLLBACK PHASE 3
-- Organisation columns
ALTER TABLE "organisations"
    DROP COLUMN IF EXISTS "type",
    DROP COLUMN IF EXISTS "org_status",
    DROP COLUMN IF EXISTS "subscription_tier",
    DROP COLUMN IF EXISTS "settings",
    DROP COLUMN IF EXISTS "branding",
    DROP COLUMN IF EXISTS "billing_email",
    DROP COLUMN IF EXISTS "billing_address",
    DROP COLUMN IF EXISTS "trial_ends_at",
    DROP COLUMN IF EXISTS "cancelled_at";

-- User columns
ALTER TABLE "users"
    DROP COLUMN IF EXISTS "first_name",
    DROP COLUMN IF EXISTS "last_name",
    DROP COLUMN IF EXISTS "user_status",
    DROP COLUMN IF EXISTS "job_title",
    DROP COLUMN IF EXISTS "department",
    DROP COLUMN IF EXISTS "hire_date",
    DROP COLUMN IF EXISTS "avatar_url",
    DROP COLUMN IF EXISTS "preferences",
    DROP COLUMN IF EXISTS "last_login_at",
    DROP COLUMN IF EXISTS "invited_at",
    DROP COLUMN IF EXISTS "invited_by",
    DROP COLUMN IF EXISTS "activated_at";

-- Wine columns
ALTER TABLE "wines"
    DROP COLUMN IF EXISTS "price",
    DROP COLUMN IF EXISTS "tasting_notes",
    DROP COLUMN IF EXISTS "food_pairings",
    DROP COLUMN IF EXISTS "serving_temperature",
    DROP COLUMN IF EXISTS "decanting_time",
    DROP COLUMN IF EXISTS "image_url",
    DROP COLUMN IF EXISTS "status",
    DROP COLUMN IF EXISTS "created_by",
    DROP COLUMN IF EXISTS "updated_by",
    DROP COLUMN IF EXISTS "published_at",
    DROP COLUMN IF EXISTS "published_by",
    DROP COLUMN IF EXISTS "archived_at",
    DROP COLUMN IF EXISTS "archived_by",
    DROP COLUMN IF EXISTS "review_requested_at",
    DROP COLUMN IF EXISTS "review_requested_by";

-- Module columns
ALTER TABLE "modules"
    DROP COLUMN IF EXISTS "tier",
    DROP COLUMN IF EXISTS "status",
    DROP COLUMN IF EXISTS "version",
    DROP COLUMN IF EXISTS "created_by",
    DROP COLUMN IF EXISTS "updated_by",
    DROP COLUMN IF EXISTS "published_at",
    DROP COLUMN IF EXISTS "published_by",
    DROP COLUMN IF EXISTS "archived_at",
    DROP COLUMN IF EXISTS "archived_by";

-- Quiz columns
ALTER TABLE "quizzes"
    DROP COLUMN IF EXISTS "tier",
    DROP COLUMN IF EXISTS "status",
    DROP COLUMN IF EXISTS "version",
    DROP COLUMN IF EXISTS "created_by",
    DROP COLUMN IF EXISTS "updated_by",
    DROP COLUMN IF EXISTS "published_at",
    DROP COLUMN IF EXISTS "published_by",
    DROP COLUMN IF EXISTS "archived_at",
    DROP COLUMN IF EXISTS "archived_by";

-- Question columns
ALTER TABLE "questions"
    DROP COLUMN IF EXISTS "difficulty",
    DROP COLUMN IF EXISTS "related_wine_ids";

-- Scenario columns
ALTER TABLE "scenarios"
    DROP COLUMN IF EXISTS "status",
    DROP COLUMN IF EXISTS "version",
    DROP COLUMN IF EXISTS "created_by",
    DROP COLUMN IF EXISTS "updated_by",
    DROP COLUMN IF EXISTS "published_at",
    DROP COLUMN IF EXISTS "published_by",
    DROP COLUMN IF EXISTS "archived_at",
    DROP COLUMN IF EXISTS "archived_by";

-- Other columns
ALTER TABLE "lessons" DROP COLUMN IF EXISTS "deleted_at";
ALTER TABLE "scenario_wines" DROP COLUMN IF EXISTS "sort_order";
ALTER TABLE "certifications" DROP COLUMN IF EXISTS "certificate_number";
ALTER TABLE "certifications" DROP COLUMN IF EXISTS "certificate_url";
ALTER TABLE "certifications" DROP COLUMN IF EXISTS "expires_at";
ALTER TABLE "quiz_attempts" DROP COLUMN IF EXISTS "time_spent_seconds";
ALTER TABLE "quiz_attempts" DROP COLUMN IF EXISTS "completed_at";
ALTER TABLE "quiz_attempts" DROP COLUMN IF EXISTS "expires_at";
```

---

## 6. Phase 4: Create Indexes

**Risk Level:** LOW
**Reversible:** YES
**Downtime:** NONE (using CONCURRENTLY)

### 6.1 SQL Commands

```sql
-- ============================================
-- PHASE 4: CREATE INDEXES
-- ============================================

-- Use CONCURRENTLY to avoid locking (requires not in transaction)

-- Subscriptions
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_subscriptions_organisation_id"
    ON "subscriptions"("organisation_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_subscriptions_status"
    ON "subscriptions"("status");

-- User sessions
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_user_sessions_user_id"
    ON "user_sessions"("user_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_user_sessions_expires_at"
    ON "user_sessions"("expires_at");

-- Password reset tokens
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_password_reset_tokens_user_id"
    ON "password_reset_tokens"("user_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_password_reset_tokens_expires_at"
    ON "password_reset_tokens"("expires_at");

-- Content versions
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_content_versions_content"
    ON "content_versions"("content_type", "content_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_content_versions_created_by"
    ON "content_versions"("created_by");

-- Content reviews
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_content_reviews_content"
    ON "content_reviews"("content_type", "content_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_content_reviews_reviewer"
    ON "content_reviews"("reviewer_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_content_reviews_status"
    ON "content_reviews"("status");

-- Review comments
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_review_comments_review_id"
    ON "review_comments"("review_id");

-- User progress
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_user_progress_user_id"
    ON "user_progress"("user_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_user_progress_content"
    ON "user_progress"("content_type", "content_id");

-- Quiz answers
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_quiz_answers_attempt_id"
    ON "quiz_answers"("attempt_id");

-- Scenario attempt choices
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_scenario_attempt_choices_attempt_id"
    ON "scenario_attempt_choices"("attempt_id");

-- Imports
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_imports_organisation_id"
    ON "imports"("organisation_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_imports_status"
    ON "imports"("status");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_imports_created_at"
    ON "imports"("created_at");

-- Import rows
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_import_rows_import_id"
    ON "import_rows"("import_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_import_rows_status"
    ON "import_rows"("status");

-- Exports
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_exports_organisation_id"
    ON "exports"("organisation_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_exports_status"
    ON "exports"("status");

-- Audit logs (critical for performance)
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_audit_logs_timestamp"
    ON "audit_logs"("timestamp" DESC);
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_audit_logs_organisation_id"
    ON "audit_logs"("organisation_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_audit_logs_user_id"
    ON "audit_logs"("user_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_audit_logs_resource"
    ON "audit_logs"("resource_type", "resource_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_audit_logs_org_time"
    ON "audit_logs"("organisation_id", "timestamp" DESC);

-- Notifications
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_notifications_user_id"
    ON "notifications"("user_id");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_notifications_user_read"
    ON "notifications"("user_id", "read");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_notifications_created_at"
    ON "notifications"("created_at" DESC);

-- Jobs
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_jobs_status"
    ON "jobs"("status");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_jobs_type"
    ON "jobs"("type");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_jobs_scheduled_at"
    ON "jobs"("scheduled_at");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_jobs_pending"
    ON "jobs"("status", "scheduled_at") WHERE "status" = 'PENDING';

-- Content status indexes (existing tables)
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_wines_status"
    ON "wines"("status");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_modules_status"
    ON "modules"("status");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_quizzes_status"
    ON "quizzes"("status");
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_scenarios_status"
    ON "scenarios"("status");

-- User status index
CREATE INDEX CONCURRENTLY IF NOT EXISTS "idx_users_user_status"
    ON "users"("user_status");

-- Certification unique constraint
CREATE UNIQUE INDEX CONCURRENTLY IF NOT EXISTS "idx_certifications_certificate_number"
    ON "certifications"("certificate_number") WHERE "certificate_number" IS NOT NULL;
```

### 6.2 Rollback Commands

```sql
-- ROLLBACK PHASE 4
DROP INDEX CONCURRENTLY IF EXISTS "idx_subscriptions_organisation_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_subscriptions_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_user_sessions_user_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_user_sessions_expires_at";
DROP INDEX CONCURRENTLY IF EXISTS "idx_password_reset_tokens_user_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_password_reset_tokens_expires_at";
DROP INDEX CONCURRENTLY IF EXISTS "idx_content_versions_content";
DROP INDEX CONCURRENTLY IF EXISTS "idx_content_versions_created_by";
DROP INDEX CONCURRENTLY IF EXISTS "idx_content_reviews_content";
DROP INDEX CONCURRENTLY IF EXISTS "idx_content_reviews_reviewer";
DROP INDEX CONCURRENTLY IF EXISTS "idx_content_reviews_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_review_comments_review_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_user_progress_user_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_user_progress_content";
DROP INDEX CONCURRENTLY IF EXISTS "idx_quiz_answers_attempt_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_scenario_attempt_choices_attempt_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_imports_organisation_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_imports_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_imports_created_at";
DROP INDEX CONCURRENTLY IF EXISTS "idx_import_rows_import_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_import_rows_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_exports_organisation_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_exports_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_audit_logs_timestamp";
DROP INDEX CONCURRENTLY IF EXISTS "idx_audit_logs_organisation_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_audit_logs_user_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_audit_logs_resource";
DROP INDEX CONCURRENTLY IF EXISTS "idx_audit_logs_org_time";
DROP INDEX CONCURRENTLY IF EXISTS "idx_notifications_user_id";
DROP INDEX CONCURRENTLY IF EXISTS "idx_notifications_user_read";
DROP INDEX CONCURRENTLY IF EXISTS "idx_notifications_created_at";
DROP INDEX CONCURRENTLY IF EXISTS "idx_jobs_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_jobs_type";
DROP INDEX CONCURRENTLY IF EXISTS "idx_jobs_scheduled_at";
DROP INDEX CONCURRENTLY IF EXISTS "idx_jobs_pending";
DROP INDEX CONCURRENTLY IF EXISTS "idx_wines_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_modules_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_quizzes_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_scenarios_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_users_user_status";
DROP INDEX CONCURRENTLY IF EXISTS "idx_certifications_certificate_number";
```

---

## 7. Phase 5: Backfill Existing Data

**Risk Level:** LOW
**Reversible:** N/A (data updates)
**Downtime:** NONE

### 7.1 SQL Commands

```sql
-- ============================================
-- PHASE 5: BACKFILL EXISTING DATA
-- ============================================

-- 5.1 Set content status to PUBLISHED for existing content
UPDATE "wines"
SET "status" = 'PUBLISHED',
    "published_at" = "created_at",
    "version" = 1
WHERE "status" IS NULL;

UPDATE "modules"
SET "status" = 'PUBLISHED',
    "published_at" = "created_at",
    "version" = 1
WHERE "status" IS NULL;

UPDATE "quizzes"
SET "status" = 'PUBLISHED',
    "published_at" = "created_at",
    "version" = 1
WHERE "status" IS NULL;

UPDATE "scenarios"
SET "status" = 'PUBLISHED',
    "published_at" = "created_at",
    "version" = 1
WHERE "status" IS NULL;

-- 5.2 Set user status based on suspendedAt
UPDATE "users"
SET "user_status" = CASE
    WHEN "suspended_at" IS NOT NULL THEN 'SUSPENDED'
    ELSE 'ACTIVE'
END
WHERE "user_status" IS NULL;

-- 5.3 Set organisation status based on suspendedAt
UPDATE "organisations"
SET "org_status" = CASE
    WHEN "suspended_at" IS NOT NULL THEN 'SUSPENDED'
    ELSE 'ACTIVE'
END
WHERE "org_status" IS NULL;

-- 5.4 Generate certificate numbers for existing certifications
UPDATE "certifications"
SET "certificate_number" =
    'SS-' || UPPER(SUBSTRING("level"::text FROM 1 FOR 1)) ||
    '-' || TO_CHAR("awarded_at", 'YYYYMMDD') ||
    '-' || SUBSTRING("id"::text FROM 1 FOR 8)
WHERE "certificate_number" IS NULL;

-- 5.5 Set default difficulty for questions
UPDATE "questions"
SET "difficulty" = 'medium'
WHERE "difficulty" IS NULL;

-- 5.6 Set default tier for modules
UPDATE "modules"
SET "tier" = 'bronze'
WHERE "tier" IS NULL;

-- 5.7 Set default tier for quizzes
UPDATE "quizzes"
SET "tier" = 'bronze'
WHERE "tier" IS NULL;
```

---

## 8. Phase 6: Data Migration (JSON → Tables)

**Risk Level:** MEDIUM
**Reversible:** YES (keep JSON fields)
**Downtime:** NONE

### 8.1 QuizAttempt.answers → QuizAnswer

```sql
-- ============================================
-- PHASE 6.1: MIGRATE QUIZ ANSWERS
-- ============================================

-- Create function to migrate JSON answers
CREATE OR REPLACE FUNCTION migrate_quiz_answers()
RETURNS INTEGER AS $$
DECLARE
    attempt RECORD;
    answer_data JSONB;
    question_id UUID;
    option_id UUID;
    migrated_count INTEGER := 0;
BEGIN
    FOR attempt IN
        SELECT id, answers
        FROM quiz_attempts
        WHERE answers IS NOT NULL
          AND answers != '{}'::jsonb
    LOOP
        FOR question_id, option_id IN
            SELECT key::uuid, value::uuid
            FROM jsonb_each_text(attempt.answers)
        LOOP
            INSERT INTO quiz_answers (
                attempt_id,
                question_id,
                selected_option_ids,
                is_correct
            )
            SELECT
                attempt.id,
                question_id,
                ARRAY[option_id],
                COALESCE(o.is_correct, false)
            FROM options o
            WHERE o.id = option_id
            ON CONFLICT (attempt_id, question_id) DO NOTHING;

            migrated_count := migrated_count + 1;
        END LOOP;
    END LOOP;

    RETURN migrated_count;
END;
$$ LANGUAGE plpgsql;

-- Run migration
SELECT migrate_quiz_answers();

-- Cleanup function
DROP FUNCTION migrate_quiz_answers();
```

### 8.2 ScenarioAttempt.choicesMade → ScenarioAttemptChoice

```sql
-- ============================================
-- PHASE 6.2: MIGRATE SCENARIO CHOICES
-- ============================================

-- Create function to migrate JSON choices
CREATE OR REPLACE FUNCTION migrate_scenario_choices()
RETURNS INTEGER AS $$
DECLARE
    attempt RECORD;
    choice_id UUID;
    choice_data RECORD;
    migrated_count INTEGER := 0;
BEGIN
    FOR attempt IN
        SELECT id, choices_made
        FROM scenario_attempts
        WHERE choices_made IS NOT NULL
          AND jsonb_array_length(choices_made) > 0
    LOOP
        FOR choice_id IN
            SELECT jsonb_array_elements_text(attempt.choices_made)::uuid
        LOOP
            SELECT sc.step_id, sc.points
            INTO choice_data
            FROM scenario_choices sc
            WHERE sc.id = choice_id;

            IF choice_data.step_id IS NOT NULL THEN
                INSERT INTO scenario_attempt_choices (
                    attempt_id,
                    step_id,
                    choice_id,
                    points_awarded
                )
                VALUES (
                    attempt.id,
                    choice_data.step_id,
                    choice_id,
                    COALESCE(choice_data.points, 0)
                )
                ON CONFLICT DO NOTHING;

                migrated_count := migrated_count + 1;
            END IF;
        END LOOP;
    END LOOP;

    RETURN migrated_count;
END;
$$ LANGUAGE plpgsql;

-- Run migration
SELECT migrate_scenario_choices();

-- Cleanup function
DROP FUNCTION migrate_scenario_choices();
```

---

## 9. Post-Migration Verification

### 9.1 Verification Queries

```sql
-- ============================================
-- POST-MIGRATION VERIFICATION
-- ============================================

-- Check enum creation
SELECT typname FROM pg_type WHERE typname IN (
    'ContentStatus', 'UserStatus', 'OrganisationType',
    'ImportStatus', 'JobStatus', 'NotificationType'
);

-- Check new tables exist
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public'
AND table_name IN (
    'subscriptions', 'user_sessions', 'content_versions',
    'content_reviews', 'audit_logs', 'notifications', 'jobs'
);

-- Check content status distribution
SELECT status, COUNT(*) FROM wines GROUP BY status;
SELECT status, COUNT(*) FROM modules GROUP BY status;
SELECT status, COUNT(*) FROM quizzes GROUP BY status;
SELECT status, COUNT(*) FROM scenarios GROUP BY status;

-- Check user status distribution
SELECT user_status, COUNT(*) FROM users GROUP BY user_status;

-- Check quiz answer migration
SELECT
    (SELECT COUNT(*) FROM quiz_attempts WHERE answers IS NOT NULL) as json_attempts,
    (SELECT COUNT(DISTINCT attempt_id) FROM quiz_answers) as migrated_attempts;

-- Check index creation
SELECT indexname FROM pg_indexes
WHERE schemaname = 'public'
AND indexname LIKE 'idx_%'
ORDER BY indexname;
```

### 9.2 Prisma Verification

```bash
# Pull schema from database
npx prisma db pull

# Generate client
npx prisma generate

# Verify schema matches
npx prisma validate
```

---

## 10. Execution Timeline

| Step | Duration | Command |
|------|----------|---------|
| Backup database | 2-5 min | `pg_dump` |
| Phase 1: Enums | < 1 min | SQL |
| Phase 2: Tables | < 1 min | SQL |
| Phase 3: Columns | < 1 min | SQL |
| Phase 4: Indexes | 2-5 min | SQL (CONCURRENTLY) |
| Phase 5: Backfill | 1-5 min | SQL (depends on data volume) |
| Phase 6: Migration | 2-10 min | SQL (depends on data volume) |
| Verification | 2-5 min | SQL + Prisma |
| **Total** | **10-30 min** | — |

---

## 11. Rollback Procedure

### 11.1 Full Rollback (All Phases)

```bash
#!/bin/bash
# Full rollback script - run in reverse order

echo "=== FULL ROLLBACK ==="

# Phase 4: Drop indexes
psql "$DATABASE_URL" -f rollback_phase4_indexes.sql

# Phase 3: Drop columns
psql "$DATABASE_URL" -f rollback_phase3_columns.sql

# Phase 2: Drop tables
psql "$DATABASE_URL" -f rollback_phase2_tables.sql

# Phase 1: Drop enums (only if tables dropped)
psql "$DATABASE_URL" -f rollback_phase1_enums.sql

# Restore Prisma schema
cp prisma/schema.prisma.backup prisma/schema.prisma
npx prisma generate

echo "=== ROLLBACK COMPLETE ==="
```

### 11.2 Partial Rollback (Keep Tables, Remove Columns)

Use if you need to keep new tables but remove changes to existing tables.

---

*Document generated: 2026-01-21*
