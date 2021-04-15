# Disease-nando (高月)

- Parameter
  - queryId (Required)
- Output
  - [ {?ID ?label ?related ?synonym ?definition ?upper_class ?upper_label } ]
    
## Parameters

* `queryId` 必須パラメータ。データを表示するIDを一つ入れる
  * default: 1200010
  * example: 1200010

## Endpoint

https://integbio.jp/rdf/sparql


## `data`

```sparql
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?id ?label ?label_jp ?description ?mondo ?source (GROUP_CONCAT(?altLabel,",") AS ?altLabel_s)
?upper_id ?upper_label
WHERE {
 GRAPH <http://nanbyodata.jp/ontology/nando> { 
   {{#if queryId}}
     VALUES ?nando { nando:{{queryId}} }
   {{/if}}
   ?nando dcterms:identifier ?id;
          rdfs:label ?label.     
   FILTER(lang(?label)= "en")
   ?nando rdfs:label ?label_jp.
   FILTER(lang(?label_jp)= "ja")
   OPTIONAL{
   ?nando dcterms:description ?description;
          skos:closeMatch ?mondo;
          dcterms:source ?source;
          skos:altLabel ?altLabel;
          rdfs:subClassOf ?upper.
   ?upper rdfs:label ?upper_label;
          dcterms:identifier ?upper_id.
   FILTER(lang(?upper_label)= "en")}
  }}
```
## `return`
```javascript

({ data }) =>  {
    return data.results.bindings.map((d) => ({
      ID: d.id.value,
      label_e: d.label.value,
      label_j: d.label_jp.value,
      synonym_label: d.altLabel_s.value,
      description: d.description.value,
      mondo: d.mondo.value,
      source: d.source.value,
      upper: d.upper_id.value,
      upper_label: d.upper_label.value
        }));
 };

```