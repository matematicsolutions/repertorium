# Coverage

Coverage means the data that is actually in the corpus - not the data that exists at the source.

## Polish corpus

1,492,530 documents from seven public sources:

| `source` | What it holds |
|---|---|
| `saos` | common courts, via the SAOS database |
| `sn` | Supreme Court of Poland |
| `nsa` | administrative courts |
| `kio` | National Appeals Chamber (public procurement) |
| `eureka` | tax interpretations of the Ministry of Finance |
| `eli` | legislation from the Sejm ELI API (Dziennik Ustaw, Monitor Polski) |
| `uodo` | decisions of the Polish data protection authority |

Every document keeps the identifier of the source it came from; see `provenance` in the [response contract](response-contract.md).

## EU corpus

236,944 documents: EU legislation and CJEU case law, identified by CELEX.

## Relations

- judgment → judgment and judgment or interpretation → statutory provision, in the Polish corpus
- Polish document → EU law, through the PL-EU bridge (1,443,748 links)
- dated amendments of provisions (`zmiana`, `uchylenie`, `dodanie`), used for the `zmiany_przepisu` warning

## Asking the corpus what it holds

The exact composition is not frozen in this file. It is returned by the service itself:

```json
{ "tool": "what_is_missing", "arguments": { "jurisdiction": "pl" } }
```

The answer carries the build date, document counts per `source` and `doc_type`, and the distribution of unresolved citations by reason. Use it before drawing a conclusion from an empty search.

## Known limits

- **Snapshot, not live.** A judgment published after the build is not in the corpus until the next build.
- **One language per document.** Language versions are separate documents, and we do not hold every act in every language. `what_is_missing` reports the split once it has been measured and tells you when it has not; the [response contract](response-contract.md) shows how to reach an act whose language version you do not have.
- **Repealed acts withheld by default.** `search_law` leaves out acts the source marks as repealed and returns everything else, case law included. Acts whose status the source does not state are SHOWN: in the Polish corpus that is every major code and most statutes (the Civil Code, the Criminal Code and the VAT act among them), so hiding them would hide Polish law itself. `in_force_only: false` returns the withheld documents alongside the rest; `hidden_by_status` counts what the default view held back.
- **No point-in-time wording.** The corpus knows that a provision was amended and when; it does not return how it read on a past date.
- **ECLI** is not yet populated in the Polish corpus; see [identifiers](identifiers.md).
- **Other jurisdictions** are not served by this interface. Corpus editions for other countries are discussed per project.
