# Low-Level Design — Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3-LLD |
| **Version** | 1.0 |
| **Date** | 2026-01-20 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Classification** | CONFIDENTIAL |
| **Related Documents** | SS-WS3-HLD, SS-WS3-EARS, SS-WS3-PRD |

---

## CONFIDENTIALITY NOTICE

This document contains proprietary and confidential information including patent-pending algorithms for the Learning Content Engine. Distribution is restricted to authorised personnel only.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Core API Service Design](#2-core-api-service-design)
3. [Learning Engine Service Design](#3-learning-engine-service-design)
4. [Content Service Design](#4-content-service-design)
5. [Authentication Service Design](#5-authentication-service-design)
6. [Notification Service Design](#6-notification-service-design)
7. [Database Design](#7-database-design)
8. [Caching Design](#8-caching-design)
9. [Queue Design](#9-queue-design)
10. [Error Handling](#10-error-handling)
11. [Testing Strategy](#11-testing-strategy)
12. [Appendices](#12-appendices)

---

## 1. Introduction

### 1.1 Purpose and Scope

This Low-Level Design (LLD) document provides detailed implementation specifications for the Sommelier Spark platform. It translates the High-Level Design (SS-WS3-HLD) into actionable development guidance including:

- Directory structures and code organisation
- API endpoint specifications with request/response schemas
- Service layer interfaces and implementations
- Database schemas with indexes and constraints
- Algorithm pseudocode for patent-pending components
- Error handling patterns and codes
- Testing strategies and coverage requirements

### 1.2 Relationship to HLD

| HLD Section | LLD Coverage |
|-------------|--------------|
| Container Architecture | Service directory structures, deployment configs |
| Component Architecture | Class designs, interface definitions |
| Data Architecture | Full database schemas, indexes, migrations |
| Integration Architecture | API specifications, event schemas |
| Security Architecture | Auth implementation, middleware code |

### 1.3 Conventions

| Convention | Description |
|------------|-------------|
| **TypeScript** | Node.js services use TypeScript with strict mode |
| **Python** | Learning Engine uses Python 3.11 with type hints |
| **Naming** | camelCase for code, snake_case for database |
| **IDs** | UUIDs for all entity identifiers |
| **Dates** | ISO 8601 format, stored as UTC |

---

## 2. Core API Service Design

### 2.1 Directory Structure

```
src/
├── controllers/
│   ├── auth.controller.ts
│   ├── user.controller.ts
│   ├── organisation.controller.ts
│   ├── wine.controller.ts
│   ├── module.controller.ts
│   ├── quiz.controller.ts
│   ├── scenario.controller.ts
│   ├── progress.controller.ts
│   ├── import.controller.ts
│   └── report.controller.ts
├── services/
│   ├── auth.service.ts
│   ├── content.service.ts
│   ├── workflow.service.ts
│   ├── generation.service.ts
│   ├── notification.service.ts
│   └── audit.service.ts
├── repositories/
│   ├── base.repository.ts
│   ├── user.repository.ts
│   ├── organisation.repository.ts
│   ├── wine.repository.ts
│   ├── module.repository.ts
│   ├── quiz.repository.ts
│   ├── scenario.repository.ts
│   ├── progress.repository.ts
│   └── audit.repository.ts
├── middleware/
│   ├── auth.middleware.ts
│   ├── tenant.middleware.ts
│   ├── validation.middleware.ts
│   ├── rate-limit.middleware.ts
│   └── audit.middleware.ts
├── models/
│   ├── user.model.ts
│   ├── organisation.model.ts
│   ├── wine.model.ts
│   ├── module.model.ts
│   ├── quiz.model.ts
│   ├── scenario.model.ts
│   └── progress.model.ts
├── dto/
│   ├── request/
│   │   ├── create-wine.dto.ts
│   │   ├── update-wine.dto.ts
│   │   └── ...
│   └── response/
│       ├── wine.response.ts
│       ├── paginated.response.ts
│       └── ...
├── validators/
│   ├── wine.validator.ts
│   ├── user.validator.ts
│   └── ...
├── utils/
│   ├── errors.ts
│   ├── logger.ts
│   ├── response.ts
│   └── pagination.ts
├── config/
│   ├── database.ts
│   ├── redis.ts
│   ├── queue.ts
│   └── app.ts
├── types/
│   ├── express.d.ts
│   ├── enums.ts
│   └── interfaces.ts
├── app.ts
└── server.ts
```

### 2.2 Controller Specifications

#### 2.2.1 Wine Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/wines | listWines | User | List wines (paginated) |
| GET | /api/v1/wines/:id | getWine | User | Get wine by ID |
| POST | /api/v1/wines | createWine | Author | Create wine |
| PUT | /api/v1/wines/:id | updateWine | Author | Update wine |
| DELETE | /api/v1/wines/:id | deleteWine | Admin | Soft delete wine |
| POST | /api/v1/wines/:id/submit | submitWine | Author | Submit for review |
| POST | /api/v1/wines/:id/approve | approveWine | Expert | Approve wine |
| POST | /api/v1/wines/:id/publish | publishWine | Admin | Publish wine |
| POST | /api/v1/wines/:id/archive | archiveWine | Admin | Archive wine |

**Request DTOs:**

```typescript
// dto/request/create-wine.dto.ts
interface CreateWineDto {
  name: string;                    // Required, 1-255 chars
  producer?: string;               // Optional, 1-255 chars
  vintage?: number;                // Optional, 1900-current year
  region: string;                  // Required, 1-100 chars
  country: string;                 // Required, 2-char ISO code
  wineType: WineType;              // Required, enum
  grapeVarieties: string[];        // Required, 1-10 items
  priceTier: PriceTier;            // Required, enum
  price?: number;                  // Optional, > 0
  tastingNotes?: string;           // Optional, max 2000 chars
  foodPairings?: string[];         // Optional, 0-20 items
  servingTemperature?: string;     // Optional, e.g., "16-18°C"
  decantingTime?: string;          // Optional, e.g., "2-3 hours"
}

// dto/request/update-wine.dto.ts
interface UpdateWineDto extends Partial<CreateWineDto> {
  // All fields optional for partial updates
}

// dto/request/list-wines.dto.ts
interface ListWinesQuery {
  page?: number;                   // Default: 1
  pageSize?: number;               // Default: 20, max: 100
  sort?: string;                   // e.g., "name:asc", "createdAt:desc"
  status?: ContentStatus;          // Filter by status
  wineType?: WineType;             // Filter by type
  region?: string;                 // Filter by region
  search?: string;                 // Full-text search
}
```

**Response DTOs:**

```typescript
// dto/response/wine.response.ts
interface WineResponse {
  id: string;
  name: string;
  producer: string | null;
  vintage: number | null;
  region: string;
  country: string;
  wineType: WineType;
  grapeVarieties: string[];
  priceTier: PriceTier;
  price: number | null;
  tastingNotes: string | null;
  foodPairings: string[];
  servingTemperature: string | null;
  decantingTime: string | null;
  status: ContentStatus;
  version: number;
  createdAt: string;
  updatedAt: string;
  createdBy: UserSummary;
  _links: {
    self: string;
    module?: string;
  };
}

// dto/response/paginated.response.ts
interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    pageSize: number;
    totalPages: number;
    totalItems: number;
  };
  links: {
    self: string;
    first: string;
    prev: string | null;
    next: string | null;
    last: string;
  };
}
```

**Controller Implementation:**

```typescript
// controllers/wine.controller.ts
import { Router, Request, Response, NextFunction } from 'express';
import { WineService } from '../services/wine.service';
import { validateRequest } from '../middleware/validation.middleware';
import { requireAuth, requireRole } from '../middleware/auth.middleware';
import { createWineSchema, updateWineSchema, listWinesSchema } from '../validators/wine.validator';

export class WineController {
  private router: Router;
  private wineService: WineService;

  constructor(wineService: WineService) {
    this.router = Router();
    this.wineService = wineService;
    this.initRoutes();
  }

  private initRoutes(): void {
    this.router.get(
      '/',
      requireAuth,
      validateRequest(listWinesSchema, 'query'),
      this.listWines.bind(this)
    );

    this.router.get(
      '/:id',
      requireAuth,
      this.getWine.bind(this)
    );

    this.router.post(
      '/',
      requireAuth,
      requireRole(['CONTENT_AUTHOR', 'CONTENT_ADMIN', 'ORG_ADMIN']),
      validateRequest(createWineSchema),
      this.createWine.bind(this)
    );

    this.router.put(
      '/:id',
      requireAuth,
      requireRole(['CONTENT_AUTHOR', 'CONTENT_ADMIN', 'ORG_ADMIN']),
      validateRequest(updateWineSchema),
      this.updateWine.bind(this)
    );

    this.router.delete(
      '/:id',
      requireAuth,
      requireRole(['CONTENT_ADMIN', 'ORG_ADMIN']),
      this.deleteWine.bind(this)
    );

    this.router.post(
      '/:id/submit',
      requireAuth,
      requireRole(['CONTENT_AUTHOR', 'CONTENT_ADMIN']),
      this.submitWine.bind(this)
    );

    this.router.post(
      '/:id/approve',
      requireAuth,
      requireRole(['DOMAIN_EXPERT', 'CONTENT_ADMIN']),
      this.approveWine.bind(this)
    );

    this.router.post(
      '/:id/publish',
      requireAuth,
      requireRole(['CONTENT_ADMIN', 'ORG_ADMIN']),
      this.publishWine.bind(this)
    );
  }

  async listWines(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const result = await this.wineService.listWines(
        req.query as ListWinesQuery,
        req.tenantContext
      );
      res.json(result);
    } catch (error) {
      next(error);
    }
  }

  async getWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.getWine(
        req.params.id,
        req.tenantContext
      );
      res.json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  async createWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.createWine(
        req.body,
        req.tenantContext
      );
      res.status(201).json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  async updateWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.updateWine(
        req.params.id,
        req.body,
        req.tenantContext
      );
      res.json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  async deleteWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      await this.wineService.deleteWine(req.params.id, req.tenantContext);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }

  async submitWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.submitForReview(
        req.params.id,
        req.tenantContext
      );
      res.json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  async approveWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.approve(
        req.params.id,
        req.body.feedback,
        req.tenantContext
      );
      res.json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  async publishWine(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const wine = await this.wineService.publish(
        req.params.id,
        req.tenantContext
      );
      res.json({ data: wine });
    } catch (error) {
      next(error);
    }
  }

  getRouter(): Router {
    return this.router;
  }
}
```

#### 2.2.2 User Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/users | listUsers | Manager | List users |
| GET | /api/v1/users/:id | getUser | Manager | Get user by ID |
| GET | /api/v1/users/me | getCurrentUser | User | Get current user |
| POST | /api/v1/users | createUser | Admin | Create user |
| PUT | /api/v1/users/:id | updateUser | Admin | Update user |
| DELETE | /api/v1/users/:id | deleteUser | Admin | Remove user |
| POST | /api/v1/users/invite | inviteUser | Manager | Invite user |
| POST | /api/v1/users/:id/suspend | suspendUser | Admin | Suspend user |
| POST | /api/v1/users/:id/reactivate | reactivateUser | Admin | Reactivate user |

#### 2.2.3 Organisation Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/organisations | listOrganisations | SysAdmin | List all orgs |
| GET | /api/v1/organisations/:id | getOrganisation | OrgAdmin | Get org by ID |
| GET | /api/v1/organisations/current | getCurrentOrg | User | Get current org |
| PUT | /api/v1/organisations/:id | updateOrganisation | OrgAdmin | Update org |
| GET | /api/v1/organisations/:id/brands | listBrands | Manager | List brands |
| POST | /api/v1/organisations/:id/brands | createBrand | OrgAdmin | Create brand |
| GET | /api/v1/brands/:id/locations | listLocations | Manager | List locations |
| POST | /api/v1/brands/:id/locations | createLocation | Manager | Create location |

#### 2.2.4 Module Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/modules | listModules | User | List modules |
| GET | /api/v1/modules/:id | getModule | User | Get module |
| GET | /api/v1/modules/:id/lessons | listLessons | User | List lessons |
| GET | /api/v1/lessons/:id | getLesson | User | Get lesson |
| POST | /api/v1/modules | createModule | Author | Create module |
| PUT | /api/v1/modules/:id | updateModule | Author | Update module |
| POST | /api/v1/modules/:id/lessons | createLesson | Author | Create lesson |
| PUT | /api/v1/lessons/:id | updateLesson | Author | Update lesson |

#### 2.2.5 Quiz Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/quizzes | listQuizzes | User | List quizzes |
| GET | /api/v1/quizzes/:id | getQuiz | User | Get quiz |
| POST | /api/v1/quizzes/:id/attempts | startAttempt | User | Start quiz attempt |
| GET | /api/v1/attempts/:id | getAttempt | User | Get attempt |
| POST | /api/v1/attempts/:id/answers | submitAnswer | User | Submit answer |
| POST | /api/v1/attempts/:id/submit | submitAttempt | User | Complete attempt |
| GET | /api/v1/attempts/:id/results | getResults | User | Get results |

#### 2.2.6 Scenario Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/scenarios | listScenarios | User | List scenarios |
| GET | /api/v1/scenarios/:id | getScenario | User | Get scenario |
| POST | /api/v1/scenarios/:id/attempts | startAttempt | User | Start scenario |
| POST | /api/v1/scenario-attempts/:id/choices | submitChoice | User | Submit choice |
| POST | /api/v1/scenario-attempts/:id/complete | completeAttempt | User | Complete scenario |

#### 2.2.7 Progress Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/progress | getMyProgress | User | Get own progress |
| GET | /api/v1/progress/users/:id | getUserProgress | Manager | Get user progress |
| GET | /api/v1/progress/summary | getProgressSummary | Manager | Team summary |
| POST | /api/v1/progress/lessons/:id/complete | completeLesson | User | Mark complete |
| GET | /api/v1/certifications | getMyCertifications | User | Get certifications |

#### 2.2.8 Import Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| POST | /api/v1/imports/wines | uploadWineImport | Manager | Upload CSV |
| GET | /api/v1/imports/:id | getImportStatus | Manager | Get import status |
| GET | /api/v1/imports/:id/preview | getImportPreview | Manager | Preview import |
| POST | /api/v1/imports/:id/confirm | confirmImport | Manager | Confirm import |
| POST | /api/v1/imports/:id/cancel | cancelImport | Manager | Cancel import |
| GET | /api/v1/imports | listImports | Manager | List imports |

#### 2.2.9 Report Controller

**Route Definitions:**

| Method | Path | Handler | Auth | Description |
|--------|------|---------|------|-------------|
| GET | /api/v1/reports/progress | getProgressReport | Manager | Progress report |
| GET | /api/v1/reports/certification | getCertificationReport | Manager | Cert report |
| GET | /api/v1/reports/engagement | getEngagementReport | Manager | Engagement report |
| GET | /api/v1/reports/content | getContentReport | Admin | Content report |
| POST | /api/v1/reports/export | exportReport | Manager | Export to CSV/PDF |

### 2.3 Service Layer Design

#### 2.3.1 Content Service Interface

```typescript
// services/content.service.ts
interface ContentService {
  // Wine operations
  createWine(dto: CreateWineDto, context: TenantContext): Promise<Wine>;
  updateWine(id: string, dto: UpdateWineDto, context: TenantContext): Promise<Wine>;
  getWine(id: string, context: TenantContext): Promise<Wine>;
  listWines(query: ListWinesQuery, context: TenantContext): Promise<PaginatedResult<Wine>>;
  deleteWine(id: string, context: TenantContext): Promise<void>;

  // State transitions
  submitForReview(id: string, context: TenantContext): Promise<Wine>;
  approve(id: string, feedback: string, context: TenantContext): Promise<Wine>;
  reject(id: string, feedback: string, context: TenantContext): Promise<Wine>;
  publish(id: string, context: TenantContext): Promise<Wine>;
  archive(id: string, context: TenantContext): Promise<Wine>;

  // Module operations
  createModule(dto: CreateModuleDto, context: TenantContext): Promise<Module>;
  updateModule(id: string, dto: UpdateModuleDto, context: TenantContext): Promise<Module>;
  getModule(id: string, context: TenantContext): Promise<Module>;
  listModules(query: ListModulesQuery, context: TenantContext): Promise<PaginatedResult<Module>>;

  // Lesson operations
  createLesson(moduleId: string, dto: CreateLessonDto, context: TenantContext): Promise<Lesson>;
  updateLesson(id: string, dto: UpdateLessonDto, context: TenantContext): Promise<Lesson>;
  getLesson(id: string, context: TenantContext): Promise<Lesson>;
}
```

**Implementation:**

```typescript
// services/content.service.impl.ts
export class ContentServiceImpl implements ContentService {
  constructor(
    private wineRepo: WineRepository,
    private moduleRepo: ModuleRepository,
    private lessonRepo: LessonRepository,
    private workflowService: WorkflowService,
    private auditService: AuditService,
    private cacheService: CacheService
  ) {}

  async createWine(dto: CreateWineDto, context: TenantContext): Promise<Wine> {
    // Validate unique name within organisation
    const existing = await this.wineRepo.findByName(
      dto.name,
      dto.vintage,
      context.organisationId
    );
    if (existing) {
      throw new ConflictError('Wine with this name and vintage already exists');
    }

    // Create wine with DRAFT status
    const wine = await this.wineRepo.create({
      ...dto,
      organisationId: context.organisationId,
      status: ContentStatus.DRAFT,
      version: 1,
      createdBy: context.userId,
      updatedBy: context.userId
    });

    // Audit log
    await this.auditService.log({
      action: AuditAction.CREATED,
      resourceType: 'wine',
      resourceId: wine.id,
      userId: context.userId,
      organisationId: context.organisationId,
      newState: wine
    });

    return wine;
  }

  async submitForReview(id: string, context: TenantContext): Promise<Wine> {
    const wine = await this.getWine(id, context);

    // Validate state transition
    if (wine.status !== ContentStatus.DRAFT) {
      throw new ConflictError('Only draft wines can be submitted for review');
    }

    // Validate content completeness
    this.validateWineCompleteness(wine);

    // Execute transition
    const updated = await this.workflowService.transition(
      wine,
      WorkflowAction.SUBMIT,
      context
    );

    // Invalidate cache
    await this.cacheService.invalidateWine(id, context.organisationId);

    return updated;
  }

  private validateWineCompleteness(wine: Wine): void {
    const errors: string[] = [];

    if (!wine.name) errors.push('Name is required');
    if (!wine.region) errors.push('Region is required');
    if (!wine.country) errors.push('Country is required');
    if (!wine.wineType) errors.push('Wine type is required');
    if (!wine.grapeVarieties?.length) errors.push('At least one grape variety is required');

    if (errors.length > 0) {
      throw new ValidationError('Wine is incomplete', { fields: errors });
    }
  }
}
```

#### 2.3.2 Workflow Service Interface

```typescript
// services/workflow.service.ts
interface WorkflowService {
  transition(
    content: Content,
    action: WorkflowAction,
    context: TenantContext
  ): Promise<Content>;

  getAvailableActions(
    content: Content,
    context: TenantContext
  ): Promise<WorkflowAction[]>;

  assignReviewer(
    contentId: string,
    reviewerId: string,
    context: TenantContext
  ): Promise<void>;
}

enum WorkflowAction {
  SUBMIT = 'submit',
  APPROVE = 'approve',
  REJECT = 'reject',
  PUBLISH = 'publish',
  ARCHIVE = 'archive',
  RESTORE = 'restore',
  REVISE = 'revise'
}
```

**Implementation:**

```typescript
// services/workflow.service.impl.ts
export class WorkflowServiceImpl implements WorkflowService {
  private transitions: TransitionMap = {
    [ContentStatus.DRAFT]: {
      [WorkflowAction.SUBMIT]: {
        target: ContentStatus.REVIEW,
        roles: [UserRole.CONTENT_AUTHOR, UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.REVIEW]: {
      [WorkflowAction.APPROVE]: {
        target: ContentStatus.APPROVED,
        roles: [UserRole.DOMAIN_EXPERT, UserRole.CONTENT_ADMIN]
      },
      [WorkflowAction.REJECT]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.DOMAIN_EXPERT, UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.APPROVED]: {
      [WorkflowAction.PUBLISH]: {
        target: ContentStatus.PUBLISHED,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN]
      },
      [WorkflowAction.REJECT]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.PUBLISHED]: {
      [WorkflowAction.ARCHIVE]: {
        target: ContentStatus.ARCHIVED,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN]
      },
      [WorkflowAction.REVISE]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_AUTHOR, UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.ARCHIVED]: {
      [WorkflowAction.RESTORE]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN]
      }
    }
  };

  async transition(
    content: Content,
    action: WorkflowAction,
    context: TenantContext
  ): Promise<Content> {
    // 1. Get transition definition
    const transition = this.transitions[content.status]?.[action];
    if (!transition) {
      throw new ConflictError(
        `Action '${action}' not allowed from status '${content.status}'`
      );
    }

    // 2. Check permission
    if (!transition.roles.includes(context.userRole)) {
      throw new ForbiddenError(
        `Role '${context.userRole}' cannot perform '${action}'`
      );
    }

    // 3. Execute pre-transition hooks
    await this.executePreHooks(content, action, context);

    // 4. Update status
    const updated = await this.contentRepo.updateStatus(
      content.id,
      transition.target,
      context
    );

    // 5. Execute post-transition hooks
    await this.executePostHooks(updated, action, context);

    // 6. Send notifications
    await this.notifyTransition(updated, action, context);

    // 7. Create audit entry
    await this.auditService.log({
      action: `workflow.${action}`,
      resourceType: content.type,
      resourceId: content.id,
      userId: context.userId,
      organisationId: context.organisationId,
      previousState: { status: content.status },
      newState: { status: updated.status }
    });

    return updated;
  }
}
```

#### 2.3.3 Generation Service Interface

```typescript
// services/generation.service.ts
interface GenerationService {
  generateCurriculum(
    organisationId: string,
    options?: CurriculumOptions
  ): Promise<GenerationJob>;

  generateQuiz(
    moduleId: string,
    tier: Tier,
    options?: QuizOptions
  ): Promise<GenerationJob>;

  generateScenarios(
    organisationId: string,
    tier: Tier,
    count: number
  ): Promise<GenerationJob>;

  getJobStatus(jobId: string): Promise<GenerationJob>;

  cancelJob(jobId: string): Promise<void>;
}

interface GenerationJob {
  id: string;
  type: GenerationType;
  status: JobStatus;
  organisationId: string;
  progress: number;
  result?: GenerationResult;
  error?: string;
  createdAt: Date;
  completedAt?: Date;
}
```

### 2.4 Repository Pattern

#### 2.4.1 Base Repository

```typescript
// repositories/base.repository.ts
interface BaseRepository<T> {
  findById(id: string, tenantId: string): Promise<T | null>;
  findAll(
    filters: Record<string, any>,
    pagination: Pagination,
    tenantId: string
  ): Promise<PaginatedResult<T>>;
  create(entity: Partial<T>, tenantId: string): Promise<T>;
  update(id: string, entity: Partial<T>, tenantId: string): Promise<T>;
  softDelete(id: string, tenantId: string): Promise<void>;
  hardDelete(id: string, tenantId: string): Promise<void>;
  count(filters: Record<string, any>, tenantId: string): Promise<number>;
}

abstract class BaseRepositoryImpl<T> implements BaseRepository<T> {
  constructor(
    protected db: Database,
    protected tableName: string
  ) {}

  async findById(id: string, tenantId: string): Promise<T | null> {
    const result = await this.db.query(
      `SELECT * FROM ${this.tableName}
       WHERE id = $1 AND organisation_id = $2 AND deleted_at IS NULL`,
      [id, tenantId]
    );
    return result.rows[0] || null;
  }

  async findAll(
    filters: Record<string, any>,
    pagination: Pagination,
    tenantId: string
  ): Promise<PaginatedResult<T>> {
    const { where, params } = this.buildWhereClause(filters, tenantId);
    const offset = (pagination.page - 1) * pagination.pageSize;

    const countQuery = `SELECT COUNT(*) FROM ${this.tableName} ${where}`;
    const countResult = await this.db.query(countQuery, params);
    const totalItems = parseInt(countResult.rows[0].count, 10);

    const dataQuery = `
      SELECT * FROM ${this.tableName}
      ${where}
      ORDER BY ${pagination.sort || 'created_at DESC'}
      LIMIT $${params.length + 1} OFFSET $${params.length + 2}
    `;
    const dataResult = await this.db.query(
      dataQuery,
      [...params, pagination.pageSize, offset]
    );

    return {
      data: dataResult.rows,
      meta: {
        page: pagination.page,
        pageSize: pagination.pageSize,
        totalPages: Math.ceil(totalItems / pagination.pageSize),
        totalItems
      }
    };
  }

  async create(entity: Partial<T>, tenantId: string): Promise<T> {
    const entityWithTenant = {
      ...entity,
      id: uuid(),
      organisation_id: tenantId,
      created_at: new Date(),
      updated_at: new Date()
    };

    const { columns, values, placeholders } = this.buildInsert(entityWithTenant);

    const result = await this.db.query(
      `INSERT INTO ${this.tableName} (${columns})
       VALUES (${placeholders})
       RETURNING *`,
      values
    );

    return result.rows[0];
  }

  async update(id: string, entity: Partial<T>, tenantId: string): Promise<T> {
    const entityWithTimestamp = {
      ...entity,
      updated_at: new Date()
    };

    const { setClause, values } = this.buildUpdate(entityWithTimestamp);

    const result = await this.db.query(
      `UPDATE ${this.tableName}
       SET ${setClause}
       WHERE id = $${values.length + 1} AND organisation_id = $${values.length + 2}
       RETURNING *`,
      [...values, id, tenantId]
    );

    if (result.rows.length === 0) {
      throw new NotFoundError(`${this.tableName} not found`);
    }

    return result.rows[0];
  }

  async softDelete(id: string, tenantId: string): Promise<void> {
    const result = await this.db.query(
      `UPDATE ${this.tableName}
       SET deleted_at = NOW()
       WHERE id = $1 AND organisation_id = $2 AND deleted_at IS NULL`,
      [id, tenantId]
    );

    if (result.rowCount === 0) {
      throw new NotFoundError(`${this.tableName} not found`);
    }
  }
}
```

### 2.5 Middleware Specifications

#### 2.5.1 Auth Middleware

```typescript
// middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { UnauthorizedError, ForbiddenError } from '../utils/errors';

interface JWTPayload {
  sub: string;
  org: string;
  role: UserRole;
  tier: SubscriptionTier;
  iat: number;
  exp: number;
  jti: string;
}

export const requireAuth = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    // 1. Extract token from header
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer ')) {
      throw new UnauthorizedError('No token provided');
    }

    const token = authHeader.substring(7);

    // 2. Verify token
    const payload = jwt.verify(token, process.env.JWT_PUBLIC_KEY, {
      algorithms: ['RS256'],
      issuer: 'sommelier-spark',
      audience: 'sommelier-spark-api'
    }) as JWTPayload;

    // 3. Check token not blacklisted
    const isBlacklisted = await redisClient.get(`blacklist:${payload.jti}`);
    if (isBlacklisted) {
      throw new UnauthorizedError('Token has been revoked');
    }

    // 4. Attach user context to request
    req.user = {
      id: payload.sub,
      organisationId: payload.org,
      role: payload.role,
      subscriptionTier: payload.tier
    };

    req.tenantContext = {
      userId: payload.sub,
      organisationId: payload.org,
      userRole: payload.role
    };

    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      next(new UnauthorizedError('Token has expired'));
    } else if (error instanceof jwt.JsonWebTokenError) {
      next(new UnauthorizedError('Invalid token'));
    } else {
      next(error);
    }
  }
};

export const requireRole = (allowedRoles: UserRole[]) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    if (!req.user) {
      return next(new UnauthorizedError('Authentication required'));
    }

    if (!allowedRoles.includes(req.user.role)) {
      return next(new ForbiddenError('Insufficient permissions'));
    }

    next();
  };
};
```

#### 2.5.2 Tenant Middleware

```typescript
// middleware/tenant.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ForbiddenError } from '../utils/errors';

export const tenantMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    const organisationId = req.user?.organisationId;

    if (!organisationId) {
      throw new ForbiddenError('No organisation context');
    }

    // Verify organisation exists and is active
    const org = await organisationRepo.findById(organisationId);

    if (!org) {
      throw new ForbiddenError('Organisation not found');
    }

    if (org.status === 'SUSPENDED') {
      throw new ForbiddenError('Organisation is suspended');
    }

    if (org.status === 'CANCELLED') {
      throw new ForbiddenError('Organisation subscription has been cancelled');
    }

    // Set tenant context for database queries
    await db.query("SET app.current_organisation = $1", [organisationId]);

    // Attach org context to request
    req.organisation = org;
    req.tenantContext = {
      ...req.tenantContext,
      organisationId,
      subscriptionTier: org.subscriptionTier
    };

    next();
  } catch (error) {
    next(error);
  }
};
```

#### 2.5.3 Validation Middleware

```typescript
// middleware/validation.middleware.ts
import { Request, Response, NextFunction } from 'express';
import Joi from 'joi';
import { ValidationError } from '../utils/errors';

export const validateRequest = (
  schema: Joi.ObjectSchema,
  property: 'body' | 'query' | 'params' = 'body'
) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const { error, value } = schema.validate(req[property], {
      abortEarly: false,
      stripUnknown: true
    });

    if (error) {
      const fields = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));

      return next(new ValidationError('Validation failed', { fields }));
    }

    // Replace with validated/sanitized value
    req[property] = value;
    next();
  };
};
```

#### 2.5.4 Rate Limit Middleware

```typescript
// middleware/rate-limit.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { RateLimitError } from '../utils/errors';

interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
  keyGenerator: (req: Request) => string;
}

export const rateLimitMiddleware = (config: RateLimitConfig) => {
  return async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    const key = config.keyGenerator(req);
    const redisKey = `ratelimit:${key}`;

    try {
      const current = await redisClient.incr(redisKey);

      if (current === 1) {
        await redisClient.expire(redisKey, Math.ceil(config.windowMs / 1000));
      }

      // Set rate limit headers
      res.setHeader('X-RateLimit-Limit', config.maxRequests);
      res.setHeader('X-RateLimit-Remaining', Math.max(0, config.maxRequests - current));

      if (current > config.maxRequests) {
        const ttl = await redisClient.ttl(redisKey);
        res.setHeader('Retry-After', ttl);
        throw new RateLimitError('Too many requests');
      }

      next();
    } catch (error) {
      next(error);
    }
  };
};

// Default configurations
export const userRateLimit = rateLimitMiddleware({
  windowMs: 60000, // 1 minute
  maxRequests: 100,
  keyGenerator: (req) => `user:${req.user?.id}`
});

export const orgRateLimit = rateLimitMiddleware({
  windowMs: 60000,
  maxRequests: 1000,
  keyGenerator: (req) => `org:${req.user?.organisationId}`
});
```

#### 2.5.5 Audit Middleware

```typescript
// middleware/audit.middleware.ts
import { Request, Response, NextFunction } from 'express';

export const auditMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  const startTime = Date.now();

  // Generate request ID
  req.requestId = req.headers['x-request-id'] as string || uuid();

  // Log request
  logger.info({
    type: 'request',
    requestId: req.requestId,
    method: req.method,
    path: req.path,
    userId: req.user?.id,
    organisationId: req.user?.organisationId,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  });

  // Capture response
  const originalSend = res.send;
  res.send = function(body: any): Response {
    const duration = Date.now() - startTime;

    logger.info({
      type: 'response',
      requestId: req.requestId,
      statusCode: res.statusCode,
      duration,
      userId: req.user?.id,
      organisationId: req.user?.organisationId
    });

    return originalSend.call(this, body);
  };

  next();
};
```

---

## 3. Learning Engine Service Design

**CONFIDENTIAL — Patent-Pending Technology**

### 3.1 Directory Structure

```
src/
├── api/
│   ├── __init__.py
│   ├── routes.py
│   ├── handlers.py
│   └── schemas.py
├── transformers/
│   ├── __init__.py
│   ├── wine_list_parser.py
│   ├── curriculum_generator.py
│   └── module_structurer.py
├── generators/
│   ├── __init__.py
│   ├── quiz_generator.py
│   ├── question_generator.py
│   ├── distractor_generator.py
│   ├── scenario_generator.py
│   ├── dialogue_generator.py
│   └── persona_generator.py
├── adapters/
│   ├── __init__.py
│   ├── performance_analyser.py
│   ├── gap_identifier.py
│   ├── path_optimiser.py
│   └── mastery_detector.py
├── templates/
│   ├── __init__.py
│   ├── question_templates.py
│   └── scenario_templates.py
├── validators/
│   ├── __init__.py
│   ├── content_validator.py
│   └── quality_checker.py
├── models/
│   ├── __init__.py
│   ├── wine.py
│   ├── curriculum.py
│   ├── question.py
│   └── scenario.py
├── services/
│   ├── __init__.py
│   ├── template_service.py
│   ├── cache_service.py
│   └── metrics_service.py
├── utils/
│   ├── __init__.py
│   ├── logger.py
│   └── errors.py
├── config/
│   ├── __init__.py
│   └── settings.py
├── main.py
└── requirements.txt
```

### 3.2 Curriculum Generation Algorithm

**CONFIDENTIAL — Patent-Pending**

```python
# transformers/curriculum_generator.py
from typing import List, Optional
from dataclasses import dataclass
from models.wine import Wine
from models.curriculum import Curriculum, Module, Lesson

@dataclass
class CurriculumGeneratorConfig:
    min_wines_per_module: int = 3
    max_wines_per_module: int = 15
    min_lessons_per_module: int = 3
    max_lessons_per_module: int = 7
    lesson_duration_minutes: int = 10

class CurriculumGenerator:
    """
    Transforms a wine list into a structured learning curriculum.

    This is the core transformation engine that converts raw wine data
    into a comprehensive, tiered training programme.
    """

    def __init__(self, config: CurriculumGeneratorConfig = None):
        self.config = config or CurriculumGeneratorConfig()
        self.template_service = TemplateService()

    def generate(
        self,
        wines: List[Wine],
        org_settings: OrgSettings
    ) -> Curriculum:
        """
        Main generation entry point.

        Steps:
        1. Analyse wine list composition
        2. Identify gaps and specialities
        3. Generate module structure
        4. Assign wines to modules
        5. Generate lessons per module
        6. Generate quizzes per tier
        7. Generate scenarios
        8. Build learning paths
        9. Validate coverage
        10. Return complete curriculum
        """
        # Step 1: Analyse composition
        analysis = self._analyse_composition(wines)

        # Step 2: Identify gaps
        gaps = self._identify_gaps(analysis, org_settings)

        # Step 3: Generate module structure
        module_templates = self._generate_module_structure(analysis, gaps)

        # Step 4-5: Create modules with lessons
        modules = []
        for template in module_templates:
            module_wines = self._select_wines_for_module(wines, template)
            lessons = self._generate_lessons(module_wines, template)

            module = Module(
                id=generate_uuid(),
                title=template.title.format(org_name=org_settings.name),
                description=template.description,
                tier=template.tier,
                category=template.category,
                lessons=lessons,
                wines=module_wines,
                estimated_time=sum(l.duration for l in lessons),
                order=template.order
            )
            modules.append(module)

        # Step 6-8: Generate assessments and paths
        quizzes = self._generate_quizzes(modules, wines)
        scenarios = self._generate_scenarios(wines, org_settings)
        paths = self._build_learning_paths(modules, quizzes, scenarios)

        # Step 9: Validate coverage
        coverage = self._validate_coverage(wines, modules, quizzes)

        return Curriculum(
            id=generate_uuid(),
            organisation_id=org_settings.organisation_id,
            modules=modules,
            quizzes=quizzes,
            scenarios=scenarios,
            learning_paths=paths,
            coverage_report=coverage,
            version=1,
            created_at=datetime.utcnow()
        )

    def _analyse_composition(self, wines: List[Wine]) -> WineAnalysis:
        """
        Analyse wine list by category, region, grape, and price.
        """
        analysis = WineAnalysis()

        for wine in wines:
            # Category counts
            analysis.category_counts[wine.wine_type] += 1

            # Region mapping
            if wine.region not in analysis.regions:
                analysis.regions[wine.region] = RegionInfo(
                    name=wine.region,
                    country=wine.country,
                    wine_count=0,
                    wine_types=set()
                )
            analysis.regions[wine.region].wine_count += 1
            analysis.regions[wine.region].wine_types.add(wine.wine_type)

            # Grape varieties
            for grape in wine.grape_varieties:
                analysis.grape_counts[grape] += 1

            # Price distribution
            analysis.price_counts[wine.price_tier] += 1
            analysis.price_range.update(wine.price)

        return analysis

    def _generate_module_structure(
        self,
        analysis: WineAnalysis,
        gaps: List[Gap]
    ) -> List[ModuleTemplate]:
        """
        Create module structure based on analysis.

        Standard structure (10-200 wines):
        - Module 1: Welcome & Orientation (Bronze)
        - Module 2: Red Wines (Bronze)
        - Module 3: White Wines (Bronze)
        - Module 4: Sparkling & Special (Bronze)
        - Module 5: Regional Deep Dive (Silver)
        - Module 6: Food Pairing (Silver)
        - Module 7: Guest Preferences (Silver)
        - Module 8: Advanced Tasting (Gold)
        - Module 9: Premium Service (Gold)
        - Module 10: Difficult Situations (Gold)
        """
        templates = []
        order = 1

        # Always add welcome module
        templates.append(ModuleTemplate(
            title="Welcome to {org_name} Wine Collection",
            category=ModuleCategory.ORIENTATION,
            tier=Tier.BRONZE,
            order=order
        ))
        order += 1

        # Add category modules based on wine counts
        if analysis.category_counts.get(WineType.RED, 0) >= 3:
            templates.append(ModuleTemplate(
                title="Your Red Wine Selection",
                category=ModuleCategory.RED_WINES,
                tier=Tier.BRONZE,
                order=order,
                filter_fn=lambda w: w.wine_type == WineType.RED
            ))
            order += 1

        if analysis.category_counts.get(WineType.WHITE, 0) >= 3:
            templates.append(ModuleTemplate(
                title="Your White Wine Selection",
                category=ModuleCategory.WHITE_WINES,
                tier=Tier.BRONZE,
                order=order,
                filter_fn=lambda w: w.wine_type == WineType.WHITE
            ))
            order += 1

        # Add sparkling/special if present
        sparkling_count = sum([
            analysis.category_counts.get(WineType.SPARKLING, 0),
            analysis.category_counts.get(WineType.ROSE, 0),
            analysis.category_counts.get(WineType.DESSERT, 0),
            analysis.category_counts.get(WineType.FORTIFIED, 0)
        ])

        if sparkling_count >= 2:
            templates.append(ModuleTemplate(
                title="Sparkling & Special Wines",
                category=ModuleCategory.SPECIAL,
                tier=Tier.BRONZE,
                order=order
            ))
            order += 1

        # Silver tier modules
        templates.extend([
            ModuleTemplate(
                title="Regional Deep Dive",
                category=ModuleCategory.REGIONAL,
                tier=Tier.SILVER,
                order=order
            ),
            ModuleTemplate(
                title="Food Pairing at {org_name}",
                category=ModuleCategory.PAIRING,
                tier=Tier.SILVER,
                order=order + 1
            ),
            ModuleTemplate(
                title="Guest Preference Discovery",
                category=ModuleCategory.SERVICE,
                tier=Tier.SILVER,
                order=order + 2
            )
        ])
        order += 3

        # Gold tier modules
        templates.extend([
            ModuleTemplate(
                title="Advanced Tasting & Description",
                category=ModuleCategory.TASTING,
                tier=Tier.GOLD,
                order=order
            ),
            ModuleTemplate(
                title="Premium Wine Service",
                category=ModuleCategory.PREMIUM,
                tier=Tier.GOLD,
                order=order + 1
            ),
            ModuleTemplate(
                title="Mastering Difficult Situations",
                category=ModuleCategory.ADVANCED,
                tier=Tier.GOLD,
                order=order + 2
            )
        ])

        return templates
```

### 3.3 Question Generation Algorithm

**CONFIDENTIAL — Patent-Pending**

```python
# generators/question_generator.py
from typing import List, Set
from models.question import Question, QuestionTemplate
from models.wine import Wine

class QuestionGenerator:
    """
    Generates quiz questions from wine attributes.

    Uses template-based generation with distractor rules
    to create valid, calibrated assessment questions.
    """

    # Template IDs by tier
    BRONZE_TEMPLATES = [
        'QT-ID-001', 'QT-ID-002', 'QT-ID-003', 'QT-ID-005',
        'QT-PA-001', 'QT-SV-001', 'QT-TF-001'
    ]
    SILVER_TEMPLATES = [
        'QT-ID-004', 'QT-PA-002', 'QT-TN-001', 'QT-TN-002',
        'QT-SV-002', 'QT-SV-003', 'QT-AD-003'
    ]
    GOLD_TEMPLATES = [
        'QT-PA-003', 'QT-TN-003', 'QT-AD-001', 'QT-AD-002'
    ]

    def __init__(self):
        self.distractor_generator = DistractorGenerator()
        self.template_service = TemplateService()

    def generate_questions(
        self,
        wines: List[Wine],
        tier: Tier,
        count: int,
        used_templates: Set[str] = None
    ) -> List[Question]:
        """
        Generate questions for a quiz.

        Steps:
        1. Select appropriate templates for tier
        2. For each question:
           a. Select candidate wines
           b. Choose template (rotate usage)
           c. Generate question text
           d. Select correct answer
           e. Generate distractors
           f. Validate question
        3. Balance question types
        4. Ensure wine coverage
        5. Return question set
        """
        used_templates = used_templates or set()
        questions = []
        wine_usage = {w.id: 0 for w in wines}

        templates = self._get_templates_for_tier(tier)

        for i in range(count):
            # Select template (rotate to ensure variety)
            template = self._select_template(templates, used_templates)
            used_templates.add(template.id)

            # Select wine(s) for question
            target_wines = self._select_wines_for_question(
                wines, template, wine_usage
            )

            # Generate question
            question = self._generate_single_question(
                template, target_wines, wines, tier
            )

            if question and self._validate_question(question):
                questions.append(question)
                for w in target_wines:
                    wine_usage[w.id] += 1

        return questions

    def _select_template(
        self,
        templates: List[QuestionTemplate],
        used: Set[str]
    ) -> QuestionTemplate:
        """
        Select template based on tier and prior usage.
        Prioritise unused templates for variety.
        """
        unused = [t for t in templates if t.id not in used]
        if unused:
            return random.choice(unused)

        # Reset and reuse if all used
        used.clear()
        return random.choice(templates)

    def _generate_single_question(
        self,
        template: QuestionTemplate,
        target_wines: List[Wine],
        all_wines: List[Wine],
        tier: Tier
    ) -> Question:
        """
        Generate a single question from template.
        """
        target_wine = target_wines[0]

        # Fill template parameters
        question_text = template.text.format(
            wine_name=target_wine.name,
            region=target_wine.region,
            grape=target_wine.grape_varieties[0] if target_wine.grape_varieties else '',
            country=target_wine.country,
            producer=target_wine.producer or '',
            wine_type=target_wine.wine_type.value
        )

        # Get correct answer
        correct_answer = self._get_correct_answer(template, target_wine)

        # Generate distractors
        distractors = self.distractor_generator.generate(
            correct_answer=correct_answer,
            attribute=template.attribute,
            target_wine=target_wine,
            wine_pool=all_wines,
            tier=tier,
            count=3
        )

        # Create options (shuffle)
        options = [
            QuestionOption(text=correct_answer, is_correct=True),
            *[QuestionOption(text=d, is_correct=False) for d in distractors]
        ]
        random.shuffle(options)

        return Question(
            id=generate_uuid(),
            template_id=template.id,
            text=question_text,
            options=options,
            tier=tier,
            wine_ids=[w.id for w in target_wines],
            explanation=self._generate_explanation(template, target_wine, correct_answer)
        )
```

### 3.4 Distractor Generation Algorithm

**CONFIDENTIAL — Patent-Pending**

```python
# generators/distractor_generator.py
class DistractorGenerator:
    """
    Generates plausible wrong answers using rule-based selection.

    Rules (from SS-WS3.0-LE-CGR):
    - DG-001: Same category, different item
    - DG-002: Common confusion pairs
    - DG-003: Similar names
    - DG-004: Geographic neighbours
    - DG-005: Same producer, different wine
    - DG-006: Opposite characteristics
    - DG-007: Same grape, different region
    - DG-008: Adjacent price tier
    """

    CONFUSION_PAIRS = {
        'Chablis': ['Chardonnay (grape)', 'Chablis (grape)'],
        'Champagne': ['Sparkling wine', 'Prosecco'],
        'Pinot Grigio': ['Pinot Gris'],
        'Pouilly-Fumé': ['Pouilly-Fuissé'],
        'Côte-Rôtie': ['Côtes du Rhône']
    }

    GEOGRAPHIC_NEIGHBOURS = {
        'Bordeaux': ['Burgundy', 'Rhône', 'Loire'],
        'Burgundy': ['Beaujolais', 'Champagne', 'Alsace'],
        'Champagne': ['Burgundy', 'Alsace'],
        'Rioja': ['Ribera del Duero', 'Navarra'],
        'Tuscany': ['Piedmont', 'Veneto']
    }

    def generate(
        self,
        correct_answer: str,
        attribute: str,
        target_wine: Wine,
        wine_pool: List[Wine],
        tier: Tier,
        count: int = 3
    ) -> List[str]:
        """
        Generate distractors based on rules appropriate for tier.

        Bronze: Obvious differences (DG-001, DG-004, DG-006)
        Silver: Plausible mistakes (DG-002, DG-003, DG-005, DG-007)
        Gold: Expert-level traps (DG-008, DG-009, DG-010, DG-012)
        """
        candidates = []
        rules = self._get_rules_for_tier(tier)

        for rule in rules:
            rule_candidates = self._apply_rule(
                rule, correct_answer, attribute, target_wine, wine_pool
            )
            candidates.extend(rule_candidates)

        # Remove duplicates and correct answer
        candidates = list(set(candidates) - {correct_answer})

        # Score candidates by plausibility
        scored = [(c, self._score_plausibility(c, correct_answer, tier))
                  for c in candidates]
        scored.sort(key=lambda x: x[1], reverse=True)

        # Select top N
        selected = [c for c, _ in scored[:count]]

        # Fill with random if not enough
        while len(selected) < count:
            fallback = self._generate_fallback_distractor(
                attribute, target_wine, wine_pool, selected + [correct_answer]
            )
            if fallback:
                selected.append(fallback)
            else:
                break

        return selected[:count]

    def _apply_rule(
        self,
        rule: str,
        correct: str,
        attribute: str,
        target: Wine,
        pool: List[Wine]
    ) -> List[str]:
        """Apply a specific distractor rule."""

        if rule == 'DG-001':  # Same category
            return self._same_category_distractors(correct, attribute, pool)

        elif rule == 'DG-002':  # Common confusion
            return self.CONFUSION_PAIRS.get(correct, [])

        elif rule == 'DG-003':  # Similar names
            return self._similar_name_distractors(correct, pool)

        elif rule == 'DG-004':  # Geographic neighbours
            if attribute == 'region':
                return self.GEOGRAPHIC_NEIGHBOURS.get(target.region, [])
            return []

        elif rule == 'DG-005':  # Same producer
            return [w.name for w in pool
                    if w.producer == target.producer and w.id != target.id]

        elif rule == 'DG-006':  # Opposite characteristics
            return self._opposite_characteristic_distractors(target, pool)

        elif rule == 'DG-007':  # Same grape, different region
            return [w.region for w in pool
                    if set(w.grape_varieties) & set(target.grape_varieties)
                    and w.region != target.region]

        return []
```

### 3.5 Scenario Generation Algorithm

**CONFIDENTIAL — Patent-Pending**

```python
# generators/scenario_generator.py
class ScenarioGenerator:
    """
    Generates interactive customer service scenarios.
    """

    def generate_scenario(
        self,
        template: ScenarioTemplate,
        wines: List[Wine],
        menu: Optional[Menu],
        tier: Tier
    ) -> Scenario:
        """
        Generate a complete scenario.

        Steps:
        1. Generate customer persona
        2. Select relevant wines
        3. Build opening situation
        4. Generate decision tree
        5. Assign scores
        6. Validate paths
        7. Return scenario
        """
        # Step 1: Generate persona
        persona = self.persona_generator.generate(
            template.persona_type,
            tier
        )

        # Step 2: Select wines
        scenario_wines = self._select_wines_for_scenario(
            template, wines, persona
        )

        # Step 3: Build opening
        opening = self._build_opening(template, persona, scenario_wines, menu)

        # Step 4: Generate decision tree
        decision_tree = self._build_decision_tree(
            template, persona, scenario_wines, menu, tier
        )

        # Step 5: Assign scores
        self._assign_scores(decision_tree, tier)

        # Step 6: Validate
        self._validate_all_paths(decision_tree)

        return Scenario(
            id=generate_uuid(),
            template_id=template.id,
            title=template.title.format(**persona.to_dict()),
            tier=tier,
            persona=persona,
            opening=opening,
            steps=decision_tree,
            wines_featured=[w.id for w in scenario_wines],
            estimated_time=template.estimated_time,
            success_criteria=template.success_criteria
        )

    def _build_decision_tree(
        self,
        template: ScenarioTemplate,
        persona: Persona,
        wines: List[Wine],
        menu: Optional[Menu],
        tier: Tier
    ) -> List[ScenarioStep]:
        """Build branching decision tree."""
        steps = []

        for step_template in template.step_templates:
            step = ScenarioStep(
                id=generate_uuid(),
                number=step_template.number,
                situation=step_template.situation.format(
                    customer_name=persona.name,
                    **self._get_context_vars(wines, menu)
                ),
                choices=[]
            )

            # Generate choices
            for choice_template in step_template.choice_templates:
                choice = self._generate_choice(
                    choice_template, persona, wines, menu, tier
                )
                step.choices.append(choice)

            steps.append(step)

        return steps
```

### 3.6 Adaptive Learning Algorithms

**CONFIDENTIAL — Patent-Pending**

```python
# adapters/performance_analyser.py
class PerformanceAnalyser:
    """
    Analyses learner performance to identify patterns.
    """

    def analyse(
        self,
        user_id: str,
        attempts: List[Attempt]
    ) -> PerformanceProfile:
        """
        Build comprehensive performance profile.
        """
        profile = PerformanceProfile(user_id=user_id)

        # Calculate overall proficiency
        profile.overall_score = self._calculate_overall_score(attempts)

        # Calculate category scores
        profile.category_scores = self._calculate_category_scores(attempts)

        # Identify trend
        profile.trend = self._calculate_trend(attempts)

        # Find weak areas
        profile.weak_areas = self._identify_weak_areas(profile.category_scores)

        # Time patterns
        profile.optimal_time = self._analyse_time_patterns(attempts)

        return profile

    def _calculate_category_scores(
        self,
        attempts: List[Attempt]
    ) -> Dict[str, float]:
        """Calculate proficiency by category."""
        category_results = defaultdict(list)

        for attempt in attempts:
            for answer in attempt.answers:
                category = answer.question.category
                category_results[category].append(answer.is_correct)

        return {
            cat: sum(results) / len(results) * 100
            for cat, results in category_results.items()
        }

# adapters/gap_identifier.py
class GapIdentifier:
    """
    Identifies knowledge gaps for targeted learning.
    """

    THRESHOLD_LOW = 60  # Below this is a gap
    THRESHOLD_MEDIUM = 75

    def identify_gaps(
        self,
        profile: PerformanceProfile
    ) -> List[Gap]:
        """Find areas needing improvement."""
        gaps = []

        for category, score in profile.category_scores.items():
            if score < self.THRESHOLD_LOW:
                gaps.append(Gap(
                    type=GapType.CATEGORY,
                    target=category,
                    severity=GapSeverity.HIGH,
                    score=score
                ))
            elif score < self.THRESHOLD_MEDIUM:
                gaps.append(Gap(
                    type=GapType.CATEGORY,
                    target=category,
                    severity=GapSeverity.MEDIUM,
                    score=score
                ))

        # Sort by severity and score
        gaps.sort(key=lambda g: (g.severity.value, g.score))

        return gaps

# adapters/mastery_detector.py
class MasteryDetector:
    """
    Detects when learner has mastered content.
    """

    MASTERY_THRESHOLD = 0.85  # 85%
    CONSISTENCY_REQUIRED = 3   # Consecutive attempts

    def check_mastery(
        self,
        category: str,
        attempts: List[Attempt]
    ) -> bool:
        """
        Determine if mastery achieved.

        Criteria:
        - Score >= 85% on last 3 attempts
        - No declining trend
        - Coverage of all sub-topics
        """
        # Filter to category
        category_attempts = [
            a for a in attempts
            if a.category == category
        ]

        if len(category_attempts) < self.CONSISTENCY_REQUIRED:
            return False

        # Check last N attempts
        recent = category_attempts[-self.CONSISTENCY_REQUIRED:]

        # All above threshold?
        if not all(a.score >= self.MASTERY_THRESHOLD for a in recent):
            return False

        # No declining trend?
        scores = [a.score for a in recent]
        if self._is_declining(scores):
            return False

        return True
```

---

## 4. Content Service Design

### 4.1 Workflow Engine

```typescript
// services/workflow-engine.ts
interface TransitionConfig {
  target: ContentStatus;
  roles: UserRole[];
  preHooks?: string[];
  postHooks?: string[];
}

type TransitionMap = {
  [status in ContentStatus]?: {
    [action in WorkflowAction]?: TransitionConfig;
  };
};

export class WorkflowEngine {
  private transitions: TransitionMap = {
    [ContentStatus.DRAFT]: {
      [WorkflowAction.SUBMIT]: {
        target: ContentStatus.REVIEW,
        roles: [UserRole.CONTENT_AUTHOR, UserRole.CONTENT_ADMIN],
        preHooks: ['validateCompleteness'],
        postHooks: ['notifyReviewers', 'assignReviewer']
      },
      [WorkflowAction.DELETE]: {
        target: ContentStatus.DELETED,
        roles: [UserRole.CONTENT_AUTHOR, UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.REVIEW]: {
      [WorkflowAction.APPROVE]: {
        target: ContentStatus.APPROVED,
        roles: [UserRole.DOMAIN_EXPERT, UserRole.CONTENT_ADMIN],
        postHooks: ['recordApproval', 'notifyAuthor']
      },
      [WorkflowAction.REJECT]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.DOMAIN_EXPERT, UserRole.CONTENT_ADMIN],
        postHooks: ['recordRejection', 'notifyAuthorRejection']
      }
    },
    [ContentStatus.APPROVED]: {
      [WorkflowAction.PUBLISH]: {
        target: ContentStatus.PUBLISHED,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN],
        preHooks: ['validatePublishReady'],
        postHooks: ['incrementVersion', 'invalidateCache', 'notifyStakeholders']
      },
      [WorkflowAction.REJECT]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_ADMIN]
      }
    },
    [ContentStatus.PUBLISHED]: {
      [WorkflowAction.ARCHIVE]: {
        target: ContentStatus.ARCHIVED,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN],
        postHooks: ['archiveRelatedContent']
      },
      [WorkflowAction.REVISE]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_AUTHOR, UserRole.CONTENT_ADMIN],
        postHooks: ['createRevision']
      }
    },
    [ContentStatus.ARCHIVED]: {
      [WorkflowAction.RESTORE]: {
        target: ContentStatus.DRAFT,
        roles: [UserRole.CONTENT_ADMIN, UserRole.ORG_ADMIN]
      }
    }
  };

  async transition(
    content: Content,
    action: WorkflowAction,
    actor: User,
    context: WorkflowContext
  ): Promise<Content> {
    // 1. Get transition config
    const config = this.transitions[content.status]?.[action];
    if (!config) {
      throw new ConflictError(
        `Action '${action}' not allowed from '${content.status}'`
      );
    }

    // 2. Check permission
    if (!config.roles.includes(actor.role)) {
      throw new ForbiddenError(`Insufficient permissions for ${action}`);
    }

    // 3. Execute pre-hooks
    if (config.preHooks) {
      for (const hook of config.preHooks) {
        await this.executeHook(hook, content, context);
      }
    }

    // 4. Update status
    const updated = await this.contentRepo.update(content.id, {
      status: config.target,
      updatedBy: actor.id,
      updatedAt: new Date(),
      [`${action}At`]: new Date(),
      [`${action}By`]: actor.id
    }, context.organisationId);

    // 5. Execute post-hooks
    if (config.postHooks) {
      for (const hook of config.postHooks) {
        await this.executeHook(hook, updated, context);
      }
    }

    // 6. Audit log
    await this.auditService.log({
      action: `content.${action}`,
      resourceType: content.type,
      resourceId: content.id,
      userId: actor.id,
      organisationId: context.organisationId,
      previousState: { status: content.status },
      newState: { status: updated.status },
      metadata: context.metadata
    });

    return updated;
  }
}
```

### 4.2 Import Processor

```typescript
// services/import-processor.ts
export class ImportProcessor {
  constructor(
    private wineRepo: WineRepository,
    private importRepo: ImportRepository,
    private queueService: QueueService,
    private validationService: ValidationService
  ) {}

  async processWineImport(
    file: Buffer,
    options: ImportOptions,
    context: TenantContext
  ): Promise<ImportResult> {
    // 1. Parse CSV
    const rows = await this.parseCSV(file);

    // 2. Validate headers
    const headerErrors = this.validateHeaders(rows[0], WINE_IMPORT_SCHEMA);
    if (headerErrors.length > 0) {
      return {
        success: false,
        phase: 'header_validation',
        errors: headerErrors
      };
    }

    // 3. Validate each row
    const validationResults = await this.validateRows(
      rows.slice(1),
      WINE_IMPORT_SCHEMA,
      context
    );

    // 4. Check for duplicates
    const duplicates = await this.findDuplicates(
      validationResults.valid,
      context.organisationId
    );

    // 5. Apply duplicate strategy
    const toImport = this.applyDuplicateStrategy(
      validationResults.valid,
      duplicates,
      options.duplicateStrategy
    );

    // 6. Create import record
    const importRecord = await this.importRepo.create({
      organisationId: context.organisationId,
      userId: context.userId,
      filename: options.filename,
      totalRows: rows.length - 1,
      validRows: toImport.length,
      errorRows: validationResults.errors.length,
      status: ImportStatus.PENDING_CONFIRMATION,
      preview: this.generatePreview(toImport, validationResults.errors)
    });

    // 7. Return preview if requested
    if (options.previewOnly) {
      return {
        success: true,
        importId: importRecord.id,
        preview: importRecord.preview,
        requiresConfirmation: true
      };
    }

    // 8. Execute import
    return this.executeImport(importRecord.id, toImport, context);
  }

  async executeImport(
    importId: string,
    wines: WineImportRow[],
    context: TenantContext
  ): Promise<ImportResult> {
    // Update status
    await this.importRepo.updateStatus(importId, ImportStatus.PROCESSING);

    try {
      // Import in transaction
      const imported = await this.db.transaction(async (tx) => {
        const results = [];

        for (const wine of wines) {
          const created = await this.wineRepo.create({
            ...this.mapImportRowToWine(wine),
            organisationId: context.organisationId,
            status: ContentStatus.DRAFT,
            createdBy: context.userId
          }, tx);
          results.push(created);
        }

        return results;
      });

      // Update import record
      await this.importRepo.update(importId, {
        status: ImportStatus.COMPLETED,
        importedCount: imported.length,
        completedAt: new Date()
      });

      // Trigger curriculum generation
      await this.queueService.enqueue({
        type: JobType.GENERATE_CURRICULUM,
        data: {
          organisationId: context.organisationId,
          trigger: 'wine_import',
          importId
        }
      });

      return {
        success: true,
        importId,
        imported: imported.length,
        curriculumJobQueued: true
      };
    } catch (error) {
      await this.importRepo.update(importId, {
        status: ImportStatus.FAILED,
        error: error.message
      });
      throw error;
    }
  }
}
```

### 4.3 Version Manager

```typescript
// services/version-manager.ts
export class VersionManager {
  async createVersion(
    content: Content,
    changeType: 'major' | 'minor'
  ): Promise<ContentVersion> {
    // Calculate new version
    const newVersion = this.incrementVersion(content.version, changeType);

    // Create snapshot
    const snapshot = this.createSnapshot(content);

    // Store version record
    const version = await this.versionRepo.create({
      contentId: content.id,
      contentType: content.type,
      version: newVersion,
      snapshot,
      createdBy: content.updatedBy,
      createdAt: new Date()
    });

    // Update content version
    await this.contentRepo.updateVersion(content.id, newVersion);

    return version;
  }

  async restoreVersion(
    contentId: string,
    versionId: string,
    context: TenantContext
  ): Promise<Content> {
    // Fetch version
    const version = await this.versionRepo.findById(versionId);
    if (!version || version.contentId !== contentId) {
      throw new NotFoundError('Version not found');
    }

    // Create new version from snapshot
    const restored = await this.contentRepo.update(
      contentId,
      {
        ...version.snapshot,
        status: ContentStatus.DRAFT,
        updatedBy: context.userId
      },
      context.organisationId
    );

    // Create new version record
    await this.createVersion(restored, 'major');

    // Audit log
    await this.auditService.log({
      action: 'content.restored',
      resourceType: version.contentType,
      resourceId: contentId,
      userId: context.userId,
      metadata: { restoredFromVersion: version.version }
    });

    return restored;
  }

  private incrementVersion(
    current: number,
    changeType: 'major' | 'minor'
  ): number {
    if (changeType === 'major') {
      return Math.floor(current) + 1;
    }
    return current + 0.1;
  }
}
```

---

## 5. Authentication Service Design

### 5.1 JWT Structure

```typescript
// types/auth.ts
interface JWTAccessPayload {
  sub: string;           // User ID
  org: string;           // Organisation ID
  role: UserRole;        // User role
  tier: SubscriptionTier; // Org subscription
  iat: number;           // Issued at
  exp: number;           // Expiry (15 minutes)
  jti: string;           // Token ID
}

interface JWTRefreshPayload {
  sub: string;           // User ID
  jti: string;           // Token ID
  fam: string;           // Token family (for rotation)
  exp: number;           // Expiry (7 days)
}

// Token configuration
const TOKEN_CONFIG = {
  access: {
    expiresIn: '15m',
    algorithm: 'RS256'
  },
  refresh: {
    expiresIn: '7d',
    algorithm: 'RS256'
  }
};
```

### 5.2 Auth Service Implementation

```typescript
// services/auth.service.ts
export class AuthService {
  constructor(
    private userRepo: UserRepository,
    private orgRepo: OrganisationRepository,
    private tokenRepo: TokenRepository,
    private auditService: AuditService
  ) {}

  async login(email: string, password: string): Promise<AuthResponse> {
    // 1. Find user
    const user = await this.userRepo.findByEmail(email);
    if (!user) {
      await this.recordFailedAttempt(null, email);
      throw new UnauthorizedError('Invalid credentials');
    }

    // 2. Check account status
    if (user.status === UserStatus.SUSPENDED) {
      throw new ForbiddenError('Account suspended');
    }
    if (user.status === UserStatus.PENDING) {
      throw new ForbiddenError('Account not activated');
    }

    // 3. Check lockout
    if (await this.isLockedOut(user.id)) {
      throw new ForbiddenError('Account locked. Try again later.');
    }

    // 4. Check organisation status
    const org = await this.orgRepo.findById(user.organisationId);
    if (org.status === OrgStatus.SUSPENDED) {
      throw new ForbiddenError('Organisation suspended');
    }

    // 5. Verify password
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) {
      await this.recordFailedAttempt(user.id, email);
      throw new UnauthorizedError('Invalid credentials');
    }

    // 6. Generate tokens
    const tokenFamily = generateUuid();
    const accessToken = this.generateAccessToken(user, org);
    const refreshToken = this.generateRefreshToken(user, tokenFamily);

    // 7. Store refresh token
    await this.tokenRepo.storeRefreshToken({
      jti: this.extractJti(refreshToken),
      userId: user.id,
      family: tokenFamily,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
    });

    // 8. Clear failed attempts
    await this.clearFailedAttempts(user.id);

    // 9. Update last login
    await this.userRepo.updateLastLogin(user.id);

    // 10. Audit log
    await this.auditService.log({
      action: 'user.login',
      resourceType: 'user',
      resourceId: user.id,
      userId: user.id,
      organisationId: user.organisationId
    });

    return {
      accessToken,
      refreshToken,
      expiresIn: 900, // 15 minutes
      user: this.sanitizeUser(user)
    };
  }

  async refresh(refreshToken: string): Promise<AuthResponse> {
    // 1. Verify token
    let payload: JWTRefreshPayload;
    try {
      payload = jwt.verify(refreshToken, process.env.JWT_PUBLIC_KEY) as JWTRefreshPayload;
    } catch {
      throw new UnauthorizedError('Invalid refresh token');
    }

    // 2. Check token exists and not revoked
    const storedToken = await this.tokenRepo.findByJti(payload.jti);
    if (!storedToken || storedToken.revokedAt) {
      // Possible token reuse attack - revoke entire family
      await this.tokenRepo.revokeFamily(payload.fam);
      throw new UnauthorizedError('Token has been revoked');
    }

    // 3. Get user
    const user = await this.userRepo.findById(payload.sub);
    if (!user || user.status !== UserStatus.ACTIVE) {
      throw new UnauthorizedError('User not found or inactive');
    }

    // 4. Get org
    const org = await this.orgRepo.findById(user.organisationId);

    // 5. Rotate tokens
    await this.tokenRepo.revoke(payload.jti);

    const newAccessToken = this.generateAccessToken(user, org);
    const newRefreshToken = this.generateRefreshToken(user, payload.fam);

    await this.tokenRepo.storeRefreshToken({
      jti: this.extractJti(newRefreshToken),
      userId: user.id,
      family: payload.fam,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
    });

    return {
      accessToken: newAccessToken,
      refreshToken: newRefreshToken,
      expiresIn: 900,
      user: this.sanitizeUser(user)
    };
  }

  async logout(userId: string, tokenJti: string): Promise<void> {
    // Revoke refresh token
    await this.tokenRepo.revoke(tokenJti);

    // Blacklist access token (optional, for immediate invalidation)
    await this.redisClient.setex(
      `blacklist:${tokenJti}`,
      900, // 15 minutes (access token lifetime)
      '1'
    );

    // Audit log
    await this.auditService.log({
      action: 'user.logout',
      resourceType: 'user',
      resourceId: userId,
      userId
    });
  }

  private generateAccessToken(user: User, org: Organisation): string {
    return jwt.sign(
      {
        sub: user.id,
        org: org.id,
        role: user.role,
        tier: org.subscriptionTier
      },
      process.env.JWT_PRIVATE_KEY,
      {
        algorithm: 'RS256',
        expiresIn: '15m',
        issuer: 'sommelier-spark',
        audience: 'sommelier-spark-api',
        jwtid: generateUuid()
      }
    );
  }

  private async isLockedOut(userId: string): Promise<boolean> {
    const attempts = await this.redisClient.get(`lockout:${userId}`);
    return parseInt(attempts || '0', 10) >= 5;
  }

  private async recordFailedAttempt(userId: string | null, email: string): Promise<void> {
    if (userId) {
      const key = `lockout:${userId}`;
      await this.redisClient.incr(key);
      await this.redisClient.expire(key, 900); // 15 minute lockout
    }

    await this.auditService.log({
      action: 'auth.failed',
      resourceType: 'auth',
      metadata: { email }
    });
  }
}
```

---

## 6. Notification Service Design

### 6.1 Notification Types

```typescript
// types/notification.ts
enum NotificationType {
  // User lifecycle
  USER_INVITED = 'user.invited',
  USER_ACTIVATED = 'user.activated',
  PASSWORD_RESET = 'password.reset',
  PASSWORD_CHANGED = 'password.changed',

  // Content workflow
  CONTENT_SUBMITTED = 'content.submitted',
  CONTENT_APPROVED = 'content.approved',
  CONTENT_REJECTED = 'content.rejected',
  CONTENT_PUBLISHED = 'content.published',
  REVIEW_ASSIGNED = 'review.assigned',
  REVIEW_OVERDUE = 'review.overdue',

  // Learning
  QUIZ_PASSED = 'quiz.passed',
  QUIZ_FAILED = 'quiz.failed',
  CERTIFICATION_ACHIEVED = 'certification.achieved',
  MILESTONE_REACHED = 'milestone.reached',

  // Admin
  IMPORT_COMPLETE = 'import.complete',
  IMPORT_FAILED = 'import.failed',
  CURRICULUM_GENERATED = 'curriculum.generated',
  SUBSCRIPTION_EXPIRING = 'subscription.expiring'
}

interface NotificationTemplate {
  type: NotificationType;
  channels: ('email' | 'inapp' | 'push')[];
  subject: string;
  body: string;
  priority: 'low' | 'normal' | 'high';
}
```

### 6.2 Notification Service

```typescript
// services/notification.service.ts
export class NotificationService {
  async send(
    type: NotificationType,
    recipient: User,
    data: Record<string, any>
  ): Promise<void> {
    // 1. Load template
    const template = await this.templateRepo.findByType(type);
    if (!template) {
      logger.warn(`No template for notification type: ${type}`);
      return;
    }

    // 2. Check user preferences
    const preferences = await this.getPreferences(recipient.id);

    // 3. Render content
    const rendered = this.renderTemplate(template, data);

    // 4. Create notification record
    const notification = await this.notificationRepo.create({
      userId: recipient.id,
      type,
      title: rendered.subject,
      body: rendered.body,
      data,
      read: false
    });

    // 5. Send via enabled channels
    const deliveries = [];

    if (template.channels.includes('email') && preferences.emailEnabled) {
      deliveries.push(
        this.emailService.send({
          to: recipient.email,
          subject: rendered.subject,
          html: rendered.emailHtml,
          text: rendered.emailText
        })
      );
    }

    if (template.channels.includes('inapp')) {
      this.eventEmitter.emit('notification.created', notification);
    }

    await Promise.allSettled(deliveries);
  }

  private renderTemplate(
    template: NotificationTemplate,
    data: Record<string, any>
  ): RenderedNotification {
    const render = (text: string) => {
      return text.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] || '');
    };

    return {
      subject: render(template.subject),
      body: render(template.body),
      emailHtml: render(template.emailHtml),
      emailText: render(template.emailText)
    };
  }
}
```

---

## 7. Database Design

### 7.1 Core Schema

```sql
-- Organisations
CREATE TABLE organisations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    subscription_tier VARCHAR(50) NOT NULL DEFAULT 'STARTER',
    status VARCHAR(50) NOT NULL DEFAULT 'ACTIVE',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Brands
CREATE TABLE brands (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(organisation_id, name)
);

-- Locations
CREATE TABLE locations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID NOT NULL REFERENCES brands(id),
    name VARCHAR(255) NOT NULL,
    address TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Users
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role VARCHAR(50) NOT NULL DEFAULT 'LEARNER',
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING',
    location_id UUID REFERENCES locations(id),
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP,
    UNIQUE(email)
);

-- Wines
CREATE TABLE wines (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    name VARCHAR(255) NOT NULL,
    producer VARCHAR(255),
    vintage INTEGER,
    region VARCHAR(100) NOT NULL,
    country CHAR(2) NOT NULL,
    wine_type VARCHAR(50) NOT NULL,
    grape_varieties TEXT[] NOT NULL,
    price_tier VARCHAR(50) NOT NULL,
    price DECIMAL(10,2),
    tasting_notes TEXT,
    food_pairings TEXT[],
    serving_temperature VARCHAR(50),
    decanting_time VARCHAR(50),
    status VARCHAR(50) NOT NULL DEFAULT 'DRAFT',
    version INTEGER NOT NULL DEFAULT 1,
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    published_at TIMESTAMP,
    published_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Modules
CREATE TABLE modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(50) NOT NULL,
    tier VARCHAR(20) NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0,
    estimated_time INTEGER, -- minutes
    status VARCHAR(50) NOT NULL DEFAULT 'DRAFT',
    version INTEGER NOT NULL DEFAULT 1,
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Lessons
CREATE TABLE lessons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    module_id UUID NOT NULL REFERENCES modules(id),
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0,
    duration INTEGER, -- minutes
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Module-Wine relationships
CREATE TABLE module_wines (
    module_id UUID REFERENCES modules(id),
    wine_id UUID REFERENCES wines(id),
    PRIMARY KEY (module_id, wine_id)
);

-- Quizzes
CREATE TABLE quizzes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    module_id UUID REFERENCES modules(id),
    title VARCHAR(255) NOT NULL,
    tier VARCHAR(20) NOT NULL,
    passing_score INTEGER NOT NULL DEFAULT 70,
    time_limit INTEGER, -- seconds
    status VARCHAR(50) NOT NULL DEFAULT 'DRAFT',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Questions
CREATE TABLE questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    quiz_id UUID NOT NULL REFERENCES quizzes(id),
    template_id VARCHAR(50),
    text TEXT NOT NULL,
    explanation TEXT,
    sort_order INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Question Options
CREATE TABLE question_options (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_id UUID NOT NULL REFERENCES questions(id),
    text TEXT NOT NULL,
    is_correct BOOLEAN NOT NULL DEFAULT FALSE,
    sort_order INTEGER NOT NULL DEFAULT 0
);

-- Question-Wine relationships
CREATE TABLE question_wines (
    question_id UUID REFERENCES questions(id),
    wine_id UUID REFERENCES wines(id),
    PRIMARY KEY (question_id, wine_id)
);

-- Scenarios
CREATE TABLE scenarios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID NOT NULL REFERENCES organisations(id),
    template_id VARCHAR(50),
    title VARCHAR(255) NOT NULL,
    tier VARCHAR(20) NOT NULL,
    persona JSONB NOT NULL,
    opening TEXT NOT NULL,
    estimated_time INTEGER,
    status VARCHAR(50) NOT NULL DEFAULT 'DRAFT',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Scenario Steps
CREATE TABLE scenario_steps (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    scenario_id UUID NOT NULL REFERENCES scenarios(id),
    step_number INTEGER NOT NULL,
    situation TEXT NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0
);

-- Scenario Choices
CREATE TABLE scenario_choices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    step_id UUID NOT NULL REFERENCES scenario_steps(id),
    text TEXT NOT NULL,
    points INTEGER NOT NULL DEFAULT 0,
    feedback TEXT,
    next_step_id UUID REFERENCES scenario_steps(id),
    is_ending BOOLEAN DEFAULT FALSE,
    sort_order INTEGER NOT NULL DEFAULT 0
);

-- User Progress
CREATE TABLE user_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    content_type VARCHAR(50) NOT NULL, -- 'lesson', 'quiz', 'scenario'
    content_id UUID NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'NOT_STARTED',
    score INTEGER,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, content_type, content_id)
);

-- Quiz Attempts
CREATE TABLE quiz_attempts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    quiz_id UUID NOT NULL REFERENCES quizzes(id),
    status VARCHAR(50) NOT NULL DEFAULT 'IN_PROGRESS',
    score INTEGER,
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    time_spent INTEGER -- seconds
);

-- Quiz Answers
CREATE TABLE quiz_answers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    attempt_id UUID NOT NULL REFERENCES quiz_attempts(id),
    question_id UUID NOT NULL REFERENCES questions(id),
    selected_option_id UUID REFERENCES question_options(id),
    is_correct BOOLEAN,
    answered_at TIMESTAMP DEFAULT NOW()
);

-- Certifications
CREATE TABLE certifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    tier VARCHAR(20) NOT NULL,
    achieved_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    certificate_url VARCHAR(500)
);

-- Audit Logs
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organisation_id UUID REFERENCES organisations(id),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    previous_state JSONB,
    new_state JSONB,
    metadata JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 7.2 Indexes

```sql
-- Performance indexes
CREATE INDEX idx_wines_org_status ON wines(organisation_id, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_wines_org_type ON wines(organisation_id, wine_type) WHERE deleted_at IS NULL;
CREATE INDEX idx_wines_org_region ON wines(organisation_id, region) WHERE deleted_at IS NULL;
CREATE INDEX idx_modules_org_tier ON modules(organisation_id, tier) WHERE deleted_at IS NULL;
CREATE INDEX idx_progress_user ON user_progress(user_id, content_type);
CREATE INDEX idx_attempts_user_quiz ON quiz_attempts(user_id, quiz_id);
CREATE INDEX idx_certifications_user ON certifications(user_id, tier);
CREATE INDEX idx_audit_org_date ON audit_logs(organisation_id, created_at DESC);
CREATE INDEX idx_audit_user_date ON audit_logs(user_id, created_at DESC);

-- Full-text search
CREATE INDEX idx_wines_search ON wines USING gin(
    to_tsvector('english', coalesce(name, '') || ' ' ||
    coalesce(producer, '') || ' ' || coalesce(region, ''))
);
```

### 7.3 Row-Level Security

```sql
-- Enable RLS
ALTER TABLE wines ENABLE ROW LEVEL SECURITY;
ALTER TABLE modules ENABLE ROW LEVEL SECURITY;
ALTER TABLE quizzes ENABLE ROW LEVEL SECURITY;
ALTER TABLE scenarios ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY tenant_isolation_wines ON wines
    USING (organisation_id = current_setting('app.current_organisation')::uuid);

CREATE POLICY tenant_isolation_modules ON modules
    USING (organisation_id = current_setting('app.current_organisation')::uuid);

CREATE POLICY tenant_isolation_users ON users
    USING (organisation_id = current_setting('app.current_organisation')::uuid);
```

---

## 8. Caching Design

### 8.1 Cache Keys

```typescript
// utils/cache-keys.ts
export const CacheKeys = {
  // Sessions
  session: (userId: string) => `session:${userId}`,
  tokenBlacklist: (jti: string) => `blacklist:${jti}`,

  // User data
  userPermissions: (userId: string) => `permissions:${userId}`,
  userProfile: (userId: string) => `user:${userId}`,

  // Content
  wine: (orgId: string, wineId: string) => `wine:${orgId}:${wineId}`,
  wineList: (orgId: string, hash: string) => `wines:${orgId}:${hash}`,
  module: (orgId: string, moduleId: string) => `module:${orgId}:${moduleId}`,
  curriculum: (orgId: string) => `curriculum:${orgId}`,

  // Progress
  userProgress: (userId: string) => `progress:${userId}`,

  // Organisation
  orgConfig: (orgId: string) => `org:${orgId}:config`,
  orgStats: (orgId: string) => `org:${orgId}:stats`,

  // Locks
  importLock: (orgId: string) => `lock:import:${orgId}`,
  generationLock: (orgId: string) => `lock:generation:${orgId}`,

  // Rate limiting
  rateLimit: (key: string) => `ratelimit:${key}`,
  lockout: (userId: string) => `lockout:${userId}`
};

export const CacheTTL = {
  session: 8 * 60 * 60,      // 8 hours
  tokenBlacklist: 15 * 60,   // 15 minutes
  content: 60 * 60,          // 1 hour
  curriculum: 24 * 60 * 60,  // 24 hours
  orgConfig: 60 * 60,        // 1 hour
  progress: 5 * 60           // 5 minutes
};
```

### 8.2 Cache Service

```typescript
// services/cache.service.ts
export class CacheService {
  constructor(private redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const data = JSON.stringify(value);
    if (ttl) {
      await this.redis.setex(key, ttl, data);
    } else {
      await this.redis.set(key, data);
    }
  }

  async del(key: string): Promise<void> {
    await this.redis.del(key);
  }

  async delPattern(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }

  async invalidateWine(wineId: string, orgId: string): Promise<void> {
    await this.del(CacheKeys.wine(orgId, wineId));
    await this.delPattern(CacheKeys.wineList(orgId, '*'));
  }

  async invalidateCurriculum(orgId: string): Promise<void> {
    await this.del(CacheKeys.curriculum(orgId));
    await this.delPattern(`module:${orgId}:*`);
  }
}
```

---

## 9. Queue Design

### 9.1 Job Types

```typescript
// types/queue.ts
enum JobType {
  // Content generation
  GENERATE_CURRICULUM = 'generation.curriculum',
  GENERATE_QUESTIONS = 'generation.questions',
  GENERATE_SCENARIOS = 'generation.scenarios',

  // Import
  PROCESS_IMPORT = 'import.process',

  // Notifications
  SEND_EMAIL = 'notification.email',
  SEND_DIGEST = 'notification.digest',

  // Maintenance
  CLEANUP_DELETED = 'maintenance.cleanup',
  ARCHIVE_OLD_DATA = 'maintenance.archive',
  REFRESH_STATS = 'maintenance.stats'
}

interface Job<T = any> {
  id: string;
  type: JobType;
  data: T;
  priority: number;
  attempts: number;
  maxAttempts: number;
  createdAt: Date;
  scheduledAt?: Date;
}
```

### 9.2 Queue Service

```typescript
// services/queue.service.ts
export class QueueService {
  private readonly BACKOFF_DELAYS = [60, 300, 900, 1800, 3600]; // seconds

  async enqueue<T>(job: Omit<Job<T>, 'id' | 'attempts' | 'createdAt'>): Promise<string> {
    const fullJob: Job<T> = {
      ...job,
      id: generateUuid(),
      attempts: 0,
      maxAttempts: job.maxAttempts || 3,
      createdAt: new Date()
    };

    if (job.scheduledAt && job.scheduledAt > new Date()) {
      await this.redis.zadd(
        'jobs:scheduled',
        job.scheduledAt.getTime(),
        JSON.stringify(fullJob)
      );
    } else {
      await this.redis.lpush(
        `jobs:${job.type}`,
        JSON.stringify(fullJob)
      );
    }

    return fullJob.id;
  }

  async process(type: JobType, handler: JobHandler): Promise<void> {
    while (true) {
      const data = await this.redis.brpop(`jobs:${type}`, 30);
      if (!data) continue;

      const job: Job = JSON.parse(data[1]);

      try {
        await handler(job);
        await this.markComplete(job.id);
      } catch (error) {
        await this.handleFailure(job, error);
      }
    }
  }

  private async handleFailure(job: Job, error: Error): Promise<void> {
    job.attempts++;

    if (job.attempts >= job.maxAttempts) {
      await this.markFailed(job.id, error);
      await this.notifyFailure(job, error);
    } else {
      const delay = this.BACKOFF_DELAYS[
        Math.min(job.attempts - 1, this.BACKOFF_DELAYS.length - 1)
      ];
      await this.reschedule(job, delay);
    }
  }
}
```

---

## 10. Error Handling

### 10.1 Error Classes

```typescript
// utils/errors.ts
export abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly code: string;

  constructor(
    message: string,
    public readonly details?: Record<string, any>
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      error: {
        code: this.code,
        message: this.message,
        details: this.details
      }
    };
  }
}

export class ValidationError extends AppError {
  readonly statusCode = 400;
  readonly code = 'VALIDATION_ERROR';
}

export class UnauthorizedError extends AppError {
  readonly statusCode = 401;
  readonly code = 'UNAUTHORIZED';
}

export class ForbiddenError extends AppError {
  readonly statusCode = 403;
  readonly code = 'FORBIDDEN';
}

export class NotFoundError extends AppError {
  readonly statusCode = 404;
  readonly code = 'NOT_FOUND';
}

export class ConflictError extends AppError {
  readonly statusCode = 409;
  readonly code = 'CONFLICT';
}

export class RateLimitError extends AppError {
  readonly statusCode = 429;
  readonly code = 'RATE_LIMIT_EXCEEDED';
}

export class InternalError extends AppError {
  readonly statusCode = 500;
  readonly code = 'INTERNAL_ERROR';
}
```

### 10.2 Error Handler Middleware

```typescript
// middleware/error-handler.ts
export const errorHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Log error
  logger.error({
    type: 'error',
    requestId: req.requestId,
    error: {
      name: error.name,
      message: error.message,
      stack: error.stack
    },
    userId: req.user?.id,
    path: req.path
  });

  // Handle AppError
  if (error instanceof AppError) {
    res.status(error.statusCode).json({
      error: {
        code: error.code,
        message: error.message,
        details: error.details,
        requestId: req.requestId
      }
    });
    return;
  }

  // Handle unknown errors
  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
      requestId: req.requestId
    }
  });
};
```

---

## 11. Testing Strategy

### 11.1 Unit Tests

```typescript
// __tests__/services/question-generator.test.ts
describe('QuestionGenerator', () => {
  let generator: QuestionGenerator;
  let mockWines: Wine[];

  beforeEach(() => {
    generator = new QuestionGenerator();
    mockWines = createMockWines(10);
  });

  describe('generateQuestions', () => {
    it('should generate correct number of questions', async () => {
      const questions = await generator.generateQuestions(
        mockWines, Tier.BRONZE, 10
      );
      expect(questions).toHaveLength(10);
    });

    it('should respect tier difficulty', async () => {
      const bronzeQuestions = await generator.generateQuestions(
        mockWines, Tier.BRONZE, 5
      );
      const goldQuestions = await generator.generateQuestions(
        mockWines, Tier.GOLD, 5
      );

      bronzeQuestions.forEach(q => {
        expect(['QT-ID-001', 'QT-ID-002', 'QT-PA-001']).toContain(q.templateId);
      });
    });

    it('should ensure wine coverage', async () => {
      const questions = await generator.generateQuestions(
        mockWines, Tier.BRONZE, 20
      );
      const wineIds = new Set(questions.flatMap(q => q.wineIds));
      expect(wineIds.size).toBeGreaterThanOrEqual(mockWines.length * 0.5);
    });
  });

  describe('_generateDistractors', () => {
    it('should generate 3 distractors', async () => {
      const distractors = await generator['_generateDistractors'](
        'Bordeaux', 'region', mockWines[0], mockWines
      );
      expect(distractors).toHaveLength(3);
    });

    it('should not include correct answer', async () => {
      const distractors = await generator['_generateDistractors'](
        'Bordeaux', 'region', mockWines[0], mockWines
      );
      expect(distractors).not.toContain('Bordeaux');
    });
  });
});
```

### 11.2 Integration Tests

```typescript
// __tests__/integration/wine-import.test.ts
describe('Wine Import Flow', () => {
  let app: Express;
  let authToken: string;

  beforeAll(async () => {
    app = await createTestApp();
    authToken = await getTestToken('MANAGER');
  });

  beforeEach(async () => {
    await cleanDatabase();
    await seedTestOrganisation();
  });

  it('should import valid CSV and trigger curriculum generation', async () => {
    // 1. Upload CSV
    const uploadResponse = await request(app)
      .post('/api/v1/imports/wines')
      .set('Authorization', `Bearer ${authToken}`)
      .attach('file', 'fixtures/valid-wines.csv');

    expect(uploadResponse.status).toBe(200);
    expect(uploadResponse.body.importId).toBeDefined();
    expect(uploadResponse.body.preview.validRows).toBe(10);

    // 2. Confirm import
    const confirmResponse = await request(app)
      .post(`/api/v1/imports/${uploadResponse.body.importId}/confirm`)
      .set('Authorization', `Bearer ${authToken}`);

    expect(confirmResponse.status).toBe(200);
    expect(confirmResponse.body.imported).toBe(10);

    // 3. Verify wines created
    const winesResponse = await request(app)
      .get('/api/v1/wines')
      .set('Authorization', `Bearer ${authToken}`);

    expect(winesResponse.body.data).toHaveLength(10);

    // 4. Verify curriculum job queued
    const job = await getQueuedJob(JobType.GENERATE_CURRICULUM);
    expect(job).toBeDefined();
  });
});
```

### 11.3 Coverage Requirements

| Component | Target | Measured By |
|-----------|--------|-------------|
| Services | 80% | Line coverage |
| Controllers | 70% | Line coverage |
| Repositories | 60% | Line coverage |
| Utils | 90% | Line coverage |
| Learning Engine | 85% | Line coverage |

---

## 12. Appendices

### 12.1 API Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | State conflict |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |

### 12.2 Enumerations

```typescript
enum ContentStatus {
  DRAFT = 'DRAFT',
  REVIEW = 'REVIEW',
  APPROVED = 'APPROVED',
  PUBLISHED = 'PUBLISHED',
  ARCHIVED = 'ARCHIVED'
}

enum UserRole {
  LEARNER = 'LEARNER',
  MANAGER = 'MANAGER',
  CONTENT_AUTHOR = 'CONTENT_AUTHOR',
  CONTENT_ADMIN = 'CONTENT_ADMIN',
  DOMAIN_EXPERT = 'DOMAIN_EXPERT',
  ORG_ADMIN = 'ORG_ADMIN',
  SYSTEM_ADMIN = 'SYSTEM_ADMIN'
}

enum Tier {
  BRONZE = 'BRONZE',
  SILVER = 'SILVER',
  GOLD = 'GOLD'
}

enum WineType {
  RED = 'RED',
  WHITE = 'WHITE',
  ROSE = 'ROSE',
  SPARKLING = 'SPARKLING',
  DESSERT = 'DESSERT',
  FORTIFIED = 'FORTIFIED'
}

enum PriceTier {
  ENTRY = 'ENTRY',
  PREMIUM = 'PREMIUM',
  FINE = 'FINE',
  LUXURY = 'LUXURY'
}
```

### 12.3 Reference Documents

| Document | ID | Relevance |
|----------|-----|-----------|
| High-Level Design | SS-WS3-HLD | Architecture context |
| EARS Specification | SS-WS3-EARS | Requirements traceability |
| Content Domain Model | SS-WS3.0-CDM | Entity definitions |
| Content Lifecycle | SS-WS3.0-CLS | Workflow states |
| Generation Rules | SS-WS3.0-LE-CGR | Algorithm templates |
| Content Mapping | SS-WS3.0-LE-CLM | Transformation rules |

### 12.4 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-20 | Obi Wan | Initial draft |

---

*End of Low-Level Design Document*

**CONFIDENTIAL — Sommelier Spark**
