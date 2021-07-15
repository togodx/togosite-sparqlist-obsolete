# show_graphs グラフ一覧を表示

Display all graph names in the endpoint.

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