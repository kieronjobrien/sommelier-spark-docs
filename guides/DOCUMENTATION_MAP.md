# Sommelier Spark — Documentation Map

Visual representation of documentation relationships and reading paths.

---

## Ecosystem Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      SOMMELIER SPARK DOCUMENTATION ECOSYSTEM                     │
└─────────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────────┐
                              │  DOCUMENTATION HUB  │
                              │ sommelier-spark-docs│
                              │                     │
                              │  • README           │
                              │  • Consolidated     │
                              │    Index            │
                              │  • Glossary         │
                              │  • Style Guide      │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
                    ▼                    │                    ▼
     ┌──────────────────────┐           │       ┌──────────────────────┐
     │     iOS PROJECT      │           │       │     WEB PROJECT      │
     │  sommelier_spark_ios │           │       │   sommelier_spark    │
     │                      │           │       │                      │
     │  ~/Projects/         │           │       │  ~/Projects/         │
     │  sommelier_spark_ios/│           │       │  sommelier_spark/    │
     └──────────────────────┘           │       └──────────────────────┘
                                        │
                              Shared Concepts:
                              • Certification System
                              • Wine Library
                              • Quizzes & Scenarios
                              • Glossary Terms
```

---

## Project Documentation Structure

### iOS Project (`sommelier_spark_ios/docs/`)

```
docs/
├── README.md                     ← Entry point
├── DOCUMENTATION_INDEX.md        ← Complete inventory
│
├── user/
│   └── User_Guide_iOS.md         ← End user documentation
│
├── setup/                        ← Developer onboarding
│   ├── Developer_Setup_Guide_Apple_Silicon.md
│   ├── Developer_Setup_Guide_Intel.md
│   ├── iOS_Device_Configuration_Guide.md
│   ├── Build_Deploy_Runbook.md
│   ├── Browser_Console_Guide.md
│   ├── Troubleshooting_Guide.md
│   └── Quick_Reference_Card.md
│
├── system/                       ← Architecture documentation
│   ├── Quiz_Unlock_System_Documentation.md
│   ├── Scenario_Unlock_System.md
│   ├── Reset_System_Documentation.md
│   └── Safe_Area_System_Documentation.md
│
├── specs/                        ← Requirements
│   ├── iOS_Single_User_Specification.md
│   └── iOS_EARS_Specification.md
│
├── testing/                      ← Test documentation
│   ├── iOS_Test_Suite_v2.md
│   ├── Quiz_Unlock_Test_Plan.md
│   └── Manual_Test_Checklist.md
│
├── guides/                       ← Implementation guides
│   ├── iOS_Implementation_Guide_v2.0.md
│   └── UX_Fixes_Step_by_Step_Instructions.md
│
├── reports/                      ← Status reports
│   └── UX_Fixes_Completion_Report.md
│
├── prompts/                      ← AI prompts
│   ├── Claude_Code_Prompt_Quiz_Unlock_Option_A.md
│   ├── Claude_Code_App_Prompt.md
│   └── Claude_Code_Tests_Prompt.md
│
└── operations/
    └── Content_Operating_Model.md
```

### Web Project (`sommelier_spark/docs/`)

```
docs/
├── DOCUMENTATION_INDEX.md        ← Entry point
├── README-v0.2.3.md              ← Developer overview
├── M1-SETUP.md                   ← Apple Silicon setup
│
├── user/
│   ├── User_Guide_Web.md         ← End user documentation
│   ├── User_Guide_Web.docx
│   ├── User_Guide_Web.html
│   └── User_Guide_Web.pdf
│
├── admin/
│   ├── Administrator_Guide.md    ← Admin documentation
│   ├── Administrator_Guide.docx
│   ├── Administrator_Guide.html
│   └── Administrator_Guide.pdf
│
├── wine-import-schema.md         ← Technical reference
├── admin-guide-adding-organization-content.md
└── Cross_Platform_Tracking_Requirements.md
```

---

## Reading Paths

### New iOS Developer

```
┌─────────────────────┐
│ 1. Setup Guide      │ (Apple Silicon or Intel)
│    - Install tools  │
│    - Clone repo     │
│    - Verify env     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. Device Config    │
│    - Enable Dev Mode│
│    - Trust settings │
│    - Signing setup  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. Build Runbook    │
│    - Dev workflow   │
│    - Deploy to sim  │
│    - Deploy to device
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 4. Quick Reference  │ ← Bookmark this!
│    - Commands       │
│    - Shortcuts      │
│    - Quick fixes    │
└─────────────────────┘
```

### New Web Developer

```
┌─────────────────────┐
│ 1. README-v0.2.3    │
│    - Tech stack     │
│    - Prerequisites  │
│    - Install steps  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. M1-SETUP         │ (if Apple Silicon)
│    - Specific config│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. CHANGELOG        │
│    - Recent changes │
│    - Version info   │
└─────────────────────┘
```

### New Administrator

```
┌─────────────────────┐
│ 1. Admin Guide      │ (Full read-through)
│    - Roles          │
│    - Org structure  │
│    - User mgmt      │
│    - Content mgmt   │
│    - Reports        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. Wine Import      │
│    Schema           │
│    - CSV format     │
│    - Field specs    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. Adding Org       │
│    Content          │
│    - Workflow       │
│    - Best practices │
└─────────────────────┘
```

### End User (iOS)

```
┌─────────────────────┐
│ 1. Welcome          │ (Sections 1-2)
│    - What is SS     │
│    - Getting started│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. Explore          │ (As needed)
│    - Wine Library   │
│    - Modules        │
│    - Quizzes        │
│    - Scenarios      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. Reference        │
│    - FAQ            │
│    - Glossary       │
│    - Tips           │
└─────────────────────┘
```

### End User (Web)

```
┌─────────────────────┐
│ 1. Welcome          │ (Sections 1-2)
│    - Introduction   │
│    - Logging in     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. Core Features    │
│    - Dashboard      │
│    - Wine Library   │
│    - Training       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. Certification    │
│    - Requirements   │
│    - Quizzes        │
│    - Scenarios      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 4. Reference        │
│    - FAQ            │
│    - Glossary       │
└─────────────────────┘
```

---

## Document Dependencies

### iOS Project Dependencies

```
Setup Guide (Apple Silicon/Intel)
       │
       ├──► iOS Device Configuration
       │           │
       │           ▼
       └──► Build & Deploy Runbook ◄──► Quick Reference Card
                    │
                    ▼
              Troubleshooting Guide
                    │
                    ▼
             Browser Console Guide
```

### Web Project Dependencies

```
README-v0.2.3 ──► M1-SETUP (optional)
     │
     ▼
User Guide Web ◄───────► Administrator Guide
     │                          │
     │                          ▼
     │                   Wine Import Schema
     │                          │
     │                          ▼
     │                   Adding Org Content
     │
     └──────────► Glossary (shared terms)
```

### Cross-Project Dependencies

```
                    ┌─────────────────┐
                    │  Shared Concepts │
                    │                 │
                    │  • Certification│
                    │  • Wine Library │
                    │  • Quizzes      │
                    │  • Scenarios    │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
   │iOS User Guide│  │Web User Guide│  │ Admin Guide  │
   └──────────────┘  └──────────────┘  └──────────────┘
```

---

## Document Boundaries

### What Lives Where

| Content Type | Location | Reason |
|--------------|----------|--------|
| iOS-specific user features | iOS User Guide | Platform-specific |
| Web-specific user features | Web User Guide | Platform-specific |
| Admin features | Admin Guide (Web) | Web-only functionality |
| iOS dev tooling | iOS Setup Docs | Capacitor/Xcode specific |
| Web dev setup | Web README | Next.js/PostgreSQL specific |
| Shared glossary | Hub | Cross-project reference |
| Doc standards | Hub | Applies to all projects |

### Cross-Platform Considerations

| Topic | iOS Approach | Web Approach |
|-------|--------------|--------------|
| Data Storage | localStorage (offline) | PostgreSQL (server) |
| Authentication | None (single user) | NextAuth.js |
| Progress Sync | Device-local only | Server-synced |
| Content Updates | App update required | Real-time |

> **Note:** Progress does not currently sync between iOS and Web platforms. Users should choose one platform for consistency.

---

*Sommelier Spark Documentation Map — January 2026*
