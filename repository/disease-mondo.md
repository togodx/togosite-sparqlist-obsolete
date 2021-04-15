# Disease (高月)

- Parameter
  - queryId (Required)
- Output
  - [ {?ID ?label ?related ?synonym ?definition ?upper_class ?upper_label } ]
    
## Parameters

* `queryId` 必須パラメータ。データを表示するIDを一つ入れる
  * default: 0005043
  * example: 0005043

## Endpoint

https://integbio.jp/rdf/bioportal/sparql

## `data`

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT STR(?id) AS ?id_s STR(?label) AS ?label_s (GROUP_CONCAT(?related, ", ") AS ?related_s) (GROUP_CONCAT(?synonym, ", ") AS ?synonym_s) 
STR(?difinition) AS ?dif_s ?upper_class_s STR(?upper_label) AS ?upper_label_s
FROM <http://integbio.jp/rdf/mirror/bioportal/mondo>
WHERE {
{{#if queryId}}
  VALUES ?mondo { mondo:{{queryId}} }
{{/if}}
  ?mondo
    oboinowl:id ?id ;
    rdfs:label ?label ;
    obo:IAO_0000115 ?difinition .
  OPTIONAL {
    ?mondo
      oboinowl:hasDbXref ?related ;
      oboinowl:hasExactSynonym ?synonym ;
      rdfs:subClassOf ?upper_class .
    ?upper_class rdfs:label ?upper_label .
    BIND(STRAFTER(STR(?upper_class), "http://purl.obolibrary.org/obo/MONDO_") AS ?upper_class_s)
  }
}

```
## `return`
```javascript

({ data }) =>  {
    return data.results.bindings.map((d) => ({
      ID: d.id_s.value,
      label: d.label_s.value,
      synonym_label: d.synonym_s.value,
      difinition: d.dif_s.value,
      related_databases: d.related_s.value,
      Upper_classID: d.upper_class_s.value,
      Upper_class_label: d.upper_label_s.value
    }));
 };

```
  
## MEMO
-Author
 - Takatsuki