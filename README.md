# StoreTechScan agent skill

`SKILL.md` teaches an AI agent how to use [StoreTechScan](https://storetechscan.ytriple.dev): bulk-check ecommerce store domains for storefront platform, installed apps, public Shopify catalog prices, prospect fit and public role contacts, over REST or MCP, with a one-time 25-domain free trial per IP that needs no key (first credit pack purchase adds +25 bonus credits: Starter $9 = 525).

The file is a mirror of the live version at <https://storetechscan.ytriple.dev/skill.md>, which is generated from the server and is always the source of truth.

## Install

```bash
npx skills add userydev/storetechscan-skill
```

Or copy `SKILL.md` into your agent's skills directory (Claude Code: `.claude/skills/storetechscan/SKILL.md`).

## Related

- Quickstart for agents: <https://storetechscan.ytriple.dev/llms.txt>
- OpenAPI 3.1: <https://storetechscan.ytriple.dev/openapi.json>
- MCP server (Streamable HTTP): `https://storetechscan.ytriple.dev/mcp` · registry id `dev.ytriple.storetechscan/storetechscan`
- Sample response: <https://storetechscan.ytriple.dev/samples/inspect-response.json>

## License

MIT
