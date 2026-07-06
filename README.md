# GenAI Product Description Rewriter

A content transformation pipeline that rewrites product descriptions at scale using an LLM, built to solve a real SEO problem: duplicate content across sibling websites.

## The Problem

Two websites (CtP and GfH) shared the exact same product descriptions for their gifts. Search engines penalize duplicate content, so one of the two sites was consistently getting ranked lower even though the products themselves were legitimate and distinct. The fix wasn't new content, it was rephrasing the existing descriptions so each site reads as unique, while keeping every product fact, name, and number intact.

This project automates that rephrasing across an entire product catalog, going from a single spreadsheet of HTML-formatted descriptions to a fully rewritten, SEO-safe dataset, then serves the resulting model through a simple API and UI.

## How It Works

```
Gather data → Extract relevant fields → Clean & transform HTML (BeautifulSoup)
→ Build sample dataset → Experiment across LLMs → Evaluate → Refine prompts
→ Finalize model → Process full dataset → Convert rewritten text back to HTML
```

**1. Understand the problem before touching data.** The requirement was clear: rephrase descriptions well enough that search engines treat each site as having unique content, without losing any product information.

**2. Gather and scope the data.** The source was a single Google Sheet containing everything about the CtP catalog. I explored it manually first, filtered down to the rows and columns actually needed, and dropped anything irrelevant, partly to keep the pipeline focused, partly to make sure no unrelated confidential data flowed downstream.

**3. Clean the input.** Descriptions came wrapped in HTML. I used BeautifulSoup to strip the markup before sending anything to an LLM, since raw HTML tags add token cost for zero semantic value and make the model's job harder for no reason.

**4. Build a small sample first.** Rather than run the full catalog through every candidate model (expensive and slow), I pulled a 50-row sample for experimentation. This is where trying new prompts or swapping models is cheap.

**5. Shortlist and test models.** Candidates included both open-source (Gemini, Mistral, Qwen, DeepSeek, Phi-3) and proprietary options (GPT). The goal was to find the cheapest model that still met the quality bar, only reaching for a paid model like GPT if the open-source options fell short.

**6. Evaluate objectively.** Each candidate was scored on semantic similarity, ROUGE/BLEU overlap, length ratio, and entity/number preservation, then averaged into a single comparable score per model.

**7. Track everything in MLflow.** Every prompt version and hyperparameter combination was logged, so results were comparable and nothing got lost between iterations.

**8. Pick a model.** GPT-4o-mini came out ahead, it performed consistently well across every metric and struck the best balance between cost and quality.

**9. Process at scale, cheaply.** The full dataset was run through OpenAI's Batch API to cut costs, with stateful checkpointing and retry logic so a failure partway through didn't mean starting over, plus fallback handling for individual record failures.

**10. Convert back to HTML and ship.** Rewritten text was converted back into the original HTML structure and served via a FastAPI backend with a Streamlit front end for demoing and future inference.

## Why the Batch API

The Batch API doesn't reduce how many records get processed, it reduces how many round trips your application makes to do it.

| | Regular API | Batch API |
|---|---|---|
| Records processed | 1,000 | 1,000 |
| App-to-API calls | ~1,000 | ~2-5 |
| Response time | Immediate | Up to 24 hours |
| Cost | Standard | Discounted |

In practice, a batch job looks like: upload a `.jsonl` file of requests → create the batch → poll for status a few times → download results. For a job with no real-time requirement, that trade (slower turnaround for lower cost and fewer calls) is an easy one to make.

## Keeping Track of Long-Running Jobs

Three small files did most of the heavy lifting for reliability:

| File | Stores | Example |
|---|---|---|
| `cache_file` | Already-computed results | text hash → API response |
| `progress_file` | Where processing left off | `{"last_index": 523}` |
| `state_file` | Batch job metadata | batch IDs, statuses |

Think of it as: **cache** = "I've already done this," **progress** = "I was last on item #523," **state** = "here's every batch job I've submitted and where it stands." Together they mean a crashed or interrupted run can resume instead of restarting from scratch.

## Choosing a Model: Cost vs. Quality

Each product description runs roughly 180-230 tokens, plus a ~600-token prompt and a 200-250 token response, so token cost per model was the main lever.

| Model | Notes |
|---|---|
| GPT-4.1-nano | Cheapest starting point |
| GPT-4o-mini | **Chosen** — best balance of cost and quality |
| GPT-5 mini | Higher quality, notably more expensive ($0.25/1M input, $2/1M output) |
| Gemini 1.5 Flash | Free tier via Google AI Studio, 1,500 requests/day |
| Gemini 2.0 Flash | Solid quality, still budget-friendly |

Prompt caching also helped where supported: prompts over ~1,024 tokens are sometimes cached automatically, which lowers latency and, depending on the model, cost too. Caching behavior varies enough by provider that it's worth checking before locking in a model.

## Evaluation Approach

Model quality wasn't judged on vibes. Four checks ran on every candidate output:

- **Semantic similarity** – does the meaning hold up after rewriting?
- **ROUGE / BLEU** – how much n-gram overlap remains with the original?
- **Length ratio** – did the rewrite blow up or shrink unreasonably?
- **Entity & number preservation** – using spaCy's NER to confirm product names, brands, and figures survived the rewrite untouched.

These four scores were averaged into one number per model, which is what ultimately separated GPT-4o-mini from the pack.

## Experiment Tracking with MLflow

MLflow ran underneath the whole pipeline rather than being bolted on after the fact:

- **Autologging** (`mlflow.autolog()`) captured parameters, metrics, and models with no manual instrumentation.
- **Manual logging** (`mlflow.log_param()`, `mlflow.log_metric()`) covered the custom evaluation scores specific to this task.
- **Model Registry** tracked versions as prompts and configs changed, with clear staging/production/archived states.
- **Reproducibility** was handled through MLflow Projects, which capture the exact environment and entry points so someone else (or a future me) can rerun this identically.

## Tech Stack

**Backend & serving**
`FastAPI` · `uvicorn` · `pyngrok` · `nest-asyncio` · `jinja2` · `python-multipart` · `streamlit`

**LLM & NLP**
`openai` · `transformers` (`AutoTokenizer`, `AutoModelForCausalLM`) · `accelerate` · `bitsandbytes` · `sentencepiece` · `sentence-transformers` · `spacy` (`en_core_web_sm`)

**Evaluation & tracking**
`rouge-score` · `mlflow`

**Data & utilities**
`openpyxl` · `BeautifulSoup` · `hashlib` (data integrity checks via file hashes) · `pickle` (caching and object persistence) · `os` · `json`

**Local demo tunnel**
`ngrok` — chosen over alternatives like Cloudflare Tunnel because it needs zero registration or custom domain setup; one command exposes the local FastAPI server for a quick demo.

**Frontend**
`HTML + Tailwind CSS` — kept the whole interface in a single file with no build step or separate stylesheets, which is all a lightweight demo UI needs.

## Possible Extensions

- Swap the Streamlit demo for a proper Tailwind CSS + FastAPI frontend
- Deploy the model behind a permanent REST API rather than an ngrok tunnel
- Expand MLflow usage into full production-grade tracing for the LLM pipeline

## Appendix: A Few Concepts Worth Knowing

**Batch API vs. Regular API** — same number of records processed either way; batch just means far fewer calls from your application and a discounted rate, at the cost of turnaround time.

**Embeddings vs. token embeddings** — a GPT model already converts input tokens into internal vector representations as part of generating a response; it doesn't need help for that. An *embedding model* is a separate, specialized tool that produces one reusable semantic vector for an entire piece of text, meant for search, retrieval, and clustering, not for generation.

**Pickle** — a way to serialize a Python object (a list, dict, trained model, whatever) to disk as a byte stream, and load it back later. Used here mainly for caching results so identical inputs don't get reprocessed.
