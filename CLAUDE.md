# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A full-stack RAG (Retrieval-Augmented Generation) chatbot for answering questions about course materials. Uses ChromaDB for vector storage, Anthropic's Claude for AI generation, and FastAPI for the backend.

## Commands

**Always use `uv` for package management, never `pip`.**

```bash
# Install dependencies
uv sync

# Run the application (starts on http://localhost:8000)
./run.sh

# Manual start with hot reload
cd backend && uv run uvicorn app:app --reload --port 8000
```

## Architecture

### Backend (Python/FastAPI)

The backend follows a layered architecture:

```
app.py (FastAPI endpoints)
    ↓
rag_system.py (RAG orchestrator)
    ↓
┌─────────────────┬─────────────────┬──────────────────┐
│ document_       │ vector_store.py │ ai_generator.py  │
│ processor.py    │ (ChromaDB)      │ (Claude API)     │
└─────────────────┴─────────────────┴──────────────────┘
```

**Key modules:**
- `app.py` - FastAPI server, serves frontend, handles `/api/query` and `/api/courses`
- `rag_system.py` - Orchestrates document loading, search, and generation
- `ai_generator.py` - Claude API integration with tool-use support
- `vector_store.py` - ChromaDB wrapper with two collections: `course_catalog` (metadata) and `course_content` (chunks)
- `document_processor.py` - Parses course documents, chunks text (800 chars, 100 overlap)
- `search_tools.py` - Tool definitions that Claude can invoke for semantic search
- `session_manager.py` - In-memory conversation history (last 2 exchanges per session)

### Frontend (Vanilla JS)

Single-page app in `/frontend/` with chat interface and course sidebar. No build step required.

### Document Format

Course documents in `/docs/` follow a specific format:
- Line 1: Course Title
- Line 2: Course Link
- Line 3: Instructor
- Subsequent lines: Content with "Lesson N: Title" markers

## Configuration

Environment variables (set in `.env`):
- `ANTHROPIC_API_KEY` - Required

Defaults in `config.py`:
- Model: `claude-sonnet-4-20250514`
- Embedding: `all-MiniLM-L6-v2`
- ChromaDB path: `./chroma_db`

## API Endpoints

- `POST /api/query` - Query the RAG system with `{query, session_id?}`
- `GET /api/courses` - Get course catalog stats
- `GET /` - Serve frontend

## Key Patterns

- Claude uses tool-use to invoke `search_course_content(query, course_name, lesson_number)` for semantic search
- Course name resolution uses fuzzy vector matching
- Sessions maintain conversation context in memory (not persisted)
- On startup, app auto-loads all documents from `/docs/` folder
