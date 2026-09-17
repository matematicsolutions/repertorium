# Identifiers and standards

What the interface implements today, and what it does not. A standard is listed as implemented only when it appears in the data the service returns.

| Scheme | Status | In the interface |
|---|---|---|
| **CELEX** | implemented | EU documents: `celex:31993L0104:en` (number and language) |
| **ELI** | implemented as an identifier prefix | Polish legislation: `eli:DU/1964/93` (publisher / year / position). This is the Polish ELI key, not a full ELI URI |
| **Source identifiers** | implemented | `saos:205994` and similar - the source's own id behind a source prefix; some carry the source's URN |
| **ECLI** | not yet | the merging logic can use it, but the Polish corpus currently holds no ECLI values |
| **Akoma Ntoso** | not implemented | documents are returned as plain text with character offsets |
| **W3C PROV** | not implemented | `provenance` is a MateMatic field set (`document_id`, `source`, `court`), not a PROV-O serialization |
| **MLOS** | not implemented | - |

## Why the table says "not implemented"

Some national connectors in this organization do read Akoma Ntoso or native ELI URIs at their source (for example Finland, Luxembourg, Italy). Repertorium's interface does not expose those formats, and saying otherwise here would be a claim about architecture that a response would contradict.

Mapping `provenance` to W3C PROV and exposing full ELI URIs are reasonable next steps. They will be listed as implemented when a response carries them.
