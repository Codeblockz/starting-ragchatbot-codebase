# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start - runs the full application
./run.sh

# Manual start - development mode with auto-reload
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package-name
```

### Environment Setup
- Create `.env` file in root with: `ANTHROPIC_API_KEY=your_key_here`
- Application runs on http://localhost:8000
- API docs available at http://localhost:8000/docs

## Architecture Overview

This is a **Course Materials RAG System** - a full-stack web application for intelligent querying of educational content using semantic search and AI generation.

### Core Component Flow
```
Document Files → DocumentProcessor → VectorStore → RAGSystem → AI Response
                     ↓                   ↓
            Text Chunking        ChromaDB Collections
```

### Key Components

**RAGSystem** (`rag_system.py`) - Main orchestrator that coordinates all components
- Initializes and manages all subsystems
- Handles document ingestion pipeline
- Processes user queries through the full RAG workflow

**DocumentProcessor** (`document_processor.py`) - Parses structured course documents
- Expects format: `Course Title: [title]`, `Course Link: [url]`, `Course Instructor: [instructor]`
- Identifies lesson markers: `Lesson [number]: [title]`
- Performs sentence-aware chunking with configurable overlap
- Adds contextual prefixes to chunks for better retrieval

**VectorStore** (`vector_store.py`) - Dual ChromaDB collection system
- `course_catalog`: Course metadata for semantic course name resolution
- `course_content`: Chunked content with lesson/course filtering
- Supports fuzzy course name matching and precise lesson filtering

**AIGenerator** (`ai_generator.py`) - Claude API integration for response generation
- Uses function calling for search tool integration
- Maintains conversation context through SessionManager

**SessionManager** (`session_manager.py`) - Conversation history management
- Tracks user questions and AI responses
- Configurable history length for context window management

### Search Architecture

The system uses a two-stage search approach:
1. **Course Resolution**: Semantic matching of user-provided course names against catalog
2. **Content Search**: Vector similarity search with optional course/lesson filtering

### Configuration

All settings centralized in `config.py`:
- `CHUNK_SIZE`: Text chunk size for vector storage (default: 800)
- `CHUNK_OVERLAP`: Character overlap between chunks (default: 100) 
- `MAX_RESULTS`: Maximum search results returned (default: 5)
- `MAX_HISTORY`: Conversation history length (default: 2)
- `CHROMA_PATH`: ChromaDB persistence location (default: "./chroma_db")

### Document Structure

Course documents must follow this format:
```
Course Title: Introduction to Python
Course Link: https://example.com/course
Course Instructor: Dr. Smith

Lesson 0: Getting Started
Lesson Link: https://example.com/lesson0
[lesson content...]

Lesson 1: Variables and Data Types
[lesson content...]
```

### Web Interface

- **Frontend**: Vanilla HTML/CSS/JS in `frontend/` directory
- **Backend**: FastAPI with automatic static file serving
- **API**: RESTful endpoints with OpenAPI documentation
- **CORS**: Configured for development with wildcard origins

### Data Flow

1. Course documents processed into structured Course/Lesson objects
2. Content chunked with lesson context preservation
3. Dual storage: metadata in catalog, content in vector collection
4. User queries trigger semantic search with optional filtering
5. Retrieved chunks provided to Claude for contextualized response generation
- always use uv to run the server. do not use pip directly