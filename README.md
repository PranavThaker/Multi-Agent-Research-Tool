# 🧠 Multi-Agent Research Tool

An autonomous research pipeline that takes a single topic and returns a structured, fact-checked report — built by orchestrating four cooperating LLM agents (Search → Read → Write → Critique) instead of one monolithic prompt.

## Overview

Ask it any topic, and the tool will:
1. **Search** the live web for recent, reliable sources
2. **Read** the most relevant source in depth by scraping its full content
3. **Write** a structured research report from the gathered material
4. **Critique** the report against a strict evaluation rubric and return a score with actionable feedback

The output isn't just a summary — it's a self-reviewed report with sources cited, giving the user both the answer and a sense of how trustworthy/complete it is.

## Why a multi-agent pipeline?

A single LLM call asked to "research and write about X" tends to hallucinate sources or skip depth. This project splits the task the way a human research team would — a scout who finds sources, an analyst who reads deeply, a writer who drafts, and an editor who pressure-tests the draft — with each agent given only the tools and context it needs for its job. This keeps outputs grounded in real, verifiable URLs and adds a built-in quality gate before the report is considered final.

## Architecture

```
                 ┌───────────────────┐
   topic ───────▶│   Search Agent     │  (Tavily web_search tool)
                 └─────────┬──────────┘
                           │ ranked sources (titles, URLs, snippets)
                           ▼
                 ┌───────────────────┐
                 │   Reader Agent     │  (BeautifulSoup scrape_url tool)
                 └─────────┬──────────┘
                           │ deep content from the most relevant source
                           ▼
                 ┌───────────────────┐
                 │   Writer Chain     │  (LLM, structured prompt)
                 └─────────┬──────────┘
                           │ draft report (intro, findings, conclusion, sources)
                           ▼
                 ┌───────────────────┐
                 │   Critic Chain     │  (LLM, rubric-based review)
                 └─────────┬──────────┘
                           ▼
                 score + strengths + areas to improve
```

Each agent is a self-contained LangChain `create_agent` instance (for the tool-using Search/Reader agents) or an LLM chain (for the Writer/Critic, which only need reasoning, not tools). The `pipeline.py` orchestrator passes state between them sequentially, printing progress at each step.

## Tech stack

| Layer | Tool |
|---|---|
| Agent framework | LangChain (`create_agent`) |
| LLM inference | Groq (`openai/gpt-oss-120b`) via `langchain-groq` |
| Web search | Tavily Search API |
| Web scraping | Requests + BeautifulSoup4 |
| Prompting | LangChain `ChatPromptTemplate` + `StrOutputParser` |
| Config | `python-dotenv` |
| CLI output | `rich` |

## Project structure

```
Multi-Agent-Research-Tool/
├── agents.py        # Agent + chain definitions (Search, Reader, Writer, Critic)
├── tools.py          # Tool implementations: web_search (Tavily), scrape_url (BS4)
├── pipeline.py        # Orchestrates the 4-stage pipeline end-to-end
├── requirements.txt
└── .gitignore
```

## Setup

**1. Clone and install dependencies**
```bash
git clone https://github.com/PranavThaker/Multi-Agent-Research-Tool.git
cd Multi-Agent-Research-Tool
pip install -r requirements.txt
```

**2. Configure API keys**

Create a `.env` file in the project root:
```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

**3. Run**
```bash
python pipeline.py
```
You'll be prompted to enter a research topic, and the pipeline will stream its progress through each of the four stages in the terminal.

## Example

```
Enter a research topic: Impact of quantum computing on cryptography

==================================================
Step 1 - Search agent is working ...
==================================================
...
==================================================
Step 4 - Critic is reviewing the report ...
==================================================

Score: 8/10
Strengths:
- Clear structure with well-supported key findings
- Sources are accurately preserved and cited
...
```
