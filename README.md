# Autonomous Document Agent

![Python](https://img.shields.io/badge/Python-3.9%2B-3776ab?style=flat-square&logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=flat-square&logo=fastapi)
![Gemini](https://img.shields.io/badge/Gemini-LLM-FF6B35?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

> **Turn natural language requests into polished business documents automatically.** An autonomous AI agent that plans its own tasks, executes them intelligently, reviews its own work, and delivers production-ready `.docx` files.

## 🎯 Overview

This project implements a fully autonomous document generation agent that:

- **Understands** your request in plain English
- **Plans** its own task breakdown and document structure
- **Executes** multi-step workflows to generate content
- **Reflects** on its own work and iterates for quality
- **Produces** polished, professional Microsoft Word documents

No templates. No hardcoded workflows. Each request gets a custom, intelligent plan tailored to what you actually need.

### Perfect for:

- **Project Plans** — Teams need a structured roadmap? The agent creates one with timelines, milestones, and resource allocation.
- **Business Proposals** — Describe what you're pitching; the agent builds a persuasive proposal with assumptions, approach, and next steps.
- **Meeting Minutes** — Vague notes become structured minutes with action items, owners, and decisions clearly marked.
- **SOPs & Design Docs** — Complex processes get distilled into clear, step-by-step documentation.
- **Ambiguous Requests** — The agent doesn't ask clarifying questions; it makes reasonable assumptions, states them upfront, and moves forward.

---

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- A free Gemini API key (get one in 30 seconds at [aistudio.google.com/apikey](https://aistudio.google.com/apikey))

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/agent-docx-builder.git
cd agent-docx-builder

# Create and activate virtual environment
python -m venv venv

# On Windows:
venv\Scripts\activate

# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Configuration

Create a `.env` file from the template:

```bash
cp .env.example .env
```

Open `.env` in Notepad (or your editor) and add your Gemini API key:

```env
# .env file
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GEMINI_MODEL=gemini-2.5-flash
LOG_LEVEL=INFO
OUTPUT_DIR=outputs
```

**Getting your Gemini API key:**
1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Sign up (free) or log in with Google
3. Click **Create API Key**
4. Copy the entire key (starts with `gsk_`)
5. Paste into `.env` next to `GEMINI_API_KEY=`
6. Save the file

### Run Locally

```powershell
# Start the server (Windows PowerShell)
uvicorn main:app --reload

# On macOS/Linux:
# uvicorn main:app --reload

# You should see:
# INFO:     Uvicorn running on http://127.0.0.1:8000
# INFO:     Application startup complete.
```

Open your browser to **`http://127.0.0.1:8000`** and start using it!

---

## 🔌 API Usage

### Via Web UI (Easiest)

1. Open `http://127.0.0.1:8000`
2. Enter your request in the textarea
3. Click **Run agent**
4. Watch the task console animate through each step
5. Download your polished `.docx` when complete

### Via cURL (Command Line)

```bash
# Simple request (Windows PowerShell)
curl.exe -X POST http://127.0.0.1:8000/agent `
  -H "Content-Type: application/json" `
  -d '{\"request\": \"Draft a project plan for migrating our CRM over the next quarter. Team is 4 engineers and 1 PM, budget is $60,000, deadline end of Q3.\"}'

# On macOS/Linux (bash):
curl -X POST http://127.0.0.1:8000/agent \
  -H "Content-Type: application/json" \
  -d '{"request": "Draft a project plan for migrating our CRM over the next quarter. Team is 4 engineers and 1 PM, budget is $60,000, deadline end of Q3."}'
```

**Response:**
```json
{
  "doc_type": "project_plan",
  "title": "Customer Database Migration to New CRM Project Plan",
  "reasoning": "A project plan is appropriate...",
  "assumptions": [
    "The new CRM is already selected and compatible",
    "The team has necessary expertise",
    "The budget covers all expenses",
    "End of Q3 deadline is feasible"
  ],
  "tasks": [
    {
      "id": "T1",
      "name": "Clarify scope and assumptions",
      "description": "Review project scope, timeline, and budget",
      "status": "done"
    },
    {
      "id": "T2",
      "name": "Assess database and CRM requirements",
      "description": "Gather information about systems",
      "status": "done"
    }
  ],
  "sections": [
    {"id": "S1", "heading": "Executive Summary"},
    {"id": "S2", "heading": "Current State Assessment"},
    {"id": "S3", "heading": "Migration Strategy"},
    {"id": "S4", "heading": "Project Timeline"},
    {"id": "S5", "heading": "Resource Allocation"},
    {"id": "S6", "heading": "Risk Management"},
    {"id": "S7", "heading": "Testing & QA Plan"}
  ],
  "reflection": {
    "overall_assessment": "The draft provides a comprehensive plan with clear sections and realistic timelines.",
    "issues_found": [],
    "revised_sections": []
  },
  "document_filename": "project_plan_a1b2c3d4.docx",
  "download_url": "/files/project_plan_a1b2c3d4.docx"
}
```

### Via Python

```python
import requests

url = "http://127.0.0.1:8000/agent"
headers = {"Content-Type": "application/json"}
payload = {
    "request": "Create meeting minutes for our Q2 all-hands meeting discussing headcount growth and product launch timeline."
}

response = requests.post(url, json=payload, headers=headers)
result = response.json()

print(f"Document Type: {result['doc_type']}")
print(f"Title: {result['title']}")
print(f"Download: {result['download_url']}")

# Download the document
doc_response = requests.get(f"http://127.0.0.1:8000{result['download_url']}")
with open(f"output_{result['document_filename']}", "wb") as f:
    f.write(doc_response.content)
print(f"✅ Saved: output_{result['document_filename']}")
```

### API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/agent` | Submit a request and generate a document |
| `GET` | `/files/{filename}` | Download a generated `.docx` file |
| `GET` | `/health` | Health check (for monitoring/Render) |
| `GET` | `/` | Serve the web UI |

---

## 🧠 How It Works

### The Agent Pipeline

```
User Request
    ↓
[1] VALIDATE → Check length, scope, guardrails
    ↓
[2] PLAN → Decide doc type, tasks, sections, assumptions
    ↓
[3] EXECUTE → Generate content for each section (per-section LLM calls)
    ↓
[4] REFLECT → Self-review: critique draft, flag issues, rewrite sections
    ↓
[5] BUILD → Convert to polished Word document with styling
    ↓
Word Document + Metadata JSON
```

### Example: Handling Ambiguous Requests

**Input:**
> "We need a product launch brief but haven't decided what product yet—somewhere between a habit tracker and social app, launch in 6-10 weeks, marketing wants big splash but engineering wants quiet soft launch."

**Agent's Response:**
- ✅ **Doc Type Decision:** `project_plan` (best for uncertainty)
- ✅ **Explicit Assumptions:**
  - "Product will be hybrid habit-tracking + social accountability app"
  - "Launch date: 8 weeks (midpoint)"
  - "Marketing and engineering will collaborate on hybrid launch strategy"
  - "Mock data for budget/metrics where specifics unavailable"
- ✅ **Task Breakdown:** 7 concrete tasks (not templates)
- ✅ **Self-Review:** Reflection flags "inconsistent budgets" and revises the budget section
- ✅ **Output:** Professional plan with phased launch, realistic budgets ($250k), clear action items

---

## 🏗️ Architecture

### Core Modules

| Module | Purpose |
|--------|---------|
| `agent/llm_client.py` | Gemini API wrapper with retry logic & robust JSON extraction |
| `agent/validator.py` | Request validation and guardrails (length, scope) |
| `agent/planner.py` | Autonomous planning: decides doc type, generates tasks, assumes missing info |
| `agent/executor.py` | Executes plan: generates each section, tracks task status |
| `agent/reflection.py` | **[Engineering Improvement]** Self-review: critiques draft, rewrites sections |
| `agent/docx_builder.py` | Converts final content to styled Word document |
| `agent/orchestrator.py` | Orchestrates the full pipeline |
| `main.py` | FastAPI routes and endpoints |
| `static/index.html` | Single-page web UI |

### Design Principles

- **Modular:** Each stage isolated; swap components without affecting others
- **Autonomous:** Agent makes decisions; no follow-up questions
- **Resilient:** Retry logic, fallback plans, graceful degradation
- **Auditable:** Task execution logs, reflection notes, explicit assumptions
- **Fast:** Per-section generation allows parallelization

---

## 📊 Example Use Cases

### 1. Project Plan (Well-Defined)
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "request": "Draft a project plan for migrating our customer database to a new CRM over Q3. Team is 4 engineers and 1 PM, budget is $60,000, deadline is end of Q3."
  }'
```

**Output:** 7-section plan with timeline, milestones, resource allocation, risks, testing, and task appendix.

### 2. Business Proposal (Vague)
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "request": "Help us write a proposal for an AI-powered analytics platform for mid-market companies. Still figuring out exact features and pricing."
  }'
```

**Output:** Proposal with assumed features, pricing tiers, competitive analysis, and next steps.

### 3. Meeting Minutes (Unstructured)
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "request": "Create structured meeting minutes: Q3 hiring (20 new engineers), paused feature X (tech debt), pushed product launch to Sept, assigned Bob to security audit, agreed no headcount cuts this year."
  }'
```

**Output:** Professional minutes with decisions, action items (with owners), and next meeting date.

---

## 🚀 Deployment

### Option 1: Render (Easiest, Free)

```bash
# 1. Push to GitHub
git add .
git commit -m "Deploy autonomous document agent"
git push origin main

# 2. Go to render.com and create new service
# 3. Connect your GitHub repo
# 4. Render auto-detects render.yaml and deploys
# 5. Add env var GEMINI_API_KEY in Render dashboard
# 6. Done! 🎉
```

**Live at:** `https://<your-service-name>.onrender.com`

### Option 2: Docker

```bash
# Build
docker build -t agent-docx-builder .

# Run
docker run -e GEMINI_API_KEY=your_key -p 8000:8000 agent-docx-builder
```

### Option 3: Local/VPS with Systemd

```bash
# Create service file
sudo nano /etc/systemd/system/agent-docx.service
```

Paste:
```ini
[Unit]
Description=Autonomous Document Agent
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/agent-docx-builder
EnvironmentFile=/home/ubuntu/agent-docx-builder/.env
ExecStart=/home/ubuntu/agent-docx-builder/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

Then:
```bash
sudo systemctl daemon-reload
sudo systemctl enable agent-docx
sudo systemctl start agent-docx
```

---

## 🧪 Testing

### Test Case 1: Standard Request
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{"request": "Draft a project plan for migrating our customer database to a new CRM over the next quarter. Team is 4 engineers and 1 PM, budget is roughly $60,000, hard deadline is end of Q3."}'
```

**Expected:** `doc_type: project_plan` with 7 sections, minimal assumptions, concrete timeline.

### Test Case 2: Ambiguous Request
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{"request": "We need a brief for a product launch but the team hasn'"'"'t agreed on the product yet—somewhere between a habit tracker and social app, launch in 6-10 weeks, marketing wants big splash but engineering wants quiet soft launch."}'
```

**Expected:** `doc_type: project_plan`, 4+ explicit assumptions, reflection stage flags issues and revises sections.

### Test Case 3: Validation (Should Fail)
```bash
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{"request": "hi"}'
```

**Expected:** `422 Unprocessable Entity` with message "Request is too short".

---

## 🔧 Configuration

### .env File Setup

**Step-by-step:**

1. **Get Gemini API Key:**
   - Visit [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
   - Sign up/login with Google
   - Click "Create API Key"
   - Copy the key (starts with `gsk_`)

2. **Create .env file:**
   ```bash
   cp .env.example .env
   ```

3. **Edit .env:**
   - Open `.env` in Notepad
   - Find `GEMINI_API_KEY=your_google-genai_api_key_here`
   - Replace with your actual key: `GEMINI_API_KEY=AIzaSyxxxxx`
   - Save file

4. **Verify:**
   ```bash
   # On Windows PowerShell:
   Get-Content .env | Select-String GEMINI_API_KEY
   
   # On macOS/Linux:
   grep GEMINI_API_KEY .env
   ```

### Optional Customization

Edit `agent/validator.py` to adjust guardrails:

```python
MIN_LENGTH = 8         # Minimum request length (chars)
MAX_LENGTH = 4000      # Maximum request length
OUT_OF_SCOPE_HINTS = ( # Requests with these are rejected
    "write malware",
    "hack into",
)
```

---

## 🎯 Key Features

### ✅ Autonomous Planning
- No templates—agent analyzes request and decides:
  - What document type best fits
  - What tasks are needed
  - What sections are required
  - What assumptions to make if info is missing

### ✅ Intelligent Execution
- One LLM call per section (parallelizable, traceable)
- Task status tracking (pending → in_progress → done/failed)
- Real content generation (not placeholders)
- Fallback plan if LLM is unavailable

### ✅ Self-Review (Reflection)
The agent critiques its own draft:
- Flags inconsistencies between sections
- Identifies missing or weak content
- Reruns only sections that need improvement
- Provides audit trail of what was checked and changed

### ✅ Professional Output
Polished Word documents (`.docx`) with:
- Title page with metadata
- Real headings and styles
- Bullet lists and formatting
- Task execution appendix
- Professional fonts and spacing

### ✅ Resilient Pipeline
- Request validation before LLM calls
- Retry logic with exponential backoff
- Graceful fallback if LLM unreachable
- Comprehensive logging and debugging

---

## 📊 Performance

| Stage | Time | Details |
|-------|------|---------|
| Planning | 2-3s | 1 LLM call to decide doc type & structure |
| Execution | 10-20s | 5-8 LLM calls (one per section) |
| Reflection | 3-5s | 1 LLM call for self-review |
| Build | <1s | Convert to Word doc |
| **Total** | **15-30s** | From request to downloadable `.docx` |

Gemini's free tier is fast; main bottleneck is network latency, not inference.

---

## 🐛 Troubleshooting

### "TypeError: Client.__init__() got an unexpected keyword argument 'proxies'"
```bash
pip install "httpx==0.27.2" --force-reinstall
```

### "GEMINI_API_KEY is not set"
1. Create `.env` file in project root
2. Add: `GEMINI_API_KEY=your_actual_key_here`
3. Restart server with `uvicorn main:app --reload`

### "LLM call failed after 3 attempts"
- Gemini API temporarily down or rate limit exceeded
- Agent automatically falls back to rule-based plan
- Check [Google Cloud status](https://status.cloud.google.com)
- Retry after a moment

### "Module not found: google.genai"
```bash
pip install -r requirements.txt
```

### Port 8000 already in use
```bash
# Use a different port
uvicorn main:app --reload --port 8001
# Then visit: http://127.0.0.1:8001
```

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🙌 Contributing

Contributions welcome! Areas of interest:
- Additional document types (newsletters, email campaigns, specs)
- Alternative LLM providers (Claude, Gemini, Ollama)
- UI improvements
- Performance optimizations
- Better error messages

```bash
git checkout -b feature/your-idea
git commit -am "Add: your improvement"
git push origin feature/your-idea
```

---

## 🔗 Resources

- **FastAPI Docs:** https://fastapi.tiangolo.com/
- **Google AI Studio:** https://aistudio.google.com/
- **Python-docx:** https://python-docx.readthedocs.io/
- **Pydantic:** https://docs.pydantic.dev/

---

**Built with ❤️ to automate the boring parts of document creation.**

**Questions?** Open an issue or reach out on [discussions](../../discussions).
