# HPOの詳細情報を出す(三橋・高月)

## Parameters
* `hp`
    * default:
    * example:0001250

## Endpoint

https://integbio.jp/togosite/sparql



## `data`

```sparql
PREFIX hp: <http://identifiers.org/HP/>
PREFIX go: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT ?hp ?hpo ?definition ?alt_id ?exac_synonym ?obo_ns ?related_synonym ?comment ?label ?hp_seealso ?subclass
WHERE {
  VALUES ?hpo { <http://purl.obolibrary.org/obo/HP_0030432> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo>{ 
    OPTIONAL {
    ?hpo go:id ?hp.
    ?hpo obo:IAO_0000115 ?definition.
    ?hpo go:hasAlternativeId ?alt_id.
    ?hpo go:hasDbXref ?dbxref.
    ?hpo go:hasExactSynonym ?exac_synonym.
    ?hpo go:hasOBONamespace ?obo_ns.
    ?hpo go:hasRelatedSynonym ?related_synonym.
    ?hpo rdfs:comment ?comment.
    ?hpo rdfs:label ?label.
    ?hpo rdfs:seeAlso ?hp_seealso.
    ?hpo rdfs:subClassOf ?subclass.
     }
   }
} 
```

## `return`
```javascript

({ data }) =>  {
    return data.results.bindings.map((d) => ({
      hp: d.hp.value,
      hpo: d.hpo.value, 
      definition: d.definition.value,
      alt_id: d.alt_id.value,
      exac_synonym: d.exac_synonym.value,  
      obo_ns: d.obo_ns.value,
      related_synonym: d.related_synonym.value,  
      comment: d.comment.value,
      label: d.label.value,
//      hp_seealso: d.hp_seealso.value,
      subclass: d.subclass.value
    }));
 };

```
  
## MEMO
-Author
 - Takatsuki