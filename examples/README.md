# Examples

`<endpoint>` and `<token>` come with your access. Do not commit a token: in the MCP form it is part of the URL.

## Connect an MCP client

In Claude, add a custom connector with the URL:

```
https://<endpoint>/mcp/<token>
```

Any MCP client that speaks Streamable HTTP takes the same URL. Check the connection with `what_is_missing` - it is the cheapest call and tells you what the corpus holds.

## Call REST

```bash
curl -s https://<endpoint>/v1/search_law \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"query": "odpowiedzialnosc czlonka zarzadu", "jurisdiction": "pl", "limit": 5}'
```

## Read a response defensively

A client that treats every non-error response as complete will present half an answer as a whole one. Branch on `coverage_status` first:

```python
def hits(envelope):
    status = envelope["coverage_status"]
    if status == "ok":
        return envelope["result"], None
    if status == "partial":
        # results are real but incomplete - show them, and say so
        return envelope["result"], envelope.get("coverage_note")
    if status == "search_unavailable":
        # result is None: this is not "nothing found"
        raise RuntimeError(envelope.get("coverage_note"))
    # not_found, no_coverage, quota refusals...: nothing was searched or returned
    return [], envelope.get("coverage_note") or status
```

## Follow a citation to its passage

```python
cites = call("get_citations", {"document_id": "saos:205994", "direction": "outgoing"})
for c in cites["result"]:
    if c["coverage_status"] != "resolved":
        print("unresolved:", c["cited_text"], "-", c["coverage_status"])
        continue
    loc = c["citation_locator"]
    passage = call("get_document", {
        "document_id": loc["document_id"],
        "offset": loc["offset_start"],
        "length": loc["offset_end"] - loc["offset_start"],
    })
```
