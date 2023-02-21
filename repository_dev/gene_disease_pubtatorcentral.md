# PubTator Central RDFを対象として、NCBI GeneID, Disease ID(MeSH), PubMedIDの関係を取得する

## Endpoint

https://integbio.jp/rdf/ncbi/sparql

## Parameters

* `gene_id`
  * default: 59272
  * examples: 57406, 59272, ...
* `disease_id`
  * default: C000657245
  * examples: D000086382, D000086402

## `Query`

```sparql
PREFIX oa: <http://www.w3.org/ns/oa#>
PREFIX dcterms: <http://purl.org/dc/terms/>

{{#if gene_id}}
SELECT DISTINCT ?d_id (count(?pmid) as ?c)
 {{else if disease_id}}
SELECT DISTINCT ?g_id (count(?pmid) as ?c)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/pubtator_central>
WHERE {
{{#if gene_id}}
  VALUES ?g_id { URI(CONCAT("http://identifiers.org/ncbigene/",{{gene_id}})) }
 {{else if disease_id}}
  VALUES ?d_id { URI(CONCAT("http://identifiers.org/mesh/",{{disease_id}})) }
{{/if}}
  [ oa:hasTarget ?pmid ;
    oa:hasBody ?g_id ;
    dcterms:subject "Gene" ] .
  [ oa:hasTarget ?pmid ;
    oa:hasBody ?d_id ;
    dcterms:subject "Disease" ] .
}
GROUP BY ?d_id
ORDER BY DESC(?c)
LIMIT 10
```

## `Return`

```javascript
({Query})=>{
  return Query.results.bindings[0]
}