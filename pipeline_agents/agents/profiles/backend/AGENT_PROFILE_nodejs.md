# Agent Profile: Backend Node.js

## Profile Identity

- Profile ID: nodejs
- Domain: backend
- Stack: Node.js/TypeScript
- Extends: `backend/AGENT_PROFILE_backend-base.md`
- Scope: Node.js applications with TypeScript
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent

This profile is **mandatory** for any feature assigned to the `backend` domain with **Node.js + TypeScript** stack.

---

## Inheritance

This profile extends `backend/AGENT_PROFILE_backend-base.md` with Node.js specific rules.

General backend rules (layered architecture, error handling, testing, observability) still apply.

---

## Supported Languages and Frameworks

**Mandatory:**
- Runtime: Node.js 20 LTS (or 18 LTS)
- Language: TypeScript 5.x
- Framework: Express.js or Fastify
- Package manager: pnpm (preferred) or npm
- Build: tsup/esbuild or tsc

**The agent MUST NOT:**
- Use plain JavaScript (must use TypeScript)
- Mix CommonJS and ESM modules
- Use deprecated frameworks (Hapi, Koa without explicit reason)

---

## Project Structure (Standard)

```
src/
├── config/          # Configuration files
├── controllers/     # Request handlers
├── services/        # Business logic
├── repositories/    # Data access
├── models/          # Type definitions and schemas
├── middleware/      # Express middleware
├── routes/          # Route definitions
├── dto/             # Request/Response types
│   ├── request.ts
│   └── response.ts
├── errors/          # Custom error classes
├── utils/           # Utility functions
└── app.ts           # Application entry
```

**The agent MUST:**
- Place files in correct directories
- Use barrel exports (index.ts) for clean imports
- Follow TypeScript naming conventions

---

## Node.js Specific Patterns

### Controllers

```typescript
// controllers/NoteController.ts
import { Request, Response, NextFunction } from 'express';
import { NoteService } from '../services/NoteService';
import { NoteCreateRequestDto, NoteUpdateRequestDto } from '../dto/request';
import { asyncHandler } from '../utils/asyncHandler';

export class NoteController {
  constructor(private readonly noteService: NoteService) {}

  getAll = asyncHandler(async (req: Request, res: Response) => {
    const notes = await this.noteService.findAll();
    res.json(notes);
  });

  getById = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    const note = await this.noteService.findById(Number(id));
    if (!note) {
      throw new NotFoundError(`Note with id ${id} not found`);
    }
    res.json(note);
  });

  create = asyncHandler(async (req: Request, res: Response) => {
    const dto: NoteCreateRequestDto = req.body;
    const note = await this.noteService.create(dto);
    res.status(201).json(note);
  });

  update = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    const dto: NoteUpdateRequestDto = req.body;
    const note = await this.noteService.update(Number(id), dto);
    res.json(note);
  });

  delete = asyncHandler(async (req: Request, res: Response) => {
    const { id } = req.params;
    await this.noteService.delete(Number(id));
    res.status(204).send();
  });
}
```

**Rules:**
- Use async/await (no callback hell)
- Use `asyncHandler` wrapper for error handling
- Validate requests before passing to services
- Return appropriate HTTP status codes
- Use class-based controllers

### Services

```typescript
// services/NoteService.ts
import { NoteRepository } from '../repositories/NoteRepository';
import { TagRepository } from '../repositories/TagRepository';
import { NoteCreateRequestDto, NoteUpdateRequestDto } from '../dto/request';
import { NoteResponseDto } from '../dto/response';
import { NotFoundError } from '../errors/NotFoundError';

export class NoteService {
  constructor(
    private readonly noteRepository: NoteRepository,
    private readonly tagRepository: TagRepository,
  ) {}

  async findAll(): Promise<NoteResponseDto[]> {
    const notes = await this.noteRepository.findAll();
    return notes.map(note => this.toResponseDto(note));
  }

  async findById(id: number): Promise<NoteResponseDto | null> {
    const note = await this.noteRepository.findById(id);
    return note ? this.toResponseDto(note) : null;
  }

  async create(dto: NoteCreateRequestDto): Promise<NoteResponseDto> {
    const entity = this.toEntity(dto);
    const note = await this.noteRepository.create(entity);
    return this.toResponseDto(note);
  }

  async update(id: number, dto: NoteUpdateRequestDto): Promise<NoteResponseDto> {
    const existing = await this.noteRepository.findById(id);
    if (!existing) {
      throw new NotFoundError(`Note with id ${id} not found`);
    }
    const updated = { ...existing, ...dto };
    const note = await this.noteRepository.update(id, updated);
    return this.toResponseDto(note);
  }

  async delete(id: number): Promise<void> {
    const existing = await this.noteRepository.findById(id);
    if (!existing) {
      throw new NotFoundError(`Note with id ${id} not found`);
    }
    await this.noteRepository.delete(id);
  }

  private toResponseDto(note: any): NoteResponseDto {
    // Mapping logic
    return {
      id: note.id,
      title: note.title,
      content: note.content,
      // ...
    };
  }

  private toEntity(dto: NoteCreateRequestDto): any {
    // Mapping logic
    return {
      title: dto.title,
      content: dto.content,
      // ...
    };
  }
}
```

**Rules:**
- Business logic goes here, NOT in controllers
- Use dependency injection via constructor
- Keep methods focused and single-purpose
- Handle domain-specific validation

### Repositories

```typescript
// repositories/NoteRepository.ts
import { Pool } from 'pg'; // or other DB client
import { Note } from '../models/Note';

export class NoteRepository {
  constructor(private readonly db: Pool) {}

  async findAll(): Promise<Note[]> {
    const result = await this.db.query(
      'SELECT * FROM notes ORDER BY created_at DESC'
    );
    return result.rows;
  }

  async findById(id: number): Promise<Note | null> {
    const result = await this.db.query(
      'SELECT * FROM notes WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  }

  async create(note: Partial<Note>): Promise<Note> {
    const result = await this.db.query(
      'INSERT INTO notes (user_id, title, content, color) VALUES ($1, $2, $3, $4) RETURNING *',
      [note.userId, note.title, note.content, note.color]
    );
    return result.rows[0];
  }

  async update(id: number, note: Partial<Note>): Promise<Note> {
    const result = await this.db.query(
      'UPDATE notes SET title = $1, content = $2, color = $3, updated_at = NOW() WHERE id = $4 RETURNING *',
      [note.title, note.content, note.color, id]
    );
    return result.rows[0];
  }

  async delete(id: number): Promise<void> {
    await this.db.query('DELETE FROM notes WHERE id = $1', [id]);
  }
}
```

**Rules:**
- Use parameterized queries to prevent SQL injection
- Use query builder (Knex) or ORM (Prisma, TypeORM, Drizzle)
- Handle database errors appropriately
- Use transactions for multi-step operations

### Routes

```typescript
// routes/noteRoutes.ts
import { Router } from 'express';
import { NoteController } from '../controllers/NoteController';
import { authenticate } from '../middleware/auth';
import { validate } from '../middleware/validate';
import { noteCreateSchema, noteUpdateSchema } from '../validators/schemas';

export function createNoteRoutes(controller: NoteController): Router {
  const router = Router();

  router.get('/', controller.getAll);
  router.get('/:id', controller.getById);
  router.post('/', authenticate, validate(noteCreateSchema), controller.create);
  router.put('/:id', authenticate, validate(noteUpdateSchema), controller.update);
  router.delete('/:id', authenticate, controller.delete);

  return router;
}
```

---

## TypeScript Specific Rules

**The agent MUST:**
- Use strict mode in `tsconfig.json`
- Enable all strict checks (strictNullChecks, noImplicitAny, etc.)
- Use interfaces for public API, types for internal
- Prefer `const` assertions for literal types
- Use utility types (Partial, Pick, Omit, etc.)
- Use generic types for reusable components

**The agent MUST NOT:**
- Use `any` without strong justification
- Use `// @ts-ignore` to silence errors
- Ignore TypeScript compiler errors
- Use loose type definitions

---

## Database Guidelines

**For SQL databases (PostgreSQL, MySQL):**
- Use query builder (Knex.js) or ORM (Prisma, TypeORM, Drizzle)
- Use migrations (Knex, Prisma, or custom)
- Use connection pooling (pg-pool)
- Parameterize ALL queries

**For NoSQL (MongoDB):**
- Use ODM (Mongoose)
- Define schemas with validation
- Use indexes for query performance
- Use transactions for multi-document operations

**The agent MUST NOT:**
- Concatenate strings for queries (SQL injection risk)
- Forget to close connections
- Put business logic in database layer

---

## Security

**Authentication:**
- Use JWT with RS256 (asymmetric) or HS256 (symmetric)
- Store refresh tokens in database
- Use httpOnly cookies for token storage
- Implement token rotation for refresh tokens

**Authorization:**
- Use middleware for route protection
- Implement role-based access control (RBAC)
- Use least privilege principle

**Common vulnerabilities:**
- Validate ALL input
- Sanitize output (XSS prevention)
- Use Helmet.js for security headers
- Implement rate limiting
- Use CORS properly
- Keep dependencies updated

---

## Error Handling

**The agent MUST:**
- Create custom error classes extending Error
- Use error handling middleware
- Return standardized error responses

```typescript
// errors/AppError.ts
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message: string) {
    super(404, message);
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(409, message);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(401, message);
  }
}

export class ForbiddenError extends AppError {
  constructor(message: string = 'Forbidden') {
    super(403, message);
  }
}

// middleware/errorHandler.ts
export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void {
  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      timestamp: new Date().toISOString(),
      status: err.statusCode,
      error: err.name,
      message: err.message,
      path: req.path,
    });
    return;
  }

  // Unexpected errors
  console.error('Unexpected error:', err);
  res.status(500).json({
    timestamp: new Date().toISOString(),
    status: 500,
    error: 'Internal Server Error',
    message: 'An unexpected error occurred',
    path: req.path,
  });
}
```

---

## Validation

**The agent MUST:**
- Use validation library (Zod, Joi, Yup, class-validator)
- Validate request bodies, query params, route params
- Return validation errors as 400 with field messages

```typescript
// validators/schemas.ts
import { z } from 'zod';

export const noteCreateSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().max(10000).optional(),
  color: z.enum(['RED', 'ORANGE', 'YELLOW', 'GREEN', 'BLUE', 'PURPLE', 'PINK', 'GRAY']).default('GRAY'),
  tagIds: z.array(z.number()).optional(),
});

export const noteUpdateSchema = z.object({
  title: z.string().min(1).max(200).optional(),
  content: z.string().max(10000).optional(),
  color: z.enum(['RED', 'ORANGE', 'YELLOW', 'GREEN', 'BLUE', 'PURPLE', 'PINK', 'GRAY']).optional(),
}).refine(data => Object.keys(data).length > 0, {
  message: 'At least one field must be provided',
});

// middleware/validate.ts
export function validate(schema: z.ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      res.status(400).json({
        timestamp: new Date().toISOString(),
        status: 400,
        error: 'Validation Error',
        message: 'Invalid request data',
        details: result.error.errors,
        path: req.path,
      });
      return;
    }
    req.body = result.data;
    next();
  };
}
```

---

## Testing Strategy

**Unit Tests:**
- Use Vitest or Jest
- Mock external dependencies
- Test services in isolation
- Use test factories for test data

**Integration Tests:**
- Use Supertest for API testing
- Use test database (Docker or in-memory)
- Clean database between tests
- Test happy path and error cases

**Coverage targets:**
- Services: 90%+
- Controllers: 80%+
- Repositories: 70%+

```typescript
// services/NoteService.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { NoteService } from './NoteService';
import { NoteRepository } from '../repositories/NoteRepository';

describe('NoteService', () => {
  let service: NoteService;
  let mockRepo: NoteRepository;

  beforeEach(() => {
    mockRepo = {
      findById: vi.fn(),
      findAll: vi.fn(),
      create: vi.fn(),
      update: vi.fn(),
      delete: vi.fn(),
    } as any;
    service = new NoteService(mockRepo);
  });

  describe('findById', () => {
    it('should return a note when found', async () => {
      const mockNote = { id: 1, title: 'Test' };
      vi.mocked(mockRepo.findById).mockResolvedValue(mockNote);

      const result = await service.findById(1);

      expect(result).toEqual({ id: 1, title: 'Test' });
      expect(mockRepo.findById).toHaveBeenCalledWith(1);
    });

    it('should return null when not found', async () => {
      vi.mocked(mockRepo.findById).mockResolvedValue(null);

      const result = await service.findById(999);

      expect(result).toBeNull();
    });
  });
});
```

---

## Non-Functional Requirements

**Performance:**
- API response p95 < 500ms for CRUD
- API response p95 < 1000ms for search
- Use cluster mode for multi-core
- Implement caching where appropriate (Redis)

**Observability:**
- Use Winston or Pino for logging
- Structured logging (JSON)
- Correlation IDs for request tracing
- Expose metrics for Prometheus

**Code Quality:**
- Use ESLint with strict rules
- Use Prettier for formatting
- Enable type checking in CI
- Use Husky for pre-commit hooks

---

## Dependency Management

**The agent MUST:**
- Use pnpm (preferred) or npm
- Lock package versions
- Use `engines` field in package.json
- Regular dependency audits

**The agent MUST NOT:**
- Add dependencies without justification
- Ignore security vulnerabilities
- Use deprecated packages

---

## Forbidden Practices (Node.js Specific)

The agent MUST NOT:

- Use `var` (use `const` or `let`)
- Mix callback and promise patterns
- Use `any` type
- Ignore TypeScript errors
- Create circular dependencies
- Block event loop with synchronous operations
- Use `require()` in ESM projects
- Forget error handling in promises
- Use `console.log` (use logger)
- Hard-code configuration (use environment variables)

---

## Profile Authority

This profile:
- Extends `backend/AGENT_PROFILE_backend-base.md`
- Is specific to Node.js + TypeScript projects
- Overrides general backend rules where Node.js has specific patterns
- Is subordinate only to `ARCHITECTURE_OVERVIEW.md` and explicit roadmap instructions

Failure to comply with this profile is grounds for rejection during review or verification.

---

## Version

- **Version:** 1.0
- **Date:** 2025-01-24
- **Extends:** backend-base v1.0
