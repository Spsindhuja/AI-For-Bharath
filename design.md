# Bharat Content Transformer - Technical Design Document

## Project Overview

**Project Name:** Bharat Content Transformer  
**Version:** 1.0  
**Last Updated:** February 14, 2026  
**Status:** Hackathon Design

---

## System Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Flutter/React Web App (Responsive UI)                   │  │
│  │  - File Upload Interface                                 │  │
│  │  - Content Dashboard                                     │  │
│  │  - Preview & Download Components                         │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ HTTPS/REST API
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  FastAPI Backend (Python)                                │  │
│  │  - API Gateway & Routing                                 │  │
│  │  - Authentication & Authorization                        │  │
│  │  - Request Validation & Rate Limiting                    │  │
│  │  - Job Queue Management (Celery/Redis)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Processing Layer                           │
│  ┌──────────────┬──────────────┬──────────────┬─────────────┐  │
│  │ PDF Processor│Video Processor│AI Orchestrator│Media Gen   │  │
│  │ (PyPDF2)     │ (FFmpeg)      │ (Gemini API)  │(PIL/Canvas)│  │
│  └──────────────┴──────────────┴──────────────┴─────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          AI Layer                                │
│  ┌──────────────┬──────────────┬──────────────┬─────────────┐  │
│  │ Gemini API   │ Whisper ASR  │ XTTS Voice   │ Translation │  │
│  │ (Summary/    │ (Speech-to-  │ (Text-to-    │ (IndicTrans)│  │
│  │  Script Gen) │  Text)        │  Speech)     │             │  │
│  └──────────────┴──────────────┴──────────────┴─────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Storage Layer                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Firebase                                                 │  │
│  │  - Firestore (Metadata & User Data)                      │  │
│  │  - Cloud Storage (Files & Generated Content)             │  │
│  │  - Authentication (User Management)                      │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Frontend:**
- Framework: Flutter (mobile-first) OR React (web-first)
- State Management: Provider (Flutter) / Redux (React)
- HTTP Client: Dio (Flutter) / Axios (React)

**Backend:**
- Framework: FastAPI (Python 3.10+)
- Task Queue: Celery with Redis
- API Documentation: OpenAPI/Swagger (auto-generated)

**AI/ML Services:**
- Gemini API: Content summarization, script generation
- Whisper ASR: Audio transcription from videos
- XTTS: Multilingual text-to-speech
- IndicTrans2: Indic language translation

**Storage & Infrastructure:**
- Firebase Firestore: NoSQL database
- Firebase Cloud Storage: File storage
- Firebase Authentication: User management
- Redis: Caching and job queue

**Media Processing:**
- FFmpeg: Video/audio processing
- PyPDF2/pdfplumber: PDF text extraction
- Pillow (PIL): Image manipulation
- MoviePy: Video editing (optional)

---

## Component Breakdown

### 1. Frontend Application

#### 1.1 Flutter/React Components

**Core Modules:**
- `UploadModule`: File upload with drag-drop, progress tracking
- `DashboardModule`: User content library and history
- `ProcessingModule`: Real-time status updates via WebSocket
- `PreviewModule`: Content preview and editing
- `DownloadModule`: Export in multiple formats
- `SettingsModule`: Language preferences, user profile

**Key Features:**
- Responsive design (mobile, tablet, desktop)
- Offline capability for viewing processed content
- Progressive Web App (PWA) support
- Multi-language UI (English + Indic languages)

#### 1.2 State Management

```javascript
// Example State Structure
{
  user: {
    id, email, preferences, subscription
  },
  uploads: {
    active: [], completed: [], failed: []
  },
  processing: {
    jobId, status, progress, estimatedTime
  },
  content: {
    summaries: [], translations: [], posters: [], scripts: []
  }
}
```

### 2. Backend API (FastAPI)

#### 2.1 Core Services

**AuthService:**
- Firebase token verification
- JWT token generation
- Role-based access control

**UploadService:**
- File validation and virus scanning
- Chunked upload support
- Firebase Storage integration

**ProcessingService:**
- Job queue management
- Task orchestration
- Status tracking and notifications

**ContentService:**
- CRUD operations for processed content
- Metadata management
- Content retrieval and filtering

**ExportService:**
- Format conversion
- File packaging
- Download link generation

#### 2.2 Middleware

- CORS configuration
- Request logging
- Error handling
- Rate limiting (per user/IP)
- Request size validation

### 3. AI Processing Pipeline

#### 3.1 PDF Processing Pipeline

```
PDF Upload → Text Extraction → Gemini Summarization → Translation → 
TTS Generation → Poster Creation → Storage
```

**Components:**
- `PDFExtractor`: Extract text using PyPDF2/pdfplumber
- `TextCleaner`: Remove artifacts, normalize formatting
- `SummaryGenerator`: Call Gemini API with prompts
- `TranslationEngine`: Batch translate to selected languages
- `TTSGenerator`: Generate audio using XTTS
- `PosterCreator`: Template-based image generation

#### 3.2 Video Processing Pipeline

```
Video Upload → Audio Extraction (FFmpeg) → Whisper Transcription → 
Gemini Summarization → Translation → TTS → Script Generation → Storage
```

**Components:**
- `VideoValidator`: Check format, duration, size
- `AudioExtractor`: FFmpeg audio extraction
- `WhisperTranscriber`: Speech-to-text conversion
- `ContentAnalyzer`: Identify key moments
- `ScriptGenerator`: Create engaging short-form scripts

#### 3.3 Text Processing Pipeline

```
Text Input → Gemini Summarization → Translation → TTS → 
Poster/Script Generation → Storage
```

### 4. Storage Architecture

#### 4.1 Firestore Collections

```
users/
  {userId}/
    - email, name, createdAt, preferences
    - subscription: {plan, expiresAt}

uploads/
  {uploadId}/
    - userId, fileName, fileType, fileSize
    - status, uploadedAt, processedAt
    - storageUrl

content/
  {contentId}/
    - uploadId, userId, contentType
    - originalText, summary
    - translations: {lang: text}
    - audioUrls: {lang: url}
    - posterUrls: []
    - scripts: []
    - createdAt, updatedAt

jobs/
  {jobId}/
    - uploadId, userId, taskType
    - status, progress, startedAt, completedAt
    - error, retryCount
```

#### 4.2 Cloud Storage Structure

```
/uploads/{userId}/{uploadId}/
  - original.{ext}

/processed/{userId}/{contentId}/
  - summary.txt
  - translations/
    - hi.txt
    - ta.txt
    - ...
  - audio/
    - hi.mp3
    - ta.mp3
    - ...
  - posters/
    - poster_1.png
    - poster_2.png
  - scripts/
    - script_v1.txt
```

---

## Data Flow

### Upload and Processing Flow

```
1. User uploads file via frontend
   ↓
2. Frontend sends file to /api/upload endpoint
   ↓
3. Backend validates file and uploads to Firebase Storage
   ↓
4. Backend creates job in Firestore and adds to Celery queue
   ↓
5. Celery worker picks up job
   ↓
6. Worker executes processing pipeline:
   - Extract content
   - Call Gemini API for summarization
   - Translate to selected languages
   - Generate audio with XTTS
   - Create posters/scripts
   ↓
7. Worker saves results to Firebase Storage
   ↓
8. Worker updates Firestore with metadata and URLs
   ↓
9. Frontend receives WebSocket notification
   ↓
10. User views/downloads processed content
```

### Real-time Status Updates

```
Backend (Celery Worker) → Redis Pub/Sub → WebSocket Server → Frontend
```

---

## API Design

### Base URL
```
Production: https://api.bharatcontent.app/v1
Development: http://localhost:8000/v1
```

### Authentication
```
Header: Authorization: Bearer <firebase_token>
```

### Core Endpoints

#### Upload Endpoints

```http
POST /api/upload
Content-Type: multipart/form-data

Request:
{
  "file": <binary>,
  "contentType": "pdf|video|text",
  "targetLanguages": ["hi", "ta", "te"],
  "options": {
    "summaryLength": "short|medium|long",
    "generatePoster": true,
    "generateScript": true,
    "voiceGender": "male|female"
  }
}

Response: 201 Created
{
  "uploadId": "uuid",
  "jobId": "uuid",
  "status": "queued",
  "estimatedTime": 120
}
```

```http
GET /api/upload/{uploadId}/status

Response: 200 OK
{
  "uploadId": "uuid",
  "status": "processing|completed|failed",
  "progress": 75,
  "currentStep": "generating_audio",
  "estimatedTimeRemaining": 30
}
```

#### Content Endpoints

```http
GET /api/content/{contentId}

Response: 200 OK
{
  "contentId": "uuid",
  "uploadId": "uuid",
  "summary": "...",
  "translations": {
    "hi": "...",
    "ta": "..."
  },
  "audioUrls": {
    "hi": "https://...",
    "ta": "https://..."
  },
  "posterUrls": ["https://...", "https://..."],
  "scripts": [{
    "version": 1,
    "content": "...",
    "duration": 60
  }],
  "createdAt": "2026-02-14T10:30:00Z"
}
```

```http
GET /api/content/list
Query params: ?page=1&limit=20&type=pdf|video|text

Response: 200 OK
{
  "items": [...],
  "total": 100,
  "page": 1,
  "pages": 5
}
```

```http
DELETE /api/content/{contentId}

Response: 204 No Content
```

#### Export Endpoints

```http
POST /api/export/{contentId}

Request:
{
  "format": "zip|json",
  "include": ["summary", "translations", "audio", "posters", "scripts"]
}

Response: 200 OK
{
  "downloadUrl": "https://...",
  "expiresAt": "2026-02-15T10:30:00Z"
}
```

#### User Endpoints

```http
GET /api/user/profile

Response: 200 OK
{
  "userId": "uuid",
  "email": "user@example.com",
  "preferences": {
    "defaultLanguages": ["hi", "ta"],
    "defaultSummaryLength": "medium"
  },
  "usage": {
    "uploadsThisMonth": 45,
    "storageUsed": 1024000000
  }
}
```

```http
PUT /api/user/preferences

Request:
{
  "defaultLanguages": ["hi", "ta", "te"],
  "defaultSummaryLength": "short"
}

Response: 200 OK
```

### WebSocket Endpoints

```
WS /ws/jobs/{jobId}

Messages:
{
  "type": "progress",
  "jobId": "uuid",
  "progress": 50,
  "step": "translating",
  "message": "Translating to Hindi..."
}

{
  "type": "completed",
  "jobId": "uuid",
  "contentId": "uuid"
}

{
  "type": "error",
  "jobId": "uuid",
  "error": "Processing failed: ..."
}
```

---

## AI Pipeline Details

### 1. Gemini API Integration

**Use Cases:**
- Content summarization
- Short video script generation
- Content enhancement

**Implementation:**
```python
import google.generativeai as genai

class GeminiService:
    def __init__(self, api_key):
        genai.configure(api_key=api_key)
        self.model = genai.GenerativeModel('gemini-pro')
    
    async def summarize(self, text, length="medium"):
        prompt = f"""
        Summarize the following content in {length} length.
        Make it engaging and preserve key information.
        
        Content: {text}
        """
        response = await self.model.generate_content_async(prompt)
        return response.text
    
    async def generate_script(self, summary, duration=60):
        prompt = f"""
        Create a {duration}-second engaging video script from this summary.
        Include: Hook, Main Content, Call-to-Action.
        Format: Scene-by-scene breakdown.
        
        Summary: {summary}
        """
        response = await self.model.generate_content_async(prompt)
        return response.text
```

### 2. Whisper ASR Integration

**Implementation:**
```python
import whisper

class WhisperService:
    def __init__(self, model_size="base"):
        self.model = whisper.load_model(model_size)
    
    def transcribe(self, audio_path, language=None):
        result = self.model.transcribe(
            audio_path,
            language=language,
            task="transcribe"
        )
        return {
            "text": result["text"],
            "segments": result["segments"],
            "language": result["language"]
        }
```

### 3. XTTS Voice Synthesis

**Implementation:**
```python
from TTS.api import TTS

class XTTSService:
    def __init__(self):
        self.tts = TTS("tts_models/multilingual/multi-dataset/xtts_v2")
    
    async def generate_speech(self, text, language, output_path, speaker_wav=None):
        self.tts.tts_to_file(
            text=text,
            language=language,
            file_path=output_path,
            speaker_wav=speaker_wav  # Optional: clone voice
        )
        return output_path
```

### 4. Translation Pipeline

**Implementation:**
```python
from indicTrans import IndicTranslator

class TranslationService:
    def __init__(self):
        self.translator = IndicTranslator()
    
    async def translate_batch(self, text, target_languages):
        results = {}
        for lang in target_languages:
            results[lang] = await self.translator.translate(
                text,
                source="en",
                target=lang
            )
        return results
```

### 5. Poster Generation

**Implementation:**
```python
from PIL import Image, ImageDraw, ImageFont

class PosterService:
    def __init__(self, templates_dir):
        self.templates_dir = templates_dir
    
    def create_poster(self, text, template="default", language="en"):
        # Load template
        img = Image.open(f"{self.templates_dir}/{template}.png")
        draw = ImageDraw.Draw(img)
        
        # Load font for language
        font = ImageFont.truetype(f"fonts/{language}.ttf", 48)
        
        # Add text with word wrapping
        self._draw_text_wrapped(draw, text, font, img.width)
        
        return img
```

---

## Deployment Plan

### Development Environment

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - FIREBASE_CREDENTIALS=/secrets/firebase.json
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend:/app
      - ./secrets:/secrets
    depends_on:
      - redis
  
  celery:
    build: ./backend
    command: celery -A app.celery worker --loglevel=info
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
```

### Production Deployment

**Option 1: Cloud Run (Google Cloud)**
```
Backend: Deploy FastAPI as containerized service
Celery Workers: Deploy as separate Cloud Run jobs
Redis: Use Cloud Memorystore
Frontend: Deploy to Firebase Hosting or Vercel
```

**Option 2: AWS**
```
Backend: ECS Fargate or Lambda (API Gateway)
Celery Workers: ECS Fargate with auto-scaling
Redis: ElastiCache
Frontend: S3 + CloudFront or Amplify
```

**Option 3: Heroku (Quick Hackathon Deploy)**
```
Backend: Heroku Dyno (Standard)
Celery: Heroku Worker Dyno
Redis: Heroku Redis add-on
Frontend: Vercel or Netlify
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          cd backend
          pip install -r requirements.txt
          pytest
  
  deploy-backend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy bharat-content-api \
            --image gcr.io/$PROJECT_ID/backend \
            --platform managed \
            --region asia-south1
  
  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Firebase
        run: |
          npm install -g firebase-tools
          firebase deploy --only hosting
```

### Environment Variables

```bash
# Backend .env
FIREBASE_CREDENTIALS=<path_to_json>
GEMINI_API_KEY=<api_key>
REDIS_URL=redis://localhost:6379
CELERY_BROKER_URL=redis://localhost:6379
CELERY_RESULT_BACKEND=redis://localhost:6379
MAX_UPLOAD_SIZE=524288000  # 500MB
ALLOWED_ORIGINS=https://app.bharatcontent.app
```

---

## Scalability Considerations

### 1. Horizontal Scaling

**Backend API:**
- Stateless design for easy replication
- Load balancer distribution (Cloud Load Balancing)
- Auto-scaling based on CPU/memory (50-80% threshold)

**Celery Workers:**
- Separate worker pools for different task types
- Priority queues (high/medium/low)
- Auto-scaling based on queue length

### 2. Caching Strategy

**Redis Caching:**
```python
# Cache frequently accessed content
cache_key = f"content:{content_id}"
ttl = 3600  # 1 hour

# Cache translation results
translation_key = f"translation:{text_hash}:{target_lang}"
ttl = 86400  # 24 hours
```

**CDN Caching:**
- Firebase Storage with CDN for static assets
- Cache-Control headers for generated content
- Edge caching for posters and audio files

### 3. Database Optimization

**Firestore Indexing:**
```
Composite indexes:
- userId + createdAt (descending)
- userId + contentType + createdAt
- status + createdAt
```

**Query Optimization:**
- Pagination for list endpoints
- Limit fields in responses
- Use Firestore listeners for real-time updates

### 4. Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/upload")
@limiter.limit("10/minute")  # 10 uploads per minute
async def upload_file():
    pass
```

### 5. Cost Optimization

**AI API Usage:**
- Batch requests where possible
- Cache common translations
- Use smaller Whisper models for short audio
- Implement request deduplication

**Storage:**
- Automatic file deletion after 30 days
- Compress audio files (MP3 128kbps)
- Optimize poster image sizes
- Use Firebase Storage lifecycle rules

### 6. Monitoring & Observability

**Metrics to Track:**
- API response times (p50, p95, p99)
- Job processing times by type
- Error rates and types
- Queue lengths and wait times
- Storage usage per user
- AI API costs and usage

**Tools:**
- Google Cloud Monitoring / AWS CloudWatch
- Sentry for error tracking
- Custom dashboards for business metrics
- Logging: Structured JSON logs

### 7. Performance Targets

```
API Response Times:
- Upload endpoint: < 2s
- Status check: < 200ms
- Content retrieval: < 500ms

Processing Times:
- PDF (50 pages): < 60s
- Video (5 min): < 180s
- Translation per language: < 10s
- TTS generation: < 15s
- Poster creation: < 10s

Concurrent Users:
- Phase 1 (Hackathon): 100 concurrent
- Phase 2 (Launch): 1000 concurrent
- Phase 3 (Scale): 10000 concurrent
```

---

## Security Considerations

### 1. Input Validation
- File type whitelist
- File size limits
- Content scanning for malicious files
- SQL injection prevention (parameterized queries)
- XSS prevention (sanitize user inputs)

### 2. Authentication & Authorization
- Firebase Authentication integration
- JWT token validation
- Role-based access control (RBAC)
- API key rotation policy

### 3. Data Protection
- HTTPS/TLS for all communications
- Encryption at rest (Firebase default)
- Secure credential storage (Secret Manager)
- PII data handling compliance

### 4. API Security
- CORS configuration
- Rate limiting per user/IP
- Request size limits
- API versioning

---

## Testing Strategy

### Unit Tests
- Service layer functions
- API endpoint handlers
- Data validation logic
- AI pipeline components

### Integration Tests
- End-to-end processing flows
- Firebase integration
- External API integrations
- WebSocket connections

### Load Tests
- Concurrent upload handling
- Queue processing under load
- Database query performance
- API response times

---

## Project Structure

```
bharat-content-transformer/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── api/
│   │   │   ├── upload.py
│   │   │   ├── content.py
│   │   │   ├── user.py
│   │   │   └── export.py
│   │   ├── services/
│   │   │   ├── gemini_service.py
│   │   │   ├── whisper_service.py
│   │   │   ├── xtts_service.py
│   │   │   ├── translation_service.py
│   │   │   ├── poster_service.py
│   │   │   └── firebase_service.py
│   │   ├── workers/
│   │   │   ├── celery_app.py
│   │   │   ├── pdf_worker.py
│   │   │   ├── video_worker.py
│   │   │   └── text_worker.py
│   │   ├── models/
│   │   │   └── schemas.py
│   │   └── utils/
│   │       ├── validators.py
│   │       └── helpers.py
│   ├── tests/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── store/
│   │   └── utils/
│   ├── public/
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
├── requirements.md
├── design.md
└── README.md
```

---

## Timeline & Milestones

**Day 1-2: Setup & Core Infrastructure**
- Project scaffolding
- Firebase setup
- FastAPI boilerplate
- Basic frontend structure

**Day 3-4: Core Processing Pipeline**
- PDF processing
- Gemini integration
- Translation pipeline
- Basic UI for upload

**Day 5-6: Advanced Features**
- Video processing with Whisper
- XTTS voice generation
- Poster generation
- Script generation

**Day 7: Polish & Deploy**
- Testing and bug fixes
- UI/UX improvements
- Deployment to production
- Documentation

---

**Document Version:** 1.0  
**Author:** Development Team  
**Status:** Ready for Implementation
