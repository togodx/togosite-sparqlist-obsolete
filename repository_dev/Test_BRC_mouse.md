# 理研BRCのマウスのテスト

## Endpoint

https://knowledge.brc.riken.jp/sparql

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX brs: <http://metadb.riken.jp/db/bioresource_schema/brs_>

SELECT ?category_label (COUNT (?category) AS ?count)
WHERE {
   GRAPH <http://metadb.riken.jp/db/xsearch_animal> { 
    ?brcID brs:strainTypeLink ?category.
    ?category rdfs:label ?category_label.
  }
} 
 ORDER BY DESC(?count)
```