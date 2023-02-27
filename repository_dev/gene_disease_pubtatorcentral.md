# PubTator Central RDFを対象として、NCBI GeneIDとDisease ID(MeSH)の関係を、関連するPubMedIDの件数降順で取得する （山本）

## Description
PubTator Centralから取得してRDF化したデータに対し、NCBI GeneID または Disease ID(MeSH) のいずれか、もしくは両者を引数として与える。
* NCBI GeneIDが与えられた場合は、関連PubMedIDの多い順にDisease ID(MeSH)を、PubMedIDの件数と共に表示する。
* Disease ID(MeSH)が与えられた場合は、NCBI GeneIDについて表示する。
* 両者が与えられた場合は、それらを含むPubMedIDの件数を表示する。

## Endpoint

https://integbio.jp/rdf/ncbi/sparql

## Parameters

* `gene_id` ( NCBI Gene ID )
  * default: 59272
  * examples: 57406, 59272, ...
* `disease_id` ( MeSH ID )
  * default: C000657245
  * examples: D000086382, D000086402

## `gene_id`
```javascript
({gene_id})=>{
  if( gene_id == "" ){
    return "?g_id"
  } else {
    return "<http://identifiers.org/ncbigene/" + gene_id + ">"
  }
}
```

## `disease_id`
```javascript
({disease_id})=>{
  if( disease_id == "" ){
    return "?d_id"
  } else {
    return "<http://identifiers.org/mesh/" + disease_id + ">"
  }
}
```

## `Query`

```sparql
PREFIX oa: <http://www.w3.org/ns/oa#>
PREFIX dcterms: <http://purl.org/dc/terms/>

{{#if gene_id == "?g_id"}}
SELECT DISTINCT ?g_id (count(?pmid) as ?c)
{{else if disease_id == "?d_id"}}
SELECT DISTINCT ?d_id (count(?pmid) as ?c)
{{else}}
SELECT DISTINCT (count(?pmid) as ?c)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/pubtator_central>
WHERE {
  [ oa:hasTarget ?pmid ;
    oa:hasBody {{gene_id}} ;
    dcterms:subject "Gene" ] .
  [ oa:hasTarget ?pmid ;
    oa:hasBody {{disease_id}} ;
    dcterms:subject "Disease" ] .
}
{{#if gene_id == "?g_id"}}
GROUP BY ?g_id
{{else if disease_id}}
GROUP BY ?d_id
{{/if}}
ORDER BY DESC(?c)
LIMIT 10
```

## `Return`

```javascript
({Query})=>{
  if (Query.head.vars[0] == "c"){
     return Query.results.bindings.map(d=>d.c.value)[0]
  } else if (Query.head.vars[0] == "g_id"){
     return Query.results.bindings.map(d=>d.g_id.value).join(",")
  } else if (Query.head.vars[0] == "d_id") {
     return Query.results.bindings.map(d=>d.d_id.value).join(",")
  } else{
    return Query.head.vars[0]
  }
}