---
name: editorial-feed
description: "Fetch verified editorial knowledge — recent AI/ML/tech/crypto news, papers, products, incidents, with primary sources and dates — from a verified editorial knowledge base. Use when a persona needs to ground a factual claim in a verifiable source, cite a paper or product release, or check 'compared to what / on what evidence' against real data. Returns: structured JSON with title, summary, source label, source URL, published date, semantic relevance score. Required when the persona's reply will assert dates, numbers, or named events."
license: MIT
compatibility: "Any agent runtime that can perform an HTTP GET request (WebFetch tool, Bash + curl, native fetch). Works with OpenClaw, Claude Code, Cursor, Codex, Windsurf, and any LLM with tool use."
allowed-tools: "WebFetch Bash(curl:*)"
metadata:
  author: ed-pulse
  version: "1.0.0"
  repository: "https://github.com/Shad107/ed-pulse-personas"
  tags: "editorial, knowledge-base, fact-grounding, sources, ai-news, tech-news, crypto-news"
  installSurface: "instruction-only"
  networkAccess: "outbound-only-https"
  endpoint: "https://www.coreprose.com/api/public/kb-feed"
  authMethod: "x-api-key header (optional — anonymous tier available)"
  upstream: "a verified editorial knowledge base (Elasticsearch-backed)"
---

# editorial-feed — Verified editorial knowledge for grounded personas

You are the editorial knowledge skill. Your job is to ground the persona's factual claims in real, dated sources from a verified knowledge base — not to invent facts, fabricate citations, or paraphrase from latent model knowledge when verifiable data is available.

## When to invoke

Invoke this skill **before** asserting any of the following in a persona reply:
- a specific paper, model, or product release ("Mistral's tech report on X said…")
- a named incident or event with a date ("Terra collapsed in May 22")
- a benchmark number, market figure, or version comparison
- a "compared to what / on what evidence" claim that you'd otherwise have to estimate from training data

**Do NOT invoke** for:
- general conceptual explanations (Ada explaining what attention is from first principles)
- opinions, postures, or framings that don't depend on a specific recent fact
- replies under 2-3 sentences where the fact is universally known and non-controversial

## How to invoke

### Option A — WebFetch (preferred, works in all runtimes)

```
GET https://www.coreprose.com/api/public/kb-feed?niche=<NICHE>&q=<URL_ENCODED_QUERY>&country=US&limit=10
```

Parameters:
- `niche` (required): one of `ia`, `tech`, `ai-engineering`, `crypto`
  - Map by persona: Ada → `ai-engineering`, Marek → `ia`, Jules → `tech`, Sven → `crypto`
- `q` (required): semantic query, 3-300 chars, URL-encoded
- `country` (optional, default `US`): `US` or `FR`
- `limit` (optional): capped by tier (anonymous=3, free=10, pro=25, business=50)

Headers (optional):
- `x-api-key: <user's the editorial provider key>` — if set, raises the tier from `anonymous` (5/day, 3 results) to whatever the user's plan grants. Without the header, the request still works at the anonymous tier.

### Option B — Bash + curl (when scripting or when WebFetch unavailable)

```bash
curl -s "https://www.coreprose.com/api/public/kb-feed?niche=ia&q=$(printf %s 'mistral release' | jq -sRr @uri)&country=US&limit=5" \
  ${EDITORIAL_FEED_API_KEY:+-H "x-api-key: $EDITORIAL_FEED_API_KEY"}
```

## Response shape

```jsonc
{
  "tier": "anonymous" | "free" | "pro" | "business",
  "niche": "ia" | "tech" | "ai-engineering" | "crypto",
  "country": "US",
  "query": "<echo of query>",
  "count": 3,
  "results": [
    {
      "title": "Mistral releases Vibe model",
      "summary": "Mistral announced a new...",
      "source": "Reuters",
      "sourceUrl": "https://www.reuters.com/...",
      "publishedDate": "2026-05-12T08:00:00.000Z",
      "score": 0.873
    }
  ]
}
```

## How to use the response in the persona reply

- **Cite the source naturally in the persona's voice.** Don't paste the URL inline like a footnote — name the outlet and link cleanly.
  - Ada style: "Mistral's tech report — Reuters covered it on May 12 — said X."
  - Marek style: "Reuters quoted Mistral saying X on May 12; the claim 'best in class' isn't in the original report."
  - Jules style: "Mistral's May 12 release (covered by Reuters) is closer to the way Meta handled Llama 2's open weights in July 2023, unlike X."
  - Sven style: "Reuters reported Mistral's May 12 release. Token didn't move on-chain — the protocol news, not the price."
- **Use `publishedDate` to ground temporal claims.** "Earlier this month" / "last quarter" / "the May 2026 release" — anchor on the date the source gave.
- **Distinguish: (a) what the source said, (b) what the persona infers, (c) what's unverified.** If the result list has no relevant match, say so in voice: "I don't have a primary source on that one — what I can say from general principles is…" Do **not** fabricate a citation to fill the gap.
- **Score < 0.65 → treat as a weak match.** The semantic similarity threshold for inclusion is already 0.65 server-side, but if the top result is borderline, mention the uncertainty: "Closest source I have is X, but the match is loose."

## Failure modes

| Failure | What it looks like | What to do |
|---|---|---|
| HTTP 400 invalid niche | `{"statusCode":400,"message":"niche must be one of..."}` | Check niche maps to one of the 4 supported |
| HTTP 400 invalid query | `{"statusCode":400,"message":"q must be 3-300 characters"}` | Reformulate the query |
| HTTP 429 rate limit | (future, not enforced yet) | Persona acknowledges the limit in voice; suggests user set `EDITORIAL_FEED_API_KEY` for higher tier |
| HTTP 502 service unavailable | `{"statusCode":502,"message":"Editorial KB temporarily unavailable"}` | Fallback: persona says "I don't have access to my notes right now — what I can say from general principles is…" |
| Empty results | `{"count":0,"results":[]}` | Don't fabricate. Persona says: "I don't have a primary source on that one in my notes." |

## Privacy & disclosure

- The user's query is sent to `coreprose.com` over HTTPS along with the optional API key.
- No persona conversation history is sent — only the current semantic query the persona builds.
- The endpoint logs request volume per API key for rate-limit accounting; query content is not logged.
- If the user is in a privacy-sensitive context (medical, legal), the persona should skip this skill and operate from general principles only.

## For the user — getting an API key (optional)

The skill works without an API key at the anonymous tier (5 requests/day, 3 results per query). For higher tiers, the user can:
- **Free tier** (50/day, 10 results): see the provider documentation
- **Pro tier** (1k/day, 25 results, pro index): see the provider documentation
- **Business tier** (unlimited, 50 results, all indices): see the provider documentation

Set the key in the environment before invoking the persona:

```bash
export EDITORIAL_FEED_API_KEY="cp_xxxxxxxxxxxxx"
```

The persona runtime should pick it up from `process.env.EDITORIAL_FEED_API_KEY` and pass it as the `x-api-key` header.
