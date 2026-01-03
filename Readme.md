NotebookLM Clone: A Self-Hosted AI Research Assistant
This project is a robust, self-hosted AI-powered research assistant inspired by Google's NotebookLM. It enables you to create a personalized knowledge base from diverse sources—PDFs, web URLs, and plain text—and engage in intelligent, cited conversations with your documents. When your local knowledge base lacks the necessary information, the assistant seamlessly offers to perform a web search, delivering synthesized responses with credible citations.
The system is orchestrated using n8n and deployed locally via Docker, ensuring complete data ownership and privacy. A visually engaging demonstration of the UI is recommended—consider recording a short GIF and saving it as demo.gif in your repository to showcase the project in action.
Failed to load imageView link
✨ Key Features

Multi-Source Ingestion: Seamlessly integrate knowledge from PDF files, web page URLs, or plain text notes.
Conversational Q&A: Engage in natural, context-aware dialogues with your documents, with AI responses grounded in your provided sources.
Extractive Citations: Receive direct quotes from your documents to support AI answers, enhancing transparency and verifiability.
Smart Fallback to Online Search: Automatically suggest web searches when local data is insufficient, providing synthesized answers with online citations.
Centralized Logging: A dedicated logging workflow captures every key operation, simplifying monitoring and debugging.
Intuitive Web UI: A user-friendly, browser-based interface for managing sources and interacting with the AI.
100% Self-Hosted & Free: Operates entirely on your local machine using Docker, leveraging open-source components (Ollama, n8n, Postgres) at no cost.

🛠️ Technology Stack

Orchestration: n8n
AI Models: Ollama (supporting gemma:2b, llama3:8b, and mxbai-embed-large)
Database: PostgreSQL with pgvector extension for semantic search
Database UI: Adminer
Containerization: Docker & Docker Compose
Web Search: SerpAPI (integrated via LangChain)
PDF Parsing: PDF.co API
Frontend: HTML, CSS, and vanilla JavaScript

🚀 Getting Started
Follow these steps to deploy the project on your local machine.
Prerequisites

Git
Docker and Docker Compose

1. Clone the Repository
Open your terminal and execute the following commands:
bashgit clone https://github.com/MuhammadOwais268/NotebookLM-Clone.git
cd NotebookLM-Clone
2. Configuration (Critical Step)
This project relies on environment variables for secure configuration. You’ll need to obtain free API keys from external services.
A. Create a .env File
In the project root, create a .env file and populate it with the contents of .env.example (provided below):
env# --- PostgreSQL Credentials ---
# Customize these values as needed.
POSTGRES_USER=user
POSTGRES_PASSWORD=password
POSTGRES_DB=notebooklm_db

# --- API Keys (Obtain from respective services) ---
# Register at https://pdf.co/ for PDF.co API key
PDFCO_API_KEY=YOUR_PDF.CO_API_KEY

# Register at https://serper.dev/ for Serper API key
SERPER_API_KEY=YOUR_SERPER_API_KEY

# Register at https://ai.google.dev/ for Gemini API key
GEMINI_API_KEY=YOUR_GEMINI_API_KEY
B. Obtain API Keys

PDF.co: Sign up for a free account at PDF.co to acquire your API key for PDF parsing.
Serper: Sign up at serper.dev to obtain your Google Search API key.
Gemini: Create a free account at Google AI Studio to get your Gemini API key.

C. Update the .env File
Insert your newly acquired API keys into the .env file.
D. Update docker-compose.yml
Enhance security and professionalism by using environment variables in docker-compose.yml. Replace its contents with the following:
yamlservices:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_service
    restart: always
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres_db
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - WEBHOOK_CORS_ALLOWED_ORIGINS=*
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres_db
    networks:
      - notebooklm_net

  postgres_db:
    image: pgvector/pgvector:pg15
    container_name: postgres_db
    restart: always
    ports:
      - "5433:5432"
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./scripts/db_init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - notebooklm_net
    command: postgres -c log_statement=all -c log_destination=stderr

  ollama:
    image: ollama/ollama
    container_name: ollama_cpu
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    networks:
      - notebooklm_net

  adminer:
    image: adminer
    restart: always
    ports:
      - "8080:8080"
    networks:
      - notebooklm_net

volumes:
  n8n_data:
  db_data:
  ollama_data:

networks:
  notebooklm_net:
    driver: bridge
3. Launch the Application
With the configuration complete, start the services:
bashdocker compose up -d
This command downloads the required images and launches the containers. Initial setup may take a few minutes.
4. Pull Ollama Models
Download the AI models used by the workflows:
bashdocker compose exec ollama ollama pull gemma:2b
docker compose exec ollama ollama pull llama3:8b
docker compose exec ollama ollama pull mxbai-embed-large
5. Import and Configure n8n Workflows
Complete the setup by configuring n8n.

Access the n8n UI at http://localhost:5678 and set up an owner account on your first visit.
A. Create Credentials in n8n:

Navigate to the "Credentials" tab.
Click "Add credential" and create entries for:

PDF.co API
SerpAPI
Google Gemini (PaLM) API


Select the respective API keys from your .env file.


B. Import the Workflows:

Go to the "Workflows" tab.
Click "Add workflow" -> "Import from file..."
Import the workflow JSON files from the /n8n_workflows directory in this order:

[UTIL] Logger.json
[API] Ingestion & Indexing.json
[API] Chat with Documents.json
[API] Online Search.json


Open each imported workflow, connect the credentials in the relevant nodes, and activate them using the toggle at the top.



💻 How to Use the Application
Once all services are running and workflows are active, open app.html in your web browser.

Add a Document: Use the form in the left column to input text notes, scrape a URL, or upload a PDF.
View Sources: The "Available Notes" list on the right displays all content in your knowledge base.
Chat: Use the "Chat with your Documents" form to ask questions. If the answer isn’t available locally, the UI will prompt you to initiate an online search.

🏛️ Project Architecture
The application employs a modular, API-driven architecture with four distinct n8n workflows:

[API] Ingestion & Indexing: The primary pipeline for adding knowledge. It processes PDFs, URLs, and text, chunks the content, generates embeddings, and stores everything in the database.
[API] Chat with Documents: A Retrieval-Augmented Generation (RAG) pipeline that performs vector searches on the database to provide context-aware, cited answers, with fallback logic to prompt for online searches.
[API] Online Search: An AI agent workflow leveraging SerpAPI to search the live web and synthesize cited answers for queries beyond the local knowledge base.
[UTIL] Logger: A centralized utility workflow that logs data from other workflows into the workflow_logs table for monitoring and debugging.


Enhancements Made

Professional Tone: Replaced casual language (e.g., "simply open") with formal phrasing (e.g., "open app.html in your web browser") and improved consistency.
Structure: Organized content with clear headings, subheadings, and bullet points for readability.
Clarity: Added detailed steps and explanations (e.g., workflow import order, API key acquisition) while avoiding redundancy.
Visual Suggestion: Emphasized the demo GIF recommendation with a note to encourage its creation.
Technical Precision: Corrected minor inconsistencies (e.g., ensured docker compose syntax matches modern usage) and clarified environment variable usage.
Documentation Style: Aligned with common open-source readme conventions (e.g., code blocks with language identifiers, consistent formatting).

Notes

The code assumes app.html is the frontend file (based on your previous context). If it differs, update the "How to Use" section accordingly.
The demo GIF suggestion is included but not implemented here; you’ll need to record and add it manually.
API keys and workflow files (/n8n_workflows) are referenced but not provided; ensure they are available in your repository.

If you need further refinements (e.g., adding a contribution guide, troubleshooting section, or integrating the UI code directly), let me know!



Where to look / how to access

Frontend UI: http://localhost:8000/index.html
n8n UI: http://localhost:5678
Adminer (DB GUI): http://localhost:8080
Ollama API: http://localhost:11434
Postgres (from host): port 5433


run index.html services by 
```bash
cd /home/owais/NotebookLM-Clone
python3 -m http.server 8000
```

---

## 🔧 Complete Setup & Run Procedure (Step-by-Step)

Follow this comprehensive guide to set up and run the NotebookLM Clone from scratch.

### Prerequisites Checklist
- [ ] Docker and Docker Compose installed
- [ ] Git installed
- [ ] Python 3 installed (for serving frontend)
- [ ] API keys obtained (PDF.co, Serper, Gemini)

### Step 1: Stop Any Running Containers (Clean Slate)
```bash
cd /home/owais/NotebookLM-Clone
docker compose down
```
This ensures a fresh start by stopping and removing all project containers.

### Step 2: Start All Services
```bash
docker compose up -d
```
This command starts:
- **n8n** on port 5678
- **Postgres** (with pgvector) on port 5433
- **Ollama** on port 11434
- **Adminer** (DB GUI) on port 8080

**Wait 30-60 seconds** for all services to fully initialize.

**Verify containers are running:**
```bash
docker ps --filter name=n8n_service --filter name=postgres_db --filter name=ollama_cpu
```

### Step 3: Access n8n and Create Owner Account
1. Open your browser and navigate to: **http://localhost:5678**
2. On first visit, you'll be prompted to create an owner account
3. Fill in:
   - Email
   - Password (set a strong password)
   - First Name / Last Name
4. Click **Get Started**

### Step 4: Create Postgres Credential in n8n

#### 4.1 Navigate to Credentials
- In n8n UI, click **Credentials** in the left sidebar
- Click **Add Credential** button (top right)

#### 4.2 Create Postgres Credential
1. Search for and select **Postgres**
2. Fill in the following details:
   - **Host:** `postgres_db` (service name inside Docker network)
   - **Port:** `5432` (internal container port)
   - **Database:** `notebooklm_db`
   - **User:** `user`
   - **Password:** `password`
   - **SSL Mode:** `disable` (or leave default)
3. Click **Save**
4. Give it a recognizable name like "Postgres NotebookLM"

**💡 Important:** Use `postgres_db` as host (not `localhost`) because n8n runs inside Docker and connects via the internal network.

### Step 5: Create Google Gemini (PaLM) Credential in n8n

#### 5.1 Navigate to Credentials
- Click **Add Credential** again

#### 5.2 Create Gemini Credential
1. Search for and select **Google PaLM API** or **Google Gemini**
2. Enter your **Gemini API Key** (from https://ai.google.dev/)
3. Click **Save**
4. Name it "Gemini NotebookLM"

**📝 Note:** You need to obtain this API key from Google AI Studio beforehand.

### Step 6: Create PostgreSQL Database and Notes Table

#### 6.1 Connect to Postgres
From your host terminal:
```bash
PGPASSWORD=password psql -h localhost -p 5433 -U user -d postgres -c "CREATE DATABASE notebooklm_db;" || true
```

#### 6.2 Enable pgvector Extension and Create Table
```bash
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db <<'SQL'
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE IF NOT EXISTS notes (
  id serial PRIMARY KEY,
  title text,
  summary text,
  content text,
  vector vector(1536)
);
SQL
```

#### 6.3 Verify Table Creation
```bash
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db -c "\dt"
```
You should see the `notes` table listed.

### Step 7: Download Required Ollama Models

Check the README or workflow files to identify required models. Based on the project architecture, download these models:

#### 7.1 Download Embedding Model
```bash
docker compose exec ollama ollama pull mxbai-embed-large:latest
```

#### 7.2 Download Chat/Generation Models
```bash
docker compose exec ollama ollama pull gemma:2b
docker compose exec ollama ollama pull llama3.2:1b
```

**⏱️ Note:** Model downloads can take 5-15 minutes depending on your internet speed. Each command will show download progress.

#### 7.3 Verify Models Are Downloaded
```bash
docker compose exec ollama ollama ls
```
or check via API:
```bash
curl -sS http://localhost:11434/v1/models | jq .
```

You should see:
- `mxbai-embed-large:latest`
- `gemma:2b`
- `llama3.2:1b`

### Step 8: Import n8n Workflows

#### 8.1 Navigate to Workflows
- In n8n UI, click **Workflows** in the left sidebar
- Click **Add workflow** → **Import from file...**

#### 8.2 Import Workflows in Order
Import the following JSON files from `/n8n workflows/` directory:

1. **Ingestion & Indexing.json**
2. **Chat with Document.json**
3. **online search.json**

For each workflow:
1. Click **Import from file...**
2. Select the JSON file
3. After import, **open the workflow**
4. Click on nodes that require credentials (e.g., Postgres nodes)
5. Select the credentials you created earlier from the dropdown
6. Click **Save** (top right)
7. **Activate** the workflow using the toggle switch (top right)

### Step 9: Run the Frontend Server

#### 9.1 Start Python HTTP Server
From your terminal:
```bash
cd /home/owais/NotebookLM-Clone
python3 -m http.server 8000
```

**Keep this terminal open** — the server runs in foreground.

#### 9.2 Alternative: Run in Background
If you want to run the server in the background:
```bash
cd /home/owais/NotebookLM-Clone
nohup python3 -m http.server 8000 > /tmp/notebooklm_frontend.log 2>&1 &
```

To stop the background server later:
```bash
pkill -f "python3 -m http.server 8000"
```

### Step 10: Access the UI

Open your browser and navigate to:
- **Frontend UI:** http://localhost:8000/index.html
- **n8n UI:** http://localhost:5678
- **Adminer (DB GUI):** http://localhost:8080
- **Ollama API:** http://localhost:11434

### Step 11: Test the Application

#### 11.1 Add a Test Document
1. In the frontend UI (http://localhost:8000/index.html)
2. Click **Add Source** button
3. Select **Plain Text** as type
4. Enter a test note:
   ```
   This is a test document for the NotebookLM clone. 
   It demonstrates the ingestion and retrieval capabilities.
   ```
5. Click **Submit to Webhook**
6. Wait for success message
7. Click **Add to List**

#### 11.2 Query Your Document
1. Select the document from the sources list
2. In the chat input, type: "What is this document about?"
3. Click **Send**
4. You should receive an AI-generated answer with citations

#### 11.3 Verify Data in Database
```bash
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db -c \
"SELECT id, title, summary FROM notes ORDER BY id DESC LIMIT 5;"
```

---

## 🔍 Troubleshooting Common Issues

### Issue 1: "Tenant or user not found" Error
**Cause:** n8n Postgres credential is pointing to wrong host/port.

**Solution:**
- Edit the Postgres credential in n8n
- Ensure host is `postgres_db` (not `localhost`)
- Port should be `5432` (internal container port)

### Issue 2: Ollama Models Not Found
**Cause:** Models not downloaded or wrong model names in workflow.

**Solution:**
```bash
# List currently downloaded models
docker compose exec ollama ollama ls

# Pull missing models
docker compose exec ollama ollama pull <model-name>
```

### Issue 3: Frontend Can't Connect to n8n Webhooks
**Cause:** CORS or webhook path mismatch.

**Solution:**
- Verify n8n is running: `docker ps --filter name=n8n_service`
- Check webhook paths in `index.html` match the paths in n8n workflows
- Ensure `WEBHOOK_CORS_ALLOWED_ORIGINS=*` is set in docker-compose.yml

### Issue 4: pgvector Extension Missing
**Cause:** Extension not created in database.

**Solution:**
```bash
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db -c \
"CREATE EXTENSION IF NOT EXISTS vector;"
```

### Issue 5: Port Already in Use
**Cause:** Another service is using port 5678, 5433, 8000, 8080, or 11434.

**Solution:**
```bash
# Check what's using the port (example for 5678)
sudo ss -ltnp | grep :5678

# Kill the process or change the port in docker-compose.yml
```

---

## 📊 Service Access Summary

| Service | URL | Purpose |
|---------|-----|---------|
| Frontend UI | http://localhost:8000/index.html | Main application interface |
| n8n Workflows | http://localhost:5678 | Workflow management & credentials |
| Adminer | http://localhost:8080 | Database visual admin |
| Ollama API | http://localhost:11434 | LLM inference endpoint |
| Postgres | localhost:5433 | Database (psql access from host) |

---

## 🛑 Stopping the Application

### Stop Docker Services
```bash
cd /home/owais/NotebookLM-Clone
docker compose down
```

### Stop Frontend Server
If running in foreground: Press `Ctrl+C`

If running in background:
```bash
pkill -f "python3 -m http.server 8000"
```

---

## 🔄 Quick Restart Commands

```bash
# Full restart
cd /home/owais/NotebookLM-Clone
docker compose down
docker compose up -d

# Wait 30 seconds, then start frontend
python3 -m http.server 8000
```

---

## 📝 Useful Maintenance Commands

### View Container Logs
```bash
# n8n logs
docker logs n8n_service --tail 200

# Postgres logs
docker logs postgres_db --tail 200

# Ollama logs
docker logs ollama_cpu --tail 200
```

### Database Queries
```bash
# Count notes in database
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db -c \
"SELECT COUNT(*) FROM notes;"

# View recent notes
PGPASSWORD=password psql -h localhost -p 5433 -U user -d notebooklm_db -c \
"SELECT id, title, LEFT(summary, 100) FROM notes ORDER BY id DESC LIMIT 10;"
```

### Check System Resources
```bash
# Docker container resource usage
docker stats --no-stream

# Check disk space used by Docker
docker system df
```

---

## 🎯 Next Steps

After successful setup:
1. ✅ Import additional documents (PDFs, URLs, text)
2. ✅ Configure additional n8n workflows for online search
3. ✅ Customize the frontend UI (`index.html`)
4. ✅ Set up backup scripts for Postgres data
5. ✅ Explore advanced querying with vector similarity

**Happy researching with your NotebookLM Clone! 🚀**