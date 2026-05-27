# Navi OS

**AI-powered data analysis, live web research, and business intelligence — from a single prompt.**

[Try Navi →](https://bi-agent-v2.vercel.app)

---

## What is Navi?

Navi is a self-correcting AI agent that turns natural language prompts into complete analytical workflows. Upload a dataset and ask a question, or point it at a live website — Navi writes its own Python, executes it, catches errors, fixes them automatically, and returns structured results with charts and calculations.

It is not a wrapper around a language model. It is a stateful reasoning engine with a planning layer, an ethics gate, a memory system, and a self-correction loop that runs without user intervention.

---

## What Navi can do

### Data Analysis
- Analyze datasets of any size from a single prompt — tested on 541,909 rows
- Generate publication-quality charts with a unified dark theme
- Perform statistical analysis, segmentation, and trend detection
- Handle CSV and Excel files including non-UTF-8 encodings (Latin-1, Windows-1252)

### Live Web Research
- Scrape live data from public websites using requests + BeautifulSoup
- Synthesize information from multiple sources into structured reports
- Tested on financial data sites, news aggregators, and market data feeds

### Business Intelligence
- Revenue timelines with month-over-month growth annotations
- Geographic market breakdowns and customer segmentation
- Cancellation and churn analysis
- Strategic synthesis with findings and recommended actions

---

## How it works

Every request runs through a multi-stage pipeline:

```
Memory Recall → Ethics Gate → Planner → Skill Creator → Executor → Auditor → Response
```

1. **Memory Recall** — retrieves relevant context from past sessions via vector search
2. **Ethics Gate** — evaluates the request before any code runs; blocks malicious requests at the infrastructure level
3. **Planner** — determines the approach and manages retry logic
4. **Skill Creator** — writes complete, executable Python tailored to the specific task
5. **Executor** — runs the skill in an isolated thread with a 90-second timeout
6. **Auditor** — validates the output; triggers a rewrite if results are incomplete or incorrect
7. **Response** — synthesizes a professional Markdown report with verified calculations

If a skill fails, Navi diagnoses the error, searches for solutions, and rewrites the skill automatically — without any input from the user.

---

## Privacy model

Navi is built on a privacy-first architecture. This is not a policy — it is a design decision baked into the infrastructure.

**What Navi does not store:**
- User identity or personal information beyond authentication
- Uploaded file contents — files are processed in memory and discarded when the session ends
- Personal details entered in prompts

**What Navi's memory system stores:**
- Semantic embeddings of task types and analytical approaches — not who performed them
- Example: *"revenue analysis on retail dataset returned peak month November"* — not the user who ran it
- No names, company names, or identifying information are intentionally captured

**Practical guidance:**
Describe your data by type rather than identity. Write *"analyze this sales dataset"* rather than including company names, client names, or personal details in the prompt. Treat prompts like search queries.

The ethics layer intercepts and blocks malicious requests before any code executes. Navi does not log what was blocked or who attempted it — the infrastructure handles it without human review.

---

## Enterprise architecture — Navi Subagent Library *(coming soon)*

For organizations that need AI agents trained on proprietary data, Navi OS offers a bring-your-own-cloud deployment model.

**How it works:**
- Subagents are deployed into the organization's own AWS VPC
- Company data never leaves their infrastructure — Navi's servers never see it
- Each subagent inherits the full Navi OS architecture including the ethics gate and self-correction loop
- Organizations control their own IAM permissions, storage, and configuration
- Up to 5 specialized subagents per library (e.g. marketing analyst, stock analyst, operations, and more)
- Deployment is automated via Terraform templates — no manual AWS configuration required

**Why this matters:**
The primary barrier to enterprise AI adoption is trust — specifically, whether a company's proprietary data is safe in a third-party system. Navi's architecture removes that concern entirely. There is no data to leak because Navi never holds it. The organization's data lives in their own virtual cloud environment under their own access controls.

This is not a data processing agreement or a security audit. It is an architectural guarantee.

---

## Pricing

| Plan | Queries | Price |
|------|---------|-------|
| Free | 3 queries | $0 |
| Pro | Unlimited | $29/month |

Cancel anytime through the billing portal. Sign in with your existing gmail account.

---

## Writing effective prompts

**Be specific about what you want back**

Instead of: *"analyze my data"*

Try: *"Calculate monthly revenue, identify the top 5 products by margin, and create a bar chart ranked by profit"*

**Name the visualizations you want**

Navi generates charts based on explicit requests. Specify chart type — bar, line, scatter, pie — and what dimensions to use.

**For web scraping, provide the full URL**

Instead of: *"scrape crypto prices"*

Try: *"Go to https://coinmarketcap.com/gainers-losers/ and scrape the top 10 gainers with price and 24h change"*

---

## Architecture

Built with:

| Component | Technology |
|-----------|-----------|
| Agent framework | LangGraph |
| Reasoning | Claude Sonnet (Anthropic) |
| Fast routing | Groq / Llama 3.1 8B |
| Vector memory | Pinecone |
| Backend API | FastAPI with SSE streaming |
| Frontend | Next.js |
| Compute | HuggingFace Spaces (Docker, 16GB RAM) |
| Frontend hosting | Vercel |

The backend repository is private. The architecture described here is accurate and complete.

---

## Roadmap

- [ ] Subagent library — specialized agents in customer-owned AWS VPCs
- [ ] One-click Terraform deployment for enterprise subagent setup
- [ ] Multi-source analysis — combine uploaded datasets with live web data
- [ ] Expanded file format support — JSON, Parquet, database connections
- [ ] Machine learning primitives — regression, classification, clustering
- [ ] API access for programmatic integration

---

## Contact

Built by Jordan Wright — solo founder.

For business inquiries, collaboration, or feedback: connect via LinkedIn.

---

*Navi OS — Privacy-first AI agent for data analysis and research.*
