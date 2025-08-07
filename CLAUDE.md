# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack **Retrieval-Augmented Generation (RAG) chatbot system** for course materials. The system uses ChromaDB for vector storage, Anthropic's Claude for AI generation with tool calling, and provides a web interface for user interaction.

## Development Commands

### Setup and Dependencies
```bash
uv sync                          # Install Python dependencies
echo "ANTHROPIC_API_KEY=sk-..." > .env  # Set up environment variables
```

### Running the Application
```bash
./run.sh                         # Quick start (recommended)
# OR manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture Overview

### Core RAG Architecture
The system uses a **tool-based RAG approach** where Claude dynamically decides when to search for information:

1. **RAG Orchestrator** (`rag_system.py`) - Central coordinator managing all components
2. **AI Generator** (`ai_generator.py`) - Claude integration with tool calling capability  
3. **Search Tools** (`search_tools.py`) - Extensible tool system for course content search
4. **Vector Store** (`vector_store.py`) - ChromaDB with dual-collection design
5. **Document Processor** (`document_processor.py`) - Course material chunking and metadata extraction

### Key Architectural Decisions

**Tool-Based RAG**: Uses Claude's tool calling instead of traditional retrieve-then-generate. The AI decides when to search based on query context.

**Dual-Collection Vector Store**: 
- `course_catalog` - Course metadata and titles for semantic course name resolution
- `course_content` - Actual course chunks with lesson-level filtering

**Session Management**: Maintains conversation context with configurable history limits via `session_manager.py`.

## Component Interactions

### Query Flow
1. Frontend (`script.js`) sends POST to `/api/query`
2. FastAPI (`app.py`) validates request and calls RAG system
3. RAG system (`rag_system.py`) generates response using Claude + tools
4. Claude may invoke `CourseSearchTool` for content retrieval
5. Vector store performs semantic search with course/lesson filtering
6. Response assembled with sources and returned to frontend

### Document Processing
- Course documents in `/docs/` are automatically loaded on startup
- Documents parsed for metadata (title, instructor, lessons)
- Content chunked with sentence-aware boundaries (800 chars, 100 overlap)
- Stored in ChromaDB with embeddings using `all-MiniLM-L6-v2`

## Configuration

### Environment Variables (.env)
```bash
ANTHROPIC_API_KEY=your_key_here  # Required for Claude API
```

### Key Configuration Points (`config.py`)
- Chunk size/overlap for document processing
- Maximum results returned from vector search
- Claude model selection and parameters
- ChromaDB storage path

## Frontend Architecture

**Single-page application** with vanilla JavaScript:
- Real-time chat interface with markdown rendering
- Course statistics sidebar with collapsible sections
- Session management and loading states
- Source attribution with collapsible details

## Development Notes

### Adding New Search Tools
1. Inherit from `Tool` class in `search_tools.py`
2. Implement `get_tool_definition()` and `execute()` methods
3. Register tool in RAG system initialization

### Document Format
Course documents should follow this structure:
```
Course Title: [Title]
Instructor: [Name]
Course Link: [URL]

Lesson 1: [Title]
Link: [URL]
[Content...]

Lesson 2: [Title]
Link: [URL]
[Content...]
```

### Vector Store Collections
- Use `add_course_metadata()` for course information
- Use `add_course_content()` for searchable content chunks
- Search with `search(query, course_name, lesson_number)` for filtering

### API Response Format
All API responses use Pydantic models:
- `QueryResponse`: answer, sources, session_id
- `CourseStats`: total_courses, course_titles

The system automatically handles session creation, source tracking, and conversation history management.