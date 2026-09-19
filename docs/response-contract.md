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
| `coverage_note` | no | one human-readable sentence explaining a non-`ok` status, or a caveat on an `ok` answer (e.g. `trace_eu_origin` links taken from a consolidated text) |
| `failed_shards` | no | only with `partial` or `search_unavailable`: which storage units did not respond |
| `in_force_only` | no | on `search_law`: whether the answer was limited to acts in force |
| `hidden_by_status` | no | on `search_law`: `{ "nie_obowiazuje": n, "nieustalony": n }` - how many documents the default view withheld, counted separately for repealed acts and for acts whose status the source does not state. Since 2026-09-19 the second count is `0`: acts without a stated status are shown, not withheld |
| `available_in` | no | on `not_found`: the same act under an identifier we do hold, in another language |
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

Offsets are character positions counted from the start of the whole document text, in search hits, in citations and in `get_document` alike. `get_document` with `offset: 4128, length: 274` returns exactly that passage, including for long documents stored in several parts (`czesc`, numbered from 0).

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
  "status_zywotnosci": "obowiazuje",
  "jezyki_w_korpusie": ["pl", "en"],
  "ranking": "dokumentowy-d3",
  "provenance": [{ "document_id": "saos:205994", "source": "saos", "court": "..." }],
  "scalono_zrodel": 1,
  "czesc": 0,
  "czesci_razem": 1
}
```

`provenance` lists where the document came from. `scalono_zrodel` is the number of source records merged into this entry; on the hosted service it is currently always `1`, with one `provenance` item. Long documents are stored in parts: `czesc` (from 0) of `czesci_razem`; the locator is always relative to the whole document.

On the envelope level, `sources[].provenance` is currently an empty array on search results; the per-hit `provenance` is the one to read.

`status_zywotnosci` is one of `obowiazuje`, `nie_obowiazuje`, `nieustalony`, `bez_statusu`. It is not `act_status`: `act_status` repeats what the source says, and for a consolidated text the source says nothing, so it is `null`. A consolidated text takes `status_zywotnosci` from the act it consolidates, so it stays in the default in-force view. Case law is `bez_statusu` and the in-force filter never applies to it.

`jezyki_w_korpusie` lists the language versions of that act we hold. A hit with `ranking: "odwolanie-do-aktu"` did not come from the text index: an address in the query resolved to it directly - an EU act number, a Polish code or statute with an article (`392 kc`, `art. 23 uoozp`), or a Journal of Laws position (`Dz.U. 2023 poz. 955`). Such a hit carries no `score`. When a specific article was asked for, its `snippet` is the opening of that article and the locator points at it; otherwise there is no snippet. It is also the way to reach an act whose language version you asked for in a different language.

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
