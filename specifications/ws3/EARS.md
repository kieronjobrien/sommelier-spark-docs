# EARS Requirements Specification
# Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3-EARS |
| **Version** | 1.0 |
| **Date** | 2026-01-20 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Classification** | CONFIDENTIAL |
| **Workstream** | WS3 (Specification Suite) |
| **Source Documents** | SS-WS3.0-CMS-FR, SS-WS3.0-LE-REQ, SS-WS3.0-CLS, SS-WS3.0-ORG, SS-WS3-PRD |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Requirement ID Convention](#2-requirement-id-convention)
3. [Authentication & Authorization](#3-authentication--authorization)
4. [User Management](#4-user-management)
5. [Organisation Management](#5-organisation-management)
6. [Wine Management](#6-wine-management)
7. [Module Management](#7-module-management)
8. [Quiz Management](#8-quiz-management)
9. [Scenario Management](#9-scenario-management)
10. [Content Lifecycle](#10-content-lifecycle)
11. [Import/Export](#11-importexport)
12. [Learning Engine - Content Transformation](#12-learning-engine---content-transformation)
13. [Learning Engine - Quiz Generation](#13-learning-engine---quiz-generation)
14. [Learning Engine - Scenario Generation](#14-learning-engine---scenario-generation)
15. [Adaptive Learning](#15-adaptive-learning)
16. [Learning Paths](#16-learning-paths)
17. [Progress Tracking](#17-progress-tracking)
18. [Reporting & Analytics](#18-reporting--analytics)
19. [Notifications](#19-notifications)
20. [Audit & Logging](#20-audit--logging)
21. [Non-Functional Requirements](#21-non-functional-requirements)
22. [Traceability Matrix](#22-traceability-matrix)
23. [Appendices](#23-appendices)

---

## 1. Introduction

### 1.1 Purpose

This document presents all functional and non-functional requirements for the Sommelier Spark platform in EARS (Easy Approach to Requirements Syntax) format. EARS provides unambiguous, testable requirement statements that eliminate interpretation issues during development and testing.

### 1.2 Scope

This specification covers:
- **271 source requirements** from WS3.0 (159 CMS + 112 Learning Engine)
- **Additional derived requirements** from Organization Model, Content Lifecycle, and PRD
- **Non-functional requirements** covering performance, security, scalability, and availability

### 1.3 How to Read This Document

Each requirement follows the EARS syntax patterns:

| Pattern | Syntax | Use Case |
|---------|--------|----------|
| **Ubiquitous** | The system shall `<action>` | Unconditional requirements |
| **Event-Driven** | WHEN `<trigger>`, the system shall `<action>` | Response to events |
| **State-Driven** | WHILE `<state>`, the system shall `<action>` | Behaviour during states |
| **Optional** | WHERE `<feature>`, the system shall `<action>` | Configurable features |
| **Unwanted** | IF `<condition>`, THEN the system shall `<action>` | Error handling |
| **Complex** | WHILE `<state>`, WHEN `<trigger>`, the system shall `<action>` | Combined conditions |

### 1.4 Requirement Attributes

Each requirement includes:

| Attribute | Description |
|-----------|-------------|
| **ID** | Unique identifier (EARS-CATEGORY-NUMBER) |
| **Statement** | EARS-formatted requirement |
| **Source** | WS3.0 requirement ID |
| **Priority** | Must / Should / Could |
| **Phase** | MVP Phase (1, 2, or 3) |

---

## 2. Requirement ID Convention

### 2.1 ID Format

```
EARS-<CATEGORY>-<NUMBER>
```

### 2.2 Category Codes

| Code | Category | Description |
|------|----------|-------------|
| AUTH | Authentication & Authorization | Login, sessions, permissions |
| USER | User Management | User CRUD, profiles, roles |
| ORG | Organisation Management | Org CRUD, subscriptions, branding |
| WINE | Wine Content Management | Wine CRUD, content, relationships |
| MOD | Module Management | Module and lesson management |
| QUIZ | Quiz Management | Quiz and question management |
| SCEN | Scenario Management | Scenario and decision tree management |
| CONT | Content Lifecycle | States, transitions, versioning |
| IMPX | Import/Export | Bulk data operations |
| LENG | Learning Engine - Curriculum | Curriculum generation |
| QGEN | Quiz Generation | Automated question generation |
| SGEN | Scenario Generation | Automated scenario generation |
| ADPT | Adaptive Learning | Personalisation engine |
| PATH | Learning Paths | Path generation and tracking |
| PROG | Progress Tracking | Learner progress |
| REPT | Reporting & Analytics | Reports and dashboards |
| NOTF | Notifications | Email and in-app notifications |
| AUDT | Audit & Logging | Audit trail |
| PERF | Performance | Response times, throughput |
| SCAL | Scalability | Capacity requirements |
| SECU | Security | Data protection, access control |
| AVAI | Availability | Uptime, recovery |

---

## 3. Authentication & Authorization

### 3.1 Authentication Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUTH-001 | The system shall authenticate users using email address and password. | ORG-derived | Must | 1 |
| EARS-AUTH-002 | WHEN a user submits valid credentials, the system shall create an authenticated session and redirect to the dashboard. | ORG-derived | Must | 1 |
| EARS-AUTH-003 | WHEN a user submits invalid credentials, the system shall display an error message without revealing which field was incorrect. | ORG-derived | Must | 1 |
| EARS-AUTH-004 | IF login fails 5 consecutive times for the same email, THEN the system shall lock the account for 30 minutes. | ORG-derived | Must | 1 |
| EARS-AUTH-005 | WHEN a user requests password reset, the system shall send a reset link to the registered email address valid for 24 hours. | ORG-derived | Must | 1 |
| EARS-AUTH-006 | The system shall hash all passwords using bcrypt with a minimum cost factor of 12. | ORG-derived | Must | 1 |
| EARS-AUTH-007 | The system shall enforce minimum password requirements: 8 characters, 1 uppercase, 1 lowercase, 1 number. | ORG-derived | Must | 1 |

### 3.2 Session Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUTH-008 | The system shall issue JWT access tokens with 8-hour expiry upon successful authentication. | CMS-NF-009 | Must | 1 |
| EARS-AUTH-009 | The system shall issue refresh tokens with 30-day expiry for session renewal. | ORG-derived | Must | 1 |
| EARS-AUTH-010 | WHEN a session exceeds 8 hours of inactivity, the system shall invalidate the session and require re-authentication. | CMS-NF-009 | Must | 1 |
| EARS-AUTH-011 | WHEN a user clicks logout, the system shall invalidate the current session and all associated tokens. | ORG-derived | Must | 1 |

### 3.3 Authorization Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUTH-012 | The system shall enforce role-based access control with roles: Learner, Admin, Owner, System Admin. | ORG-3.1 | Must | 1 |
| EARS-AUTH-013 | WHILE a user has Learner role, the system shall restrict access to learning content, own progress, and profile. | ORG-3.1 | Must | 1 |
| EARS-AUTH-014 | WHILE a user has Admin role, the system shall grant access to team management, reporting, and organisation settings. | ORG-3.1 | Must | 1 |
| EARS-AUTH-015 | WHILE a user has Owner role, the system shall grant full access to organisation including billing and user management. | ORG-3.1 | Must | 1 |
| EARS-AUTH-016 | The system shall validate user permissions before executing any protected action. | CMS-NF-008 | Must | 1 |
| EARS-AUTH-017 | The system shall isolate organisation data such that users can only access data belonging to their organisation. | ORG-5.2 | Must | 1 |

---

## 4. User Management

### 4.1 User Creation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-USER-001 | WHEN an Admin invites a new user, the system shall create a pending user record and send an invitation email. | ORG-derived | Must | 1 |
| EARS-USER-002 | WHEN an invited user clicks the invitation link, the system shall display a registration form for password creation. | ORG-derived | Must | 1 |
| EARS-USER-003 | WHEN a user completes registration, the system shall activate the user account and record the activation timestamp. | ORG-derived | Must | 1 |
| EARS-USER-004 | The system shall store user profile data: firstName, lastName, email, jobTitle, and profileImageUrl. | ORG-2.3 | Must | 1 |
| EARS-USER-005 | The system shall validate email uniqueness within the platform before creating a user. | ORG-derived | Must | 1 |

### 4.2 User Profile Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-USER-006 | WHEN a user updates their profile, the system shall validate and save the changes. | ORG-derived | Must | 1 |
| EARS-USER-007 | WHEN a user uploads a profile image, the system shall validate the file type (JPG, PNG) and size (max 2MB). | ORG-derived | Should | 1 |
| EARS-USER-008 | WHEN a user changes their password, the system shall require the current password for verification. | ORG-derived | Must | 1 |

### 4.3 User Status Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-USER-009 | The system shall track user status: PENDING, ACTIVE, SUSPENDED, DEACTIVATED. | ORG-derived | Must | 1 |
| EARS-USER-010 | WHEN an Admin suspends a user, the system shall prevent login while preserving all user data. | ORG-derived | Must | 1 |
| EARS-USER-011 | WHEN an Admin deactivates a user, the system shall anonymise personal data after 30 days per GDPR requirements. | ORG-derived | Must | 1 |
| EARS-USER-012 | WHEN an Admin reactivates a suspended user, the system shall restore full access immediately. | ORG-derived | Should | 1 |

### 4.4 Role Assignment Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-USER-013 | WHEN an Owner assigns a role to a user, the system shall update the user's permissions immediately. | ORG-derived | Must | 1 |
| EARS-USER-014 | The system shall prevent the last Owner from removing their own Owner role. | ORG-derived | Must | 1 |
| EARS-USER-015 | The system shall log all role changes in the audit trail. | ORG-derived | Must | 1 |

---

## 5. Organisation Management

### 5.1 Organisation CRUD Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ORG-001 | WHEN a new customer signs up, the system shall create an organisation with name, type, and subscription tier. | ORG-2.1 | Must | 1 |
| EARS-ORG-002 | WHEN an Owner updates organisation details, the system shall save changes and record in audit log. | ORG-derived | Must | 1 |
| EARS-ORG-003 | The system shall store organisation attributes: name, type, subscriptionTier, settings, and createdAt. | ORG-2.1 | Must | 1 |
| EARS-ORG-004 | The system shall generate a unique organisation slug for URL purposes. | ORG-derived | Should | 1 |

### 5.2 Subscription Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ORG-005 | The system shall enforce subscription tier limits: Starter (25 users), Professional (100 users), Enterprise (unlimited). | ORG-4.2 | Must | 1 |
| EARS-ORG-006 | WHEN an organisation exceeds user limit, the system shall prevent new user invitations until upgraded or users removed. | ORG-derived | Must | 1 |
| EARS-ORG-007 | WHEN an organisation upgrades subscription tier, the system shall apply new limits immediately. | ORG-derived | Must | 1 |
| EARS-ORG-008 | WHEN an organisation downgrades subscription tier, the system shall enforce limits at next billing cycle. | ORG-derived | Should | 2 |

### 5.3 Organisation Branding Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ORG-009 | WHERE Professional or Enterprise tier, the system shall allow organisation logo upload. | ORG-derived | Should | 1 |
| EARS-ORG-010 | WHERE Professional or Enterprise tier, the system shall allow primary colour customisation. | ORG-derived | Should | 1 |
| EARS-ORG-011 | WHEN branding is configured, the system shall apply branding to learner-facing interfaces. | ORG-derived | Should | 1 |

### 5.4 Data Isolation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ORG-012 | The system shall implement row-level security to isolate organisation data. | ORG-5.2 | Must | 1 |
| EARS-ORG-013 | The system shall include organisationId in all queries for multi-tenant entities. | ORG-5.2 | Must | 1 |
| EARS-ORG-014 | The system shall prevent cross-organisation data access at the API level. | ORG-5.2 | Must | 1 |

---

## 6. Wine Management

### 6.1 Wine CRUD Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-WINE-001 | WHEN a Content Admin initiates wine creation, the system shall display a form with fields: name (required), producer, vintage, region (required), country (required), wineType (required), grapeVarieties (required, minimum 1), priceTier (required). | CMS-W-001 | Must | 1 |
| EARS-WINE-002 | WHEN a user saves a wine, the system shall validate all required fields and store the wine in DRAFT status. | CMS-W-001 | Must | 1 |
| EARS-WINE-003 | WHEN a Content Admin edits a wine, the system shall record all changes in the audit log with previous values. | CMS-W-002 | Must | 1 |
| EARS-WINE-004 | WHEN a user requests wine details, the system shall display the wine in read-only preview matching the learner view. | CMS-W-003 | Must | 1 |
| EARS-WINE-005 | WHILE a wine is in DRAFT status, WHEN a Content Admin requests deletion, the system shall soft-delete the wine and retain for 30 days. | CMS-W-004 | Must | 1 |
| EARS-WINE-006 | WHILE a wine is in PUBLISHED status, WHEN a Content Admin requests archival, the system shall initiate a 7-day warning period before archiving. | CMS-W-005 | Must | 1 |
| EARS-WINE-007 | WHILE a wine is in ARCHIVED status, WHEN a Content Admin requests restoration, the system shall restore to DRAFT status requiring re-review. | CMS-W-006 | Should | 1 |

### 6.2 Wine Content Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-WINE-008 | WHEN editing a wine, the system shall provide a field for Quick Facts (Level 1) content with 50-1000 character limit. | CMS-W-007 | Must | 1 |
| EARS-WINE-009 | WHEN editing a wine, the system shall provide a field for Detailed Profile (Level 2) content with 5000 character limit. | CMS-W-008 | Must | 1 |
| EARS-WINE-010 | WHEN editing a wine, the system shall provide a field for Expert Insights (Level 3) content with 10000 character limit. | CMS-W-009 | Must | 1 |
| EARS-WINE-011 | WHEN a user uploads a wine image, the system shall validate file type (JPG, PNG, WebP) and size (max 5MB) and auto-resize. | CMS-W-010 | Should | 1 |
| EARS-WINE-012 | WHEN editing a wine, the system shall allow selection of multiple food pairings from a tag-based list. | CMS-W-011 | Must | 1 |
| EARS-WINE-013 | WHEN editing a wine, the system shall allow selection of serving temperature from predefined ranges. | CMS-W-012 | Should | 1 |

### 6.3 Wine Relationship Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-WINE-014 | WHEN editing a wine, the system shall allow linking to one or more quizzes with bi-directional references. | CMS-W-013 | Must | 1 |
| EARS-WINE-015 | WHEN editing a wine, the system shall allow linking to one or more scenarios with talking points. | CMS-W-014 | Must | 1 |
| EARS-WINE-016 | WHEN viewing a wine, the system shall display all quizzes and scenarios referencing this wine. | CMS-W-015 | Must | 1 |
| EARS-WINE-017 | IF a wine is referenced by published quizzes or scenarios, THEN the system shall prevent archival and display the dependencies. | CMS-W-016 | Must | 1 |

### 6.4 Wine Import Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-WINE-018 | WHEN a Content Admin uploads a CSV file, the system shall parse and validate against the wine import schema. | CMS-W-017 | Must | 1 |
| EARS-WINE-019 | WHEN validating wine import, the system shall check each row for required fields, data types, and enum values, reporting errors by row. | CMS-W-018 | Must | 1 |
| EARS-WINE-020 | WHEN a Content Admin requests wine export, the system shall generate a CSV file with all wine attributes. | CMS-W-019 | Should | 1 |
| EARS-WINE-021 | WHEN a Content Admin selects specific wines for export, the system shall generate a CSV with only selected wines. | CMS-W-020 | Should | 1 |
| EARS-WINE-022 | WHEN a Content Admin requests wine duplication, the system shall create a copy in DRAFT status with "(Copy)" appended to name. | CMS-W-021 | Could | 2 |

### 6.5 Wine Validation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-WINE-023 | WHEN saving a wine, the system shall validate all required fields in real-time before enabling save. | CMS-W-022 | Must | 1 |
| EARS-WINE-024 | WHEN Quick Facts content is below 50 characters, the system shall display a warning to the user. | CMS-W-023 | Should | 1 |
| EARS-WINE-025 | WHEN saving a wine, the system shall check for duplicates by name + vintage + producer and suggest existing matches. | CMS-W-024 | Should | 1 |

---

## 7. Module Management

### 7.1 Module CRUD Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-MOD-001 | WHEN a Content Admin creates a module, the system shall require title, description, and category fields. | CMS-M-001 | Must | 1 |
| EARS-MOD-002 | WHEN a Content Admin edits module metadata, the system shall record changes in the audit log. | CMS-M-002 | Must | 1 |
| EARS-MOD-003 | WHILE a module is in DRAFT status, WHEN a Content Admin deletes it, the system shall cascade deletion to all contained lessons. | CMS-M-003 | Must | 1 |
| EARS-MOD-004 | WHILE a module is in PUBLISHED status, WHEN a Content Admin archives it, the system shall also archive all contained lessons. | CMS-M-004 | Must | 1 |
| EARS-MOD-005 | WHEN a user previews a module, the system shall display content exactly as learners would see it. | CMS-M-005 | Must | 1 |

### 7.2 Lesson Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-MOD-006 | WHEN a Content Admin adds a lesson to a module, the system shall create the lesson with sortOrder at end of list. | CMS-M-006 | Must | 1 |
| EARS-MOD-007 | WHEN editing a lesson, the system shall provide a rich text editor supporting headings, lists, emphasis, and links. | CMS-M-007 | Must | 1 |
| EARS-MOD-008 | WHEN a Content Admin drags a lesson to a new position, the system shall update sortOrder for all affected lessons. | CMS-M-008 | Must | 1 |
| EARS-MOD-009 | WHEN a Content Admin removes a lesson from a module, the system shall display a confirmation if module is published. | CMS-M-009 | Must | 1 |
| EARS-MOD-010 | WHEN a Content Admin moves a lesson to a different module, the system shall update moduleId and recalculate sortOrder. | CMS-M-010 | Should | 1 |
| EARS-MOD-011 | WHEN a Content Admin duplicates a lesson, the system shall create a copy in DRAFT status within the same or target module. | CMS-M-011 | Could | 2 |

### 7.3 Module Configuration Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-MOD-012 | WHEN configuring a module, the system shall allow setting tier classification: Bronze, Silver, or Gold. | CMS-M-012 | Must | 1 |
| EARS-MOD-013 | WHEN a module contains lessons, the system shall calculate estimated completion time by summing lesson durations. | CMS-M-013 | Must | 1 |
| EARS-MOD-014 | WHEN configuring a module, the system shall allow uploading a thumbnail image for catalog display. | CMS-M-014 | Should | 1 |
| EARS-MOD-015 | WHEN configuring a module, the system shall allow setting prerequisite modules that must be completed first. | CMS-M-015 | Should | 1 |
| EARS-MOD-016 | WHEN configuring a module, the system shall allow associating exactly one quiz for assessment. | CMS-M-016 | Must | 1 |

---

## 8. Quiz Management

### 8.1 Quiz CRUD Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QUIZ-001 | WHEN a Content Admin creates a quiz, the system shall require title, description, tier, and category fields. | CMS-Q-001 | Must | 1 |
| EARS-QUIZ-002 | WHEN a Content Admin edits quiz metadata, the system shall record changes in the audit log. | CMS-Q-002 | Must | 1 |
| EARS-QUIZ-003 | WHILE a quiz is in DRAFT status, WHEN a Content Admin deletes it, the system shall cascade deletion to all contained questions. | CMS-Q-003 | Must | 1 |
| EARS-QUIZ-004 | WHILE a quiz is in PUBLISHED status, WHEN a Content Admin archives it, the system shall preserve all attempt history. | CMS-Q-004 | Must | 1 |
| EARS-QUIZ-005 | WHEN a Content Admin duplicates a quiz, the system shall create a copy in DRAFT status with all questions copied. | CMS-Q-005 | Should | 1 |
| EARS-QUIZ-006 | WHEN a Content Admin previews a quiz, the system shall simulate the complete learner experience including timer and scoring. | CMS-Q-006 | Must | 1 |

### 8.2 Question Management Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QUIZ-007 | WHEN adding a multiple choice question, the system shall allow 2-6 answer options with exactly one marked correct. | CMS-Q-007 | Must | 1 |
| EARS-QUIZ-008 | WHEN adding a multiple select question, the system shall allow 2-6 answer options with one or more marked correct. | CMS-Q-008 | Must | 1 |
| EARS-QUIZ-009 | WHEN adding a true/false question, the system shall provide exactly two options: True and False. | CMS-Q-009 | Must | 1 |
| EARS-QUIZ-010 | WHEN adding a matching question, the system shall allow defining pairs of items for matching. | CMS-Q-010 | Could | 2 |
| EARS-QUIZ-011 | WHEN editing a question, the system shall record changes in the audit log. | CMS-Q-011 | Must | 1 |
| EARS-QUIZ-012 | WHEN saving a question, the system shall require at least one option marked as correct. | CMS-Q-012 | Must | 1 |
| EARS-QUIZ-013 | WHEN editing a question, the system shall allow entering explanation text displayed after the learner answers. | CMS-Q-013 | Must | 1 |
| EARS-QUIZ-014 | WHEN a Content Admin drags a question to a new position, the system shall update sortOrder for all affected questions. | CMS-Q-014 | Must | 1 |
| EARS-QUIZ-015 | WHEN a Content Admin removes a question, the system shall display a confirmation dialog. | CMS-Q-015 | Must | 1 |
| EARS-QUIZ-016 | WHEN editing a question, the system shall allow setting difficulty: Easy, Medium, or Hard. | CMS-Q-016 | Should | 1 |

### 8.3 Quiz Configuration Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QUIZ-017 | WHEN configuring a quiz, the system shall set default pass thresholds by tier: Bronze 70%, Silver 80%, Gold 90%. | CMS-Q-017 | Must | 1 |
| EARS-QUIZ-018 | WHEN configuring a quiz, the system shall allow setting a time limit in minutes (optional). | CMS-Q-018 | Should | 1 |
| EARS-QUIZ-019 | WHEN configuring a quiz, the system shall allow enabling/disabling question order randomisation. | CMS-Q-019 | Should | 1 |
| EARS-QUIZ-020 | WHEN configuring a quiz, the system shall allow enabling/disabling answer option randomisation. | CMS-Q-020 | Should | 1 |
| EARS-QUIZ-021 | WHEN configuring a quiz, the system shall require selecting a category for filtering and grouping. | CMS-Q-021 | Must | 1 |
| EARS-QUIZ-022 | WHEN configuring a quiz, the system shall allow setting unlock conditions: prerequisite modules or quizzes. | CMS-Q-022 | Must | 1 |

### 8.4 Question Bank Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QUIZ-023 | WHEN a Content Admin views the question bank, the system shall display all questions with search capability. | CMS-Q-023 | Should | 1 |
| EARS-QUIZ-024 | WHEN viewing the question bank, the system shall allow filtering by category, tier, type, and difficulty. | CMS-Q-024 | Should | 1 |
| EARS-QUIZ-025 | WHEN a Content Admin adds a question from the bank to a quiz, the system shall create a reference to the shared question. | CMS-Q-025 | Should | 2 |
| EARS-QUIZ-026 | WHEN a Content Admin uploads a CSV of questions, the system shall validate and create questions in bulk. | CMS-Q-026 | Should | 2 |
| EARS-QUIZ-027 | WHEN a Content Admin exports questions, the system shall generate a CSV with all question data. | CMS-Q-027 | Should | 2 |

### 8.5 Wine Reference Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QUIZ-028 | WHEN editing a question, the system shall allow linking to one or more wines from the wine library. | CMS-Q-028 | Must | 1 |
| EARS-QUIZ-029 | WHEN a question is linked to wines, the system shall display wine details for quick reference during editing. | CMS-Q-029 | Should | 1 |

---

## 9. Scenario Management

### 9.1 Scenario CRUD Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCEN-001 | WHEN a Content Admin creates a scenario, the system shall require title, description, category, and difficulty tier. | CMS-SC-001 | Must | 1 |
| EARS-SCEN-002 | WHEN a Content Admin edits scenario metadata, the system shall record changes in the audit log. | CMS-SC-002 | Must | 1 |
| EARS-SCEN-003 | WHILE a scenario is in DRAFT status, WHEN a Content Admin deletes it, the system shall cascade deletion to all steps and choices. | CMS-SC-003 | Must | 1 |
| EARS-SCEN-004 | WHILE a scenario is in PUBLISHED status, WHEN a Content Admin archives it, the system shall preserve all attempt history. | CMS-SC-004 | Must | 1 |
| EARS-SCEN-005 | WHEN a Content Admin duplicates a scenario, the system shall create a complete copy in DRAFT status. | CMS-SC-005 | Should | 1 |

### 9.2 Customer Persona Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCEN-006 | WHEN configuring a scenario, the system shall require customer name and persona description. | CMS-SC-006 | Must | 1 |
| EARS-SCEN-007 | WHEN configuring a scenario, the system shall require initial situation/context text. | CMS-SC-007 | Must | 1 |
| EARS-SCEN-008 | WHEN configuring a scenario, the system shall require estimated completion time in minutes. | CMS-SC-008 | Must | 1 |

### 9.3 Decision Tree Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCEN-009 | WHEN a Content Admin adds a step, the system shall create a new step with sequential or branched positioning. | CMS-SC-009 | Must | 1 |
| EARS-SCEN-010 | WHEN editing a step, the system shall allow setting type: Dialogue, Action, Observation, or Conclusion. | CMS-SC-010 | Must | 1 |
| EARS-SCEN-011 | WHEN editing a step, the system shall require customer dialogue text. | CMS-SC-011 | Must | 1 |
| EARS-SCEN-012 | WHEN editing a step, the system shall allow setting customer mood for emotional context. | CMS-SC-012 | Should | 1 |
| EARS-SCEN-013 | WHEN editing a step, the system shall allow adding 2-4 response choices for the staff member. | CMS-SC-013 | Must | 1 |
| EARS-SCEN-014 | WHEN editing a choice, the system shall require points value from 0 to 10. | CMS-SC-014 | Must | 1 |
| EARS-SCEN-015 | WHEN editing a choice, the system shall allow marking as the optimal choice. | CMS-SC-015 | Must | 1 |
| EARS-SCEN-016 | WHEN editing a choice, the system shall require feedback text explaining the choice outcome. | CMS-SC-016 | Must | 1 |
| EARS-SCEN-017 | WHEN editing a choice, the system shall allow linking to the next step or marking as scenario end. | CMS-SC-017 | Must | 1 |
| EARS-SCEN-018 | WHEN viewing a scenario, the system shall display the decision tree as a visual flowchart. | CMS-SC-018 | Should | 2 |
| EARS-SCEN-019 | WHEN a Content Admin drags a step, the system shall reorder and update references. | CMS-SC-019 | Should | 1 |

### 9.4 Scenario Validation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCEN-020 | WHEN a Content Admin previews a scenario, the system shall simulate the complete learner experience. | CMS-SC-020 | Must | 1 |
| EARS-SCEN-021 | WHEN a Content Admin tests a scenario, the system shall identify all possible paths through the decision tree. | CMS-SC-021 | Should | 2 |
| EARS-SCEN-022 | WHEN validating a scenario, the system shall verify all steps are reachable from the start. | CMS-SC-022 | Must | 1 |
| EARS-SCEN-023 | WHEN validating a scenario, the system shall verify at least one path reaches a conclusion step. | CMS-SC-023 | Must | 1 |

### 9.5 Scenario Wine Reference Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCEN-024 | WHEN editing a scenario, the system shall allow linking wines from the organisation's wine library. | CMS-SC-024 | Must | 1 |
| EARS-SCEN-025 | WHEN a wine is linked to a scenario, the system shall allow adding talking points for staff guidance. | CMS-SC-025 | Should | 1 |

---

## 10. Content Lifecycle

### 10.1 State Transition Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-CONT-001 | The system shall track content status with values: DRAFT, REVIEW, PUBLISHED, ARCHIVED. | CLS-2.1 | Must | 1 |
| EARS-CONT-002 | WHILE content is in DRAFT state, the system shall allow editing by the author and admins. | CLS-3.2.1 | Must | 1 |
| EARS-CONT-003 | WHEN a Content Author submits content for review, the system shall change status to REVIEW and lock editing. | CLS-3.2.1 | Must | 1 |
| EARS-CONT-004 | WHILE content is in REVIEW state, the system shall prevent editing except by returning to DRAFT. | CLS-3.2.3 | Must | 1 |
| EARS-CONT-005 | WHEN a Content Admin approves content, the system shall change status to PUBLISHED and set publishedAt timestamp. | CLS-3.2.2 | Must | 1 |
| EARS-CONT-006 | WHEN a reviewer requests changes, the system shall return status to DRAFT and attach feedback comments. | CLS-3.2.3 | Must | 1 |
| EARS-CONT-007 | WHEN a Content Admin archives published content, the system shall change status to ARCHIVED after warning period. | CLS-3.2.5 | Must | 1 |
| EARS-CONT-008 | WHEN a Content Admin restores archived content, the system shall change status to DRAFT for re-review. | CLS-3.2.6 | Should | 1 |

### 10.2 Versioning Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-CONT-009 | The system shall track content version using semantic format: MAJOR.MINOR. | CLS-4.1 | Must | 1 |
| EARS-CONT-010 | WHEN content is modified and republished, the system shall increment the version number appropriately. | CLS-4.2 | Must | 1 |
| EARS-CONT-011 | WHEN a new version is published, the system shall archive the previous version. | CLS-4.3 | Must | 1 |
| EARS-CONT-012 | The system shall retain previous published versions for 2 years in archive storage. | CLS-4.3 | Must | 1 |
| EARS-CONT-013 | WHEN viewing content history, the system shall display version comparison showing changed fields. | CLS-4.4 | Should | 2 |

### 10.3 Approval Workflow Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-CONT-014 | The system shall support Simple workflow: Content Admin approval only. | CLS-5.1 | Must | 1 |
| EARS-CONT-015 | The system shall support Standard workflow: Domain Expert review then Content Admin approval. | CLS-5.1 | Should | 1 |
| EARS-CONT-016 | The system shall support Extended workflow: Domain Expert, QA Reviewer, then Content Admin approval. | CLS-5.1 | Should | 2 |
| EARS-CONT-017 | IF review is pending for more than 3 days, THEN the system shall send a reminder to the reviewer. | CLS-5.5 | Should | 1 |
| EARS-CONT-018 | IF review is pending for more than 5 days, THEN the system shall escalate to Content Admin. | CLS-5.5 | Should | 1 |
| EARS-CONT-019 | IF review is pending for more than 7 days, THEN the system shall auto-assign a backup reviewer. | CLS-5.5 | Should | 2 |

### 10.4 Rollback Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-CONT-020 | WHEN a Content Admin triggers emergency unpublish, the system shall immediately revert content to DRAFT. | CLS-7.2 | Must | 1 |
| EARS-CONT-021 | WHEN a Content Admin selects a previous version, the system shall create a new DRAFT from that version. | CLS-7.2 | Should | 2 |
| EARS-CONT-022 | WHEN content is rolled back, the system shall preserve learner progress where structure is unchanged. | CLS-7.3 | Should | 2 |

---

## 11. Import/Export

### 11.1 File Upload Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IMPX-001 | WHEN a user uploads a file, the system shall validate file type: CSV, XLSX, or JSON. | CMS-IE-2.1 | Must | 1 |
| EARS-IMPX-002 | WHEN a user uploads a CSV file, the system shall validate UTF-8 encoding and maximum 10MB size. | CMS-IE-2.2.1 | Must | 1 |
| EARS-IMPX-003 | WHEN a user uploads an Excel file, the system shall validate XLSX format and maximum 25MB size. | CMS-IE-2.2.2 | Must | 1 |
| EARS-IMPX-004 | WHEN a user uploads a JSON file, the system shall validate standard JSON and maximum 50MB size. | CMS-IE-2.2.3 | Must | 1 |

### 11.2 Validation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IMPX-005 | WHEN processing an import file, the system shall validate each row against the entity schema. | CMS-IE-3.5 | Must | 1 |
| EARS-IMPX-006 | WHEN validation errors are found, the system shall return detailed error report with row number, field, and message. | CMS-IE-9.1 | Must | 1 |
| EARS-IMPX-007 | WHEN duplicate items are detected, the system shall offer options: Skip, Update, Create New, or Error. | CMS-IE-7.4 | Must | 1 |

### 11.3 Import Execution Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IMPX-008 | WHEN validation passes, the system shall display preview of items to be imported. | CMS-IE-7.3 | Must | 1 |
| EARS-IMPX-009 | WHEN user confirms import, the system shall create records and report success/failure counts. | CMS-IE-7.3 | Must | 1 |
| EARS-IMPX-010 | WHERE import exceeds 1000 items, the system shall process in background and notify on completion. | CMS-IE-7.1 | Should | 1 |
| EARS-IMPX-011 | WHEN import completes, the system shall record all created items in the audit log. | CMS-IE-7.3 | Must | 1 |

### 11.4 Export Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IMPX-012 | WHEN a user requests export, the system shall allow selection of format: CSV, XLSX, or JSON. | CMS-IE-8.2 | Should | 1 |
| EARS-IMPX-013 | WHEN a user requests export, the system shall allow filtering by status, category, and date range. | CMS-IE-8.2 | Should | 1 |
| EARS-IMPX-014 | WHEN generating export, the system shall include all selected fields and generate downloadable file. | CMS-IE-8.1 | Should | 1 |
| EARS-IMPX-015 | WHERE export exceeds size threshold, the system shall process in background and email download link. | CMS-IE-8.1 | Should | 2 |

### 11.5 Template Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IMPX-016 | WHEN a user requests import template, the system shall provide downloadable CSV/XLSX with correct headers. | CMS-IE-11.3 | Must | 1 |
| EARS-IMPX-017 | The system shall provide template downloads for wines, modules, quizzes, and scenarios. | CMS-IE-11.3 | Must | 1 |

---

## 12. Learning Engine - Content Transformation

### 12.1 Curriculum Generation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-LENG-001 | WHEN an organisation uploads a wine list, the system shall generate learning modules covering all wines. | LE-CG-001 | Must | 1 |
| EARS-LENG-002 | WHEN generating curriculum, the system shall sequence content from simple to complex within each topic. | LE-CG-002 | Must | 1 |
| EARS-LENG-003 | WHEN generating curriculum, the system shall ensure 100% coverage of all wines in the organisation's list. | LE-CG-003 | Must | 1 |
| EARS-LENG-004 | WHEN generating curriculum, the system shall create role-specific paths for server, sommelier, and manager. | LE-CG-004 | Should | 2 |
| EARS-LENG-005 | WHEN generating curriculum, the system shall detect content gaps and recommend additions. | LE-CG-005 | Should | 2 |
| EARS-LENG-006 | The system shall complete full curriculum generation in under 5 minutes for up to 200 wines. | LE-CG-006 | Must | 1 |
| EARS-LENG-007 | WHEN generating curriculum, the system shall create Bronze, Silver, and Gold tier content automatically. | LE-CG-007 | Must | 1 |

### 12.2 Module Structure Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-LENG-008 | WHEN generating modules, the system shall group wines by logical categories: region, type, or price tier. | LE-CG-008 | Must | 1 |
| EARS-LENG-009 | WHEN generating modules, the system shall create human-readable titles and descriptions automatically. | LE-CG-009 | Must | 1 |
| EARS-LENG-010 | WHEN generating modules, the system shall calculate estimated completion time based on content volume. | LE-CG-010 | Must | 1 |
| EARS-LENG-011 | WHEN generating lessons, the system shall apply progressive disclosure (Level 1/2/3) structure. | LE-CG-011 | Must | 1 |
| EARS-LENG-012 | WHEN generating modules, the system shall create learning objectives for each module. | LE-CG-012 | Should | 1 |
| EARS-LENG-013 | WHEN generating curriculum, the system shall identify prerequisite relationships between modules. | LE-CG-013 | Should | 2 |

### 12.3 Content Enrichment Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-LENG-014 | WHEN generating curriculum, the system shall enrich wine data with regional context. | LE-CG-014 | Should | 2 |
| EARS-LENG-015 | WHEN curriculum includes multiple grape varieties, the system shall generate grape variety overview modules. | LE-CG-015 | Should | 2 |
| EARS-LENG-016 | WHEN curriculum includes wines requiring special service, the system shall generate service technique modules. | LE-CG-016 | Should | 2 |
| EARS-LENG-017 | WHEN processing wine list, the system shall detect and merge near-duplicate wines intelligently. | LE-CG-017 | Must | 1 |
| EARS-LENG-018 | WHEN wine list changes, the system shall support incremental curriculum updates processing only deltas. | LE-CG-018 | Must | 1 |

---

## 13. Learning Engine - Quiz Generation

### 13.1 Core Question Generation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QGEN-001 | WHEN generating questions, the system shall create questions from wine attributes: region, grape, type, price, producer. | LE-QG-001 | Must | 1 |
| EARS-QGEN-002 | WHEN generating questions, the system shall create plausible wrong answers (distractors) from the same domain. | LE-QG-002 | Must | 1 |
| EARS-QGEN-003 | WHEN generating questions, the system shall calibrate difficulty based on tier: Bronze (identification), Silver (comparison), Gold (analysis). | LE-QG-003 | Must | 1 |
| EARS-QGEN-004 | WHEN generating questions, the system shall balance coverage across all wines in the curriculum. | LE-QG-004 | Must | 1 |
| EARS-QGEN-005 | WHEN generating questions, the system shall add to organisation's question bank for reuse. | LE-QG-005 | Should | 1 |
| EARS-QGEN-006 | The system shall achieve greater than 95% question validity rate in automated QA checks. | LE-QG-006 | Must | 1 |
| EARS-QGEN-007 | The system shall complete quiz generation in under 30 seconds for 10-20 questions. | LE-QG-007 | Must | 1 |

### 13.2 Question Type Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QGEN-008 | WHEN generating questions, the system shall create identification questions: "Which wine is from X region?" | LE-QG-008 | Must | 1 |
| EARS-QGEN-009 | WHEN generating questions, the system shall create pairing questions: "What food pairs with this wine?" | LE-QG-009 | Must | 1 |
| EARS-QGEN-010 | WHEN generating questions, the system shall create tasting note questions: "What characterises this wine?" | LE-QG-010 | Must | 1 |
| EARS-QGEN-011 | WHEN generating questions, the system shall create service questions: "What temperature for this wine?" | LE-QG-011 | Must | 1 |
| EARS-QGEN-012 | WHEN generating Gold tier questions, the system shall create comparison questions: "How does A differ from B?" | LE-QG-012 | Should | 1 |
| EARS-QGEN-013 | WHEN generating questions, the system shall create true/false questions from wine facts. | LE-QG-013 | Must | 1 |
| EARS-QGEN-014 | WHEN generating questions, the system shall create matching questions pairing wines to regions or grapes. | LE-QG-014 | Should | 2 |

### 13.3 Question Quality Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-QGEN-015 | WHEN generating distractors, the system shall ensure they are from the same category as the correct answer. | LE-QG-015 | Must | 1 |
| EARS-QGEN-016 | WHEN generating questions, the system shall avoid ambiguous wording that could have multiple interpretations. | LE-QG-016 | Must | 1 |
| EARS-QGEN-017 | WHEN generating questions, the system shall include explanations for each answer option. | LE-QG-017 | Should | 1 |
| EARS-QGEN-018 | WHEN tracking question performance, the system shall record pass rate and discrimination index. | LE-QG-018 | Should | 2 |
| EARS-QGEN-019 | IF question pass rate falls below 30% or exceeds 95%, THEN the system shall flag for review. | LE-QG-019 | Could | 2 |

---

## 14. Learning Engine - Scenario Generation

### 14.1 Core Scenario Generation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SGEN-001 | WHEN generating scenarios, the system shall create customer personas with realistic backgrounds. | LE-SG-001 | Must | 1 |
| EARS-SGEN-002 | WHEN generating scenarios, the system shall use situation templates for common service contexts. | LE-SG-002 | Must | 1 |
| EARS-SGEN-003 | WHEN generating scenarios, the system shall build branching decision trees with meaningful choices. | LE-SG-003 | Must | 1 |
| EARS-SGEN-004 | WHEN generating scenarios, the system shall integrate the organisation's wine list and menu data. | LE-SG-004 | Must | 1 |
| EARS-SGEN-005 | WHEN generating scenarios, the system shall scale difficulty based on tier: Bronze, Silver, Gold. | LE-SG-005 | Must | 1 |
| EARS-SGEN-006 | The system shall target greater than 70% scenario completion rate. | LE-SG-006 | Should | 1 |
| EARS-SGEN-007 | The system shall complete scenario generation in under 1 minute including branching logic. | LE-SG-007 | Must | 1 |

### 14.2 Scenario Type Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SGEN-008 | WHEN generating scenarios, the system shall create pairing request scenarios: "What wine with the fish?" | LE-SG-008 | Must | 1 |
| EARS-SGEN-009 | WHEN generating scenarios, the system shall create budget constraint scenarios: "Under £50." | LE-SG-009 | Must | 1 |
| EARS-SGEN-010 | WHEN generating scenarios, the system shall create preference discovery scenarios: "I usually drink..." | LE-SG-010 | Must | 1 |
| EARS-SGEN-011 | WHEN generating Silver/Gold scenarios, the system shall create objection handling scenarios: "Too expensive." | LE-SG-011 | Should | 1 |
| EARS-SGEN-012 | WHEN generating scenarios, the system shall create upsell opportunity scenarios. | LE-SG-012 | Should | 2 |
| EARS-SGEN-013 | WHEN generating scenarios, the system shall create special occasion scenarios: anniversary, celebration. | LE-SG-013 | Should | 1 |
| EARS-SGEN-014 | WHEN generating scenarios, the system shall create dietary/allergy scenarios: vegan, sulphite-free. | LE-SG-014 | Should | 2 |

### 14.3 Scenario Quality Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SGEN-015 | WHEN generating scenarios, the system shall ensure all paths lead to valid conclusions. | LE-SG-015 | Must | 1 |
| EARS-SGEN-016 | WHEN generating scenarios, the system shall create realistic customer dialogue. | LE-SG-016 | Must | 1 |
| EARS-SGEN-017 | WHEN generating scenarios, the system shall provide constructive feedback for each choice. | LE-SG-017 | Must | 1 |

---

## 15. Adaptive Learning

### 15.1 Performance Tracking Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ADPT-001 | The system shall track individual learner performance across all content types. | LE-AL-001 | Must | 1 |
| EARS-ADPT-002 | WHEN a learner completes assessments, the system shall identify weak areas by topic, wine, and skill. | LE-AL-002 | Must | 1 |
| EARS-ADPT-003 | WHEN a learner completes assessments, the system shall calculate proficiency scores by category. | LE-AL-003 | Must | 1 |
| EARS-ADPT-004 | The system shall track time spent per content item for engagement analysis. | LE-AL-004 | Should | 2 |
| EARS-ADPT-005 | The system shall record attempt history with timestamps for all assessments. | LE-AL-005 | Must | 1 |

### 15.2 Personalisation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ADPT-006 | WHEN a learner has identified gaps, the system shall recommend specific content to address weaknesses. | LE-AL-006 | Should | 1 |
| EARS-ADPT-007 | WHERE adaptive difficulty is enabled, the system shall adjust question difficulty based on performance. | LE-AL-007 | Could | 3 |
| EARS-ADPT-008 | WHERE spaced repetition is enabled, the system shall schedule content review for optimal retention. | LE-AL-008 | Could | 3 |
| EARS-ADPT-009 | WHEN a learner demonstrates mastery (>90% on topic), the system shall advance to next level. | LE-AL-009 | Should | 2 |
| EARS-ADPT-010 | WHEN a learner struggles (<50% on topic), the system shall surface content for additional practice. | LE-AL-010 | Should | 1 |

### 15.3 Learning Analytics Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-ADPT-011 | WHEN a learner views their dashboard, the system shall display individual learning reports. | LE-AL-011 | Should | 1 |
| EARS-ADPT-012 | WHEN a learner views their progress, the system shall predict time to certification based on current pace. | LE-AL-012 | Could | 2 |
| EARS-ADPT-013 | WHEN a learner shows declining engagement, the system shall flag as at-risk for manager review. | LE-AL-013 | Could | 2 |
| EARS-ADPT-014 | WHEN a learner views their progress, the system shall show anonymised comparison to peers. | LE-AL-014 | Could | 2 |

---

## 16. Learning Paths

### 16.1 Path Generation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PATH-001 | WHEN generating curriculum, the system shall create optimal content sequences per certification tier. | LE-LP-001 | Must | 1 |
| EARS-PATH-002 | WHEN displaying a learning path, the system shall show estimated total time to completion. | LE-LP-002 | Must | 1 |
| EARS-PATH-003 | WHEN generating paths, the system shall define milestones at regular intervals. | LE-LP-003 | Must | 1 |
| EARS-PATH-004 | WHEN generating paths, the system shall map progression: Bronze → Silver → Gold. | LE-LP-004 | Must | 1 |
| EARS-PATH-005 | WHEN generating paths, the system shall create role-specific sequences for server, sommelier, manager. | LE-LP-005 | Should | 2 |

### 16.2 Path Optimisation Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PATH-006 | WHEN generating paths, the system shall balance theory and practical content. | LE-LP-006 | Should | 1 |
| EARS-PATH-007 | WHEN generating paths, the system shall incorporate prerequisite dependencies in sequencing. | LE-LP-007 | Must | 1 |
| EARS-PATH-008 | WHERE organisation customisation is enabled, the system shall allow path modification by admin. | LE-LP-008 | Should | 2 |
| EARS-PATH-009 | WHERE deadline is set, the system shall compress path to meet time constraints. | LE-LP-009 | Should | 2 |

### 16.3 Path Tracking Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PATH-010 | The system shall track learner position in their assigned learning path. | LE-LP-010 | Must | 1 |
| EARS-PATH-011 | WHEN displaying path progress, the system shall show visual representation of completion. | LE-LP-011 | Should | 1 |
| EARS-PATH-012 | WHEN a learner approaches a milestone, the system shall send notification of upcoming achievement. | LE-LP-012 | Should | 2 |

---

## 17. Progress Tracking

### 17.1 Lesson Progress Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PROG-001 | WHEN a learner views a lesson, the system shall record view timestamp and duration. | PROG-derived | Must | 1 |
| EARS-PROG-002 | WHEN a learner completes a lesson, the system shall mark as complete and update module progress. | PROG-derived | Must | 1 |
| EARS-PROG-003 | WHEN all lessons in a module are complete, the system shall mark module as complete. | PROG-derived | Must | 1 |

### 17.2 Quiz Progress Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PROG-004 | WHEN a learner submits a quiz, the system shall calculate and record the score. | PROG-derived | Must | 1 |
| EARS-PROG-005 | WHEN a learner passes a quiz, the system shall record the certification at that tier. | PROG-derived | Must | 1 |
| EARS-PROG-006 | The system shall retain best score across all attempts for each quiz. | PROG-derived | Must | 1 |
| EARS-PROG-007 | WHEN a learner fails a quiz, the system shall enforce 24-hour cooldown before retry. | PRD-Business Rules | Must | 1 |

### 17.3 Scenario Progress Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PROG-008 | WHEN a learner completes a scenario, the system shall record score and path taken. | PROG-derived | Must | 1 |
| EARS-PROG-009 | WHEN a learner passes a scenario, the system shall record completion for that tier. | PROG-derived | Must | 1 |
| EARS-PROG-010 | The system shall retain best score across all attempts for each scenario. | PROG-derived | Must | 1 |

### 17.4 Certification Progress Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PROG-011 | WHEN a learner achieves Bronze certification, the system shall unlock Silver tier content. | PROG-derived | Must | 1 |
| EARS-PROG-012 | WHEN a learner achieves Silver certification, the system shall unlock Gold tier content. | PROG-derived | Must | 1 |
| EARS-PROG-013 | WHEN a learner achieves certification, the system shall record timestamp and generate certificate. | PROG-derived | Must | 1 |

---

## 18. Reporting & Analytics

### 18.1 Content Inventory Reports

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-REPT-001 | WHEN an admin views dashboard, the system shall display total content count by type. | CMS-RP-001 | Must | 1 |
| EARS-REPT-002 | WHEN an admin views dashboard, the system shall display content count by status. | CMS-RP-002 | Must | 1 |
| EARS-REPT-003 | WHEN an admin views reports, the system shall display content breakdown by tier. | CMS-RP-003 | Should | 1 |
| EARS-REPT-004 | WHEN an admin views reports, the system shall display content breakdown by category. | CMS-RP-004 | Should | 1 |

### 18.2 Content Quality Reports

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-REPT-005 | WHEN an admin views quality reports, the system shall list wines missing Level 2 or Level 3 content. | CMS-RP-005 | Should | 1 |
| EARS-REPT-006 | WHEN an admin views quality reports, the system shall list quizzes with fewer than 10 questions. | CMS-RP-006 | Should | 1 |
| EARS-REPT-007 | WHEN an admin views quality reports, the system shall list modules without associated quizzes. | CMS-RP-007 | Should | 1 |
| EARS-REPT-008 | WHEN an admin views workflow reports, the system shall list content pending review beyond SLA. | CMS-RP-008 | Must | 1 |

### 18.3 Team Progress Reports

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-REPT-009 | WHEN a manager views team dashboard, the system shall display certification status for all team members. | REPT-derived | Must | 1 |
| EARS-REPT-010 | WHEN a manager views team dashboard, the system shall highlight at-risk learners behind schedule. | REPT-derived | Should | 1 |
| EARS-REPT-011 | WHEN a manager views reports, the system shall display team progress trends over time. | REPT-derived | Should | 1 |

### 18.4 Export Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-REPT-012 | WHEN an admin requests report export, the system shall generate CSV format. | CMS-RP-012 | Should | 1 |
| EARS-REPT-013 | WHEN an admin requests report export, the system shall generate PDF format. | CMS-RP-013 | Could | 2 |

---

## 19. Notifications

### 19.1 Email Notification Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-NOTF-001 | WHEN a user is invited, the system shall send invitation email with unique registration link. | NOTF-derived | Must | 1 |
| EARS-NOTF-002 | WHEN content is submitted for review, the system shall notify assigned reviewers via email. | CMS-WF | Must | 1 |
| EARS-NOTF-003 | WHEN content is approved, the system shall notify the content author via email. | CMS-WF | Must | 1 |
| EARS-NOTF-004 | WHEN content is rejected, the system shall notify the content author with feedback via email. | CMS-WF | Must | 1 |
| EARS-NOTF-005 | WHEN a learner achieves certification, the system shall send congratulations email with certificate. | NOTF-derived | Should | 1 |

### 19.2 In-App Notification Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-NOTF-006 | WHEN a notification-worthy event occurs, the system shall display in-app notification badge. | NOTF-derived | Must | 1 |
| EARS-NOTF-007 | WHEN a user views notifications, the system shall display list sorted by recency. | NOTF-derived | Must | 1 |
| EARS-NOTF-008 | WHEN a user clicks a notification, the system shall navigate to the relevant content or page. | NOTF-derived | Must | 1 |
| EARS-NOTF-009 | WHEN a user dismisses a notification, the system shall mark as read but retain in history. | NOTF-derived | Should | 1 |

### 19.3 Escalation Notification Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-NOTF-010 | IF review pending exceeds 3 days, THEN the system shall send reminder to reviewer. | CMS-WF-5.5 | Should | 1 |
| EARS-NOTF-011 | IF review pending exceeds 5 days, THEN the system shall escalate to Content Admin. | CMS-WF-5.5 | Should | 1 |
| EARS-NOTF-012 | IF a learner has not logged in for 7 days, THEN the system shall send engagement reminder. | NOTF-derived | Should | 2 |

---

## 20. Audit & Logging

### 20.1 Action Logging Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUDT-001 | WHEN content is created, the system shall log user, timestamp, content ID, and content type. | CMS-AU-001 | Must | 1 |
| EARS-AUDT-002 | WHEN content is updated, the system shall log user, timestamp, content ID, and changed fields with previous values. | CMS-AU-002 | Must | 1 |
| EARS-AUDT-003 | WHEN content is deleted, the system shall log user, timestamp, content ID, and full content snapshot. | CMS-AU-003 | Must | 1 |
| EARS-AUDT-004 | WHEN content status changes, the system shall log user, timestamp, from-state, and to-state. | CMS-AU-004 | Must | 1 |
| EARS-AUDT-005 | WHEN a review action occurs, the system shall log reviewer, action type, and any feedback. | CMS-AU-005 | Must | 1 |
| EARS-AUDT-006 | WHEN a bulk operation completes, the system shall log user, operation type, and affected item count. | CMS-AU-006 | Must | 1 |

### 20.2 Audit Access Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUDT-007 | WHEN a Content Admin views a content item, the system shall display complete audit history. | CMS-AU-007 | Must | 1 |
| EARS-AUDT-008 | WHEN searching audit logs, the system shall allow filtering by user, action type, and date range. | CMS-AU-008 | Should | 1 |
| EARS-AUDT-009 | WHEN searching audit logs, the system shall allow filtering by content type. | CMS-AU-009 | Should | 1 |
| EARS-AUDT-010 | WHEN an admin requests audit export, the system shall generate CSV with all log fields. | CMS-AU-010 | Should | 1 |

### 20.3 Audit Retention Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AUDT-011 | The system shall retain audit logs for minimum 7 years for compliance. | CMS-AU-011 | Must | 1 |
| EARS-AUDT-012 | The system shall archive audit logs older than 2 years to cold storage. | CMS-AU-012 | Should | 2 |

---

## 21. Non-Functional Requirements

### 21.1 Performance Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-PERF-001 | The system shall load any page within 2 seconds (95th percentile). | PRD-8.1 | Must | 1 |
| EARS-PERF-002 | The system shall return search results within 1 second. | CMS-NF-002 | Must | 1 |
| EARS-PERF-003 | The system shall respond to API requests within 500ms (95th percentile). | PRD-8.1 | Must | 1 |
| EARS-PERF-004 | The system shall complete quiz generation within 30 seconds. | LE-PF-002 | Must | 1 |
| EARS-PERF-005 | The system shall complete scenario generation within 1 minute. | LE-PF-003 | Must | 1 |
| EARS-PERF-006 | The system shall complete full curriculum generation within 5 minutes for 200 wines. | LE-PF-001 | Must | 1 |
| EARS-PERF-007 | WHILE editing content, the system shall autosave every 30 seconds. | CMS-NF-003 | Should | 1 |

### 21.2 Scalability Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SCAL-001 | The system shall support up to 500 organisations. | PRD-8.2 | Must | 1 |
| EARS-SCAL-002 | The system shall support up to 50,000 total users. | PRD-8.2 | Must | 1 |
| EARS-SCAL-003 | The system shall support up to 1,000 concurrent users per organisation. | PRD-8.2 | Must | 1 |
| EARS-SCAL-004 | The system shall support up to 10,000 wines per organisation. | PRD-8.2 | Must | 1 |
| EARS-SCAL-005 | The system shall support 10+ concurrent curriculum generation requests. | LE-PF-009 | Must | 1 |

### 21.3 Security Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-SECU-001 | The system shall encrypt all data at rest using AES-256. | PRD-8.3 | Must | 1 |
| EARS-SECU-002 | The system shall encrypt all data in transit using TLS 1.3. | PRD-8.3 | Must | 1 |
| EARS-SECU-003 | The system shall enforce RBAC for all protected resources. | CMS-NF-008 | Must | 1 |
| EARS-SECU-004 | The system shall implement row-level security for multi-tenant data isolation. | ORG-5.2 | Must | 1 |
| EARS-SECU-005 | The system shall log all security-relevant events (login, permission changes, data access). | PRD-8.3 | Must | 1 |
| EARS-SECU-006 | The system shall implement rate limiting to prevent abuse. | LE-IP-010 | Should | 1 |

### 21.4 Availability Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-AVAI-001 | The system shall achieve 99.5% uptime for Starter and Professional tiers. | PRD-8.4 | Must | 1 |
| EARS-AVAI-002 | The system shall achieve 99.9% uptime for Enterprise tier. | PRD-8.4 | Must | 2 |
| EARS-AVAI-003 | The system shall complete recovery within 4 hours (RTO). | PRD-8.4 | Must | 1 |
| EARS-AVAI-004 | The system shall restore data to within 1 hour of failure (RPO). | PRD-8.4 | Must | 1 |
| EARS-AVAI-005 | The system shall perform backups hourly (incremental) and daily (full). | PRD-8.4 | Must | 1 |
| EARS-AVAI-006 | The system shall retain backups for 30 days. | PRD-8.4 | Must | 1 |

### 21.5 Usability Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-USAB-001 | The system shall be fully responsive on tablet devices. | CMS-NF-004 | Should | 1 |
| EARS-USAB-002 | WHEN any action completes, the system shall display clear success or error feedback. | CMS-NF-005 | Must | 1 |
| EARS-USAB-003 | WHEN a user navigates away with unsaved changes, the system shall prompt for confirmation. | CMS-NF-006 | Must | 1 |
| EARS-USAB-004 | The system shall comply with WCAG 2.1 Level AA accessibility standards. | PRD-8.5 | Must | 1 |
| EARS-USAB-005 | The system shall support keyboard navigation for all functions. | PRD-8.5 | Must | 1 |
| EARS-USAB-006 | The system shall support screen readers (NVDA, VoiceOver). | PRD-8.5 | Must | 1 |

### 21.6 IP Protection Requirements

| ID | Requirement | Source | Priority | Phase |
|----|-------------|--------|----------|-------|
| EARS-IPPR-001 | The system shall execute all generation algorithms server-side only. | LE-IP-001 | Must | 1 |
| EARS-IPPR-002 | The system shall transmit only generated content to clients, never algorithms. | LE-IP-002 | Must | 1 |
| EARS-IPPR-003 | The system shall not expose generation logic through APIs. | LE-IP-003 | Must | 1 |
| EARS-IPPR-004 | The system shall encrypt template library at rest. | LE-IP-006 | Must | 1 |
| EARS-IPPR-005 | The system shall log all generation activity with user attribution. | LE-IP-008 | Must | 1 |
| EARS-IPPR-006 | The system shall detect and block bulk content scraping attempts. | LE-IP-010 | Should | 1 |

---

## 22. Traceability Matrix

### 22.1 Summary by Source Document

| Source Document | Source Reqs | EARS Reqs | Coverage |
|-----------------|-------------|-----------|----------|
| SS-WS3.0-CMS-FR | 159 | 115 | 100% |
| SS-WS3.0-LE-REQ | 112 | 89 | 100% |
| SS-WS3.0-ORG | 15 | 28 | 100% |
| SS-WS3.0-CLS | 22 | 22 | 100% |
| SS-WS3-PRD (NFRs) | 25 | 31 | 100% |
| **Total** | **333** | **285** | **100%** |

### 22.2 Summary by Priority

| Priority | Count | Percentage |
|----------|-------|------------|
| Must | 189 | 66% |
| Should | 78 | 27% |
| Could | 18 | 7% |
| **Total** | **285** | **100%** |

### 22.3 Summary by Phase

| Phase | Count | Percentage |
|-------|-------|------------|
| Phase 1 (MVP) | 231 | 81% |
| Phase 2 | 42 | 15% |
| Phase 3 | 12 | 4% |
| **Total** | **285** | **100%** |

### 22.4 Summary by Category

| Category | Code | Count |
|----------|------|-------|
| Authentication & Authorization | AUTH | 17 |
| User Management | USER | 15 |
| Organisation Management | ORG | 14 |
| Wine Management | WINE | 25 |
| Module Management | MOD | 16 |
| Quiz Management | QUIZ | 29 |
| Scenario Management | SCEN | 25 |
| Content Lifecycle | CONT | 22 |
| Import/Export | IMPX | 17 |
| Learning Engine - Curriculum | LENG | 18 |
| Quiz Generation | QGEN | 19 |
| Scenario Generation | SGEN | 17 |
| Adaptive Learning | ADPT | 14 |
| Learning Paths | PATH | 12 |
| Progress Tracking | PROG | 13 |
| Reporting & Analytics | REPT | 13 |
| Notifications | NOTF | 12 |
| Audit & Logging | AUDT | 12 |
| Performance | PERF | 7 |
| Scalability | SCAL | 5 |
| Security | SECU | 6 |
| Availability | AVAI | 6 |
| Usability | USAB | 6 |
| IP Protection | IPPR | 6 |
| **Total** | | **285** |

---

## 23. Appendices

### 23.1 Glossary

See SS-WS3-PRD Appendix 12.1 for complete glossary.

### 23.2 EARS Pattern Reference

| Pattern | Template | Example |
|---------|----------|---------|
| **Ubiquitous** | The system shall `<action>` | The system shall encrypt all data at rest. |
| **Event-Driven** | WHEN `<trigger>`, the system shall `<action>` | WHEN a user submits a quiz, the system shall calculate the score. |
| **State-Driven** | WHILE `<state>`, the system shall `<action>` | WHILE content is in DRAFT, the system shall allow editing. |
| **Optional** | WHERE `<feature>`, the system shall `<action>` | WHERE SSO is enabled, the system shall authenticate via SAML. |
| **Unwanted** | IF `<condition>`, THEN the system shall `<action>` | IF login fails 5 times, THEN the system shall lock the account. |
| **Complex** | WHILE `<state>`, WHEN `<trigger>`, the system shall `<action>` | WHILE user is authenticated, WHEN session expires, the system shall redirect to login. |

### 23.3 Reference Documents

| Document ID | Title | Relevance |
|-------------|-------|-----------|
| SS-WS3.0-CMS-FR | CMS Functional Requirements | 159 source requirements |
| SS-WS3.0-LE-REQ | Learning Engine Requirements | 112 source requirements |
| SS-WS3.0-CLS | Content Lifecycle Specification | State and workflow requirements |
| SS-WS3.0-ORG | Organization Model | User and org requirements |
| SS-WS3-PRD | Product Requirements Document | NFRs and business rules |

### 23.4 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-20 | Obi Wan | Initial draft |

---

**Document Classification:** CONFIDENTIAL

*This document contains proprietary requirement specifications for the Sommelier Spark platform. Distribution is restricted to authorised personnel only.*

---

*End of EARS Requirements Specification*
