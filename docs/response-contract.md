# Response contract

Every tool returns the same envelope.

```json
{
  "result": "...",
  "jurisdiction": "pl",
  "snapshot": "pl-2026-08",
  "sources": [],
  "coverage_status": "ok"
}
```

| Field | Always | Meaning |
|---|---|---|
| `result` | yes | the payload; `null` when there is none |
| `jurisdiction` | yes | `pl` or `eu` |
| `snapshot` | yes | the corpus state the answer comes from |
| `sources` | yes | documents the answer draws on |
| `coverage_status` | yes | see below |
| `document_id` | no | the document the call was about |
| `citation_locator` | no | where the returned text sits in its document |
| `coverage_note` | no | one human-readable sentence explaining a non-`ok` status |
| `failed_shards` | no | only with `partial` or `search_unavailable`: which storage units did not respond |
| `zywotnosc` | no | only with `get_citations(direction="incoming")` |
| `zmiany_przepisu` | no | only when a specific `provision` was asked about |

## Coverage status

A status describes what the answer is, not only whether the call succeeded.

| Status | Meaning |
|---|---|
| `ok` | the whole corpus answered |
| `partial` | some storage units did not respond; the results that came back are returned, with `coverage_note` and `failed_shards` |
| `search_unavailable` | nothing responded; `result` is `null`. This is not an empty result, it is no result |
| `not_found` | no document with that identifier in the corpus |
| `no_coverage` | the jurisdiction is not loaded in this deployment |
| `not_applicable` | the tool does not apply to this document (e.g. `trace_eu_origin` on an EU document) |
| `no_eu_link` | the PL-EU bridge knows no link from this Polish document |
| `amendments_only` | dated amendments are known; the wording on a past date is not |
| `no_version_chain` | the snapshot carries no version information for this act |
| `rate_limited`, `quota_*` | the token's limit was reached; nothing was searched |

## Locator

```json
"citation_locator": {
  "document_id": "saos:205994",
  "offset_start": 4128,
  "offset_end": 4402
}
```

Offsets are character positions. In `get_document` and in citations they count from the start of the whole document text, so `get_document` with `offset: 4128, length: 274` returns that passage.

**Known limitation:** in a search hit, offsets count from the start of the part named by `czesc`. For `czesc: 1` both bases are the same; for a later part, add the length of the preceding parts before calling `get_document`.

For citations, the locator points into the **citing** document - with `direction: "incoming"` that is the other side of the relation, not the document you asked about.

## Search hit

```json
{
  "document_id": "saos:205994",
  "title": "...",
  "doc_type": "...",
  "date": "2011-03-17",
  "court": "...",
  "act_status": null,
  "source": "saos",
  "source_url": "https://...",
  "score": 12.3456,
  "snippet": "...",
  "citation_locator": { "document_id": "saos:205994", "offset_start": 4128, "offset_end": 4402 },
  "ranking": "dokumentowy-d3",
  "provenance": [{ "document_id": "saos:205994", "source": "saos", "court": "..." }],
  "scalono_zrodel": 1,
  "czesc": 1,
  "czesci_razem": 1
}
```

`provenance` lists where the document came from. `scalono_zrodel` is the number of source records merged into this entry; on the hosted service it is currently always `1`, with one `provenance` item. Long documents are split into parts: `czesc` of `czesci_razem`.

On the envelope level, `sources[].provenance` is currently an empty array on search results; the per-hit `provenance` is the one to read.

## Citation

```json
{
  "target_document_id": "eli:DU/1964/93",
  "cited_text": "art. 415 k.c.",
  "provision": "art. 415",
  "relation": "cites_provision",
  "confidence": 0.97,
  "coverage_status": "resolved",
  "resolution_method": "...",
  "zmiany_przepisu": null,
  "citation_locator": { "document_id": "saos:205994", "offset_start": 5120, "offset_end": 5133 }
}
```

`coverage_status` on a citation is one of `resolved`, `out_of_corpus`, `in_family_unmatched`, `below_confidence`, `unparsed`. Unresolved citations are part of the result, not filtered out.

Values shown as `"..."` and the numbers in these examples are illustrative; the shapes and field names are the contract.
