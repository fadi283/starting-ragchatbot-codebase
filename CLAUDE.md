# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Installation & Setup:**
- `uv sync` - Install dependencies from pyproject.toml
- `cp .env.example .env && edit .env` - Configure Anthropic API key
- `chmod +x run.sh && ./run.sh` - Start application using provided script

**Running the Application:**
- `cd backend && uv run uvicorn app:app --reload --port 8000` - Manual start with hot reload
- Application available at: http://localhost:8000
- API documentation: http://localhost:8000/docs

**Environment:**
- Requires Python 3.13+
- Uses uv as package manager (not pip)
- Windows users should use Git Bash for shell commands
- API key stored in .env file as ANTHROPIC_API_KEY

## Code Architecture

**High-Level Structure:**
- **Backend** (`/backend`): FastAPI application providing REST API and RAG logic
- **Frontend** (`/frontend`): Static web interface (HTML/CSS/JS) served by backend
- **Docs** (`/docs`): Course materials loaded into the RAG system on startup
- **Configuration**: pyproject.toml (dependencies), .env (environment variables)

**Backend Modules:**
1. **app.py** - FastAPI application setup:
   - API endpoints: `/api/query` (POST), `/api/courses` (GET)
   - Middleware: CORS, TrustedHost
   - Static file serving for frontend
   - Startup event loads documents from `/docs`

2. **rag_system.py** - Main RAG orchestrator:
   - Coordinates document processing, vector storage, AI generation
   - Manages conversation sessions
   - Integrates search tools for tool use with Claude
   - Handles document addition and querying

3. **document_processor.py** - Document parsing:
   - Expects specific format: Course Title/Link/Instructor headers
   - Parses lessons with optional lesson links
   - Creates semantic chunks with overlap (configurable)
   - Adds contextual metadata to chunks (course/lesson info)

4. **vector_store.py** - ChromaDB integration:
   - Stores course metadata and content chunks
   - Performs similarity search for retrieval
   - Manages collections for courses and content
   - Provides methods to add/search course data

5. **ai_generator.py** - Anthropic Claude integration:
   - Handles API calls to Claude with tool use
   - Manages conversation history
   - Processes tool results for final response generation

6. **search_tools.py** - Tool definitions:
   - CourseSearchTool for vector store searches
   - ToolManager to coordinate multiple tools
   - Source tracking for response attribution

7. **session_manager.py** - Conversation history:
   - Creates and manages chat sessions
   - Stores message history with limits
   - Retrieves context for AI generation

8. **models.py** - Pydantic data models:
   - Course, Lesson, CourseChunk structures
   - API request/response models

9. **config.py** - Configuration management:
   - Centralized settings (chunk sizes, model names, paths)
   - Loaded from environment variables with defaults

**Data Flow:**
1. User query → Frontend JS → POST `/api/query`
2. Backend validates request, ensures session exists
3. RAG system creates prompt with conversation history
4. Claude decides whether to use search tool
5. If tool used: vector store search → returns relevant chunks
6. Claude synthesizes final answer using retrieved context
7. Response returned to frontend with sources and session ID
8. Frontend displays answer with markdown rendering and source collapsibles

**Document Format Expectation:**
Course documents (in `/docs`) should follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]
Lesson 0: [lesson title]
[Lesson content...]
Lesson Link: [url]  (optional)
Lesson 1: [lesson title]
[Lesson content...]
```

**Key Configuration Values:**
- CHUNK_SIZE: Target size for text chunks (~500 chars)
- CHUNK_OVERLAP: Overlap between chunks (~50 chars)
- MAX_RESULTS: Number of search results to return
- EMBEDDING_MODEL: Sentence transformer model for vectorization
- CHROMA_PATH: Directory for persistent vector storage

**Extensibility Points:**
- Add new search tools by implementing BaseTool and registering with ToolManager
- Modify chunking strategy in DocumentProcessor.chunk_text()
- Change vector store implementation in VectorStore class
- Update AI generator parameters in AIGenerator