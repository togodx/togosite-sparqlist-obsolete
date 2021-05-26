# show_graphs

Display all graph names in the endopint.
This sparqlist is monitored by uptime monitoring services, [Status cake](https://www.statuscake.com/) and [Uptimerobot](https://uptimerobot.com/).

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