# Enterprise Knowledge Assistant

A document-based **Retrieval-Augmented Generation (RAG)** application that lets users upload documents and ask natural-language questions about their content.

The project combines a **FastAPI REST API**, **OpenAI models**, **ChromaDB vector search**, and a **Streamlit web interface**. It can also be run with Docker Compose and includes Kubernetes deployment configuration.

## Features

- 📄 Upload **PDF, DOCX, and TXT** documents
- ✂️ Extract and split document text into manageable chunks
- 🔢 Generate embeddings for document chunks
- 🗃️ Store and retrieve embeddings using **ChromaDB**
- 🔎 Retrieve relevant document context for a user query
- 🤖 Generate answers using an OpenAI model
- 💬 Interactive Streamlit interface for uploading and querying documents
- 🚀 FastAPI REST API with automatic Swagger documentation
- 🐳 Docker and Docker Compose support
- ☸️ Kubernetes deployment configuration
- ❤️ Health-check endpoint for the backend
- 💾 Persistent ChromaDB storage

## How the RAG Pipeline Works

```text
                ┌──────────────────────┐
                │   User uploads docs  │
                │   PDF / DOCX / TXT   │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Document Processing  │
                │ Text extraction     │
                │ + chunking          │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ OpenAI Embeddings    │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │      ChromaDB        │
                │   Vector Database    │
                └──────────┬───────────┘
                           │
                     User question
                           │
                           ▼
                ┌──────────────────────┐
                │ Query Embedding      │
                │ + Similarity Search  │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Relevant Context     │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ OpenAI Generation    │
                │ Context + Question   │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │      Answer          │
                │  + source context    │
                └──────────────────────┘
```

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.11+ |
| Backend | FastAPI |
| AI / LLM | OpenAI API |
| Embeddings | OpenAI Embeddings |
| Vector Database | ChromaDB |
| Text Splitting | LangChain Text Splitters |
| Frontend | Streamlit |
| API Server | Uvicorn |
| Containers | Docker, Docker Compose |
| Deployment | Kubernetes |

## Project Structure

```text
enterprise-knowledge-assistant/
│
├── data/
│   └── Sample / uploaded documents
│
├── k8s/
│   └── Kubernetes configuration
│
├── src/
│   ├── main.py
│   ├── config.py
│   ├── models/
│   │   └── schemas.py
│   ├── services/
│   │   ├── document_service.py
│   │   ├── embedding_service.py
│   │   ├── retrieval_service.py
│   │   └── generation_service.py
│   └── utils/
│       └── text_processing.py
│
├── .env.example
├── .gitignore
├── Dockerfile
├── Dockerfile.streamlit
├── docker-compose.yml
├── deploy-k8s.sh
├── deploy-minikube.sh
├── MINIKUBE_GUIDE.md
├── requirements.txt
├── start.sh
└── README.md
```

## Prerequisites

For local development:

- Python 3.11 or newer
- An OpenAI API key

For Docker deployment:

- Docker Desktop with Docker Compose

For Kubernetes deployment:

- A Kubernetes cluster
- `kubectl`
- The required Kubernetes configuration in the `k8s/` directory

## Configuration

Create your environment file from the provided template.

### Linux / macOS / Git Bash

```bash
cp .env.example .env
```

### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

Then open `.env` and add your OpenAI API key:

```env
OPENAI_API_KEY=your-openai-api-key-here
```

The repository's example configuration includes:

```env
OPENAI_MODEL=gpt-4.1-mini
OPENAI_EMBEDDING_MODEL=text-embedding-ada-002

DEBUG=false
HOST=0.0.0.0
PORT=8000

CHROMA_PERSIST_DIRECTORY=./chroma_db
COLLECTION_NAME=documents

CHUNK_SIZE=1000
CHUNK_OVERLAP=200
MAX_FILE_SIZE=10485760

RETRIEVAL_K=5
SIMILARITY_THRESHOLD=0.2
```

> Never commit your real `.env` file or API key to GitHub.

## Run with Docker Compose

Docker Compose is the simplest way to run both the FastAPI backend and Streamlit frontend.

First create `.env` and configure your API key.

Then run:

```bash
docker compose up --build
```

The application will be available at:

- **Streamlit:** http://localhost:8501
- **FastAPI:** http://localhost:8000
- **Swagger API Docs:** http://localhost:8000/docs
- **Health Check:** http://localhost:8000/health

To stop the containers:

```bash
docker compose down
```

## Run with the Startup Script

The repository also contains `start.sh`, which prepares the environment and starts Docker Compose.

On Linux, macOS, Git Bash, or WSL:

```bash
./start.sh
```

The script creates `.env` from `.env.example` when needed and checks that an OpenAI API key has been configured before starting the containers.

## Local Development

Create and activate a virtual environment:

### Windows PowerShell

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create `.env` from `.env.example` and configure your OpenAI API key.

Start the FastAPI backend:

```bash
python -m uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

The backend will be available at:

```text
http://localhost:8000
```

Swagger documentation:

```text
http://localhost:8000/docs
```

> The Streamlit interface is normally started together with the backend through Docker Compose.

## API Endpoints

### Upload a Document

```http
POST /upload
```

Example:

```bash
curl -X POST "http://localhost:8000/upload" \
  -H "accept: application/json" \
  -F "file=@your-document.pdf"
```

Supported formats:

- `.pdf`
- `.docx`
- `.txt`

### Query Documents

```http
POST /query
```

Example:

```bash
curl -X POST "http://localhost:8000/query" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is the main topic of the document?",
    "max_results": 5,
    "include_sources": true
  }'
```

### Get Document Statistics

```http
GET /documents/stats
```

Example:

```bash
curl "http://localhost:8000/documents/stats"
```

### Clear Stored Documents

```http
DELETE /documents
```

Example:

```bash
curl -X DELETE "http://localhost:8000/documents"
```

### Health Check

```http
GET /health
```

Example:

```bash
curl "http://localhost:8000/health"
```

## Using the Streamlit Interface

After starting the application with Docker Compose:

1. Open http://localhost:8501
2. Upload one or more supported documents.
3. Wait for document processing to complete.
4. Enter a natural-language question.
5. The application retrieves relevant document content.
6. The OpenAI model generates an answer using the retrieved context.

## Docker Architecture

The Docker Compose setup contains two application services:

```text
                    ┌──────────────────────┐
                    │   Streamlit Frontend │
                    │      Port 8501       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    FastAPI Backend   │
                    │      Port 8000       │
                    └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    ▼                      ▼
             ┌──────────────┐      ┌──────────────┐
             │   OpenAI API │      │   ChromaDB   │
             │ LLM + Embed. │      │ Vector Store │
             └──────────────┘      └──────────────┘
```

ChromaDB data is persisted through the Docker volume defined in `docker-compose.yml`.

## Kubernetes

The repository includes Kubernetes configuration under:

```text
k8s/
```

It also contains deployment helper scripts and a Minikube guide.

Before deploying to any Kubernetes environment:

1. Review the manifests in `k8s/`.
2. Configure the required secrets securely.
3. Set the appropriate container image and environment values for your cluster.
4. Apply the manifests using `kubectl`.

Example:

```bash
kubectl apply -f k8s/
```

Do not commit real API keys or other secrets to Kubernetes manifests.

## Configuration Reference

| Variable | Purpose |
|---|---|
| `OPENAI_API_KEY` | OpenAI API authentication |
| `OPENAI_MODEL` | Model used for answer generation |
| `OPENAI_EMBEDDING_MODEL` | Model used for embeddings |
| `DEBUG` | Enables debug mode |
| `HOST` | FastAPI host |
| `PORT` | FastAPI port |
| `CHROMA_PERSIST_DIRECTORY` | ChromaDB storage location |
| `COLLECTION_NAME` | ChromaDB collection name |
| `CHUNK_SIZE` | Size of text chunks |
| `CHUNK_OVERLAP` | Overlap between chunks |
| `MAX_FILE_SIZE` | Maximum uploaded file size |
| `RETRIEVAL_K` | Number of results retrieved |
| `SIMILARITY_THRESHOLD` | Minimum similarity threshold |

## Troubleshooting

### OpenAI API key error

Make sure `.env` exists and contains a valid key:

```env
OPENAI_API_KEY=your-real-api-key
```

Restart the application after changing environment variables.

### Port already in use

If port `8000` or `8501` is already being used, stop the process using that port or change the port mapping in `docker-compose.yml`.

### No relevant results

Try:

- Uploading a document containing information related to the question.
- Asking a more specific question.
- Adjusting `RETRIEVAL_K`.
- Reviewing `SIMILARITY_THRESHOLD` in the environment configuration.

### Docker containers do not start

Try rebuilding the images:

```bash
docker compose down
docker compose build --no-cache
docker compose up
```

## Security Notes

- Keep API keys in environment variables.
- Do not commit `.env` files containing secrets.
- Review Kubernetes secrets before deployment.
- Do not expose the application publicly without appropriate authentication, authorization, network controls, and secret management.

## Future Improvements

Potential extensions include:

- Authentication and user management
- Conversation history
- Better document metadata filtering
- Multiple vector-store backends
- Reranking for improved retrieval quality
- Evaluation of retrieval and answer quality
- Streaming model responses
- Observability and usage metrics
- Cloud-native deployment and monitoring

## License

This project is released under the MIT License.
