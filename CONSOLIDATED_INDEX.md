# Sommelier Spark — Consolidated Documentation Index

| Document ID | Version | Date | Author |
|-------------|---------|------|--------|
| SS-HUB-002 | 1.0 | January 2026 | Obi Wan |

---

## Table of Contents

1. [Master Document Inventory](#1-master-document-inventory)
2. [Documents by Project](#2-documents-by-project)
3. [Documents by Audience](#3-documents-by-audience)
4. [Reading Paths by Role](#4-reading-paths-by-role)
5. [Document Relationships](#5-document-relationships)
6. [Version Tracking](#6-version-tracking)

---

## 1. Master Document Inventory

### All Documents

| Doc ID | Title | Project | Path | Audience | Version |
|--------|-------|---------|------|----------|---------|
| SS-USR-001 | User Guide (iOS) | iOS | docs/user/User_Guide_iOS.md | End Users | 1.0 |
| SS-USR-002 | User Guide (Web) | Web | docs/user/User_Guide_Web.md | End Users | 2.0 |
| SS-ADM-001 | Administrator Guide | Web | docs/admin/Administrator_Guide.md | Administrators | 1.0 |
| SS-DEV-001 | Developer Setup (Apple Silicon) | iOS | docs/setup/Developer_Setup_Guide_Apple_Silicon.md | Developers | 1.0 |
| SS-DEV-002 | Developer Setup (Intel) | iOS | docs/setup/Developer_Setup_Guide_Intel.md | Developers | 1.0 |
| SS-DEV-003 | iOS Device Configuration | iOS | docs/setup/iOS_Device_Configuration_Guide.md | Developers | 1.0 |
| SS-DEV-004 | Build & Deploy Runbook | iOS | docs/setup/Build_Deploy_Runbook.md | Developers | 1.0 |
| SS-DEV-005 | Browser Console Guide | iOS | docs/setup/Browser_Console_Guide.md | Developers | 1.0 |
| SS-DEV-006 | Troubleshooting Guide | iOS | docs/setup/Troubleshooting_Guide.md | Developers | 1.0 |
| SS-DEV-007 | Quick Reference Card | iOS | docs/setup/Quick_Reference_Card.md | Developers | 1.0 |
| SS-DOC-INDEX | iOS Documentation Index | iOS | docs/DOCUMENTATION_INDEX.md | All | 1.0 |
| SS-HUB-001 | Documentation Hub README | Hub | README.md | All | 1.0 |
| SS-HUB-002 | Consolidated Index | Hub | CONSOLIDATED_INDEX.md | All | 1.0 |
| — | Web Documentation Index | Web | docs/DOCUMENTATION_INDEX.md | All | — |
| — | Web README | Web | docs/README-v0.2.3.md | Developers | 0.2.3 |
| — | M1 Setup Guide | Web | docs/M1-SETUP.md | Developers | — |
| — | Wine Import Schema | Web | docs/wine-import-schema.md | Administrators | — |
| — | Adding Org Content | Web | docs/admin-guide-adding-organization-content.md | Administrators | — |
| — | Cross-Platform Requirements | Web | docs/Cross_Platform_Tracking_Requirements.md | Developers | — |

### iOS System Documentation

| Doc ID | Title | Path | Purpose |
|--------|-------|------|---------|
| — | Quiz Unlock System | docs/system/Quiz_Unlock_System_Documentation.md | Lesson-to-quiz mapping |
| — | Scenario Unlock System | docs/system/Scenario_Unlock_System.md | Certification-based scenario locking |
| — | Reset System | docs/system/Reset_System_Documentation.md | Settings and reset functionality |
| — | Safe Area System | docs/system/Safe_Area_System_Documentation.md | iOS safe area handling |

### iOS Specifications

| Doc ID | Title | Path | Purpose |
|--------|-------|------|---------|
| — | iOS Single User Specification | docs/specs/iOS_Single_User_Specification.md | Original feature requirements |
| — | iOS EARS Specification | docs/specs/iOS_EARS_Specification.md | Formal EARS requirements (~112 items) |

### iOS Testing Documentation

| Doc ID | Title | Path | Purpose |
|--------|-------|------|---------|
| — | iOS Test Suite v2 | docs/testing/iOS_Test_Suite_v2.md | 162 automated + 57 manual tests |
| — | Quiz Unlock Test Plan | docs/testing/Quiz_Unlock_Test_Plan.md | Quiz-specific tests |
| — | Manual Test Checklist | docs/testing/Manual_Test_Checklist.md | Printable checklist |

### iOS Guides and Reports

| Doc ID | Title | Path | Purpose |
|--------|-------|------|---------|
| — | iOS Implementation Guide v2.0 | docs/guides/iOS_Implementation_Guide_v2.0.md | Step-by-step build instructions |
| — | UX Fixes Step-by-Step | docs/guides/UX_Fixes_Step_by_Step_Instructions.md | 7-phase implementation guide |
| — | UX Fixes Completion Report | docs/reports/UX_Fixes_Completion_Report.md | 8 issues resolved |
| — | Content Operating Model | docs/operations/Content_Operating_Model.md | Content management process |

### iOS Prompts (Claude Code)

| Title | Path | Purpose |
|-------|------|---------|
| Quiz Unlock Prompt | docs/prompts/Claude_Code_Prompt_Quiz_Unlock_Option_A.md | Reusable prompt |
| App Build Prompt | docs/prompts/Claude_Code_App_Prompt.md | Full app build |
| Tests Prompt | docs/prompts/Claude_Code_Tests_Prompt.md | Test generation |

---

## 2. Documents by Project

### iOS Project (`sommelier_spark_ios`)

**Location:** `~/Projects/sommelier_spark_ios/docs/`

| Category | Count | Key Documents |
|----------|-------|---------------|
| Setup Guides | 7 | Developer Setup, Device Config, Build Runbook |
| User Docs | 1 | User Guide iOS |
| System Docs | 4 | Quiz Unlock, Scenario Unlock, Reset, Safe Area |
| Specifications | 2 | Single User Spec, EARS Specification |
| Testing | 5 | Test Suite v2, Test Plans, Checklists |
| Guides | 2 | Implementation Guide, UX Fixes |
| Reports | 2 | UX Fixes Report, Changes Report |
| Prompts | 3 | Claude Code prompts |
| **Total** | **26+** | |

### Web Project (`sommelier_spark`)

**Location:** `~/Projects/sommelier_spark/docs/`

| Category | Count | Key Documents |
|----------|-------|---------------|
| User Docs | 1 | User Guide Web (v2.0) |
| Admin Docs | 3 | Administrator Guide, Adding Org Content, Wine Import |
| Developer Docs | 2 | README, M1 Setup |
| Technical | 1 | Cross-Platform Requirements |
| WS2 Docs | 4 | Feature Gap Analysis, Sync Prioritization, Verification Report, Completion Summary |
| Index | 1 | Documentation Index |
| **Total** | **12+** | |

### Documentation Hub (`sommelier-spark-docs`)

**Location:** `~/Projects/sommelier_spark_docs/`

| Category | Count | Documents |
|----------|-------|-----------|
| Index | 2 | README, Consolidated Index |
| Shared | 3 | Glossary, Style Guide, Changelog |
| Guides | 1 | Documentation Map |
| Meta | 1 | Contributing |
| **Total** | **7** | |

---

## 3. Documents by Audience

### End Users (Staff)

| Document | Platform | Format(s) |
|----------|----------|-----------|
| User Guide iOS | iOS | .md, .docx, .pdf |
| User Guide Web | Web | .md, .docx, .pdf, .html |

### Administrators

| Document | Focus | Format(s) |
|----------|-------|-----------|
| Administrator Guide | User/content management | .md, .docx, .pdf, .html |
| Wine Import Schema | CSV bulk import | .md |
| Adding Org Content | Content workflow | .md |

### Developers (iOS)

| Document | Purpose | Priority |
|----------|---------|----------|
| Developer Setup (Apple Silicon/Intel) | Environment setup | Day 1 |
| iOS Device Configuration | Device prep | Day 1 |
| Build & Deploy Runbook | Daily workflow | Daily |
| Quick Reference Card | Command lookup | Daily |
| Troubleshooting Guide | Problem solving | As needed |
| Browser Console Guide | Debugging | As needed |

### Developers (Web)

| Document | Purpose | Priority |
|----------|---------|----------|
| README-v0.2.3.md | Project overview | Day 1 |
| M1-SETUP.md | Apple Silicon setup | Day 1 |

---

## 4. Reading Paths by Role

### New iOS Developer — First Day

```
1. Developer Setup Guide (Apple Silicon or Intel)
   ↓
2. iOS Device Configuration Guide
   ↓
3. Build & Deploy Runbook
   ↓
4. Quick Reference Card (bookmark this!)
```

### Experienced iOS Developer — Daily Reference

```
Quick Reference Card ←→ Build & Deploy Runbook
                              ↓
                     Troubleshooting Guide
                              ↓
                     Browser Console Guide
```

### New Web Developer — First Day

```
1. README-v0.2.3.md
   ↓
2. M1-SETUP.md (if Apple Silicon)
   ↓
3. CHANGELOG.md
```

### New Administrator — First Week

```
1. Administrator Guide (all sections)
   ↓
2. Wine Import Schema
   ↓
3. Adding Organization Content
```

### End User — Getting Started

```
iOS: User Guide iOS (Section 1-2, then explore)
Web: User Guide Web (Section 1-2, then explore)
```

---

## 5. Document Relationships

### Cross-Project Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SOMMELIER SPARK ECOSYSTEM                            │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────┐     ┌──────────────────────────┐
  │    sommelier_spark_ios   │     │     sommelier_spark      │
  │         (iOS App)        │     │       (Web App)          │
  │                          │     │                          │
  │  ┌────────────────────┐  │     │  ┌────────────────────┐  │
  │  │  User Guide iOS    │  │     │  │  User Guide Web    │  │
  │  │    (SS-USR-001)    │  │     │  │    (SS-USR-002)    │  │
  │  └────────────────────┘  │     │  └────────────────────┘  │
  │           │              │     │           │              │
  │  ┌────────────────────┐  │     │  ┌────────────────────┐  │
  │  │  Developer Docs    │  │     │  │  Administrator     │  │
  │  │   (SS-DEV-001-007) │  │     │  │    Guide           │  │
  │  └────────────────────┘  │     │  │    (SS-ADM-001)    │  │
  │           │              │     │  └────────────────────┘  │
  │  ┌────────────────────┐  │     │           │              │
  │  │  System Docs       │  │     │  ┌────────────────────┐  │
  │  │  Specs & Tests     │  │     │  │  Developer Docs    │  │
  │  └────────────────────┘  │     │  │  Schemas           │  │
  │                          │     │  └────────────────────┘  │
  └──────────────────────────┘     └──────────────────────────┘
               │                               │
               │                               │
               └───────────────┬───────────────┘
                               │
                               ▼
                 ┌──────────────────────────┐
                 │  sommelier-spark-docs    │
                 │    (Documentation Hub)   │
                 │                          │
                 │  • Consolidated Index    │
                 │  • Shared Glossary       │
                 │  • Style Guide           │
                 │  • Doc Map               │
                 └──────────────────────────┘
```

### Shared Concepts

| Concept | iOS Docs | Web Docs |
|---------|----------|----------|
| Certification System | User Guide, System Docs | User Guide, Admin Guide |
| Wine Library | User Guide | User Guide |
| Quizzes & Levels | User Guide, Quiz Unlock | User Guide |
| Scenarios | User Guide, Scenario Unlock | User Guide |
| Glossary Terms | User Guide | User Guide, Admin Guide |

---

## 6. Version Tracking

### Current Document Versions

| Doc ID | Title | Version | Last Updated |
|--------|-------|---------|--------------|
| SS-USR-001 | User Guide iOS | 1.0 | Jan 2026 |
| SS-USR-002 | User Guide Web | 1.0 | Jan 2026 |
| SS-ADM-001 | Administrator Guide | 1.0 | Jan 2026 |
| SS-DEV-001 | Developer Setup (Apple Silicon) | 1.0 | Jan 2026 |
| SS-DEV-002 | Developer Setup (Intel) | 1.0 | Jan 2026 |
| SS-DEV-003 | iOS Device Configuration | 1.0 | Jan 2026 |
| SS-DEV-004 | Build & Deploy Runbook | 1.0 | Jan 2026 |
| SS-DEV-005 | Browser Console Guide | 1.0 | Jan 2026 |
| SS-DEV-006 | Troubleshooting Guide | 1.0 | Jan 2026 |
| SS-DEV-007 | Quick Reference Card | 1.0 | Jan 2026 |
| SS-DOC-INDEX | iOS Documentation Index | 1.0 | Jan 2026 |
| SS-HUB-001 | Documentation Hub README | 1.0 | Jan 2026 |
| SS-HUB-002 | Consolidated Index | 1.0 | Jan 2026 |

### Application Versions

| Application | Current Version | Docs Updated |
|-------------|-----------------|--------------|
| Sommelier Spark iOS | 2.0.0 | Jan 2026 |
| Sommelier Spark Web | 2.0.0 | Jan 2026 |

### Documentation History

| Date | Changes |
|------|---------|
| Jan 2026 | **WS2 Release:** Web v2.0.0, iOS parity achieved, User Guide updated to v2.0, WS2 documentation suite (4 docs) |
| Jan 2026 | Web User Guide, Administrator Guide, Documentation Hub created |
| Jan 2026 | iOS Developer docs suite (7 documents) |
| Jan 2026 | iOS User Guide created |
| Jan 2025 | iOS System docs, Test suites |
| Dec 2025 | Initial iOS specs and implementation guides |

---

## Document Count Summary

| Project | Markdown | Word | PDF | HTML | Total Files |
|---------|----------|------|-----|------|-------------|
| iOS | 35 | 10 | 2 | 1 | 48 |
| Web | 10 | 3 | 2 | 2 | 17 |
| Hub | 7 | 0 | 0 | 0 | 7 |
| **Total** | **52** | **13** | **4** | **3** | **72** |

---

*SS-HUB-002 — Sommelier Spark Consolidated Documentation Index v1.0 — January 2026*
