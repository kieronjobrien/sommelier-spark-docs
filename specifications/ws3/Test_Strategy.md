# Test Strategy — Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3-TEST |
| **Version** | 1.0 |
| **Date** | 2026-01-21 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Classification** | CONFIDENTIAL |
| **Related Documents** | SS-WS3-PRD, SS-WS3-EARS, SS-WS3-HLD, SS-WS3-LLD, SS-WS3-API, SS-WS3-DATA |

---

## CONFIDENTIALITY NOTICE

This document contains proprietary testing specifications for the Sommelier Spark platform. Distribution is restricted to authorised personnel only.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Test Strategy](#2-test-strategy)
3. [Test Coverage Requirements](#3-test-coverage-requirements)
4. [Test Cases by Feature Area](#4-test-cases-by-feature-area)
5. [Integration Test Scenarios](#5-integration-test-scenarios)
6. [API Test Plan](#6-api-test-plan)
7. [Performance Test Plan](#7-performance-test-plan)
8. [Security Test Plan](#8-security-test-plan)
9. [Accessibility Test Plan](#9-accessibility-test-plan)
10. [UAT Test Plan](#10-uat-test-plan)
11. [Defect Management](#11-defect-management)
12. [Test Automation](#12-test-automation)
13. [Quality Gates](#13-quality-gates)
14. [Test Data Management](#14-test-data-management)
15. [Traceability Matrix](#15-traceability-matrix)
16. [Appendices](#16-appendices)

---

## 1. Executive Summary

### 1.1 Testing Objectives

The Sommelier Spark test strategy ensures the platform meets quality standards for the MVP release on **1 March 2026**. Key objectives:

1. **Functional Correctness** — Verify all 285 EARS requirements are implemented correctly
2. **Integration Integrity** — Validate component interactions and data flows
3. **Performance Compliance** — Meet response time and scalability targets
4. **Security Assurance** — Protect multi-tenant data and user credentials
5. **Accessibility Compliance** — Achieve WCAG 2.1 Level AA conformance
6. **User Acceptance** — Validate end-to-end user journeys

### 1.2 Scope

**In Scope (MVP - Phase 1):**
- 231 Phase 1 requirements
- 62 API endpoints (81 operations)
- 5 user roles: Learner, Manager, Admin, Owner, System Admin
- 3 content types: Wines, Quizzes, Scenarios
- Learning engine: Curriculum, Quiz, and Scenario generation
- Multi-tenant data isolation

**Out of Scope (Phase 2+):**
- 54 Phase 2/3 requirements
- Advanced analytics dashboards
- Mobile native applications
- Third-party LMS integrations

### 1.3 Quality Goals

| Metric | Target | Minimum Acceptable |
|--------|--------|--------------------|
| Requirements Coverage | 100% | 95% |
| Unit Test Coverage | 80% | 70% |
| Critical Path E2E Coverage | 100% | 90% |
| P1/P2 Defects at Release | 0 | 0 |
| Performance Targets Met | 100% | 95% |
| WCAG 2.1 AA Compliance | 100% | 100% |

### 1.4 Key Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Learning engine generation quality | Medium | High | Extensive UAT with real wine data |
| Multi-tenant data leakage | Low | Critical | Automated tenant isolation tests |
| Performance under load | Medium | High | Early performance testing, capacity planning |
| Third-party API failures (OpenAI) | Medium | Medium | Circuit breakers, fallback content |
| Insufficient test data diversity | Medium | Medium | Comprehensive seed data across wine types |

---

## 2. Test Strategy

### 2.1 Test Levels

| Level | Scope | Owner | Tools | Execution |
|-------|-------|-------|-------|-----------|
| **Unit** | Functions, classes, modules | Developers | Jest (TS), pytest (Python) | Every commit |
| **Integration** | Service interactions, database | Developers | Jest, supertest, testcontainers | Every PR |
| **API** | Endpoint contracts, validation | QA | Postman, Newman | Every PR |
| **E2E** | User journeys, UI flows | QA | Playwright | Nightly + pre-release |
| **Performance** | Load, stress, soak | QA/DevOps | k6, Grafana | Weekly + pre-release |
| **Security** | Vulnerabilities, penetration | Security | OWASP ZAP, Burp Suite | Sprint + pre-release |
| **Accessibility** | WCAG compliance | QA | axe-core, WAVE, manual | Sprint + pre-release |
| **UAT** | Business acceptance | Product/Users | Manual + guided scripts | Pre-release |

### 2.2 Test Types

#### 2.2.1 Functional Testing
- **Positive Testing** — Verify correct behaviour with valid inputs
- **Negative Testing** — Verify error handling with invalid inputs
- **Boundary Testing** — Test at limits (character counts, file sizes)
- **State Testing** — Verify content lifecycle transitions

#### 2.2.2 Regression Testing
- Automated suite run on every PR
- Full regression before each release
- Selective regression for hotfixes

#### 2.2.3 Smoke Testing
- Critical path verification after deployment
- 15-minute maximum execution time
- Blocks release if failures

#### 2.2.4 Integration Testing
- API-to-database interactions
- Service-to-service communication
- External API integrations (OpenAI, email)

#### 2.2.5 Contract Testing
- OpenAPI schema validation
- Request/response structure verification
- Backward compatibility checks

#### 2.2.6 Performance Testing
- Load testing — expected user volumes
- Stress testing — beyond expected capacity
- Soak testing — extended duration stability
- Spike testing — sudden load increases

#### 2.2.7 Security Testing
- OWASP Top 10 vulnerability scans
- Penetration testing (pre-release)
- Authentication/authorisation testing
- Multi-tenant isolation verification

#### 2.2.8 Accessibility Testing
- Automated axe-core scans
- Manual keyboard navigation testing
- Screen reader compatibility (NVDA, VoiceOver)
- Colour contrast verification

#### 2.2.9 Usability Testing
- Task completion observation
- Time-on-task measurement
- Error rate tracking
- User satisfaction surveys

### 2.3 Test Environments

| Environment | Purpose | Data | Refresh Frequency | Access |
|-------------|---------|------|-------------------|--------|
| **Local** | Developer testing | Seed data (fixtures) | On demand | Developers |
| **CI** | Automated test execution | Synthetic data | Per build | CI system |
| **QA** | Manual QA testing | Anonymised production snapshot | Weekly | QA team |
| **Staging** | Integration, UAT, performance | Anonymised production + synthetic | Weekly | QA, Product, Stakeholders |
| **Production** | Smoke tests only | Real data | N/A | Operations |

#### 2.3.1 Environment Configuration

```yaml
# Environment parity requirements
All environments:
  - PostgreSQL 15.x
  - Node.js 20 LTS
  - Redis 7.x
  - Same API version

Differences:
  Local:
    - Docker Compose
    - Mock external services
  CI:
    - GitHub Actions
    - Testcontainers
  QA/Staging:
    - AWS ECS
    - Real external services (sandbox)
  Production:
    - AWS ECS (production)
    - Real external services
```

---

## 3. Test Coverage Requirements

### 3.1 Code Coverage Targets

| Component | Target | Minimum | Enforcement |
|-----------|--------|---------|-------------|
| Core API Services | 80% | 70% | PR gate |
| Learning Engine | 85% | 75% | PR gate |
| Controllers | 70% | 60% | PR gate |
| Repositories | 60% | 50% | PR gate |
| Utilities | 90% | 80% | PR gate |
| UI Components | 70% | 60% | Advisory |
| E2E Critical Paths | 100% | 90% | Release gate |

### 3.2 Requirements Coverage

Based on EARS specification (SS-WS3-EARS):

| Priority | Total Requirements | Test Coverage Target | Minimum |
|----------|-------------------|---------------------|---------|
| **Must** | 189 | 100% | 100% |
| **Should** | 78 | 95% | 90% |
| **Could** | 18 | 70% | 50% |
| **Total** | 285 | 97% | 93% |

#### 3.2.1 Coverage by Phase

| Phase | Requirements | MVP Coverage |
|-------|-------------|--------------|
| Phase 1 (MVP) | 231 | 100% tested |
| Phase 2 | 42 | Deferred |
| Phase 3 | 12 | Deferred |

### 3.3 API Coverage Requirements

All 62 API paths (81 operations) must have:

| Test Type | Required | Notes |
|-----------|----------|-------|
| Happy path | Yes | Valid request, expected response |
| Validation error (400) | Yes | Invalid request body/parameters |
| Authentication error (401) | Yes | Missing/invalid token |
| Authorisation error (403) | Yes | Insufficient permissions |
| Not found error (404) | Yes (where applicable) | Non-existent resource |
| Conflict error (409) | Where applicable | Duplicate creation |
| Rate limit error (429) | Yes | Exceed rate limits |

---

## 4. Test Cases by Feature Area

### 4.1 Authentication (AUTH)

**EARS Requirements:** EARS-AUTH-001 through EARS-AUTH-017

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-AUTH-001 | Successful login with valid email and password | EARS-AUTH-001, EARS-AUTH-002 | Must | Functional |
| TC-AUTH-002 | Login failure with invalid password returns generic error | EARS-AUTH-003 | Must | Negative |
| TC-AUTH-003 | Login failure with non-existent email returns generic error | EARS-AUTH-003 | Must | Negative |
| TC-AUTH-004 | Account lockout after 5 consecutive failed attempts | EARS-AUTH-004 | Must | Security |
| TC-AUTH-005 | Locked account auto-unlocks after 30 minutes | EARS-AUTH-004 | Must | Functional |
| TC-AUTH-006 | Password reset email sent with valid token | EARS-AUTH-005 | Must | Functional |
| TC-AUTH-007 | Password reset token expires after 24 hours | EARS-AUTH-005 | Must | Security |
| TC-AUTH-008 | Password hashed with bcrypt cost factor 12+ | EARS-AUTH-006 | Must | Security |
| TC-AUTH-009 | Password rejected if < 8 chars | EARS-AUTH-007 | Must | Validation |
| TC-AUTH-010 | Password rejected if no uppercase | EARS-AUTH-007 | Must | Validation |
| TC-AUTH-011 | Password rejected if no lowercase | EARS-AUTH-007 | Must | Validation |
| TC-AUTH-012 | Password rejected if no number | EARS-AUTH-007 | Must | Validation |
| TC-AUTH-013 | JWT access token expires after 8 hours | EARS-AUTH-008, EARS-AUTH-010 | Must | Security |
| TC-AUTH-014 | Refresh token valid for 30 days | EARS-AUTH-009 | Must | Security |
| TC-AUTH-015 | Refresh token exchange returns new access token | EARS-AUTH-009 | Must | Functional |
| TC-AUTH-016 | Logout invalidates current session | EARS-AUTH-011 | Must | Functional |
| TC-AUTH-017 | Logout invalidates all associated tokens | EARS-AUTH-011 | Must | Security |
| TC-AUTH-018 | Role-based access enforced for Learner | EARS-AUTH-012, EARS-AUTH-013 | Must | Authorisation |
| TC-AUTH-019 | Role-based access enforced for Admin | EARS-AUTH-012, EARS-AUTH-014 | Must | Authorisation |
| TC-AUTH-020 | Role-based access enforced for Owner | EARS-AUTH-012, EARS-AUTH-015 | Must | Authorisation |
| TC-AUTH-021 | Permission validated before protected action | EARS-AUTH-016 | Must | Security |
| TC-AUTH-022 | User cannot access other organisation's data | EARS-AUTH-017 | Must | Security |

### 4.2 User Management (USER)

**EARS Requirements:** EARS-USER-001 through EARS-USER-015

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-USER-001 | Admin invites user, pending record created | EARS-USER-001 | Must | Functional |
| TC-USER-002 | Invitation email sent with unique link | EARS-USER-001 | Must | Integration |
| TC-USER-003 | Invited user clicks link, registration form displayed | EARS-USER-002 | Must | Functional |
| TC-USER-004 | User completes registration, account activated | EARS-USER-003 | Must | Functional |
| TC-USER-005 | Activation timestamp recorded | EARS-USER-003 | Must | Functional |
| TC-USER-006 | User profile stores required fields | EARS-USER-004 | Must | Functional |
| TC-USER-007 | Email uniqueness enforced | EARS-USER-005 | Must | Validation |
| TC-USER-008 | Duplicate email returns conflict error | EARS-USER-005 | Must | Negative |
| TC-USER-009 | User updates profile successfully | EARS-USER-006 | Must | Functional |
| TC-USER-010 | Profile image upload validates JPG/PNG | EARS-USER-007 | Should | Validation |
| TC-USER-011 | Profile image upload rejects > 2MB | EARS-USER-007 | Should | Validation |
| TC-USER-012 | Password change requires current password | EARS-USER-008 | Must | Security |
| TC-USER-013 | User status tracks PENDING, ACTIVE, SUSPENDED, DEACTIVATED | EARS-USER-009 | Must | Functional |
| TC-USER-014 | Suspended user cannot login | EARS-USER-010 | Must | Security |
| TC-USER-015 | Suspended user data preserved | EARS-USER-010 | Must | Functional |
| TC-USER-016 | Deactivated user data anonymised after 30 days | EARS-USER-011 | Must | GDPR |
| TC-USER-017 | Reactivated user access restored immediately | EARS-USER-012 | Should | Functional |
| TC-USER-018 | Role assignment updates permissions immediately | EARS-USER-013 | Must | Security |
| TC-USER-019 | Last Owner cannot remove own Owner role | EARS-USER-014 | Must | Business Rule |
| TC-USER-020 | Role changes logged in audit trail | EARS-USER-015 | Must | Audit |

### 4.3 Organisation Management (ORG)

**EARS Requirements:** EARS-ORG-001 through EARS-ORG-014

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-ORG-001 | New organisation created with required fields | EARS-ORG-001 | Must | Functional |
| TC-ORG-002 | Organisation details update logged | EARS-ORG-002 | Must | Audit |
| TC-ORG-003 | Organisation attributes stored correctly | EARS-ORG-003 | Must | Functional |
| TC-ORG-004 | Unique slug generated for URL | EARS-ORG-004 | Should | Functional |
| TC-ORG-005 | Starter tier limited to 25 users | EARS-ORG-005 | Must | Business Rule |
| TC-ORG-006 | Professional tier limited to 100 users | EARS-ORG-005 | Must | Business Rule |
| TC-ORG-007 | Enterprise tier has unlimited users | EARS-ORG-005 | Must | Business Rule |
| TC-ORG-008 | User invitation blocked when at limit | EARS-ORG-006 | Must | Business Rule |
| TC-ORG-009 | Upgrade applies new limits immediately | EARS-ORG-007 | Must | Functional |
| TC-ORG-010 | Downgrade enforces limits at next billing cycle | EARS-ORG-008 | Should | Business Rule |
| TC-ORG-011 | Logo upload available for Professional+ | EARS-ORG-009 | Should | Functional |
| TC-ORG-012 | Colour customisation available for Professional+ | EARS-ORG-010 | Should | Functional |
| TC-ORG-013 | Branding applied to learner interfaces | EARS-ORG-011 | Should | Functional |
| TC-ORG-014 | Row-level security isolates organisation data | EARS-ORG-012 | Must | Security |
| TC-ORG-015 | All queries include organisationId filter | EARS-ORG-013 | Must | Security |
| TC-ORG-016 | API prevents cross-organisation data access | EARS-ORG-014 | Must | Security |

### 4.4 Wine Management (WINE)

**EARS Requirements:** EARS-WINE-001 through EARS-WINE-025

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-WINE-001 | Wine creation form displays all required fields | EARS-WINE-001 | Must | Functional |
| TC-WINE-002 | Wine created in DRAFT status | EARS-WINE-002 | Must | Functional |
| TC-WINE-003 | Wine creation fails without required fields | EARS-WINE-002 | Must | Validation |
| TC-WINE-004 | Wine edit changes logged with previous values | EARS-WINE-003 | Must | Audit |
| TC-WINE-005 | Wine preview matches learner view | EARS-WINE-004 | Must | Functional |
| TC-WINE-006 | DRAFT wine can be deleted (soft delete) | EARS-WINE-005 | Must | Functional |
| TC-WINE-007 | Deleted wine retained for 30 days | EARS-WINE-005 | Must | Functional |
| TC-WINE-008 | PUBLISHED wine archival triggers 7-day warning | EARS-WINE-006 | Must | Business Rule |
| TC-WINE-009 | ARCHIVED wine can be restored to DRAFT | EARS-WINE-007 | Should | Functional |
| TC-WINE-010 | Quick Facts enforces 50-1000 character limit | EARS-WINE-008 | Must | Validation |
| TC-WINE-011 | Detailed Profile enforces 5000 character limit | EARS-WINE-009 | Must | Validation |
| TC-WINE-012 | Expert Insights enforces 10000 character limit | EARS-WINE-010 | Must | Validation |
| TC-WINE-013 | Wine image upload validates JPG/PNG/WebP | EARS-WINE-011 | Should | Validation |
| TC-WINE-014 | Wine image upload rejects > 5MB | EARS-WINE-011 | Should | Validation |
| TC-WINE-015 | Wine image auto-resized | EARS-WINE-011 | Should | Functional |
| TC-WINE-016 | Food pairings selected from tag list | EARS-WINE-012 | Must | Functional |
| TC-WINE-017 | Serving temperature selected from ranges | EARS-WINE-013 | Should | Functional |
| TC-WINE-018 | Wine linked to quizzes with bi-directional refs | EARS-WINE-014 | Must | Functional |
| TC-WINE-019 | Wine linked to scenarios with talking points | EARS-WINE-015 | Must | Functional |
| TC-WINE-020 | Wine view shows referencing quizzes/scenarios | EARS-WINE-016 | Must | Functional |
| TC-WINE-021 | Wine archival blocked if referenced by published content | EARS-WINE-017 | Must | Business Rule |
| TC-WINE-022 | CSV upload parsed and validated | EARS-WINE-018 | Must | Functional |
| TC-WINE-023 | CSV validation reports errors by row | EARS-WINE-019 | Must | Functional |
| TC-WINE-024 | Wine export generates CSV with all attributes | EARS-WINE-020 | Should | Functional |
| TC-WINE-025 | Selected wines export to CSV | EARS-WINE-021 | Should | Functional |
| TC-WINE-026 | Wine duplication creates DRAFT copy | EARS-WINE-022 | Could | Functional |
| TC-WINE-027 | Real-time field validation before save | EARS-WINE-023 | Must | Validation |
| TC-WINE-028 | Quick Facts < 50 chars shows warning | EARS-WINE-024 | Should | Validation |
| TC-WINE-029 | Duplicate detection by name+vintage+producer | EARS-WINE-025 | Should | Functional |

### 4.5 Module Management (MOD)

**EARS Requirements:** EARS-MOD-001 through EARS-MOD-016

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-MOD-001 | Module created with required title, description, category | EARS-MOD-001 | Must | Functional |
| TC-MOD-002 | Module edit changes logged | EARS-MOD-002 | Must | Audit |
| TC-MOD-003 | DRAFT module deletion cascades to lessons | EARS-MOD-003 | Must | Functional |
| TC-MOD-004 | PUBLISHED module archival cascades to lessons | EARS-MOD-004 | Must | Functional |
| TC-MOD-005 | Module preview matches learner view | EARS-MOD-005 | Must | Functional |
| TC-MOD-006 | Lesson added with sortOrder at end | EARS-MOD-006 | Must | Functional |
| TC-MOD-007 | Lesson editor supports rich text | EARS-MOD-007 | Must | Functional |
| TC-MOD-008 | Lesson reorder updates all sortOrders | EARS-MOD-008 | Must | Functional |
| TC-MOD-009 | Lesson removal from published module shows confirmation | EARS-MOD-009 | Must | Functional |
| TC-MOD-010 | Lesson move to different module updates references | EARS-MOD-010 | Should | Functional |
| TC-MOD-011 | Lesson duplication creates DRAFT copy | EARS-MOD-011 | Could | Functional |
| TC-MOD-012 | Module tier set to Bronze, Silver, or Gold | EARS-MOD-012 | Must | Functional |
| TC-MOD-013 | Module completion time calculated from lessons | EARS-MOD-013 | Must | Functional |
| TC-MOD-014 | Module thumbnail image upload | EARS-MOD-014 | Should | Functional |
| TC-MOD-015 | Module prerequisites enforced | EARS-MOD-015 | Should | Business Rule |
| TC-MOD-016 | Module associated with exactly one quiz | EARS-MOD-016 | Must | Functional |

### 4.6 Quiz Management (QUIZ)

**EARS Requirements:** EARS-QUIZ-001 through EARS-QUIZ-027

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-QUIZ-001 | Quiz created with required fields | EARS-QUIZ-001 | Must | Functional |
| TC-QUIZ-002 | Quiz edit changes logged | EARS-QUIZ-002 | Must | Audit |
| TC-QUIZ-003 | DRAFT quiz deletion cascades to questions | EARS-QUIZ-003 | Must | Functional |
| TC-QUIZ-004 | PUBLISHED quiz archival preserves attempt history | EARS-QUIZ-004 | Must | Functional |
| TC-QUIZ-005 | Quiz duplication copies all questions to DRAFT | EARS-QUIZ-005 | Should | Functional |
| TC-QUIZ-006 | Quiz preview simulates complete learner experience | EARS-QUIZ-006 | Must | Functional |
| TC-QUIZ-007 | Multiple choice question: 2-6 options, one correct | EARS-QUIZ-007 | Must | Validation |
| TC-QUIZ-008 | Multiple select question: 2-6 options, 1+ correct | EARS-QUIZ-008 | Must | Validation |
| TC-QUIZ-009 | True/false question: exactly 2 options | EARS-QUIZ-009 | Must | Validation |
| TC-QUIZ-010 | Matching question: pairs definition | EARS-QUIZ-010 | Could | Functional |
| TC-QUIZ-011 | Question edit changes logged | EARS-QUIZ-011 | Must | Audit |
| TC-QUIZ-012 | Question save requires correct answer marked | EARS-QUIZ-012 | Must | Validation |
| TC-QUIZ-013 | Question explanation text saved | EARS-QUIZ-013 | Must | Functional |
| TC-QUIZ-014 | Question linked to specific wines | EARS-QUIZ-014 | Must | Functional |
| TC-QUIZ-015 | Question difficulty set (Easy, Medium, Hard) | EARS-QUIZ-015 | Must | Functional |
| TC-QUIZ-016 | Quiz passing score configurable (default 70%) | EARS-QUIZ-016 | Must | Functional |
| TC-QUIZ-017 | Quiz time limit enforced | EARS-QUIZ-017 | Should | Functional |
| TC-QUIZ-018 | Question order randomised on attempt | EARS-QUIZ-018 | Should | Functional |
| TC-QUIZ-019 | Learner starts quiz, attempt record created | EARS-QUIZ-019 | Must | Functional |
| TC-QUIZ-020 | Answer selection recorded | EARS-QUIZ-020 | Must | Functional |
| TC-QUIZ-021 | Quiz timer displays remaining time | EARS-QUIZ-021 | Should | Functional |
| TC-QUIZ-022 | Quiz submission calculates score | EARS-QUIZ-022 | Must | Functional |
| TC-QUIZ-023 | Pass/fail determined by passing score threshold | EARS-QUIZ-023 | Must | Functional |
| TC-QUIZ-024 | Post-quiz shows correct answers with explanations | EARS-QUIZ-024 | Must | Functional |
| TC-QUIZ-025 | Quiz attempt logged for analytics | EARS-QUIZ-025 | Must | Functional |
| TC-QUIZ-026 | 24-hour cooldown between certification attempts | EARS-QUIZ-026 | Should | Business Rule |
| TC-QUIZ-027 | Best score retained across attempts | EARS-QUIZ-027 | Should | Functional |

### 4.7 Scenario Management (SCEN)

**EARS Requirements:** EARS-SCEN-001 through EARS-SCEN-025

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-SCEN-001 | Scenario created with required fields | EARS-SCEN-001 | Must | Functional |
| TC-SCEN-002 | Scenario edit changes logged | EARS-SCEN-002 | Must | Audit |
| TC-SCEN-003 | DRAFT scenario deletion cascades to steps | EARS-SCEN-003 | Must | Functional |
| TC-SCEN-004 | PUBLISHED scenario archival preserves attempt history | EARS-SCEN-004 | Must | Functional |
| TC-SCEN-005 | Scenario duplication copies tree to DRAFT | EARS-SCEN-005 | Should | Functional |
| TC-SCEN-006 | Customer name and persona required | EARS-SCEN-006 | Must | Validation |
| TC-SCEN-007 | Initial situation text required | EARS-SCEN-007 | Must | Validation |
| TC-SCEN-008 | Estimated completion time required | EARS-SCEN-008 | Must | Validation |
| TC-SCEN-009 | Step added with sequential/branched positioning | EARS-SCEN-009 | Must | Functional |
| TC-SCEN-010 | Step type set: Dialogue, Action, Observation, Conclusion | EARS-SCEN-010 | Must | Functional |
| TC-SCEN-011 | Step customer dialogue required | EARS-SCEN-011 | Must | Validation |
| TC-SCEN-012 | Step customer mood optional | EARS-SCEN-012 | Should | Functional |
| TC-SCEN-013 | Step has 2-4 response choices | EARS-SCEN-013 | Must | Validation |
| TC-SCEN-014 | Choice points value 0-10 | EARS-SCEN-014 | Must | Validation |
| TC-SCEN-015 | Optimal choice marking | EARS-SCEN-015 | Must | Functional |
| TC-SCEN-016 | Choice feedback text required | EARS-SCEN-016 | Must | Validation |
| TC-SCEN-017 | Choice links to next step or end | EARS-SCEN-017 | Must | Functional |
| TC-SCEN-018 | Decision tree displayed as visual flowchart | EARS-SCEN-018 | Should | Functional |
| TC-SCEN-019 | Step reorder updates references | EARS-SCEN-019 | Should | Functional |
| TC-SCEN-020 | Scenario preview simulates learner experience | EARS-SCEN-020 | Must | Functional |
| TC-SCEN-021 | Path analysis identifies all possible paths | EARS-SCEN-021 | Should | Functional |
| TC-SCEN-022 | Validation verifies all steps reachable | EARS-SCEN-022 | Must | Validation |
| TC-SCEN-023 | Validation verifies conclusion path exists | EARS-SCEN-023 | Must | Validation |
| TC-SCEN-024 | Wines linked from organisation library | EARS-SCEN-024 | Must | Functional |
| TC-SCEN-025 | Talking points added for linked wines | EARS-SCEN-025 | Should | Functional |

### 4.8 Content Lifecycle (CONT)

**EARS Requirements:** EARS-CONT-001 through EARS-CONT-022

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-CONT-001 | Content status values: DRAFT, REVIEW, PUBLISHED, ARCHIVED | EARS-CONT-001 | Must | Functional |
| TC-CONT-002 | DRAFT content editable by author and admins | EARS-CONT-002 | Must | Authorisation |
| TC-CONT-003 | Submit for review changes status to REVIEW, locks editing | EARS-CONT-003 | Must | Functional |
| TC-CONT-004 | REVIEW content not editable except by returning to DRAFT | EARS-CONT-004 | Must | Authorisation |
| TC-CONT-005 | Approval changes status to PUBLISHED, sets timestamp | EARS-CONT-005 | Must | Functional |
| TC-CONT-006 | Rejection returns to DRAFT with feedback | EARS-CONT-006 | Must | Functional |
| TC-CONT-007 | Archive after warning period | EARS-CONT-007 | Must | Functional |
| TC-CONT-008 | Restore archived to DRAFT for re-review | EARS-CONT-008 | Should | Functional |
| TC-CONT-009 | Version tracked in MAJOR.MINOR format | EARS-CONT-009 | Must | Functional |
| TC-CONT-010 | Version incremented on republish | EARS-CONT-010 | Must | Functional |
| TC-CONT-011 | Previous version archived on new publish | EARS-CONT-011 | Must | Functional |
| TC-CONT-012 | Archived versions retained 2 years | EARS-CONT-012 | Must | Compliance |
| TC-CONT-013 | Version comparison shows changed fields | EARS-CONT-013 | Should | Functional |
| TC-CONT-014 | Simple workflow: Admin approval only | EARS-CONT-014 | Must | Functional |
| TC-CONT-015 | Standard workflow: Expert + Admin approval | EARS-CONT-015 | Should | Functional |
| TC-CONT-016 | Extended workflow: Expert + QA + Admin approval | EARS-CONT-016 | Should | Functional |
| TC-CONT-017 | Review reminder after 3 days | EARS-CONT-017 | Should | Notification |
| TC-CONT-018 | Escalation to Admin after 5 days | EARS-CONT-018 | Should | Notification |
| TC-CONT-019 | Auto-assign backup reviewer after 7 days | EARS-CONT-019 | Should | Functional |
| TC-CONT-020 | Emergency unpublish reverts to DRAFT | EARS-CONT-020 | Must | Functional |
| TC-CONT-021 | Create DRAFT from previous version | EARS-CONT-021 | Should | Functional |
| TC-CONT-022 | Rollback preserves learner progress | EARS-CONT-022 | Should | Functional |

### 4.9 Import/Export (IMPX)

**EARS Requirements:** EARS-IMPX-001 through EARS-IMPX-017

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-IMPX-001 | File upload validates CSV, XLSX, JSON types | EARS-IMPX-001 | Must | Validation |
| TC-IMPX-002 | CSV validates UTF-8 encoding | EARS-IMPX-002 | Must | Validation |
| TC-IMPX-003 | CSV rejects > 10MB | EARS-IMPX-002 | Must | Validation |
| TC-IMPX-004 | XLSX validates format | EARS-IMPX-003 | Must | Validation |
| TC-IMPX-005 | XLSX rejects > 25MB | EARS-IMPX-003 | Must | Validation |
| TC-IMPX-006 | JSON validates format | EARS-IMPX-004 | Must | Validation |
| TC-IMPX-007 | JSON rejects > 50MB | EARS-IMPX-004 | Must | Validation |
| TC-IMPX-008 | Each row validated against schema | EARS-IMPX-005 | Must | Validation |
| TC-IMPX-009 | Validation errors report row, field, message | EARS-IMPX-006 | Must | Functional |
| TC-IMPX-010 | Duplicate handling: Skip, Update, Create New, Error | EARS-IMPX-007 | Must | Functional |
| TC-IMPX-011 | Validation pass shows import preview | EARS-IMPX-008 | Must | Functional |
| TC-IMPX-012 | Confirm creates records, reports counts | EARS-IMPX-009 | Must | Functional |
| TC-IMPX-013 | > 1000 items processed in background | EARS-IMPX-010 | Should | Performance |
| TC-IMPX-014 | Import completion logged in audit | EARS-IMPX-011 | Must | Audit |
| TC-IMPX-015 | Export format selection: CSV, XLSX, JSON | EARS-IMPX-012 | Should | Functional |
| TC-IMPX-016 | Export filtering by status, category, date | EARS-IMPX-013 | Should | Functional |
| TC-IMPX-017 | Export generates downloadable file | EARS-IMPX-014 | Should | Functional |
| TC-IMPX-018 | Large export processed in background | EARS-IMPX-015 | Should | Performance |
| TC-IMPX-019 | Import template download available | EARS-IMPX-016 | Must | Functional |
| TC-IMPX-020 | Templates for wines, modules, quizzes, scenarios | EARS-IMPX-017 | Must | Functional |

### 4.10 Learning Engine - Content Transformation (LENG)

**EARS Requirements:** EARS-LENG-001 through EARS-LENG-013

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-LENG-001 | Wine list upload triggers curriculum generation | EARS-LENG-001 | Must | Integration |
| TC-LENG-002 | Curriculum sequenced simple to complex | EARS-LENG-002 | Must | Functional |
| TC-LENG-003 | 100% wine coverage in curriculum | EARS-LENG-003 | Must | Functional |
| TC-LENG-004 | Role-specific paths generated | EARS-LENG-004 | Should | Functional |
| TC-LENG-005 | Content gaps detected and recommended | EARS-LENG-005 | Should | Functional |
| TC-LENG-006 | 200 wines processed in < 5 minutes | EARS-LENG-006 | Must | Performance |
| TC-LENG-007 | Bronze, Silver, Gold tiers generated | EARS-LENG-007 | Must | Functional |
| TC-LENG-008 | Wines grouped by region, type, or price | EARS-LENG-008 | Must | Functional |
| TC-LENG-009 | Human-readable titles generated | EARS-LENG-009 | Must | Functional |
| TC-LENG-010 | Curriculum respects wine associations | EARS-LENG-010 | Must | Functional |
| TC-LENG-011 | Curriculum adapts to wine list changes | EARS-LENG-011 | Should | Functional |
| TC-LENG-012 | Wine list deletion updates curriculum | EARS-LENG-012 | Should | Functional |
| TC-LENG-013 | Curriculum generation idempotent | EARS-LENG-013 | Should | Functional |

### 4.11 Learning Engine - Quiz Generation (QGEN)

**EARS Requirements:** EARS-QGEN-001 through EARS-QGEN-014

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-QGEN-001 | Quiz generated from wine attributes | EARS-QGEN-001 | Must | Functional |
| TC-QGEN-002 | Question types varied by tier | EARS-QGEN-002 | Must | Functional |
| TC-QGEN-003 | Distractors plausible but incorrect | EARS-QGEN-003 | Must | Quality |
| TC-QGEN-004 | Questions appropriately difficult | EARS-QGEN-004 | Must | Quality |
| TC-QGEN-005 | Quiz generation < 30 seconds | EARS-QGEN-005 | Must | Performance |
| TC-QGEN-006 | Questions linked to source wines | EARS-QGEN-006 | Must | Functional |
| TC-QGEN-007 | Explanations reference learning material | EARS-QGEN-007 | Should | Functional |
| TC-QGEN-008 | No duplicate questions generated | EARS-QGEN-008 | Must | Quality |
| TC-QGEN-009 | Questions grammatically correct | EARS-QGEN-009 | Must | Quality |
| TC-QGEN-010 | Tier-appropriate vocabulary | EARS-QGEN-010 | Must | Quality |
| TC-QGEN-011 | Generated quiz editable by author | EARS-QGEN-011 | Must | Functional |
| TC-QGEN-012 | Regeneration creates new questions | EARS-QGEN-012 | Should | Functional |
| TC-QGEN-013 | Question bank for future reuse | EARS-QGEN-013 | Should | Functional |
| TC-QGEN-014 | Quality metrics tracked | EARS-QGEN-014 | Should | Functional |

### 4.12 Learning Engine - Scenario Generation (SGEN)

**EARS Requirements:** EARS-SGEN-001 through EARS-SGEN-014

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-SGEN-001 | Scenarios generated for wine pairings | EARS-SGEN-001 | Must | Functional |
| TC-SGEN-002 | Customer personas realistic | EARS-SGEN-002 | Must | Quality |
| TC-SGEN-003 | Dialogue contextually appropriate | EARS-SGEN-003 | Must | Quality |
| TC-SGEN-004 | Branching paths created | EARS-SGEN-004 | Must | Functional |
| TC-SGEN-005 | Scenario generation < 1 minute | EARS-SGEN-005 | Must | Performance |
| TC-SGEN-006 | Scenarios reference relevant wines | EARS-SGEN-006 | Must | Functional |
| TC-SGEN-007 | Feedback educational and constructive | EARS-SGEN-007 | Must | Quality |
| TC-SGEN-008 | Optimal path clearly defined | EARS-SGEN-008 | Must | Functional |
| TC-SGEN-009 | All paths reach valid conclusion | EARS-SGEN-009 | Must | Quality |
| TC-SGEN-010 | Tier-appropriate complexity | EARS-SGEN-010 | Must | Functional |
| TC-SGEN-011 | Generated scenario editable | EARS-SGEN-011 | Must | Functional |
| TC-SGEN-012 | Regeneration creates new scenario | EARS-SGEN-012 | Should | Functional |
| TC-SGEN-013 | Scenario categories supported | EARS-SGEN-013 | Should | Functional |
| TC-SGEN-014 | Quality metrics tracked | EARS-SGEN-014 | Should | Functional |

### 4.13 Adaptive Learning (ADPT)

**EARS Requirements:** EARS-ADPT-001 through EARS-ADPT-010

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-ADPT-001 | Gap analysis identifies weak areas | EARS-ADPT-001 | Must | Functional |
| TC-ADPT-002 | Quiz results update weak areas | EARS-ADPT-002 | Must | Functional |
| TC-ADPT-003 | Targeted content recommended | EARS-ADPT-003 | Must | Functional |
| TC-ADPT-004 | Difficulty adapts to performance | EARS-ADPT-004 | Should | Functional |
| TC-ADPT-005 | Mastery thresholds configurable | EARS-ADPT-005 | Should | Functional |
| TC-ADPT-006 | Spaced repetition for retention | EARS-ADPT-006 | Should | Functional |
| TC-ADPT-007 | Forgetting curve modelled | EARS-ADPT-007 | Should | Functional |
| TC-ADPT-008 | Review reminders scheduled | EARS-ADPT-008 | Should | Notification |
| TC-ADPT-009 | Performance trends analysed | EARS-ADPT-009 | Should | Functional |
| TC-ADPT-010 | Adaptive recommendations accurate | EARS-ADPT-010 | Must | Quality |

### 4.14 Progress Tracking (PROG)

**EARS Requirements:** EARS-PROG-001 through EARS-PROG-015

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-PROG-001 | Lesson completion tracked | EARS-PROG-001 | Must | Functional |
| TC-PROG-002 | Quiz scores tracked | EARS-PROG-002 | Must | Functional |
| TC-PROG-003 | Scenario scores tracked | EARS-PROG-003 | Must | Functional |
| TC-PROG-004 | Module completion calculated | EARS-PROG-004 | Must | Functional |
| TC-PROG-005 | Progress percentage accurate | EARS-PROG-005 | Must | Functional |
| TC-PROG-006 | Progress synced across devices | EARS-PROG-006 | Should | Functional |
| TC-PROG-007 | Certification awarded at threshold | EARS-PROG-007 | Must | Business Rule |
| TC-PROG-008 | Bronze requires 70% overall | EARS-PROG-007 | Must | Business Rule |
| TC-PROG-009 | Silver requires Bronze + 80% | EARS-PROG-007 | Must | Business Rule |
| TC-PROG-010 | Gold requires Silver + 90% | EARS-PROG-007 | Must | Business Rule |
| TC-PROG-011 | Certificate generated with unique number | EARS-PROG-008 | Must | Functional |
| TC-PROG-012 | Certificate downloadable as PDF | EARS-PROG-009 | Should | Functional |
| TC-PROG-013 | Learning streak tracked | EARS-PROG-010 | Should | Functional |
| TC-PROG-014 | Streak notifications sent | EARS-PROG-011 | Should | Notification |
| TC-PROG-015 | Time-on-task tracked | EARS-PROG-012 | Should | Functional |

### 4.15 Reporting (REPT)

**EARS Requirements:** EARS-REPT-001 through EARS-REPT-012

| Test Case ID | Description | EARS Ref | Priority | Type |
|--------------|-------------|----------|----------|------|
| TC-REPT-001 | Team progress dashboard displays | EARS-REPT-001 | Must | Functional |
| TC-REPT-002 | Individual progress viewable | EARS-REPT-002 | Must | Functional |
| TC-REPT-003 | Certification summary by team | EARS-REPT-003 | Must | Functional |
| TC-REPT-004 | At-risk learners highlighted | EARS-REPT-004 | Should | Functional |
| TC-REPT-005 | Date range filtering | EARS-REPT-005 | Must | Functional |
| TC-REPT-006 | Export to PDF | EARS-REPT-006 | Should | Functional |
| TC-REPT-007 | Export to CSV | EARS-REPT-007 | Should | Functional |
| TC-REPT-008 | Scheduled reports | EARS-REPT-008 | Should | Functional |
| TC-REPT-009 | Organisation-wide analytics | EARS-REPT-009 | Should | Functional |
| TC-REPT-010 | Content effectiveness metrics | EARS-REPT-010 | Should | Functional |
| TC-REPT-011 | Quiz item analysis | EARS-REPT-011 | Should | Functional |
| TC-REPT-012 | Engagement metrics | EARS-REPT-012 | Should | Functional |

---

## 5. Integration Test Scenarios

### 5.1 Wine Import to Curriculum Generation

**Scenario:** Complete wine import triggers automatic curriculum generation

```gherkin
Feature: Wine Import Triggers Curriculum Generation

  Background:
    Given an organisation "Demo Venue" exists
    And user "admin@demo.com" is logged in as Admin

  Scenario: Successful wine import triggers curriculum generation
    Given I have a valid CSV file with 50 wines
    When I upload the CSV file to the import endpoint
    Then the system validates all 50 rows
    And the validation report shows 0 errors
    When I confirm the import
    Then 50 wine records are created in DRAFT status
    And a curriculum generation job is queued
    When the job completes
    Then learning modules are created covering all 50 wines
    And quizzes are generated for each module
    And scenarios are generated for wine categories
    And all generated content is in DRAFT status for review

  Scenario: Wine import with validation errors
    Given I have a CSV file with 3 invalid rows
    When I upload the CSV file
    Then the validation report shows 3 errors
    And each error includes row number, field name, and message
    When I fix the errors and re-upload
    Then validation passes
```

**Test Steps:**
1. Upload valid CSV with wines
2. Verify wines created in database
3. Verify curriculum generation job triggered
4. Wait for job completion (timeout: 5 minutes)
5. Verify modules created with correct groupings
6. Verify quizzes generated with tier-appropriate questions
7. Verify scenarios generated with realistic dialogues
8. Verify all content links to source wines

### 5.2 Learner Journey: Onboarding to Bronze Certification

**Scenario:** New learner completes onboarding and achieves Bronze certification

```gherkin
Feature: Learner Onboarding to Certification

  Scenario: Complete learner journey to Bronze certification
    Given an organisation "Demo Venue" with published curriculum
    And a user invitation sent to "sophie@demo.com"

    # Onboarding
    When Sophie clicks the invitation link
    Then she sees the registration form
    When she creates a password meeting requirements
    And accepts terms and conditions
    And completes her profile
    Then her account is activated
    And she sees the welcome tutorial

    # Learning
    When Sophie completes 3 Bronze-tier lessons
    Then her progress shows 3 lessons completed
    And the next recommended content is appropriate

    # Quiz
    When Sophie starts the Bronze certification quiz
    Then she sees quiz rules and question count
    When she answers all questions (scoring 75%)
    Then she passes the quiz (threshold: 70%)
    And she sees her results with explanations

    # Scenario
    When Sophie completes a Bronze-tier scenario
    Then she sees feedback on her choices
    And her scenario score is recorded

    # Certification
    When Sophie meets all Bronze requirements
    Then she is awarded Bronze certification
    And a certificate is generated with unique number
    And her manager is notified
```

**Test Steps:**
1. Create test organisation with published content
2. Send invitation via API
3. Simulate registration flow
4. Complete lessons via API
5. Verify progress updates
6. Complete quiz via API
7. Verify score calculation and pass/fail logic
8. Complete scenario via API
9. Verify certification awarded
10. Verify notification sent

### 5.3 Content Workflow: Draft to Published

**Scenario:** Content author creates content and it goes through approval workflow

```gherkin
Feature: Content Approval Workflow

  Scenario: Standard approval workflow
    Given user "author@demo.com" is a Content Author
    And user "expert@demo.com" is a Domain Expert
    And user "admin@demo.com" is a Content Admin

    # Creation
    When Author creates a new wine entry
    Then the wine is saved in DRAFT status
    And Author can edit the wine

    # Submit for review
    When Author submits the wine for review
    Then the status changes to REVIEW
    And editing is locked
    And Expert receives a notification

    # Expert review
    When Expert reviews and approves
    Then the review is recorded
    And Admin receives notification

    # Admin approval
    When Admin approves for publication
    Then the status changes to PUBLISHED
    And publishedAt timestamp is set
    And the wine is visible to learners
    And all parties are notified
```

**Test Steps:**
1. Create wine as author
2. Verify DRAFT status and edit capability
3. Submit for review
4. Verify status change and lock
5. Verify notification sent
6. Approve as expert
7. Verify review recorded
8. Publish as admin
9. Verify status, timestamp, visibility

### 5.4 Multi-Tenant Data Isolation

**Scenario:** Verify data isolation between organisations

```gherkin
Feature: Multi-Tenant Data Isolation

  Scenario: Users cannot access other organisation's data
    Given organisation "Venue A" with wines
    And organisation "Venue B" with wines
    And user "user-a@venue-a.com" belongs to "Venue A"
    And user "user-b@venue-b.com" belongs to "Venue B"

    # API isolation
    When User A requests wine list
    Then they receive only Venue A wines
    When User A requests Venue B wine by ID
    Then they receive 404 Not Found

    # Direct ID access attempt
    When User A attempts to update Venue B wine
    Then they receive 404 Not Found
    And no data is modified

    # Search isolation
    When User A searches for wines
    Then results contain only Venue A wines

    # Reporting isolation
    When User A views progress dashboard
    Then they see only Venue A team members
```

**Test Steps:**
1. Create two organisations with data
2. Create users in each organisation
3. Authenticate as User A
4. Attempt to list other org's wines (verify empty/404)
5. Attempt to get specific wine by ID (verify 404)
6. Attempt to update wine (verify 404)
7. Search and verify isolation
8. Repeat for all multi-tenant resources

### 5.5 Learning Engine Quality Validation

**Scenario:** Verify generated content meets quality standards

```gherkin
Feature: Learning Engine Content Quality

  Scenario: Quiz generation produces quality questions
    Given an organisation with 10 wines covering different regions
    When I trigger quiz generation for the wine module
    Then 10 questions are generated
    And each question:
      | Has clear question text |
      | Has 4 answer options |
      | Has exactly one correct answer |
      | Has plausible distractors |
      | Has explanation text |
      | Links to source wine |
    And no duplicate questions exist
    And vocabulary matches the tier level

  Scenario: Scenario generation produces quality dialogues
    Given an organisation with wines for pairing recommendations
    When I trigger scenario generation
    Then a scenario is generated with:
      | Realistic customer persona |
      | Contextual situation |
      | 4+ dialogue steps |
      | 2-4 choices per step |
      | Optimal path defined |
      | Educational feedback |
    And all paths reach a conclusion
    And referenced wines are relevant
```

---

## 6. API Test Plan

### 6.1 Endpoint Coverage Matrix

All 62 API paths require the following test coverage:

| Endpoint Category | Paths | Operations | Status |
|-------------------|-------|------------|--------|
| Authentication | 5 | 5 | Required |
| Users | 7 | 10 | Required |
| Organisations | 4 | 6 | Required |
| Wines | 8 | 12 | Required |
| Modules | 6 | 9 | Required |
| Lessons | 4 | 6 | Required |
| Quizzes | 7 | 10 | Required |
| Questions | 4 | 6 | Required |
| Scenarios | 7 | 10 | Required |
| Progress | 5 | 7 | Required |
| Reports | 3 | 4 | Required |
| Imports | 4 | 5 | Required |
| **Total** | **62** | **81** | - |

### 6.2 API Test Types

#### 6.2.1 Happy Path Tests

```javascript
// Example: Create Wine API
describe('POST /api/v1/wines', () => {
  it('creates wine with valid data', async () => {
    const response = await request(app)
      .post('/api/v1/wines')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({
        name: 'Château Margaux',
        region: 'Bordeaux',
        country: 'FR',
        wineType: 'red',
        grapeVarieties: ['Cabernet Sauvignon', 'Merlot'],
        priceTier: 'luxury'
      });

    expect(response.status).toBe(201);
    expect(response.body.data).toMatchObject({
      name: 'Château Margaux',
      status: 'DRAFT',
      version: 1
    });
    expect(response.body.data.id).toBeDefined();
  });
});
```

#### 6.2.2 Validation Error Tests (400)

```javascript
describe('POST /api/v1/wines - Validation', () => {
  it('returns 400 for missing required field', async () => {
    const response = await request(app)
      .post('/api/v1/wines')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({
        name: 'Test Wine'
        // Missing: region, country, wineType, grapeVarieties, priceTier
      });

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
    expect(response.body.error.details).toContainEqual(
      expect.objectContaining({ field: 'region' })
    );
  });

  it('returns 400 for invalid enum value', async () => {
    const response = await request(app)
      .post('/api/v1/wines')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({
        name: 'Test Wine',
        region: 'Bordeaux',
        country: 'FR',
        wineType: 'invalid_type', // Invalid enum
        grapeVarieties: ['Merlot'],
        priceTier: 'moderate'
      });

    expect(response.status).toBe(400);
    expect(response.body.error.details[0].field).toBe('wineType');
  });
});
```

#### 6.2.3 Authentication Error Tests (401)

```javascript
describe('Authentication Errors', () => {
  it('returns 401 for missing token', async () => {
    const response = await request(app)
      .get('/api/v1/wines');

    expect(response.status).toBe(401);
    expect(response.body.error.code).toBe('UNAUTHORIZED');
  });

  it('returns 401 for expired token', async () => {
    const response = await request(app)
      .get('/api/v1/wines')
      .set('Authorization', `Bearer ${expiredToken}`);

    expect(response.status).toBe(401);
    expect(response.body.error.code).toBe('TOKEN_EXPIRED');
  });

  it('returns 401 for invalid token', async () => {
    const response = await request(app)
      .get('/api/v1/wines')
      .set('Authorization', 'Bearer invalid.token.here');

    expect(response.status).toBe(401);
    expect(response.body.error.code).toBe('INVALID_TOKEN');
  });
});
```

#### 6.2.4 Authorisation Error Tests (403)

```javascript
describe('Authorisation Errors', () => {
  it('returns 403 for insufficient permissions', async () => {
    // Learner trying to create wine (requires Admin)
    const response = await request(app)
      .post('/api/v1/wines')
      .set('Authorization', `Bearer ${learnerToken}`)
      .send(validWineData);

    expect(response.status).toBe(403);
    expect(response.body.error.code).toBe('FORBIDDEN');
    expect(response.body.error.message).toContain('permission');
  });

  it('returns 403 for cross-organisation access', async () => {
    // User from Org A trying to access Org B resource
    const response = await request(app)
      .get(`/api/v1/wines/${orgBWineId}`)
      .set('Authorization', `Bearer ${orgAUserToken}`);

    expect(response.status).toBe(404); // Returns 404, not 403 for security
  });
});
```

#### 6.2.5 Not Found Tests (404)

```javascript
describe('Not Found Errors', () => {
  it('returns 404 for non-existent resource', async () => {
    const fakeId = '00000000-0000-0000-0000-000000000000';
    const response = await request(app)
      .get(`/api/v1/wines/${fakeId}`)
      .set('Authorization', `Bearer ${adminToken}`);

    expect(response.status).toBe(404);
    expect(response.body.error.code).toBe('NOT_FOUND');
  });

  it('returns 404 for deleted resource', async () => {
    const response = await request(app)
      .get(`/api/v1/wines/${deletedWineId}`)
      .set('Authorization', `Bearer ${adminToken}`);

    expect(response.status).toBe(404);
  });
});
```

### 6.3 Contract Testing

#### 6.3.1 Request Schema Validation

```javascript
describe('Request Schema Validation', () => {
  const ajv = new Ajv();
  const schema = require('./openapi.yaml');

  it('validates CreateWineRequest against schema', () => {
    const requestSchema = schema.components.schemas.CreateWineRequest;
    const validate = ajv.compile(requestSchema);

    const validRequest = {
      name: 'Test Wine',
      region: 'Bordeaux',
      country: 'FR',
      wineType: 'red',
      grapeVarieties: ['Merlot'],
      priceTier: 'moderate'
    };

    expect(validate(validRequest)).toBe(true);
  });
});
```

#### 6.3.2 Response Schema Validation

```javascript
describe('Response Schema Validation', () => {
  it('validates wine response against OpenAPI schema', async () => {
    const response = await request(app)
      .get(`/api/v1/wines/${wineId}`)
      .set('Authorization', `Bearer ${adminToken}`);

    expect(response.status).toBe(200);

    // Validate against OpenAPI schema
    const result = validateResponse(
      response.body,
      'WineResponse',
      openApiSchema
    );
    expect(result.valid).toBe(true);
  });
});
```

### 6.4 Rate Limiting Tests

```javascript
describe('Rate Limiting', () => {
  it('returns 429 when rate limit exceeded', async () => {
    const requests = Array(101).fill().map(() =>
      request(app)
        .get('/api/v1/wines')
        .set('Authorization', `Bearer ${userToken}`)
    );

    const responses = await Promise.all(requests);
    const rateLimited = responses.filter(r => r.status === 429);

    expect(rateLimited.length).toBeGreaterThan(0);
    expect(rateLimited[0].body.error.code).toBe('RATE_LIMITED');
    expect(rateLimited[0].headers['retry-after']).toBeDefined();
  });
});
```

---

## 7. Performance Test Plan

### 7.1 Performance Targets

From PRD and EARS requirements:

| Metric | Target | EARS Ref | Test Scenario |
|--------|--------|----------|---------------|
| Page load time | < 2s (p95) | EARS-PERF-001 | Dashboard, wine list, lesson view |
| Search response | < 1s | EARS-PERF-002 | Wine search, user search |
| API response | < 500ms (p95) | EARS-PERF-003 | All API endpoints |
| Quiz generation | < 30s | EARS-PERF-004 | Generate 10-question quiz |
| Scenario generation | < 1 min | EARS-PERF-005 | Generate complete scenario |
| Curriculum generation | < 5 min | EARS-PERF-006 | 200 wines full curriculum |
| Concurrent users | 1,000/org | EARS-SCAL-003 | Mixed workload |

### 7.2 Load Test Scenarios

#### 7.2.1 Baseline Load Test

```javascript
// k6 baseline test
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '10m', target: 100 }, // Steady state
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  // Mixed workload: 70% reads, 20% writes, 10% complex operations
  const rand = Math.random();

  if (rand < 0.7) {
    // Read operations
    http.get(`${BASE_URL}/api/v1/wines`, { headers: authHeaders });
  } else if (rand < 0.9) {
    // Write operations
    http.post(`${BASE_URL}/api/v1/progress`, progressPayload, { headers: authHeaders });
  } else {
    // Complex operations
    http.post(`${BASE_URL}/api/v1/quizzes/${quizId}/attempts`, {}, { headers: authHeaders });
  }

  sleep(1);
}
```

#### 7.2.2 Load Test (Expected Volume)

| Parameter | Value |
|-----------|-------|
| Duration | 30 minutes |
| Concurrent Users | 500 |
| Ramp-up | 5 minutes |
| Workload Mix | 70% reads, 30% writes |
| Think Time | 1-3 seconds |

**Success Criteria:**
- p95 response time < 500ms
- Error rate < 1%
- No memory leaks
- No connection pool exhaustion

#### 7.2.3 Stress Test (Beyond Expected)

| Parameter | Value |
|-----------|-------|
| Duration | 15 minutes |
| Concurrent Users | 1,000 |
| Ramp-up | 3 minutes |
| Workload | Aggressive (no think time) |

**Success Criteria:**
- System remains responsive
- Graceful degradation (not crashes)
- Recovery within 5 minutes of load reduction

#### 7.2.4 Spike Test

| Parameter | Value |
|-----------|-------|
| Initial Users | 50 |
| Spike Users | 500 |
| Spike Duration | 1 minute |
| Recovery Observation | 5 minutes |

**Success Criteria:**
- System handles spike without errors
- Auto-scaling triggers (if configured)
- Response times recover after spike

#### 7.2.5 Soak Test

| Parameter | Value |
|-----------|-------|
| Duration | 4 hours |
| Concurrent Users | 200 |
| Workload | Realistic mix |

**Success Criteria:**
- No memory leaks (memory stable over time)
- No connection leaks
- No degradation in response times
- Database connections stable

### 7.3 Performance Test Reports

Each performance test produces:

1. **Summary Report**
   - Test duration and parameters
   - Pass/fail against thresholds
   - Key metrics summary

2. **Detailed Metrics**
   - Response time percentiles (p50, p90, p95, p99)
   - Throughput (requests/second)
   - Error rates by endpoint
   - Resource utilisation

3. **Trend Analysis**
   - Comparison with previous runs
   - Regression identification
   - Capacity recommendations

---

## 8. Security Test Plan

### 8.1 OWASP Top 10 Coverage

| Vulnerability | Test Approach | Tools | Priority |
|---------------|---------------|-------|----------|
| **A01: Broken Access Control** | Role/permission testing, tenant isolation | Manual + Automated | Critical |
| **A02: Cryptographic Failures** | TLS verification, password storage audit | Manual + SSL Labs | Critical |
| **A03: Injection** | SQL, NoSQL, Command injection testing | OWASP ZAP, sqlmap | Critical |
| **A04: Insecure Design** | Architecture review, threat modelling | Manual | High |
| **A05: Security Misconfiguration** | Header analysis, default credentials | OWASP ZAP, Nmap | High |
| **A06: Vulnerable Components** | Dependency scanning | Snyk, npm audit | High |
| **A07: Auth Failures** | Session management, credential testing | Burp Suite | Critical |
| **A08: Data Integrity Failures** | Input validation, deserialization | Manual + Automated | High |
| **A09: Logging Failures** | Audit log review, sensitive data exposure | Manual | Medium |
| **A10: SSRF** | URL parameter testing | Burp Suite | Medium |

### 8.2 Authentication Security Tests

| Test ID | Description | Expected Result |
|---------|-------------|-----------------|
| SEC-AUTH-001 | Brute force login prevention | Account locked after 5 attempts |
| SEC-AUTH-002 | Password complexity enforcement | Weak passwords rejected |
| SEC-AUTH-003 | JWT token tampering | Invalid signature rejected |
| SEC-AUTH-004 | JWT token expiry | Expired tokens rejected |
| SEC-AUTH-005 | Session fixation prevention | New session ID on login |
| SEC-AUTH-006 | Logout token invalidation | Logged out tokens rejected |
| SEC-AUTH-007 | Password reset token security | Single-use, time-limited tokens |
| SEC-AUTH-008 | Credential enumeration prevention | Generic error messages |

### 8.3 Authorisation Security Tests

| Test ID | Description | Expected Result |
|---------|-------------|-----------------|
| SEC-AUTHZ-001 | Horizontal privilege escalation | Cannot access other users' data |
| SEC-AUTHZ-002 | Vertical privilege escalation | Cannot perform admin actions as user |
| SEC-AUTHZ-003 | Tenant boundary violation | Cannot access other organisation's data |
| SEC-AUTHZ-004 | Direct object reference | UUIDs not guessable, access checked |
| SEC-AUTHZ-005 | Role permission enforcement | Each role has correct permissions |
| SEC-AUTHZ-006 | API permission bypass | All endpoints check authorisation |

### 8.4 Injection Tests

| Test ID | Description | Payload Examples |
|---------|-------------|------------------|
| SEC-INJ-001 | SQL injection in search | `'; DROP TABLE wines; --` |
| SEC-INJ-002 | SQL injection in filters | `1 OR 1=1` |
| SEC-INJ-003 | NoSQL injection | `{"$gt": ""}` |
| SEC-INJ-004 | Command injection | `; cat /etc/passwd` |
| SEC-INJ-005 | LDAP injection | `*)(uid=*))(|(uid=*` |
| SEC-INJ-006 | XSS in content fields | `<script>alert('xss')</script>` |
| SEC-INJ-007 | Stored XSS | Persistent script injection |
| SEC-INJ-008 | HTML injection | `<img src=x onerror=alert(1)>` |

### 8.5 Multi-Tenant Isolation Tests

```javascript
describe('Multi-Tenant Security', () => {
  let orgA, orgB, userA, userB;

  beforeAll(async () => {
    // Create two organisations with data
    orgA = await createOrganisation('Org A');
    orgB = await createOrganisation('Org B');
    userA = await createUser(orgA.id);
    userB = await createUser(orgB.id);
  });

  it('prevents cross-tenant data access via API', async () => {
    const wineB = await createWine(orgB.id);

    const response = await request(app)
      .get(`/api/v1/wines/${wineB.id}`)
      .set('Authorization', `Bearer ${userA.token}`);

    expect(response.status).toBe(404);
  });

  it('prevents cross-tenant data access via direct DB', async () => {
    // Verify RLS policies
    const result = await db.query(
      'SELECT * FROM wines WHERE id = $1',
      [orgBWineId],
      { userId: userA.id, organisationId: orgA.id }
    );

    expect(result.rows).toHaveLength(0);
  });

  it('prevents tenant ID manipulation in requests', async () => {
    const response = await request(app)
      .post('/api/v1/wines')
      .set('Authorization', `Bearer ${userA.token}`)
      .send({
        ...validWineData,
        organisationId: orgB.id // Attempted manipulation
      });

    // Should either ignore or reject the organisationId
    expect(response.body.data.organisationId).toBe(orgA.id);
  });
});
```

### 8.6 Rate Limiting & DoS Prevention

| Test ID | Description | Expected Result |
|---------|-------------|-----------------|
| SEC-RATE-001 | API rate limit enforcement | 429 after 100 req/min |
| SEC-RATE-002 | Login rate limit | Slower after failures |
| SEC-RATE-003 | File upload limits | Large files rejected |
| SEC-RATE-004 | Request size limits | Oversized payloads rejected |
| SEC-RATE-005 | Concurrent connection limits | Excess connections queued |

### 8.7 Security Scan Schedule

| Scan Type | Frequency | Tool | Owner |
|-----------|-----------|------|-------|
| Dependency vulnerability | Daily | Snyk | CI/CD |
| SAST (Static analysis) | Every PR | SonarQube | CI/CD |
| DAST (Dynamic analysis) | Weekly | OWASP ZAP | Security |
| Penetration test | Pre-release | Manual + Burp | Security |
| Infrastructure scan | Monthly | AWS Inspector | DevOps |

---

## 9. Accessibility Test Plan

### 9.1 WCAG 2.1 Level AA Requirements

| Guideline | Criteria | Test Method |
|-----------|----------|-------------|
| **1.1 Text Alternatives** | All images have alt text | axe-core + manual |
| **1.2 Time-based Media** | Captions for video | Manual |
| **1.3 Adaptable** | Content structure via headings | axe-core + manual |
| **1.4 Distinguishable** | Colour contrast 4.5:1 | Colour contrast analyser |
| **2.1 Keyboard** | All functions keyboard accessible | Manual keyboard testing |
| **2.2 Enough Time** | Adjustable time limits | Manual |
| **2.3 Seizures** | No flashing content | Manual review |
| **2.4 Navigable** | Skip links, focus order | Manual |
| **3.1 Readable** | Language declaration | axe-core |
| **3.2 Predictable** | Consistent navigation | Manual |
| **3.3 Input Assistance** | Error identification, labels | axe-core + manual |
| **4.1 Compatible** | Valid HTML, ARIA | axe-core + validator |

### 9.2 Automated Accessibility Tests

```javascript
// Playwright + axe-core
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('dashboard has no accessibility violations', async ({ page }) => {
    await page.goto('/dashboard');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('wine list has no accessibility violations', async ({ page }) => {
    await page.goto('/wines');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('quiz interface is accessible', async ({ page }) => {
    await page.goto('/quiz/start');

    const results = await new AxeBuilder({ page }).analyze();

    expect(results.violations).toEqual([]);
  });
});
```

### 9.3 Manual Accessibility Tests

| Test ID | Description | Pass Criteria |
|---------|-------------|---------------|
| A11Y-KB-001 | Complete login using only keyboard | All fields reachable, form submittable |
| A11Y-KB-002 | Navigate wine list using keyboard | Can browse, select, and view wines |
| A11Y-KB-003 | Complete quiz using keyboard | Can answer all questions and submit |
| A11Y-KB-004 | Navigate scenario using keyboard | Can make all choices |
| A11Y-SR-001 | Screen reader announces page title | Title read on navigation |
| A11Y-SR-002 | Screen reader announces form errors | Errors announced when they occur |
| A11Y-SR-003 | Screen reader reads quiz questions | Questions and options clearly read |
| A11Y-SR-004 | Screen reader navigation landmarks | Regions properly identified |
| A11Y-VIS-001 | Zoom to 200% without horizontal scroll | Content reflows correctly |
| A11Y-VIS-002 | High contrast mode | All content visible |
| A11Y-COL-001 | Colour not sole indicator | Information conveyed other ways |

### 9.4 Screen Reader Testing

| Screen Reader | Browser | Platform | Priority |
|---------------|---------|----------|----------|
| NVDA | Chrome, Firefox | Windows | Primary |
| VoiceOver | Safari | macOS | Primary |
| VoiceOver | Safari | iOS | Secondary |
| TalkBack | Chrome | Android | Secondary |

---

## 10. UAT Test Plan

### 10.1 UAT Scenarios

Based on PRD User Journeys (Section 6):

| UAT ID | User Journey | Primary Actor | Acceptance Criteria |
|--------|--------------|---------------|---------------------|
| UAT-001 | First-time onboarding | Learner (Sophie) | Complete registration in < 5 min |
| UAT-002 | Daily practice session | Learner (Sophie) | Complete 10-min session smoothly |
| UAT-003 | Taking certification quiz | Learner (Sophie) | Pass/fail clearly communicated |
| UAT-004 | Completing customer scenario | Learner (Sophie) | Feedback helpful and clear |
| UAT-005 | Setting up venue | Manager (James) | Upload wines, generate curriculum |
| UAT-006 | Inviting team members | Manager (James) | Team receives and accepts invites |
| UAT-007 | Monitoring team progress | Manager (James) | Dashboard shows actionable insights |
| UAT-008 | Generating reports | Manager (James) | Reports export correctly |
| UAT-009 | Creating custom quiz | Content Author (Marcus) | Quiz created and previewed |
| UAT-010 | Creating custom scenario | Content Author (Marcus) | Scenario plays correctly |
| UAT-011 | Reviewing content for approval | Content Admin | Workflow completes correctly |

### 10.2 UAT-001: First-Time Onboarding

**User Story:** As Sophie (new learner), I want to complete registration and start learning quickly.

**Prerequisites:**
- Manager has sent invitation email
- Email received with valid link

**Test Steps:**

| Step | Action | Expected Result | Pass/Fail |
|------|--------|-----------------|-----------|
| 1 | Click invitation link | Registration page opens | |
| 2 | Create password | Validation feedback shown | |
| 3 | Accept terms | Terms displayed, checkbox works | |
| 4 | Complete profile | Fields save correctly | |
| 5 | View welcome tutorial | Tutorial plays/skippable | |
| 6 | View dashboard | Personalised content shown | |
| 7 | Start first lesson | Lesson loads correctly | |
| 8 | Complete first lesson | Progress updated | |

**Time Target:** < 15 minutes total

### 10.3 UAT-005: Setting Up Venue

**User Story:** As James (manager), I want to set up my venue and import our wine list.

**Prerequisites:**
- Organisation account created
- Manager has Owner/Admin role
- Wine list prepared in CSV

**Test Steps:**

| Step | Action | Expected Result | Pass/Fail |
|------|--------|-----------------|-----------|
| 1 | Log in as manager | Dashboard shows setup prompts | |
| 2 | Enter organisation details | Details saved, branding preview | |
| 3 | Download CSV template | Template downloads correctly | |
| 4 | Upload completed CSV | File accepted, validation runs | |
| 5 | Review validation report | Errors clearly shown | |
| 6 | Fix errors, re-upload | Validation passes | |
| 7 | Preview import | Data looks correct | |
| 8 | Confirm import | Wines created | |
| 9 | Wait for curriculum generation | Progress indicator shown | |
| 10 | Review generated content | Modules, quizzes, scenarios created | |
| 11 | Publish content | Content available to learners | |

**Time Target:** < 60 minutes for 50 wines

### 10.4 UAT Sign-Off Criteria

| Criterion | Requirement | Status |
|-----------|-------------|--------|
| All UAT scenarios executed | 11/11 complete | |
| Critical paths successful | 100% pass rate | |
| No P1 defects | 0 open | |
| No P2 defects | 0 open | |
| Performance acceptable | Meets targets | |
| Usability acceptable | User satisfaction > 4/5 | |
| Stakeholder approval | Sign-off received | |

### 10.5 UAT Participants

| Role | Name | Responsibility |
|------|------|----------------|
| Product Owner | [TBD] | Sign-off authority |
| QA Lead | [TBD] | Test coordination |
| Learner Tester | [TBD] | Execute learner journeys |
| Manager Tester | [TBD] | Execute manager journeys |
| Content Author Tester | [TBD] | Execute authoring journeys |

---

## 11. Defect Management

### 11.1 Severity Levels

| Severity | Description | Examples | Response SLA |
|----------|-------------|----------|--------------|
| **P1 - Critical** | System unusable, data loss/corruption, security breach | Login broken, data deleted, tenant leak | 4 hours |
| **P2 - High** | Major feature broken, no workaround | Quiz submission fails, import crashes | 24 hours |
| **P3 - Medium** | Feature degraded, workaround exists | Slow performance, UI glitch | 72 hours |
| **P4 - Low** | Minor issue, cosmetic | Typo, alignment issue | Next sprint |

### 11.2 Defect Classification

| Category | Description |
|----------|-------------|
| Functional | Feature doesn't work as specified |
| Performance | Fails to meet performance targets |
| Security | Vulnerability or security concern |
| Usability | Poor user experience |
| Accessibility | WCAG compliance issue |
| Integration | Component interaction failure |
| Data | Data integrity or validation issue |

### 11.3 Defect Workflow

```
┌──────────┐     ┌──────────┐     ┌────────────┐     ┌─────────┐     ┌──────────┐     ┌────────┐
│   New    │────▶│  Triage  │────▶│ In Progress│────▶│  Fixed  │────▶│ Verified │────▶│ Closed │
└──────────┘     └──────────┘     └────────────┘     └─────────┘     └──────────┘     └────────┘
                      │                  │                │                │
                      ▼                  ▼                ▼                ▼
                 ┌─────────┐      ┌──────────┐     ┌───────────┐   ┌──────────┐
                 │Duplicate│      │  Blocked │     │ Won't Fix │   │ Reopened │
                 └─────────┘      └──────────┘     └───────────┘   └──────────┘
```

### 11.4 Defect Report Template

```markdown
## Defect Report

**ID:** DEF-XXXX
**Title:** [Clear, concise description]
**Severity:** P1/P2/P3/P4
**Category:** Functional/Performance/Security/Usability/Accessibility

### Environment
- Browser:
- OS:
- Environment: QA/Staging/Production
- User Role:

### Steps to Reproduce
1.
2.
3.

### Expected Result
[What should happen]

### Actual Result
[What actually happens]

### Screenshots/Logs
[Attach evidence]

### Related Requirements
- EARS-XXX-XXX

### Additional Notes
[Any other relevant information]
```

---

## 12. Test Automation

### 12.1 Automation Scope

| Test Type | Automation Target | Current | Gap |
|-----------|-------------------|---------|-----|
| Unit Tests | 100% | 0% | 100% |
| API Tests | 100% | 0% | 100% |
| Integration Tests | 90% | 0% | 90% |
| E2E Critical Paths | 70% | 0% | 70% |
| Performance Tests | 100% | 0% | 100% |
| Accessibility Scans | 80% | 0% | 80% |

### 12.2 Test Automation Stack

| Layer | Tool | Language | Repository |
|-------|------|----------|------------|
| Unit (Backend) | Jest | TypeScript | /api |
| Unit (Frontend) | Jest + RTL | TypeScript | /web |
| API | Jest + Supertest | TypeScript | /api/tests |
| E2E | Playwright | TypeScript | /e2e |
| Performance | k6 | JavaScript | /performance |
| Security | OWASP ZAP | N/A | CI/CD |
| Accessibility | axe-core | TypeScript | /e2e |

### 12.3 CI/CD Integration

```yaml
# GitHub Actions workflow
name: Test Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run unit tests
        run: npm test -- --coverage
      - name: Check coverage threshold
        run: npm run coverage:check

  api-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
    steps:
      - uses: actions/checkout@v4
      - name: Run API tests
        run: npm run test:api

  e2e-tests:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - name: Run E2E tests
        run: npm run test:e2e

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP scan
        uses: zaproxy/action-full-scan@v0.4.0

  performance-test:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Run k6 tests
        run: npm run test:performance
```

### 12.4 Test Execution Schedule

| Test Suite | Trigger | Duration | Blocking |
|------------|---------|----------|----------|
| Unit tests | Every commit | ~2 min | Yes |
| API tests | Every PR | ~5 min | Yes |
| Integration tests | Every PR | ~10 min | Yes |
| E2E smoke | Every PR | ~5 min | Yes |
| E2E full | Nightly | ~30 min | No |
| Performance | Weekly | ~60 min | No |
| Security scan | Weekly | ~30 min | No |
| Full regression | Pre-release | ~2 hours | Yes |

---

## 13. Quality Gates

### 13.1 PR Merge Gate

| Criterion | Threshold | Enforcement |
|-----------|-----------|-------------|
| Unit tests pass | 100% | Block merge |
| API tests pass | 100% | Block merge |
| Code coverage | ≥ target (see 3.1) | Block merge |
| Linting | No errors | Block merge |
| Type checking | No errors | Block merge |
| Security scan | No critical/high | Block merge |
| PR review | ≥ 1 approval | Block merge |

### 13.2 Staging Deployment Gate

| Criterion | Threshold | Enforcement |
|-----------|-----------|-------------|
| All PR gate criteria | Met | Block deploy |
| E2E smoke tests | 100% pass | Block deploy |
| Integration tests | 100% pass | Block deploy |
| No open P1 defects | 0 | Block deploy |

### 13.3 Production Release Gate

| Criterion | Threshold | Enforcement |
|-----------|-----------|-------------|
| All staging gate criteria | Met | Block release |
| Full E2E regression | 100% pass | Block release |
| Performance tests | Meet targets | Block release |
| Security scan | No critical/high/medium | Block release |
| Accessibility scan | No violations | Block release |
| UAT sign-off | Complete | Block release |
| No open P1/P2 defects | 0 | Block release |
| Rollback plan | Documented | Block release |

### 13.4 Gate Override Process

1. **Identify blocker:** Document specific failing criterion
2. **Risk assessment:** Evaluate impact of override
3. **Approval required:** QA Lead + Tech Lead + Product Owner
4. **Document decision:** Record in release notes
5. **Remediation plan:** Schedule fix for next release

---

## 14. Test Data Management

### 14.1 Test Data Strategy

| Environment | Data Source | Characteristics |
|-------------|-------------|-----------------|
| Local | Seed scripts | Minimal, deterministic |
| CI | Fixtures | Synthetic, isolated per test |
| QA | Snapshot + anonymisation | Production-like volume |
| Staging | Snapshot + anonymisation | Production-like volume |
| Performance | Generated | High volume, diverse |

### 14.2 Seed Data Categories

```typescript
// seed/organisations.ts
export const seedOrganisations = [
  {
    name: 'Demo Restaurant',
    slug: 'demo-restaurant',
    type: 'RESTAURANT',
    tier: 'PROFESSIONAL',
    userCount: 25,
  },
  {
    name: 'Test Hotel Group',
    slug: 'test-hotel',
    type: 'HOTEL',
    tier: 'ENTERPRISE',
    userCount: 150,
  },
  // Starter tier org for limit testing
  {
    name: 'Small Wine Bar',
    slug: 'small-wine-bar',
    type: 'WINE_BAR',
    tier: 'STARTER',
    userCount: 24, // Near limit
  },
];

// seed/wines.ts
export const seedWines = [
  // Red wines - various regions
  { name: 'Château Margaux 2018', type: 'red', region: 'Bordeaux', tier: 'luxury' },
  { name: 'Barolo DOCG 2019', type: 'red', region: 'Piedmont', tier: 'premium' },
  // White wines
  { name: 'Chablis Premier Cru 2021', type: 'white', region: 'Burgundy', tier: 'premium' },
  // Sparkling
  { name: 'Dom Pérignon 2012', type: 'sparkling', region: 'Champagne', tier: 'luxury' },
  // Budget options
  { name: 'House Red', type: 'red', region: 'Various', tier: 'budget' },
  // ... 50+ wines covering all types, regions, tiers
];
```

### 14.3 Data Anonymisation Rules

| Field | Anonymisation Method |
|-------|---------------------|
| Email | `user_{id}@test.example.com` |
| Name | Faker-generated name |
| Password | Reset to test password |
| IP Address | `0.0.0.0` |
| Phone | `+1-555-000-{random}` |
| Address | Faker-generated address |

### 14.4 Test Data Refresh

| Environment | Frequency | Process | Owner |
|-------------|-----------|---------|-------|
| Local | On demand | `npm run db:seed` | Developer |
| CI | Per build | Docker container reset | CI/CD |
| QA | Weekly (Monday 2am) | Snapshot restore + anonymise | DevOps |
| Staging | Weekly (Sunday 2am) | Snapshot restore + anonymise | DevOps |

---

## 15. Traceability Matrix

### 15.1 Requirements to Test Cases

| EARS Category | Requirements | Test Cases | Coverage |
|---------------|-------------|------------|----------|
| AUTH | 17 | 22 | 100% |
| USER | 15 | 20 | 100% |
| ORG | 14 | 16 | 100% |
| WINE | 25 | 29 | 100% |
| MOD | 16 | 16 | 100% |
| QUIZ | 27 | 27 | 100% |
| SCEN | 25 | 25 | 100% |
| CONT | 22 | 22 | 100% |
| IMPX | 17 | 20 | 100% |
| LENG | 13 | 13 | 100% |
| QGEN | 14 | 14 | 100% |
| SGEN | 14 | 14 | 100% |
| ADPT | 10 | 10 | 100% |
| PROG | 15 | 15 | 100% |
| REPT | 12 | 12 | 100% |
| NFR | 29 | Integrated | 100% |
| **Total** | **285** | **275+** | **100%** |

### 15.2 Test Case Traceability Format

```markdown
| Test Case ID | EARS Requirement | PRD Section | Priority | Automated | Status |
|--------------|------------------|-------------|----------|-----------|--------|
| TC-AUTH-001 | EARS-AUTH-001, EARS-AUTH-002 | 6.1.1 | Must | Yes | Ready |
| TC-AUTH-002 | EARS-AUTH-003 | 6.1.1 | Must | Yes | Ready |
| TC-WINE-001 | EARS-WINE-001 | 6.3.1 | Must | Yes | Ready |
```

### 15.3 Coverage Verification

Before each release, verify:

1. **Requirement coverage:** Every EARS requirement has at least one test case
2. **Test execution:** All test cases have been executed
3. **Pass rate:** All Must requirements pass, >90% Should pass
4. **Traceability updated:** Matrix reflects current state

---

## 16. Appendices

### 16.1 Test Case Template

```markdown
## Test Case: TC-XXX-NNN

**Title:** [Descriptive title]

**EARS Requirement:** EARS-XXX-NNN

**Priority:** Must/Should/Could

**Type:** Functional/Security/Performance/Accessibility

**Preconditions:**
- [Condition 1]
- [Condition 2]

**Test Data:**
- [Data item 1]
- [Data item 2]

**Steps:**
| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | [Action] | [Result] |
| 2 | [Action] | [Result] |

**Postconditions:**
- [State after test]

**Automation Status:** Automated/Manual/Planned

**Notes:**
[Additional information]
```

### 16.2 Defect Report Template

See Section 11.4.

### 16.3 Test Environment Setup Guide

```bash
# Local development environment setup

# 1. Prerequisites
node --version  # v20.x required
docker --version  # Docker Desktop required

# 2. Clone repository
git clone https://github.com/sommelier-spark/api.git
cd api

# 3. Install dependencies
npm install

# 4. Start services
docker-compose up -d postgres redis

# 5. Run migrations
npm run db:migrate

# 6. Seed test data
npm run db:seed

# 7. Run tests
npm test                    # Unit tests
npm run test:api           # API tests
npm run test:e2e           # E2E tests (requires app running)

# 8. Check coverage
npm run coverage
```

### 16.4 Reference Documents

| Document ID | Title | Version |
|-------------|-------|---------|
| SS-WS3-PRD | Product Requirements Document | 1.0 |
| SS-WS3-EARS | EARS Requirements Specification | 1.0 |
| SS-WS3-HLD | High-Level Design | 1.0 |
| SS-WS3-LLD | Low-Level Design | 1.0 |
| SS-WS3-API | API Specification | 1.0 |
| SS-WS3-DATA | Data Model | 1.0 |

### 16.5 Glossary

| Term | Definition |
|------|------------|
| EARS | Easy Approach to Requirements Syntax |
| E2E | End-to-End testing |
| MVP | Minimum Viable Product |
| p95 | 95th percentile |
| RLS | Row-Level Security |
| SLA | Service Level Agreement |
| UAT | User Acceptance Testing |
| WCAG | Web Content Accessibility Guidelines |

### 16.6 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-21 | Obi Wan | Initial draft |

---

*End of Document*

**CONFIDENTIAL — Sommelier Spark**
