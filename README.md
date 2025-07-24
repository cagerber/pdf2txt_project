# PDF2TXT - PDF Processing and Text Extraction System

A modern web application for PDF processing, text extraction, and interactive viewing with coordinate-based text selection. Built with FastAPI backend, React frontend, and OpenSeadragon for high-quality PDF viewing.

## 🚀 Features

- **Interactive PDF Viewer**: High-quality PDF viewing with zoom, pan, and navigation using OpenSeadragon
- **Coordinate-based Text Extraction**: Click anywhere on the PDF to extract text at specific coordinates
- **Background Processing**: Asynchronous PDF processing using Celery workers
- **Real-time Updates**: WebSocket support for live processing status updates
- **Multiple Database Support**: SQLite for development, PostgreSQL for production
- **Modern UI**: React 18 with TypeScript, Tailwind CSS, and responsive design

## 📋 Prerequisites

Before setting up the project, ensure you have the following installed:

- **Python 3.8+** (recommended: Python 3.11)
- **Node.js 16+** and npm
- **Redis** (for Celery task queue)
- **PostgreSQL** (optional, SQLite used by default)
- **Git**

### System Dependencies

#### Ubuntu/Debian:
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv nodejs npm redis-server
sudo apt install -y libmagic1 tesseract-ocr poppler-utils  # For PDF processing
```

#### macOS:
```bash
brew install python nodejs npm redis
brew install libmagic tesseract poppler
```

#### Windows:
- Install Python from [python.org](https://python.org)
- Install Node.js from [nodejs.org](https://nodejs.org)
- Install Redis using [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) or [Redis for Windows](https://github.com/microsoftarchive/redis/releases)

## 🛠️ Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/cagerber/pdf2txt_project.git
cd pdf2txt_project
```

### 2. Backend Setup

#### Create Python Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

#### Install Python Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### Environment Configuration
```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your preferred settings
nano .env  # or use your preferred editor
```

**Key Environment Variables:**
- `DATABASE_URL`: Database connection string (defaults to SQLite)
- `REDIS_URL`: Redis connection URL (defaults to localhost:6380)
- `STORAGE_PATH`: Path for file storage (defaults to ./storage)
- `SECRET_KEY`: Change this to a secure random string for production

#### Database Setup

**Option A: SQLite (Default - Recommended for Development)**
```bash
# Create storage directories
mkdir -p storage/{documents,pages,thumbnails,cache}

# Run database migrations
alembic upgrade head
```

**Option B: PostgreSQL (Production)**
```bash
# Install PostgreSQL (if not already installed)
sudo apt install postgresql postgresql-contrib  # Ubuntu/Debian
# brew install postgresql  # macOS

# Create database and user
sudo -u postgres psql
CREATE DATABASE pdf_processor;
CREATE USER pdfuser WITH PASSWORD 'pdfpass';
GRANT ALL PRIVILEGES ON DATABASE pdf_processor TO pdfuser;
\q

# Update .env file
DATABASE_URL=postgresql://pdfuser:pdfpass@localhost:5432/pdf_processor

# Run migrations
alembic upgrade head
```

### 3. Frontend Setup

```bash
cd frontend

# Install Node.js dependencies
npm install

# Return to project root
cd ..
```

### 4. Redis Setup

#### Start Redis Server
```bash
# Ubuntu/Debian
sudo systemctl start redis-server
sudo systemctl enable redis-server

# macOS
brew services start redis

# Or run manually
redis-server --port 6380
```

#### Verify Redis is Running
```bash
redis-cli -p 6380 ping
# Should return: PONG
```

## 🚀 Running the Application

### Method 1: Manual Startup (Recommended for Development)

You'll need **4 terminal windows/tabs** for the complete setup:

#### Terminal 1: FastAPI Backend
```bash
source venv/bin/activate
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```

#### Terminal 2: Celery Worker
```bash
source venv/bin/activate
celery -A services.pdf_processor worker --loglevel=info --concurrency=2
```

#### Terminal 3: Frontend Development Server
```bash
cd frontend
npm run dev
```

#### Terminal 4: Redis (if not running as service)
```bash
redis-server --port 6380
```

### Method 2: Docker Compose (Alternative)

```bash
# Build and start all services
docker-compose up --build

# Or run in background
docker-compose up -d --build
```

## 🌐 Accessing the Application

Once all services are running:

- **Frontend**: http://localhost:5173 (Vite dev server) or http://localhost:3001 (Docker)
- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/docs (Swagger UI)
- **Alternative API Docs**: http://localhost:8000/redoc

## 🧪 Testing the Installation

### 1. Verify Backend Health
```bash
curl http://localhost:8000/health
# Should return: {"status": "healthy"}
```

### 2. Test PDF Upload and Processing

1. Open the frontend at http://localhost:5173
2. Click "Upload PDF" or drag and drop a PDF file
3. Wait for processing to complete
4. Click on the processed document to open the viewer
5. Try clicking on different parts of the PDF to extract text

### 3. Test Sample PDFs

The project includes sample PDFs in the `test/` directory:

```bash
# List available test files
ls test/
```

Upload any of these files through the web interface to test functionality.

## 🔧 Configuration

### Backend Configuration

Key settings in `api/config.py`:

- **Database**: Supports both SQLite and PostgreSQL
- **File Upload**: Max file size (default: 100MB)
- **CORS**: Configure allowed origins for frontend
- **Storage**: Customize storage paths
- **Processing**: Celery worker concurrency settings

### Frontend Configuration

Environment variables for frontend (create `frontend/.env.local`):

```bash
VITE_API_URL=http://localhost:8000
```

## 📁 Project Structure

```
pdf2txt_project/
├── api/                    # FastAPI backend
│   ├── main.py            # Main application entry
│   ├── config.py          # Configuration settings
│   ├── routes/            # API route handlers
│   ├── models/            # Database models
│   └── services/          # Business logic
├── frontend/              # React frontend
│   ├── src/
│   │   ├── components/    # React components
│   │   ├── services/      # API client
│   │   └── types/         # TypeScript types
│   └── public/
├── services/              # Background services
│   └── pdf_processor/     # Celery tasks
├── storage/               # File storage
│   ├── documents/         # Uploaded PDFs
│   ├── pages/             # Processed page images
│   ├── thumbnails/        # PDF thumbnails
│   └── cache/             # Temporary files
├── test/                  # Sample PDF files
└── alembic/               # Database migrations
```

## 🐛 Troubleshooting

### Common Issues

#### 1. Port Already in Use
```bash
# Check what's using the port
lsof -i :8000  # or :5173, :6380

# Kill the process
kill -9 <PID>
```

#### 2. Redis Connection Error
```bash
# Check if Redis is running
redis-cli -p 6380 ping

# Start Redis if not running
redis-server --port 6380
```

#### 3. Database Connection Issues
```bash
# For SQLite, ensure storage directory exists
mkdir -p storage

# For PostgreSQL, verify connection
psql -h localhost -U pdfuser -d pdf_processor
```

#### 4. Python Dependencies Issues
```bash
# Upgrade pip and reinstall
pip install --upgrade pip
pip install -r requirements.txt --force-reinstall
```

#### 5. Node.js Dependencies Issues
```bash
cd frontend
rm -rf node_modules package-lock.json
npm install
```

#### 6. PDF Processing Errors
- Ensure `tesseract-ocr` and `poppler-utils` are installed
- Check that the `storage/` directory has write permissions
- Verify Celery worker is running and connected to Redis

### Logs and Debugging

#### Backend Logs
```bash
# FastAPI logs (in terminal running uvicorn)
# Celery logs (in terminal running celery worker)
```

#### Frontend Logs
- Open browser developer tools (F12)
- Check Console tab for JavaScript errors
- Check Network tab for API request failures

#### Database Logs
```bash
# SQLite - check if file exists and has correct permissions
ls -la storage/pdf_processor.db

# PostgreSQL - check connection
pg_isready -h localhost -p 5432
```

## 🔄 Development Workflow

### Making Changes

1. **Backend Changes**: FastAPI auto-reloads with `--reload` flag
2. **Frontend Changes**: Vite auto-reloads in development mode
3. **Database Changes**: Create new Alembic migrations
   ```bash
   alembic revision --autogenerate -m "Description of changes"
   alembic upgrade head
   ```

### Adding New Dependencies

#### Backend
```bash
pip install new-package
pip freeze > requirements.txt
```

#### Frontend
```bash
cd frontend
npm install new-package
```

## 📚 API Documentation

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

### Key Endpoints

- `POST /api/v1/documents/upload` - Upload PDF
- `GET /api/v1/documents/` - List documents
- `GET /api/v1/documents/{id}` - Get document details
- `POST /api/v1/documents/{id}/extract-text` - Extract text at coordinates
- `WebSocket /ws` - Real-time processing updates

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

## 🆘 Support

If you encounter issues:

1. Check the troubleshooting section above
2. Review the logs for error messages
3. Ensure all prerequisites are installed
4. Verify all services are running
5. Check that ports are not in use by other applications

For additional help, please create an issue in the repository with:
- Your operating system
- Python and Node.js versions
- Complete error messages
- Steps to reproduce the issue