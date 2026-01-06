# mConverter

**Production-ready, asynchronous file conversion API** built with FastAPI, Celery, and modern Python standards (2026).

Convert various file types (PDF, Excel, CSV, CLI output) into structured JSON/Markdown via a distributed task queue.

## 🚀 Features

- **Asynchronous Processing**: Non-blocking file conversion using Celery task queue
- **Multiple File Formats**: Support for PDF, DOCX, Excel, CSV, Markdown, and more
- **Distributed Architecture**: Horizontally scalable with RabbitMQ and Redis
- **Production-Ready**: Docker Compose orchestration with health checks
- **Modern Python**: Built with Python 3.12+, FastAPI, and `uv` package manager
- **Type-Safe**: Full type hinting throughout the codebase
- **Structured Logging**: JSON logging for production monitoring
- **Retry Logic**: Automatic retry with exponential backoff for failed tasks

## 📋 Requirements

- **Docker & Docker Compose** (recommended)
- **Python 3.12+** (for local development)
- **uv** package manager (installed automatically in Docker)

## 🏗️ Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Client    │─────▶│  FastAPI API │─────▶│  RabbitMQ   │
└─────────────┘      └──────────────┘      └─────────────┘
                            │                      │
                            ▼                      ▼
                     ┌──────────────┐      ┌─────────────┐
                     │    Redis     │◀─────│   Celery    │
                     │   (Results)  │      │   Workers   │
                     └──────────────┘      └─────────────┘
```

### Components

1. **FastAPI API**: Receives file uploads, submits tasks, returns job IDs
2. **Celery Workers**: Process files using conversion engines
3. **RabbitMQ**: Message broker for task distribution
4. **Redis**: Result backend for storing task status and results
5. **Conversion Engines**:
   - `DocumentEngine`: PDF/DOCX using `docling`
   - `SpreadsheetEngine`: Excel/CSV using `pandas`
   - `CLIEngine`: Text formats using `pandoc`

## 📁 Project Structure

```
mConverter/
├── src/
│   ├── api/
│   │   ├── main.py           # FastAPI application
│   │   └── models.py         # Pydantic models
│   ├── core/
│   │   └── config.py         # Configuration management
│   └── workers/
│       ├── tasks.py          # Celery task definitions
│       └── engines/
│           ├── base.py       # Abstract base engine
│           ├── doc_engine.py # Document conversion
│           ├── spreadsheet_engine.py
│           ├── cli_engine.py # CLI tool wrapper
│           └── factory.py    # Engine selection
├── docker-compose.yaml       # Service orchestration
├── Dockerfile.api            # API container
├── Dockerfile.worker         # Worker container
├── pyproject.toml           # Dependencies (uv/pip)
└── .env.example             # Environment template
```

## 🚀 Quick Start

### 1. Clone and Setup

```bash
cd mConverter
cp .env.example .env
```

### 2. Start Services

```bash
docker-compose up -d
```

This starts:
- **API**: http://localhost:8000
- **RabbitMQ Management**: http://localhost:15672 (guest/guest)
- **Flower (Celery Monitor)**: http://localhost:5555
- **Redis**: localhost:6379

### 3. Check Health

```bash
curl http://localhost:8000/health
```

Expected response:
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "services": {
    "celery_workers": true,
    "redis": true
  }
}
```

## 📖 API Usage

### Upload and Convert File

```bash
curl -X POST http://localhost:8000/v1/convert \
  -F "file=@document.pdf" \
  -H "Content-Type: multipart/form-data"
```

Response:
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "submitted",
  "message": "File document.pdf submitted for conversion",
  "created_at": "2026-01-07T10:30:00Z"
}
```

### Check Conversion Status

```bash
curl http://localhost:8000/v1/status/550e8400-e29b-41d4-a716-446655440000
```

Response (when complete):
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "SUCCESS",
  "result": {
    "success": true,
    "output_format": "json",
    "data": {
      "content": {...},
      "metadata": {...}
    },
    "processing_time": 2.45
  }
}
```

### Get Supported Formats

```bash
curl http://localhost:8000/v1/supported-formats
```

## 🎯 Supported File Formats

| Category      | Extensions                    | Engine        |
|---------------|-------------------------------|---------------|
| Documents     | `.pdf`, `.docx`, `.doc`       | docling       |
| Spreadsheets  | `.xlsx`, `.xls`, `.csv`       | pandas        |
| Text          | `.md`, `.txt`, `.html`, `.rst`| pandoc        |

## 🔧 Configuration

Edit `.env` file or set environment variables:

```bash
# Application
ENVIRONMENT=production
LOG_LEVEL=INFO
LOG_FORMAT=json

# File Upload
MAX_UPLOAD_SIZE=104857600  # 100MB

# Celery Workers
WORKER_REPLICAS=4
CELERY_WORKER_PREFETCH_MULTIPLIER=4
CELERY_TASK_TIME_LIMIT=3600

# Task Retry
TASK_MAX_RETRIES=3
TASK_RETRY_BACKOFF=60
```

## 🛠️ Development

### Local Setup (without Docker)

```bash
# Install uv
pip install uv

# Install dependencies
uv pip install -e ".[dev]"

# Start RabbitMQ and Redis (Docker)
docker-compose up -d rabbitmq redis

# Run API
uvicorn src.api.main:app --reload

# Run Worker (separate terminal)
celery -A src.workers.tasks worker --loglevel=info
```

### Run Tests

```bash
pytest tests/ -v --cov
```

### Code Quality

```bash
# Format code
black src/

# Lint
ruff check src/

# Type check
mypy src/
```

## 📊 Monitoring

### Flower Dashboard

Access Celery monitoring at http://localhost:5555

Features:
- Real-time task monitoring
- Worker status and statistics
- Task history and retries
- Performance metrics

### RabbitMQ Management

Access at http://localhost:15672 (guest/guest)

Monitor:
- Queue depths
- Message rates
- Connection status

## 🐳 Docker Commands

```bash
# Start all services
docker-compose up -d

# Scale workers
docker-compose up -d --scale worker=4

# View logs
docker-compose logs -f worker
docker-compose logs -f api

# Stop all services
docker-compose down

# Rebuild after code changes
docker-compose up -d --build
```

## 🔍 Troubleshooting

### Workers not starting

```bash
# Check RabbitMQ is running
docker-compose ps rabbitmq

# Check worker logs
docker-compose logs worker
```

### File conversion fails

```bash
# Check supported formats
curl http://localhost:8000/v1/supported-formats

# Verify system dependencies in worker
docker-compose exec worker pandoc --version
docker-compose exec worker python -c "import docling"
```

### Task stuck in PENDING

```bash
# Restart workers
docker-compose restart worker

# Check Redis connection
docker-compose exec redis redis-cli ping
```

## 🎭 Production Deployment

### Recommended Settings

```bash
ENVIRONMENT=production
DEBUG=false
LOG_FORMAT=json
WORKER_REPLICAS=8
CELERY_WORKER_MAX_TASKS_PER_CHILD=100
```

### Security Considerations

1. **Change default credentials**:
   ```bash
   RABBITMQ_USER=admin
   RABBITMQ_PASSWORD=<strong-password>
   REDIS_PASSWORD=<strong-password>
   ```

2. **Enable SSL/TLS** for production APIs
3. **Set up reverse proxy** (nginx/traefik)
4. **Configure CORS** in `src/api/main.py`
5. **Enable rate limiting**
6. **Set up monitoring** (Prometheus, Grafana)

### Scaling

Horizontal scaling:
```bash
# Scale API instances
docker-compose up -d --scale api=3

# Scale workers
docker-compose up -d --scale worker=10
```

## 📝 API Documentation

Interactive API docs available at:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## 📄 License

MIT License - see LICENSE file for details

## 🙏 Acknowledgments

- **FastAPI** - Modern web framework
- **Celery** - Distributed task queue
- **docling** - Document processing
- **pandas** - Data manipulation
- **pandoc** - Universal document converter

---

**Built with ❤️ using Python 3.12+ and modern 2026 standards**
