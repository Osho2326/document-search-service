# Google Drive Document Search Service

A production-ready document search service that indexes files from Google Drive and provides full-text search capabilities using Elasticsearch.

## 📋 Project Overview

This project implements a document search service that:
- Connects to Google Drive using OAuth 2.0
- Extracts text from multiple file formats (.txt, .csv, .pdf, .png)
- Indexes documents in Elasticsearch for fast full-text search
- Provides a REST API for searching and syncing documents
- Follows clean OOP principles with modular architecture

## ✨ Features Implemented (Steps 1-10)

### Core Components

1. **Google Drive Integration** (`src/storage/gdrive_client.py`)
   - OAuth 2.0 authentication with token management
   - File listing with filtering by extension
   - File downloading and metadata retrieval
   - File existence checking

2. **Text Extraction** (`src/storage/text_extractor.py`)
   - CSV file parsing
   - Plain text with encoding detection
   - PDF extraction using PyPDF2
   - OCR for images using Tesseract
   - Apache Tika support (optional)

3. **Elasticsearch Integration** (`src/indexer/index_manager.py`)
   - Index creation with optimized mappings
   - Document indexing (single and bulk)
   - Full-text search with relevance scoring
   - Document deletion and statistics

4. **Sync Service** (`src/indexer/sync_service.py`)
   - Full synchronization of all Google Drive files
   - Single file sync
   - Deleted file cleanup
   - Progress tracking and statistics

5. **REST API** (`src/api/app.py`)
   - `GET /health` - Health check with Elasticsearch status
   - `GET /search?q=<query>` - Search documents
   - `POST /sync` - Trigger sync (all files or specific file)
   - `POST /sync/cleanup` - Cleanup deleted files
   - `GET /stats` - Index statistics

6. **Configuration Management** (`config/settings.py`)
   - Environment-based configuration
   - Validation for critical settings
   - Type-safe configuration access

7. **Logging** (`src/utils/logger.py`)
   - Colored console output
   - File logging with rotation
   - Appropriate log levels throughout

## 🏗 Architecture

```
┌─────────────┐
│   Client    │
│  (API Call) │
└──────┬──────┘
       │
┌──────▼──────┐
│  Flask API  │
│   Layer     │
└──────┬──────┘
       │
┌──────▼──────────────────┐
│   Business Logic        │
│  ┌──────┐  ┌─────────┐ │
│  │Search│  │  Sync   │ │
│  │Service  │Service  │ │
│  └──────┘  └─────────┘ │
└──────┬──────────┬───────┘
       │          │
┌──────▼──────┐  │
│Elasticsearch│  │
│   Index     │  │
└─────────────┘  │
                 │
        ┌────────▼────────┐
        │  Google Drive   │
        │  API & Storage  │
        └─────────────────┘
```

## 📦 Prerequisites

- Python 3.9+
- Elasticsearch 8.x
- Tesseract OCR (for image text extraction)
- Google Cloud Project with Drive API enabled
- OAuth 2.0 credentials (credentials.json)

## 🚀 Installation

### 1. Install Python Dependencies

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Install System Dependencies

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install tesseract-ocr tesseract-ocr-eng libmagic1
```

**macOS:**
```bash
brew install tesseract
```

**Windows:**
Download Tesseract from: https://github.com/UB-Mannheim/tesseract/wiki

### 3. Install Elasticsearch

**Using Docker (Recommended):**
```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  docker.elastic.co/elasticsearch/elasticsearch:8.11.1
```

**Verify Elasticsearch is running:**
```bash
curl http://localhost:9200
```

### 4. Setup Google Drive API

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable the Google Drive API:
   - Navigate to "APIs & Services" > "Library"
   - Search for "Google Drive API"
   - Click "Enable"
4. Create OAuth 2.0 Credentials:
   - Go to "APIs & Services" > "Credentials"
   - Click "Create Credentials" > "OAuth client ID"
   - Choose "Desktop app"
   - Download the credentials JSON file
5. Save the file as `credentials.json` in the project root

### 5. Configure Environment

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your settings (optional, defaults work for local development)
nano .env
```

**Key configuration options in `.env`:**
```env
# Google Drive
GOOGLE_CREDENTIALS_FILE=credentials.json
GOOGLE_TOKEN_FILE=token.json

# Elasticsearch
ELASTICSEARCH_HOST=localhost
ELASTICSEARCH_PORT=9200
ELASTICSEARCH_INDEX=gdrive_documents

# Flask API
FLASK_HOST=0.0.0.0
FLASK_PORT=5000
FLASK_DEBUG=False

# File Processing
MAX_FILE_SIZE_MB=100
SUPPORTED_EXTENSIONS=.txt,.csv,.pdf,.png,.jpg,.jpeg

# OCR
OCR_ENABLED=True
TESSERACT_CMD=/usr/bin/tesseract

# Logging
LOG_LEVEL=INFO
LOG_FILE=logs/app.log
```

## 🎯 Usage

### Starting the API Server

```bash
# Activate virtual environment
source venv/bin/activate

# Start the server
python -m src.api.app
```

The API will be available at `http://localhost:5000`

### First-Time Setup: Authentication

On first run, you'll need to authenticate with Google Drive:

```bash
# Trigger authentication by starting a sync
curl -X POST http://localhost:5000/sync
```

This will:
1. Open a browser for Google authentication
2. Request permission to read your Drive files
3. Save the token to `token.json` for future use

## 📖 API Documentation

### 1. Health Check
Check API and Elasticsearch health.

```bash
curl http://localhost:5000/health
```

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2024-12-24T10:30:00.000000",
  "elasticsearch": {
    "connected": true,
    "index": "gdrive_documents",
    "document_count": 150
  }
}
```

### 2. Search Documents
Search for documents containing a query term.

```bash
curl "http://localhost:5000/search?q=test&size=10"
```

**Parameters:**
- `q` (required): Search query
- `size` (optional): Maximum results (default: 100)

**Response:**
```json
{
  "query": "test",
  "total_results": 2,
  "files": [
    {
      "file_id": "1abc...",
      "file_name": "test_document.txt",
      "web_view_link": "https://drive.google.com/file/d/1abc.../view",
      "web_content_link": "https://drive.google.com/uc?id=1abc...",
      "modified_time": "2024-01-15T10:30:00Z",
      "size": 2048,
      "relevance_score": 4.52
    }
  ]
}
```

### 3. Sync Files
Trigger synchronization from Google Drive.

**Sync all files:**
```bash
curl -X POST http://localhost:5000/sync
```

**Sync specific file:**
```bash
curl -X POST http://localhost:5000/sync \
  -H "Content-Type: application/json" \
  -d '{"file_id": "YOUR_FILE_ID"}'
```

**Response:**
```json
{
  "total_files": 150,
  "indexed": 10,
  "updated": 5,
  "deleted": 2,
  "skipped": 3,
  "failed": 0,
  "duration_seconds": 45.2
}
```

### 4. Cleanup Deleted Files
Remove documents from index that no longer exist in Google Drive.

```bash
curl -X POST http://localhost:5000/sync/cleanup
```

**Response:**
```json
{
  "status": "success",
  "deleted_count": 5,
  "message": "Removed 5 deleted files from index"
}
```

### 5. Get Statistics
Get Elasticsearch index statistics.

```bash
curl http://localhost:5000/stats
```

**Response:**
```json
{
  "document_count": 150,
  "index_size_bytes": 52428800,
  "index_name": "gdrive_documents"
}
```

## 📁 Project Structure

```
gdrive-search-service/
├── src/
│   ├── api/                    # Flask REST API
│   │   ├── __init__.py
│   │   └── app.py             # API endpoints and server
│   ├── indexer/                # Elasticsearch operations
│   │   ├── __init__.py
│   │   ├── index_manager.py   # ES index management
│   │   └── sync_service.py    # Sync orchestration
│   ├── storage/                # Google Drive & text extraction
│   │   ├── __init__.py
│   │   ├── gdrive_client.py   # Google Drive API client
│   │   └── text_extractor.py  # Multi-format text extraction
│   ├── utils/                  # Utilities
│   │   ├── __init__.py
│   │   └── logger.py          # Logging configuration
│   └── __init__.py
├── config/                     # Configuration
│   ├── __init__.py
│   └── settings.py            # Environment configuration
├── tests/                      # Unit tests
│   ├── __init__.py
│   ├── conftest.py            # Test fixtures
│   └── test_api.py            # API tests
├── docs/                       # Documentation
│   └── architecture.mermaid   # Architecture diagram
├── logs/                       # Log files (created at runtime)
├── requirements.txt            # Python dependencies
├── .env.example               # Environment template
├── .gitignore                 # Git ignore rules
├── Dockerfile                 # Docker image
├── docker-compose.yml         # Docker Compose config
└── README.md                  # This file
```

## 🔧 Configuration Options

### Supported File Types
By default, the following extensions are supported:
- `.txt` - Plain text files
- `.csv` - Comma-separated values
- `.pdf` - PDF documents (PyPDF2)
- `.png`, `.jpg`, `.jpeg` - Images (Tesseract OCR)

To modify supported types, edit `SUPPORTED_EXTENSIONS` in `.env`

### File Size Limits
Default maximum file size: 100MB

Adjust in `.env`:
```env
MAX_FILE_SIZE_MB=50
```

### OCR Configuration
Enable/disable OCR for images:
```env
OCR_ENABLED=True
TESSERACT_CMD=/usr/bin/tesseract
```

**3. OCR not working**
```bash
# Verify Tesseract installation
tesseract --version

# Test OCR manually
tesseract test_image.png output

# Reinstall dependencies
pip install -r requirements.txt
```

**5. Port already in use**
```bash
# Change port in .env
FLASK_PORT=5001

```
```

## Design Decisions

### Why Elasticsearch over PostgreSQL?
- Better full-text search capabilities
- Built-in relevance scoring
- Optimized for text search operations
- Scalable for large document collections

### Why PyPDF2?
- Simpler API than alternatives
- Adequate for most PDF extraction needs
- Lightweight with fewer dependencies
- Fallback to Tika available for complex PDFs

### Why Flask?
- Lightweight and simple
- Quick to set up
- Sufficient for the API requirements
- Well-documented and widely used

## 📝 Code Quality Features

- **OOP Design**: Clean class-based architecture
- **Type Hints**: Improved code clarity and IDE support
- **Docstrings**: Comprehensive documentation for all classes and methods
- **Error Handling**: Graceful error handling throughout
- **Logging**: Detailed logging at appropriate levels
- **Separation of Concerns**: Clear module boundaries
- **Configuration Management**: Centralized configuration
- **Testing**: Unit tests with fixtures


## ✅ Assessment Requirements Met

- ✅ Connects to Google Drive using API
- ✅ Supports .csv, .txt, .pdf, .png file formats
- ✅ Extracts text content with encoding detection
- ✅ OCR support for images (Tesseract)
- ✅ Elasticsearch indexing with full-text search
- ✅ REST API with search endpoint returning file URLs
- ✅ Clean, modular OOP code
- ✅ Proper error handling and logging
- ✅ Configuration management
- ✅ Documentation with README
- ✅ Architecture design included


