# Tools

Six tools, identical over MCP and REST. Limits are clamped on the server: a client may ask for more, the server decides.

| Tool | Parameters | Returns |
|---|---|---|
| `search_law` | `query` (required), `jurisdiction` = `pl` \| `eu` (default `pl`), `limit` (default 10, max 50), `doc_type` (optional filter) | ranked documents with a snippet and a locator |
| `get_document` | `document_id` (required), `offset` (default 0), `length` (default 4000, max 20,000 characters) | one document's text window, with its offsets and total length |
| `get_citations` | `document_id` (required), `direction` = `outgoing` \| `incoming` (default `outgoing`), `limit` (default 50, max 200), `provision` (optional) | citations with resolution status and locator |
| `trace_eu_origin` | `document_id` of a Polish document (required), `limit` (default 50, max 200) | known links from that document to EU law |
| `trace_history` | `document_id` (required) | what the snapshot knows about an act's versions or amendments |
| `what_is_missing` | `jurisdiction` (default `pl`), `document_id` (optional) | unresolved citations by reason, plus corpus composition |

## Notes that change how you call them

**`trace_eu_origin` returns EU acts the Polish text CITES, not proof of transposition.** A link means the text names that EU act. For a Polish statute whose original text predates the EU citations (a code from 1964, an act from 1994), the links come from its newest consolidated text that has them, and `coverage_note` names that text. A consolidated text also quotes footnotes of the amending acts it incorporates, so a cited directive can belong to a neighbouring subject. Treat the list as a starting point for checking the transposition, not as the answer.

**Call `what_is_missing` before concluding that something is absent.** Without `document_id` it returns `sklad`: document counts per `source` and per `doc_type`, counted from the corpus. These are exactly the values the `doc_type` filter accepts. An empty result on a wrong filter value looks the same as an empty corpus.

**`get_citations` with `provision`** works on a legal act with `direction: "incoming"` and returns the judgments citing one specific provision:

```json
{ "document_id": "eli:DU/1964/93", "direction": "incoming", "provision": "art. 415" }
```

The provision is written canonically: `art. N`, then optionally `§ N`, `ust. N`, `pkt N`, `lit. x`. `art. N` alone covers all of its lower-level units.

**`get_citations` with `direction: "incoming"`** adds `zywotnosc` to the envelope: citations per year, the last citation and citations in the last five years, together with `rok_odniesienia`, the year of the latest document in the corpus. A citation count alone does not tell a live line of case law from an abandoned one. Compare `ostatnie_cytowanie` with `rok_odniesienia`, not with today's date: the end of the snapshot is not the end of the line.

**`get_citations` with `direction: "outgoing"`** marks each citation of a provision with `zmiany_przepisu` when that provision was amended after the date of the citing document. It is a warning that the interpretation may be outdated, not the historical wording of the provision.

**`trace_history`** returns the act's state on the snapshot date, with `amendments_only` when dated amendments are known (the wording on a past day is not) or `no_version_chain` when nothing is known. `known_versions` is reserved for a version chain and is currently `null`. The tool does not reconstruct how a provision read on a past date.

## MCP specifics

- Transport: Streamable HTTP, POST, JSON responses, no SSE stream, no session.
- Handshake: the 2026-07-28 `server/discover` flow and `initialize` for 2025-11-25, 2025-06-18 and 2025-03-26. Other versions are refused with `-32020`.
- Each result is returned both as `content[].text` (JSON) and as `structuredContent`.
- The token is part of the connector URL (`.../mcp/<token>`), because client connector screens often have no field for an `Authorization` header.

## REST specifics

```
POST /v1/<tool_name>
Authorization: Bearer <token>
Content-Type: application/json

{ ...tool arguments... }
```

An unknown tool returns 404 `unknown_tool`. A quota refusal returns HTTP 429 with the refusal code in the envelope.

## Tool descriptions are in Polish

The descriptions a client sees in `tools/list`, and the human-readable `coverage_note`, are written in Polish. Field names and status values are stable identifiers; a few of them are Polish words (`zywotnosc`, `zmiany_przepisu`, `sklad`) and are documented here as they are.
