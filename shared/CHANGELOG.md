# Sommelier Spark — Documentation Changelog

Version history for documentation across all Sommelier Spark projects.

---

## January 2026

### WS3.0 Complete — Core Domain Requirements

**Date:** 20 January 2026
**Project:** sommelier-spark-docs
**Tag:** ws3.0-complete

Completed WS3.0 (Work Stream 3.0) Core Domain Requirements specification suite.

**Sprints Completed:**
| Sprint | Focus | Documents |
|--------|-------|-----------|
| Sprint 1 | Content & Organization Domain | 4 |
| Sprint 2 | CMS Requirements | 4 |
| Sprint 3 | Learning Engine Requirements | 4 |

**Key Deliverables:**
- 271 total requirements (159 CMS + 112 Learning Engine)
- 47 Mermaid diagrams
- 18 question generation templates
- 12 scenario generation templates
- 18 wine attribute-to-question mappings
- 4 import/export schemas

**WS3.0 Documents:**
| Doc ID | Document |
|--------|----------|
| SS-WS3.0-CDM | Content Domain Model |
| SS-WS3.0-CLS | Content Lifecycle Specification |
| SS-WS3.0-ORG | Organization Model |
| SS-WS3.0-S1R | Sprint 1 Review Report |
| SS-WS3.0-CMS-FR | CMS Functional Requirements |
| SS-WS3.0-CMS-WF | CMS Workflow Specification |
| SS-WS3.0-CMS-IE | Content Import/Export Specification |
| SS-WS3.0-S2R | Sprint 2 Review Report |
| SS-WS3.0-LE-REQ | Learning Engine Requirements |
| SS-WS3.0-LE-CGR | Content Generation Rules |
| SS-WS3.0-LE-CLM | Content-to-Learning Mapping |
| SS-WS3.0-SUM | WS3.0 Completion Summary |

**Result:** PASS — Foundation complete for WS3 Specification Suite.

---

### WS2 Release — iOS Parity for Web App

**Date:** 3 January 2026
**Project:** sommelier_spark
**Version:** 2.0.0

Completed WS2 (Work Stream 2) sync to achieve iOS v2.0.0 feature parity for the web application.

**Features Implemented:**
- Tier-based progress tracking (Bronze/Silver/Gold)
- Quiz terminology aligned with iOS (Bronze/Silver/Gold, not Level 1/2/3)
- Pass thresholds: Bronze 70%, Silver 80%, Gold 90%
- Demo Mode (accelerate to Bronze/Silver)
- Granular reset (by category: wines, modules, quizzes, scenarios)
- Tier progress banners on all pages
- Tier progress cards on dashboard
- Bug fixes and parity alignment

**Documentation Updated:**
- User Guide Web (v1.0 → v2.0)
- Documentation Index updated
- WS2 documentation suite created (4 documents)

**WS2 Documentation:**
| Document | Description |
|----------|-------------|
| Feature Gap Analysis | iOS v2.0.0 vs Web comparison |
| Sync Prioritization | Task prioritization matrix |
| Verification Report | Testing and QA results (426/429 tests pass) |
| Completion Summary | Implementation summary |

---

### WS1 Final Documentation Review (SS-REV-002)

**Date:** 3 January 2026
**Project:** sommelier-spark-docs
**Version:** 1.0

Completed final review of all user documentation across all projects:

**Documents Reviewed:**
- User Guide iOS (SS-USR-001)
- User Guide Web (SS-USR-002)
- Administrator Guide (SS-ADM-001)
- All Hub documentation (7 files)

**Review Findings:**
- 0 Critical issues
- 0 High-severity issues
- 1 Medium issue (fixed: duplicate document ID)
- 2 Low issues (noted: platform terminology differences)

**Fixes Applied:**
- Changed CONSOLIDATED_INDEX.md document ID from SS-HUB-001 to SS-HUB-002

**Result:** PASS — Documentation suite approved for release.

---

### Documentation Hub Created

**Date:** 3 January 2026

Created the sommelier-spark-docs repository as a central documentation hub.

| Document | Purpose |
|----------|---------|
| README.md | Hub welcome and quick links |
| CONSOLIDATED_INDEX.md | Master inventory of all docs |
| CONTRIBUTING.md | How to contribute to docs |
| shared/GLOSSARY.md | Combined glossary |
| shared/STYLE_GUIDE.md | Documentation standards |
| shared/CHANGELOG.md | This file |
| guides/DOCUMENTATION_MAP.md | Visual doc relationships |

---

### Web App Administrator Guide (SS-ADM-001)

**Date:** 3 January 2026
**Project:** sommelier_spark
**Version:** 1.0

Created comprehensive administrator guide covering:
- Admin roles and permissions
- Organisation/Brand/Location management
- User management (CRUD, suspension)
- Content management (wines, modules, scenarios)
- Scope editing and visibility
- Progress monitoring and reports
- Best practices and FAQs

**Formats:** .md, .docx, .pdf, .html

---

### Web App User Guide (SS-USR-002)

**Date:** 3 January 2026
**Project:** sommelier_spark
**Version:** 1.0 (updated to 2.0 with WS2 release)

Created complete user guide for web application covering:
- Getting started and logging in
- Dashboard navigation (with tier progress cards in v2.0)
- Wine Library with progressive disclosure
- Learning modules and lessons
- 3-tier quiz system (Bronze/Silver/Gold in v2.0)
- Scenarios (Bronze/Silver/Gold)
- Certification journey
- Demo Mode and granular reset (v2.0)
- FAQs and glossary

**Formats:** .md, .docx, .pdf, .html

---

### iOS User Guide (SS-USR-001)

**Date:** January 2026
**Project:** sommelier_spark_ios
**Version:** 1.0

Created complete user guide for iOS application covering:
- App overview and navigation
- Wine Library
- Learning modules
- Quizzes and certifications
- Scenarios
- Settings and reset
- Tips and FAQs

**Formats:** .md, .docx, .pdf

---

### iOS Developer Documentation Suite

**Date:** January 2026
**Project:** sommelier_spark_ios
**Version:** 1.0

Created 7 developer documents:

| Doc ID | Title |
|--------|-------|
| SS-DEV-001 | Developer Setup Guide (Apple Silicon) |
| SS-DEV-002 | Developer Setup Guide (Intel) |
| SS-DEV-003 | iOS Device Configuration Guide |
| SS-DEV-004 | Build & Deploy Runbook |
| SS-DEV-005 | Browser Console Guide |
| SS-DEV-006 | Troubleshooting Guide |
| SS-DEV-007 | Quick Reference Card |

---

### iOS Documentation Index (SS-DOC-INDEX)

**Date:** January 2026
**Project:** sommelier_spark_ios
**Version:** 1.0

Created comprehensive documentation index for iOS project with:
- Document inventory
- Audience guide
- Reading paths
- Cross-reference matrix
- Maintenance schedule

---

## December 2025

### iOS System Documentation

**Project:** sommelier_spark_ios

| Document | Description |
|----------|-------------|
| Quiz Unlock System | Lesson-to-quiz mapping documentation |
| Scenario Unlock System | Certification-based scenario locking |
| Reset System | Settings page and reset functionality |
| Safe Area System | iOS safe area handling |

---

### iOS Specifications

**Project:** sommelier_spark_ios

| Document | Description |
|----------|-------------|
| iOS Single User Specification | Original feature requirements |
| iOS EARS Specification | Formal requirements (~112 items) |

---

### iOS Test Suites

**Project:** sommelier_spark_ios

| Document | Description |
|----------|-------------|
| iOS Test Suite v2 | 162 automated + 57 manual tests |
| Quiz Unlock Test Plan | Quiz-specific tests |
| Manual Test Checklist | Printable checklist |

---

### Web App Content Management Docs

**Project:** sommelier_spark

| Document | Description |
|----------|-------------|
| Wine Import Schema | CSV format for bulk imports |
| Adding Organization Content | Content workflow guide |
| Admin Guide v0.2.3 | Legacy admin reference |

---

## Summary Statistics

### Total Documents by Project

| Project | Count |
|---------|-------|
| sommelier_spark_ios | 35+ |
| sommelier_spark | 14+ (includes WS2 docs) |
| sommelier-spark-docs | 7 |
| WS3.0 Specifications | 12 |
| **Total** | **68+** |

### Documents by Type

| Type | Count |
|------|-------|
| User Guides | 2 |
| Admin Guides | 1 |
| Developer Guides | 7+ |
| System Docs | 4 |
| Specifications | 2 |
| Test Documentation | 5 |
| Indexes | 3 |
| Other | 15+ |

---

*Sommelier Spark Documentation Changelog — January 2026*
