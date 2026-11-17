---
name: duckduckgo-search
description: The single keyless DuckDuckGo web search path. No API key required. Use when web_search is unavailable or unsuitable, or when DuckDuckGo-specific text, news, image, or video results are wanted. Multi-provider aggregation with failover is out of scope here — use free-search-aggregator for that.
source: nousresearch/hermes-agent or openclaw/skills
pipeline_stage: DEFINE
---

# duckduckgo-search

**Pipeline Stage:** DEFINE (Stage 1 – Discover & Research)

**Purpose:** Perform free web searches using DuckDuckGo without requiring an API key. Use when web_search is unavailable or unsuitable, or when DuckDuckGo-specific results are desired.

**Boundary:** this is the **only** skill that claims the keyless DuckDuckGo trigger. `free-search-aggregator` handles 12-provider aggregation and failover and explicitly defers single DuckDuckGo queries here, so exactly one skill competes for the trigger.

**When to use:** During DEFINE stage for research-heavy pipelines when you need current information, documentation, tutorials, or any information requiring web search.

**Requirements:** ddgs CLI or Python package installed locally

**Detection flow:**
1. If ddgs CLI is installed, prefer terminal + ddgs
2. If ddgs CLI is missing, verify ddgs Python package before importing
3. Install ddgs only when DuckDuckGo search is specifically needed

**CLI usage:**
```bash
ddgs -q "search terms" -m 5 -r us-en -t w -s off -o json
```

**Python API:**
```python
from ddgs import DDGS
results = DDGS().text("search terms", max_results=5)
```

**Handoff contract:**
- Input: Search query
- Output: Search results with titles, links, snippets
- Handoff to: PLAN stage (domain-modeling → to-spec)

**Install:**
```bash
pip install ddgs
npx skills add nousresearch/hermes-agent --skill duckduckgo-search --agent '*' -y
# or
npx skills add https://github.com/openclaw/skills --skill duckduckgo-search --agent '*' -y
```
