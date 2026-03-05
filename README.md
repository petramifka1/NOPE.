# NOPE

**Deepfake, Misinformation & Scam Shield for Older Adults**

NOPE is a verification tool that helps older adults (55+) check whether online content is trustworthy. Paste any claim, URL, or image into the chat and get a plain-language verdict backed by multiple independent sources — plus scam detection that flags phishing, fraud, and social engineering tactics.

## How It Works

```
Frontend (static/index.html)  or  N8N Chat UI
              │
              ▼
         FastAPI Server (app.py)
              │
   ┌──────────┼──────────┬──────────────┐
   ▼          ▼          ▼              ▼
Pinecone   Tavily    Google Fact    Scam Analyzer
(known      (web      Check API     ├─ Pattern matching (50 scam patterns)
 misinfo)   search)                 ├─ URL safety (Google Safe Browsing)
   │          │          │          └─ Social engineering detection
   └──────────┼──────────┴──────────────┘
              ▼
   LangGraph Agent (Claude)
     ┌────────┴────────┐
     ▼                 ▼
Claude Analysis   LLM Validation
     └────────┬────────┘
              ▼
       SQLite Logging
              ▼
    Formatted response → Chat
```

### Evidence Sources

1. **Pinecone** — searches a curated knowledge base of 20 known misinformation patterns
2. **Tavily** — real-time web search for current information
3. **Google Fact Check API** — published fact-checks from trusted organizations
4. **Scam Analyzer** — pattern-matches against 50 known scam templates, detects social engineering tactics, and checks URL safety via Google Safe Browsing
5. **Image Analysis** — reverse image search (SerpAPI / Google Lens) and AI-powered image description for visual content verification

## Tech Stack

| Component | Technology |
|-----------|-----------|
| LLM | Claude (Anthropic) |
| Agent Orchestration | LangGraph |
| Frontend | Custom HTML/CSS/JS (`static/`) |
| Workflow (alternative) | N8N (built-in chat trigger) |
| Vector Store | Pinecone |
| Embeddings | OpenAI text-embedding-3-small |
| Web Search | Tavily |
| Fact-Checking | Google Fact Check API |
| Scam Detection | Custom pattern analyzer + Google Safe Browsing |
| Image Analysis | Pillow + SerpAPI (Google Lens) |
| API Server | FastAPI |
| Database | SQLite |

## Setup

### Prerequisites

- Python 3.11+
- API keys for: Anthropic, OpenAI, Pinecone, Tavily, Google Fact Check

### Installation

```bash
git clone https://github.com/Javirum/clearcheck.git
cd clearcheck
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Environment Variables

Copy the example env file and fill in your API keys:

```bash
cp .env.example .env
```

Required variables:

```
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
PINECONE_API_KEY=
PINECONE_INDEX_NAME=
TAVILY_API_KEY=
GOOGLE_FACTCHECK_API_KEY=
```

Optional (enable additional features):

```
GOOGLE_SAFE_BROWSING_API_KEY=   # URL safety checks in scam detection
SERPAPI_API_KEY=                 # Image reverse search via Google Lens
```

### Seed the Knowledge Base

```bash
python -m src.seed_knowledge_base
```

### Start the Server

```bash
python app.py
```

The server runs on `http://localhost:8000`.

- **Custom frontend:** open `http://localhost:8000/static/index.html`
- **Health check:** `GET /health`
- **N8N workflow (alternative):** import `n8n_workflow.json` into your N8N instance

### Run Evaluation

```bash
python evaluate.py
```

## Project Structure

```
clearcheck/
├── app.py                            # FastAPI server
├── evaluate.py                       # Test dataset evaluation script
├── n8n_workflow.json                 # N8N workflow (text verification)
├── n8n_image_workflow.json           # N8N workflow (image verification)
├── requirements.txt
├── Dockerfile
├── railway.toml
├── src/
│   ├── __init__.py
│   ├── config.py                     # Environment variables & constants
│   ├── schemas.py                    # Pydantic models & LangGraph state
│   ├── evidence.py                   # Pinecone, Tavily, Google Fact Check
│   ├── agent.py                      # LangGraph agent (Claude + validation)
│   ├── scam_analyzer.py              # Scam pattern matching & detection
│   ├── url_safety.py                 # Google Safe Browsing URL checker
│   ├── image_agent.py                # Image verification agent
│   ├── image_evidence.py             # Reverse image search & analysis
│   ├── audit_log.py                  # SQLite audit logging
│   ├── retry.py                      # Retry utilities
│   └── seed_knowledge_base.py        # Seed Pinecone with misinfo patterns
├── data/
│   ├── misinformation_patterns.json  # 20 curated misinfo patterns
│   ├── scam_patterns.json            # 50 known scam patterns
│   └── test_dataset.json             # 20-item test set with ground truth
├── static/
│   ├── index.html                    # Landing page & main UI
│   ├── chat.html                     # Chat interface
│   ├── css/                          # Stylesheets
│   ├── js/                           # Frontend scripts
│   └── assets/                       # Images & static assets
├── .env.example
├── PROJECT_PLAN.md
└── README.md
```

## License

MIT
