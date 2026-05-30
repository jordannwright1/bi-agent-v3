# Navi OS

**AI-powered business intelligence with persistent subagents, AWS S3 integration, and a developer API.**

[Try Navi →](https://www.navi-os.cc)

---

## What is Navi?

Navi is a self-correcting AI agent that turns natural language prompts into complete analytical workflows. Upload a dataset or connect an S3 bucket — Navi writes its own Python, executes it, catches errors, fixes them automatically, and returns structured results with charts and calculations.

It is not a wrapper around a language model. It is a stateful reasoning engine with a planning layer, an ethics gate, a memory system, and a self-correction loop that runs without user intervention.

---

## Quick Start

### 1. Sign up
Go to [www.navi-os.cc](https://www.navi-os.cc) and create an account. Free accounts include 3 analyses.

### 2. Ask a question
From the chat page, type any analytical question. Navi works without a dataset for research, web scraping, and general analysis. For data analysis, upload a file first.

### 3. Upload a file
Click the paperclip icon in the chat input and select a `.csv` or `.xlsx` file. Navi will confirm the upload with row and column counts, then you can ask questions about the data.

### 4. Read the results
Navi returns a written analysis, inline charts, and a calculations panel showing the exact numbers. Every chart is generated from your actual data — not a template.

---

## Subagents

Subagents are specialized AI analysts you create and configure. Each one has its own persona, conversation history, and optionally its own S3 data layer.

### Creating a subagent
1. Go to [www.navi-os.cc/subagents](https://www.navi-os.cc/subagents)
2. Click **New** in the top left
3. Describe the agent's specialty in plain language — e.g. *"Stock market analyst focused on earnings reports and technical indicators"* or *"E-commerce analyst specializing in customer cohort analysis and LTV"*
4. Click **Create Agent** — Navi generates a name, icon, color, and full system prompt automatically
5. The agent appears in the library on the left. Click it to open the chat

### Chatting with a subagent
The subagent chat works the same as the main Navi chat but the agent stays in its domain. If you ask a financial analyst about marketing metrics, it will note that and redirect you to the appropriate context. Upload files via the paperclip icon or load from S3 (see below).

### Free vs Pro
- **Free**: 1 subagent
- **Pro**: unlimited subagents, unlimited analyses

---

## AWS S3 Integration

Connect an S3 bucket to a subagent so it can access your data directly — no manual uploads needed. The bucket lives entirely in your own AWS account. Navi never stores your data.

### Step 1 — Select a subagent and click Connect AWS

From the subagents page, select any agent. In the header, click **Connect AWS**. This opens a three-step flow.

### Step 2 — Launch the CloudFormation stack

Click **Launch CloudFormation Stack**. This opens the AWS CloudFormation console in a new tab with everything pre-filled:

- A unique stack name tied to your subagent
- Your S3 bucket name (pre-generated, you can change it)
- Navi's service ARN (injected automatically — you never touch IAM)

In the CloudFormation console:
1. Review the pre-filled parameters — you can change the bucket name if you want
2. Check the acknowledgment box at the bottom ("I acknowledge that AWS CloudFormation might create IAM resources")
3. Click **Create stack**
4. Wait approximately 60 seconds for the status to change to `CREATE_COMPLETE`

### Step 3 — Read stack outputs

Come back to Navi. The modal will show a field for your stack name (pre-filled). Click **Read Stack Outputs**. Navi reads the CloudFormation outputs automatically — bucket name, bucket ARN, and role ARN — and saves the connection. You never copy or paste ARNs.

The subagent is now connected to your S3 bucket.

### Adding files to your S3 bucket

You can add files to the bucket from the AWS console, AWS CLI, or any S3-compatible tool.

**Via AWS Console:**
1. Go to [AWS S3](https://s3.console.aws.amazon.com)
2. Find the bucket Navi created (named `navi-<agentname>-<id>`)
3. Click **Upload** → drag and drop your CSV or Excel files → click **Upload**

**Via AWS CLI:**
```bash
aws s3 cp your-data.csv s3://your-bucket-name/your-data.csv
```

**Supported formats:** `.csv`, `.xlsx`

### Loading files in the Navi UI

Once files are in your bucket:
1. Open the subagent chat
2. Click the **database icon** (cylinder icon) in the chat input bar — this opens the S3 file picker
3. Browse your files, each shown with its name and size
4. Click **Load** next to the file you want to analyze
5. Navi confirms the load with row and column counts
6. Ask your question — the subagent analyzes the loaded file

Files are loaded on demand per session. The subagent doesn't automatically pull new files — you choose which dataset to use for each conversation. This gives you full control over what data is being analyzed.

### Security model

- The S3 bucket is private and lives in your AWS account
- Navi accesses it by assuming an IAM role scoped to that bucket only
- Access uses temporary STS credentials that expire automatically
- An ExternalId tied to your subagent ID prevents unauthorized role assumption
- All stored credentials (bucket name, role ARN) are encrypted at rest with Fernet symmetric encryption
- Revoking access is instant — delete the CloudFormation stack in AWS and the connection is severed

---

## Developer API

The Navi API gives programmatic access to the same analysis engine that powers the chat interface.

### Authentication

Generate an API key from the chat page (Pro users only — click the key icon in the top right). Pass it as a Bearer token:

```bash
curl -X POST https://www.navi-os.cc/api/v1/analyze \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "message=What are the top 5 products by revenue?" \
  -F "file=@sales_data.csv"
```

### Response format

```json
{
  "reply":        "The top 5 products by revenue are...",
  "images":       ["<base64 encoded PNG>", "..."],
  "calculations": { "total_revenue": 1234567, "top_product": "Widget A" }
}
```

- `reply` — written analysis in markdown
- `images` — array of base64-encoded PNG charts (decode and display as `data:image/png;base64,...`)
- `calculations` — key-value pairs of computed metrics

### API tiers

| Plan | Price | Calls/month |
|------|-------|-------------|
| API Starter | $99/month | 500 |
| API Business | $299/month | 2,500 |

Upgrade from the chat page — click **View plans**.

---

## Pricing

| Plan | Price | What you get |
|------|-------|-------------|
| Free | $0 | 3 analyses, 1 subagent |
| Pro | $29/month | Unlimited analyses, unlimited subagents, AWS S3 |
| API Starter | $99/month | 500 API calls/month |
| API Business | $299/month | 2,500 API calls/month |

---

## How Navi thinks

Navi runs a LangGraph state machine on every query. Here's what happens when you send a message:

1. **Planner** — decides what type of analysis is needed and whether a dataset is required
2. **Dataset Detector** — LLM-based check (not keyword matching) to avoid false positives
3. **Code Generator** — writes Python for analysis and visualization using your actual data
4. **Executor** — runs the code in an isolated subprocess
5. **Self-Corrector** — if the code fails, rewrites it with the error as context and retries (up to 3 times)
6. **Ethics Gate** — reviews the response before delivery
7. **Conversational** — synthesizes a natural language response with charts and calculations

The execution log (visible in the chat UI during processing) shows every step in real time.

---

## Tips for better results

**Be specific about what you want**
- ❌ "Analyze this data"
- ✅ "Show me monthly revenue trend for 2024 and identify the three months with the biggest growth"

**Reference column names if you know them**
- ✅ "What's the average `order_value` by `customer_segment`?"

**Ask follow-up questions**
Navi remembers the conversation. You can say "now break that down by region" and it will apply that to the previous analysis.

**Use subagents for recurring workflows**
If you run the same type of analysis regularly (weekly sales reports, monthly cohort analysis), create a subagent for it. The persona keeps the analysis focused and the S3 connection means your latest data is always one click away.

**Upload context alongside data**
You can describe your business context in plain text alongside a file upload. For example: *"This is Q3 sales data. Our target was $2M per region. Analyze performance against target."*

---

## Architecture

```
Browser → Vercel (Next.js) → HuggingFace Space (FastAPI + LangGraph)
                                      ↓
                              Pinecone (memory)
                              SQLite (subagent/key metadata)
                              S3 (user data, in their AWS account)
```

**Frontend:** Next.js 14 App Router, Clerk auth, Stripe payments, Tailwind CSS
**Backend:** FastAPI, LangGraph, Claude Sonnet (planning + analysis), Groq (fast path)
**Infrastructure:** Vercel (frontend), HuggingFace Spaces (backend), AWS S3 (user data)

---

## Local Development

```bash
# Backend
cd backend
pip install -r requirements.txt
python index.py

# Frontend
cd navi-os
npm install
npm run dev
```

**Required environment variables:**

| Variable | Description |
|----------|-------------|
| `CLAUDE_API_KEY` | Anthropic API key |
| `GROQ_API_KEY` | Groq API key (fast path) |
| `PINECONE_API_KEY` | Pinecone for memory |
| `CLERK_SECRET_KEY` | Clerk server-side key |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk client-side key |
| `HF_TOKEN` | HuggingFace token (for private Space) |
| `ADMIN_SECRET` | Secret for API key creation endpoint |
| `ENCRYPTION_KEY` | Fernet key for credential encryption |
| `NAVI_AWS_ACCESS_KEY_ID` | AWS key for cross-account STS |
| `NAVI_AWS_SECRET_ACCESS_KEY` | AWS secret for cross-account STS |
| `NAVI_SERVICE_ARN` | Navi's IAM user ARN (injected into CF template) |
| `CF_TEMPLATE_URL` | S3 URL of the CloudFormation template |

---

*Navi OS — Privacy-first AI agent for data analysis and research.*
