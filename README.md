# AI Agent Readiness Checklist

A practical, vendor-neutral reference for making a website work for AI agents —
search-grounded answer engines, autonomous browsing agents, and the crawlers
that feed them. Maintained by [Scovant](https://scovant.com).

> Scovant is an AI agent readiness testing platform that scans, simulates, and continuously monitors websites to determine how reliably AI agents can discover, understand, and interact with them.

**Data:** the recommendations below are informed by the
[Scovant Agent Web Index](https://scovant.com/research) — a monthly, open,
machine-readable measurement of agent readiness across the web
([JSON](https://scovant.com/research/agent-web-index.json) ·
[CSV](https://scovant.com/research/agent-web-index.csv) ·
[latest](https://scovant.com/research/latest.json)).

---

## 1. Know the three intents of AI crawlers

Not every "AI bot" does the same thing. Decide your policy per **intent**,
not per vendor:

| Intent | What it does | User agents |
|---|---|---|
| **Search** | Fetches pages to ground live answers, usually with citations back to you | `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`, `Applebot` |
| **Agent / user** | Fetches a page on behalf of one user's live request (browsing agents, assistants) | `ChatGPT-User`, `Claude-User`, `Perplexity-User`, `DuckAssistBot` |
| **Training** | Collects content for model training | `GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `CCBot` |

Blocking *search* and *agent* intents removes you from AI answers and breaks
agent-assisted visits. Blocking *training* is a separate, legitimate policy
choice — many sites allow the first two and opt out of the third.

- Example robots.txt allowing everything: [`examples/robots-allow-all-ai.txt`](examples/robots-allow-all-ai.txt)
- Example allowing search + agent, opting out of training: [`examples/robots-search-and-agent-only.txt`](examples/robots-search-and-agent-only.txt)

## 2. Make your declared policy match reality

A robots.txt that *allows* an agent while your WAF/CDN serves it a CAPTCHA is
worse than an explicit block — the agent wastes its attempt and your site
reads as broken, not protected.

- Check your CDN's bot-management defaults; several ship AI-crawler blocks
  that are ON unless you opt out.
- Test with a real request using the bot's user agent from outside your
  network, not just by reading the config.
- If you verify crawlers, prefer cryptographic identity (Web Bot Auth /
  HTTP message signatures, RFC 9421) over user-agent string matching —
  UA strings are trivially forged in both directions.

## 3. llms.txt

A plain-text index at `/llms.txt` telling language models what your site is
and where the high-value pages are. Low effort, growing adoption.

- Keep it generated from your real content, not hand-written — a stale
  llms.txt that advertises removed pages is an anti-signal.
- Example: [`examples/llms.txt`](examples/llms.txt)

## 4. Structured data

Agents parse markup far more reliably than rendered pixels.

- JSON-LD `Organization` / `WebSite` / `Product` / `Offer` with **stable
  `@id`s** referenced consistently across pages, so crawlers merge your
  entities instead of splitting them.
- Ship prices, availability, and variants in the markup, not only in
  JavaScript-rendered DOM.
- Client-rendered SPAs: verify your key pages have server-rendered or
  prerendered HTML — many crawlers do not execute JavaScript.

## 5. MCP (Model Context Protocol)

If you expose an agent-facing API, declare it so agents can find it:

- Discovery file at `/.well-known/mcp.json` pointing at your MCP endpoint —
  example: [`examples/well-known-mcp.json`](examples/well-known-mcp.json)
- The declared server name should match what the endpoint's `initialize`
  handshake reports.
- Tool names and descriptions are your contract: keep them stable, avoid
  reserved protocol method names, and never embed instructions to the
  calling model in tool descriptions.

## 6. WebMCP

`navigator.modelContext` — an in-page API exposing site actions to
browser-embedded agents. Early: adoption in the wild is still effectively
zero (see the Agent Web Index), but the pattern to prepare for is the same
as MCP: declared tools with honest names, descriptions, and input schemas.

## 7. UCP (Universal Commerce Protocol)

For commerce sites: a machine-readable profile declaring checkout
capabilities and signing keys, so purchasing agents can transact without
scraping. If you publish one, validate it — a schema-invalid profile is
worse than none.

## 8. Task completion, not just crawlability

The hardest failures are interactive: an agent that can *read* your site but
cannot *use* it.

- Cookie/consent banners and newsletter popups must be dismissible without
  a mouse hover or drag.
- Cart, checkout, and booking flows should work without CAPTCHA challenges
  on ordinary paths.
- Forms need real `<label>`s, native controls, and predictable validation.
- Test with an actual browsing agent completing an actual task — static
  checkers cannot see these failures.

## 9. Regressions: agent readiness decays

Every deploy can silently break any of the above. Treat agent readiness as
a **regression class**: baseline it, re-test on deploy, and diff — the same
way you treat performance budgets.

## Testing

You can verify most of this checklist by hand (curl with each bot's user
agent, a schema validator, an MCP client).

**Automated, free:** the open-source scanner
[Scovant Core](https://github.com/Scovant/scovant-core) checks the passive
parts of this list from the outside — robots and AI-crawler policy, llms.txt,
structured data, MCP/UCP discovery, security baseline — with a public scoring
model. `pipx run scovant-core scan https://your-site.example`, or the
`Scovant/scovant-core@v0` GitHub Action as a CI gate (see
[ci-examples](https://github.com/Scovant/ci-examples)).

**Task completion and regressions:** simulating real AI agents through your
flows and comparing every deploy against a baseline is what
[Scovant](https://scovant.com) does — the full rule catalog and scoring
model are public: [rules](https://scovant.com/rules) ·
[checks spec](https://github.com/Scovant/agent-readiness-checks) ·
[scoring methodology](https://scovant.com/scoring) ·
[glossary](https://scovant.com/glossary).

## Contributing

Corrections and additions welcome — especially real-world examples of
WAF/CDN configurations that block agents unintentionally. Keep entries
vendor-neutral and verifiable.

## License

[CC BY 4.0](LICENSE) — reuse freely with attribution to
[Scovant](https://scovant.com). The
[Agent Web Index dataset](https://scovant.com/research) is published under
the same license.
