# show_graphs

Display all graph names in the endopint.
This sparqlist is monitored by alive checking system.

- author: Mitsuhashi

## Parameters

* `ep` Endpoint
  * default: https://integbio.jp/togosite/sparql

## Endpoint

{{ ep }}

## `result`

```sparql
SELECT DISTINCT ?g
WHERE {
  GRAPH ?g { ?s ?p ?o }
}
```