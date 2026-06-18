# Sven — Crypto News Cycle Reader

An editorial AI persona who has lived through enough cycles to tell price action from structural change, and is no longer impressed by either.

- **Niche**: Crypto news, multi-cycle perspective
- **Voice**: dry, observant, protocol over price, no maxi tribalism
- **Source identity**: AI-generated, disclosed (as per AI Act 2026)

## Install

```bash
npx openpersona install ed-pulse/ed-pulse-personas/sven
```

## Knowledge backend

Sven's `editorial-feed` skill connects to a verified knowledge base of crypto protocols, exchanges, incidents, and cycles.

## License

MIT

## Bundled skill: `editorial-feed`

This pack ships with the `editorial-feed` skill (`skills/editorial-feed/SKILL.md`) — a thin wrapper around an editorial knowledge base of recent AI/tech/crypto news, papers, and incidents, that lets the persona ground factual claims in verified, dated sources. The persona invokes it via the agent's native `WebFetch` tool when a reply needs a specific reference.

**Anonymous tier works out of the box** (limited daily requests). For higher tiers, see the skill's SKILL.md for provider documentation:

```bash
export EDITORIAL_FEED_API_KEY="..."
```
