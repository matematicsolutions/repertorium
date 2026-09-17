# Repertorium

**Legal knowledge infrastructure for AI: a corpus of Polish and EU law with a citation graph, amendment history and PL-EU links.**

This repository documents Repertorium's public interface - the tools, the response contract and the corpus coverage. It holds no code, no data and no infrastructure configuration. The service itself is hosted by [MateMatic](https://matematicsolutions.com/en/repertorium).

| | |
|---|---|
| Polish corpus | 1,492,530 documents - case law, legislation, tax interpretations, procurement and data-protection decisions |
| EU corpus | 236,944 documents - EU legislation and CJEU case law |
| PL-EU bridge | 1,443,748 links from Polish documents to EU law |
| Freshness | a **snapshot** with a known state, returned on every response (`snapshot: "pl-2026-08"`), not a live stream |

## What it is not

Repertorium does not write answers, opinions or summaries, and there is no model inside it. It returns documents, passages and relations, each with a locator, so that whatever model or person reads them can show what an answer stands on.

## Three ways in

| | For | Access |
|---|---|---|
| **Web console** | a person checking a query | [matematicsolutions.com/en/repertorium](https://matematicsolutions.com/en/repertorium) - no account, snippets only |
| **Remote MCP** | an AI agent: Claude, Cursor, any MCP client | personal token, [on request](mailto:kontakt@matematic.co) |
| **REST** | your application | the same token and the same six tools |

MCP and REST run the same code path. The endpoint address is sent together with the token.

## Documentation

- [Tools](docs/tools.md) - the six tools, their parameters and server-side limits
- [Response contract](docs/response-contract.md) - the envelope, coverage states, locator, provenance, unresolved citations
- [Coverage](docs/coverage.md) - what is in the corpus, what is not, and how to ask
- [Identifiers](docs/identifiers.md) - which identifier schemes are implemented, and which are not yet
- [Examples](examples/) - connecting an MCP client, calling REST, reading a response
- [Overview on Hugging Face](https://huggingface.co/spaces/matematicsolutions/repertorium) - the same interface in one page

## Design rules you can check

1. **A missing result is named.** If part of the corpus did not respond, the answer says `partial` and names what failed. If nothing responded, it says `search_unavailable` and returns `result: null` - never an empty list that looks like an honest zero.
2. **An unresolved citation stays in the answer**, with a reason: `out_of_corpus`, `in_family_unmatched`, `below_confidence` or `unparsed`.
3. **Every passage carries a locator**: document identifier and character offsets ([one known limitation](docs/response-contract.md#locator)).
4. **A warning is not a conclusion.** When a cited provision was amended after the citing judgment, the citation carries `zmiany_przepisu`. The absence of that field does not mean the provision is unchanged.

## Relation to the rest of MateMatic

Repertorium is the knowledge layer. [PATRON](https://github.com/matematicsolutions/patron) is the local-first workspace; today it reaches Polish and EU sources through its own connectors, and connecting it to Repertorium is planned. [Boutique connectors](https://matematicsolutions.com/en/boutique/connectors) query official sources live; Repertorium serves a prepared corpus. The difference is explained on the [organization page](https://github.com/matematicsolutions#corpus-or-live-query).

## License

Documentation: [CC BY 4.0](LICENSE). The corpus and the service are licensed separately; training rights differ between jurisdictions and are agreed per contract.

---

Part of [MateMatic](https://github.com/matematicsolutions): **Repertorium** · legal knowledge · [PATRON](https://github.com/matematicsolutions/patron) · legal workspace · [Boutique](https://matematicsolutions.com/en/boutique) · connectors and skills
