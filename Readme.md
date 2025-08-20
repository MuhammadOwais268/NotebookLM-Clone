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