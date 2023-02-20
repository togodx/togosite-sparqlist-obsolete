# PubTator Central RDFを対象として、NCBI GeneID, Disease ID(MeSH), PubMedIDの関係を取得する

## Endpoint

https://integbio.jp/rdf/ncbi/sparql

## Parameters

* `gene_id`
  * default: 3358
  * examples: 57406, 59272, ...
* `disease_id`
  * default: D016066

## `Query`

```sparql
PREFIX oa: <http://www.w3.org/ns/oa#>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?pmid ?g_id ?d_id
FROM <http://rdf.integbio.jp/dataset/pubtator_central>
WHERE {
  BIND ((URI(CONCAT("http://identifiers.org/ncbigene/", "{{gene_id}}"))) AS ?g_id)
  BIND ((URI(CONCAT("http://identifiers.org/mesh/", "{{disease_id}}"))) AS ?d_id)
  [ oa:hasTarget ?pmid ;
    oa:hasBody ?g_id ;
    dcterms:subject "Gene" ] .
  [ oa:hasTarget ?pmid ;
    oa:hasBody ?d_id ;
    dcterms:subject "Disease" ] .
}
```

## `Return`

```javascript
({Query})=>{
  return Query.results.bindings[0]
}