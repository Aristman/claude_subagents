# Agent Profile: Backend Python

## Profile Identity

- Profile ID: python
- Domain: backend
- Stack: Python
- Extends: `backend/AGENT_PROFILE_backend-base.md`
- Scope: Python applications with Django or FastAPI
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent

This profile is **mandatory** for any feature assigned to the `backend` domain with **Python** stack.

---

## Inheritance

This profile extends `backend/AGENT_PROFILE_backend-base.md` with Python specific rules.

General backend rules (layered architecture, error handling, testing, observability) still apply.

---

## Supported Languages and Frameworks

**Mandatory:**
- Runtime: Python 3.11+ (3.12+ recommended)
- Framework: Django 5.x OR FastAPI 0.100+
- Package manager: Poetry (preferred) or uv
- Type checking: mypy (strict mode)

**The agent MUST NOT:**
- Use Python < 3.11
- Mix frameworks (Django + FastAPI) without explicit reason
- Use pip without virtual environment

---

## Project Structure

### Django Structure

```
project/
├── config/              # Django settings, urls, wsgi
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── test.py
│   ├── urls.py
│   └── wsgi.py
├── apps/                # Django applications
│   ├── notes/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   ├── services.py
│   │   └── permissions.py
│   └── auth/
├── manage.py
└── pyproject.toml
```

### FastAPI Structure

```
project/
├── app/
│   ├── main.py         # FastAPI app initialization
│   ├── config/         # Settings
│   ├── api/            # API routes
│   │   ├── v1/
│   │   │   ├── notes.py
│   │   │   ├── auth.py
│   │   │   └── dependencies.py
│   │   └── deps.py
│   ├── models/         # SQLAlchemy models
│   ├── schemas/        # Pydantic schemas
│   ├── services/       # Business logic
│   ├── core/           # Security, database
│   │   ├── security.py
│   │   ├── database.py
│   │   └── config.py
│   └── exceptions/     # Custom exceptions
├── alembic/            # Database migrations
├── tests/
└── pyproject.toml
```

---

## Django Specific Patterns

### Models

```python
# apps/notes/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Note(models.Model):
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='notes',
        db_index=True
    )
    title = models.CharField(max_length=200)
    content = models.TextField(blank=True, null=True)
    color = models.CharField(
        max_length=20,
        choices=[
            ('RED', 'Red'),
            ('ORANGE', 'Orange'),
            # ...
        ],
        default='GRAY'
    )
    tags = models.ManyToManyField(
        'tags.Tag',
        blank=True,
        related_name='notes'
    )
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['user', '-created_at']),
            models.Index(fields=['created_at']),
        ]

    def __str__(self):
        return self.title
```

### Serializers

```python
# apps/notes/serializers.py
from rest_framework import serializers
from .models import Note
from apps.tags.serializers import TagSerializer

class NoteSerializer(serializers.ModelSerializer):
    tags = TagSerializer(many=True, read_only=True)
    tag_ids = serializers.ListField(
        child=serializers.IntegerField(),
        write_only=True,
        required=False
    )

    class Meta:
        model = Note
        fields = ['id', 'title', 'content', 'color', 'tags', 'tag_ids', 'created_at', 'updated_at']
        read_only_fields = ['id', 'created_at', 'updated_at']

    def create(self, validated_data):
        tag_ids = validated_data.pop('tag_ids', [])
        note = Note.objects.create(**validated_data)
        if tag_ids:
            note.tags.set(tag_ids)
        return note
```

### Views

```python
# apps/notes/views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import Note
from .serializers import NoteSerializer
from .services import NoteService

class NoteViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = NoteSerializer

    def get_queryset(self):
        return Note.objects.filter(user=self.request.user)

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

    @action(detail=False, methods=['get'])
    def search(self, request):
        query = request.query_params.get('q', '')
        notes = NoteService.search_notes(request.user, query)
        serializer = self.get_serializer(notes, many=True)
        return Response(serializer.data)
```

### Services

```python
# apps/notes/services.py
from typing import List, Optional
from .models import Note

class NoteService:
    @staticmethod
    def search_notes(user, query: str) -> List[Note]:
        return Note.objects.filter(
            user=user,
            title__icontains=query
        ).distinct()

    @staticmethod
    def get_user_note_count(user) -> int:
        return Note.objects.filter(user=user).count()
```

---

## FastAPI Specific Patterns

### Pydantic Schemas

```python
# app/schemas/notes.py
from pydantic import BaseModel, Field, ConfigDict
from datetime import datetime
from typing import Optional, List
from enum import Enum

class NoteColor(str, Enum):
    RED = 'RED'
    ORANGE = 'ORANGE'
    YELLOW = 'YELLOW'
    GREEN = 'GREEN'
    BLUE = 'BLUE'
    PURPLE = 'PURPLE'
    PINK = 'PINK'
    GRAY = 'GRAY'

class NoteBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: Optional[str] = Field(None, max_length=10000)
    color: NoteColor = NoteColor.GRAY

class NoteCreate(NoteBase):
    tag_ids: List[int] = []

class NoteUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    content: Optional[str] = Field(None, max_length=10000)
    color: Optional[NoteColor] = None

class TagSchema(BaseModel):
    id: int
    name: str

    model_config = ConfigDict(from_attributes=True)

class NoteResponse(NoteBase):
    id: int
    tags: List[TagSchema] = []
    created_at: datetime
    updated_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### Models (SQLAlchemy)

```python
# app/models/note.py
from sqlalchemy import Column, Integer, String, Text, Enum as SQLEnum, ForeignKey, DateTime
from sqlalchemy.orm import Mapped, mapped_column, relationship
from datetime import datetime
from app.core.database import Base

class Note(Base):
    __tablename__ = "notes"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    content: Mapped[str | None] = mapped_column(Text, nullable=True)
    color: Mapped[str] = mapped_column(String(20), default='GRAY')
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
    updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    user: Mapped["User"] = relationship("User", back_populates="notes")
    tags: Mapped[List["Tag"]] = relationship("Tag", secondary="note_tags", back_populates="notes")

    __table_args__ = (
        Index('idx_notes_user_created', 'user_id', 'created_at'),
    )
```

### Routes

```python
# app/api/v1/notes.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List

from app.api.deps import get_current_user, get_db
from app.schemas.notes import NoteCreate, NoteUpdate, NoteResponse
from app.services.note_service import NoteService

router = APIRouter()

@router.get("/", response_model=List[NoteResponse])
async def get_notes(
    current_user = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    notes = await NoteService.get_user_notes(db, current_user.id)
    return notes

@router.post("/", response_model=NoteResponse, status_code=status.HTTP_201_CREATED)
async def create_note(
    note_in: NoteCreate,
    current_user = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    note = await NoteService.create_note(db, note_in, current_user.id)
    return note

@router.get("/{note_id}", response_model=NoteResponse)
async def get_note(
    note_id: int,
    current_user = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    note = await NoteService.get_note(db, note_id, current_user.id)
    if not note:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Note not found"
        )
    return note
```

### Services

```python
# app/services/note_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from typing import List, Optional

from app.models.note import Note
from app.schemas.notes import NoteCreate, NoteUpdate

class NoteService:
    @staticmethod
    async def get_user_notes(db: AsyncSession, user_id: int) -> List[Note]:
        result = await db.execute(
            select(Note)
            .where(Note.user_id == user_id)
            .order_by(Note.created_at.desc())
        )
        return result.scalars().all()

    @staticmethod
    async def get_note(db: AsyncSession, note_id: int, user_id: int) -> Optional[Note]:
        result = await db.execute(
            select(Note)
            .where(Note.id == note_id, Note.user_id == user_id)
        )
        return result.scalar_one_or_none()

    @staticmethod
    async def create_note(db: AsyncSession, note_in: NoteCreate, user_id: int) -> Note:
        note = Note(
            user_id=user_id,
            title=note_in.title,
            content=note_in.content,
            color=note_in.color.value
        )
        db.add(note)
        await db.commit()
        await db.refresh(note)
        return note
```

---

## Python Specific Rules

**The agent MUST:**
- Use type hints for all function parameters and returns
- Follow PEP 8 style guide (use Black formatter)
- Use `dataclasses` for data containers
- Use `async`/`await` for I/O operations (FastAPI)
- Use context managers for resources
- Follow explicit over implicit

**The agent MUST NOT:**
- Use `type: ignore` without justification
- Mix tabs and spaces
- Use wildcard imports (`from module import *`)
- Use mutable default arguments
- Ignore mypy errors

---

## Security

**Django:**
- Use built-in authentication system
- Use `@login_required` decorator
- Use DRF permissions
- Set `DEBUG = False` in production
- Use `ALLOWED_HOSTS`
- Use `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE` in production

**FastAPI:**
- Use OAuth2 with JWT (password flow)
- Hash passwords with bcrypt/passlib
- Use dependency injection for auth
- Implement CORS properly
- Use `python-jose` for JWT
- Use `passlib` for password hashing

**Common:**
- Validate ALL input
- Use environment variables for secrets
- Never log sensitive data
- Use parameterized queries (SQLAlchemy ORM does this)
- Keep dependencies updated

---

## Error Handling

**The agent MUST:**
- Create custom exception classes
- Use middleware for exception handling
- Return standardized error responses

```python
# app/core/exceptions.py
class AppException(Exception):
    def __init__(self, status_code: int, message: str):
        self.status_code = status_code
        self.message = message

class NotFoundError(AppException):
    def __init__(self, resource: str, id: int):
        super().__init__(404, f"{resource} with id {id} not found")

class ConflictError(AppException):
    def __init__(self, message: str):
        super().__init__(409, message)

# FastAPI exception handler
# app/core/exceptions_handlers.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "timestamp": datetime.utcnow().isoformat(),
            "status": exc.status_code,
            "error": exc.__class__.__name__,
            "message": exc.message,
            "path": request.url.path,
        }
    )
```

---

## Testing Strategy

**Framework:** pytest + pytest-asyncio (for FastAPI)

**Unit Tests:**
```python
# tests/services/test_note_service.py
import pytest
from app.services.note_service import NoteService

@pytest.mark.asyncio
async def test_get_note_found(db_session, test_user):
    # Arrange
    note = await NoteService.create_note(
        db_session,
        NoteCreate(title="Test"),
        test_user.id
    )

    # Act
    result = await NoteService.get_note(db_session, note.id, test_user.id)

    # Assert
    assert result is not None
    assert result.title == "Test"
```

**Integration Tests:**
```python
# tests/api/test_notes.py
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_note(auth_headers):
    response = client.post(
        "/api/v1/notes/",
        json={"title": "Test Note"},
        headers=auth_headers
    )
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Test Note"
```

**Coverage targets:**
- Services: 90%+
- API endpoints: 80%+

---

## Non-Functional Requirements

**Performance:**
- Use database connection pooling
- Implement caching (Redis) where appropriate
- Use select_related/prefetch_related (Django) to avoid N+1
- Use async/await (FastAPI) for concurrent operations

**Observability:**
- Use structlog for structured logging
- Log important operations (login, errors)
- Use Prometheus metrics (prometheus-fastapi-instrumentator)

**Code Quality:**
- Use Black for formatting
- Use Ruff for fast linting
- Use mypy for type checking
- Use pre-commit hooks

---

## Dependency Management

**The agent MUST:**
- Use Poetry (preferred) or uv
- Lock dependency versions
- Specify Python version in pyproject.toml
- Run `poetry check` or `uv check`

**The agent MUST NOT:**
- Use pip without virtual environment
- Ignore security vulnerabilities
- Add dependencies without justification

---

## Forbidden Practices (Python Specific)

The agent MUST NOT:

- Use `from module import *`
- Use mutable default arguments (`def foo(items=[])`)
- Mix tabs and spaces
- Ignore type hints
- Use `print()` for logging
- Suppress exceptions with bare `except:`
- Use `os.system` or `subprocess.call` without validation
- Hard-code configuration values
- Ignore async/await in FastAPI
- Use `global` variables
- Import inside functions (unless lazy loading)

---

## Profile Authority

This profile:
- Extends `backend/AGENT_PROFILE_backend-base.md`
- Is specific to Python projects (Django or FastAPI)
- Overrides general backend rules where Python has specific patterns
- Is subordinate only to `ARCHITECTURE_OVERVIEW.md` and explicit roadmap instructions

Failure to comply with this profile is grounds for rejection during review or verification.

---

## Version

- **Version:** 1.0
- **Date:** 2025-01-24
- **Extends:** backend-base v1.0
