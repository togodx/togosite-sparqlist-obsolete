# [WIP] Data source of subject Disease (takatsuki)

- https://integbio.jp/togosite/sparqlist/disease-mondo
- https://integbio.jp/togosite/sparqlist/Disease-nando
- https://integbio.jp/togosite/sparqlist/disease_hpo_details
- https://integbio.jp/togosite/sparqlist/mesh_descriptor

## Test pattern

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 0004997
  * example: MONDO_0004997(w/ nando:2200051, mesh:D002804, HP_0030432), MONDO_0005854(w/nando:1200278,2200430, mesh:D008947)
* `type`
  * default: mondo
  * example: mondo,nando,mesh,hp

## `idDict` returns an id type object

```javascript
({id, type}) => {
  var obj = {};
  switch (type) {
    case 'hp':
      obj.hp = id;
      break;
    case 'mesh':
      obj.mesh = id;
      break;
    case 'mondo':
      obj.mondo = id;
      break;
    case 'nando':
      obj.nando = id;
      break;
  }
  return obj;
}
```

## `togoid` resolve id relations

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX go: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX hp: <http://purl.obolibrary.org/obo/HP_>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?hp REPLACE(STR(?mesh_id), "https://identifiers.org/mesh/", "http://id.nlm.nih.gov/mesh/") AS ?mesh ?mondo ?nando
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
{{#if idDict.hp}}
  VALUES ?hp { <http://purl.obolibrary.org/obo/HP_{{idDict.hp}}> }
  ?mondo rdfs:seeAlso ?hp.
{{/if}}
{{#if idDict.mesh}}
  VALUES ?mesh { <https://identifiers.org/mesh/{{idDict.mesh}}> }
  ?mondo rdfs:seeAlso ?mesh.
{{/if}}
{{#if idDict.mondo}}
  VALUES ?mondo { <http://purl.obolibrary.org/obo/MONDO_{{idDict.mondo}}> }
{{/if}}
{{#if idDict.nando}}
  VALUES ?nando { <http://nanbyodata.jp/ontology/nando#{{idDict.nando}}> }
  ?mondo rdfs:seeAlso ?nando.
{{/if}}
  FILTER(STRSTARTS(STR(?mondo), "http://purl.obolibrary.org/obo/MONDO_"))  
  {
     ?mondo rdfs:seeAlso ?nando.
     FILTER(STRSTARTS(STR(?nando), "http://nanbyodata.jp/ontology/nando"))  
    }UNION {
      ?mondo rdfs:seeAlso ?mesh_id.
      FILTER(STRSTARTS(STR(?mesh_id), "https://identifiers.org/mesh/"))   # TODO: replace https: with http: after virtuoso is updated.
    }UNION {
      ?mondo rdfs:seeAlso ?hp.
      FILTER(STRSTARTS(STR(?hp), "http://purl.obolibrary.org/obo/HP_"))
    }
 }
}
```

## `togoidDict` returns an id relation object 

- Create a object from the rows of sparql query result (`Array.reduce`)
  - loop for columns of a row (`for Object.entries`)

```javascript
({togoid}) => {
  return togoid.results.bindings.reduce(
    (obj, elem) => {
      for (const [key, node] of Object.entries(elem)) {
        if (!obj[key]) obj[key] = [];
        if (!obj[key].includes(node.value)) obj[key].push(node.value);
      };
      return obj;
    }, {}
  );
}
```

## `main`

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX go: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX hp: <http://identifiers.org/HP/>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT *
WHERE {    
{{#if togoidDict.hp}}
{
  SELECT ?hpo  ?hpo_label ?hpo_definition ?hpo_alt_id ?hpo_dbxref ?hpo_comment
                 ?hpo_subclass  ?hpo_exac_synonym ?hpo_obo_ns ?hpo_related_synonym ?hpo_seealso
  WHERE {
    VALUES ?hpo { {{#each togoidDict.hp}} <{{this}}> {{/each}} }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
      ?hpo rdfs:label ?hpo_label.
      ?hpo obo:IAO_0000115 ?hpo_definition.
    OPTIONAL {
      ?hpo go:hasAlternativeId ?hpo_alt_id.
      ?hpo go:hasDbXref ?hpo_dbxref.
      ?hpo rdfs:comment ?hpo_comment.
      ?hpo rdfs:subClassOf ?hpo_subclass.
      ?hpo go:hasExactSynonym ?hpo_exac_synonym.
      ?hpo go:hasOBONamespace ?hpo_obo_ns.
      ?hpo go:hasRelatedSynonym ?hpo_related_synonym.
      ?hpo rdfs:seeAlso ?hpo_seealso.
    }
  }
 }
}                   
{{/if}}
{{#if togoidDict.mesh}}
{
  SELECT ?mesh ?mesh_label ?mesh_tree_uri ?mesh_concept ?mesh_scope_note
  WHERE { 
    VALUES ?mesh { {{#each togoidDict.mesh}} <{{this}}> {{/each}} }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/mesh> {
      ?mesh a meshv:TopicalDescriptor;
          rdfs:label ?mesh_label;
          meshv:treeNumber ?mesh_tree_uri;
          meshv:preferredConcept ?mesh_concept.
      ?mesh_concept meshv:scopeNote ?mesh_note_temp.

      BIND(IF(bound(?mesh_note_temp), ?mesh_note_temp,"null") AS ?mesh_scope_note) 

      BIND (substr(str(?mesh), 28) AS ?mesh_id)
      BIND (substr(str(?mesh_tree_uri),28) AS ?mesh_tree_temp)
   }
 }
}
{{/if}}

{{#if togoidDict.mondo}}
{
SELECT DISTINCT ?mondo_id ?mondo_label ?mondo_definition
                  (GROUP_CONCAT(DISTINCT ?related, ", ") AS ?mondo_related)
                  (GROUP_CONCAT(DISTINCT ?synonym, ", ") AS ?mondo_synonym) 
                  (GROUP_CONCAT(DISTINCT ?upper_class_s,",")  AS ?mondo_upper_class)
                  (GROUP_CONCAT(DISTINCT ?upper_label , ",") AS ?mondo_upper_label)
  WHERE { 
   VALUES ?mondo { {{#each togoidDict.mondo}} <{{this}}> {{/each}} }
   GRAPH <http://rdf.integbio.jp/dataset/togosite/mondo> {
    ?mondo oboinowl:id ?mondo_id ;
       rdfs:label ?mondo_label ;
       obo:IAO_0000115 ?mondo_definition .
    OPTIONAL {
      ?mondo oboinowl:hasDbXref ?related ;
        oboinowl:hasExactSynonym ?synonym ;
        rdfs:subClassOf ?upper_class .
      ?upper_class rdfs:label ?upper_label .
      BIND(STRAFTER(STR(?upper_class), "http://purl.obolibrary.org/obo/MONDO_") AS ?upper_class_s)
    }
  }
 }
}
{{/if}}
    
{{#if togoidDict.nando}}
{
  SELECT ?nando ?nando_id ?nando_label  ?nando_label_jp ?nando_description ?nando_mondo ?nando_source
                ?nando_altLabel ?nando_upper_label ?nando_upper_id
  WHERE { 
   VALUES ?nando { {{#each togoidDict.nando}} <{{this}}> {{/each}} }
   GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> {
    ?nando dcterms:identifier ?nando_id;
           rdfs:label ?nando_label.
    FILTER(lang(?nando_label)= "en")
    ?nando rdfs:label ?nando_label_jp.
    FILTER(lang(?nando_label_jp)= "ja")
    OPTIONAL{
      ?nando dcterms:description ?nando_description;
             skos:closeMatch ?nando_mondo;
             dcterms:source ?nando_source;
             skos:altLabel ?nando_altLabel;
             rdfs:subClassOf ?nando_upper.
      ?nando_upper rdfs:label ?nando_upper_label;
             dcterms:identifier ?nando_upper_id.
      FILTER(lang(?nando_upper_label)= "en")
    }
  }
 }
}
{{/if}}
}
```

## `return`

```javascript

({ main }) =>  {
    return main.results.bindings.map((d) => ({
      Mondo_ID: d.mondo_id.value,
      Mondo_label: d.mondo_label.value,
      Mondo_definition: d.mondo_definition?.value,
      Mondo_relatedDB: d.mondo_related?.value,
      Mondo_synonym: d.mondo_synonym?.value,
      Mondo_upperClass: d.mondo_upper_class?.value,
      Mondo_upperLabel: d.mondo_upper_label?.value,
      HPO_ID : d.hpo?.value,
      HPO_label: d.hpo_label?.value,
      HPO_definition: d.hpo_definition?.value,
      HPO_altID: d.hpo_alt_id?.value,
      HPO_relatedDB: d.hpo_dbxref?.value,
      HPO_comment: d.hpo_comment?.value,
      HPO_upperClass: d.hpo_subclass?.value,
      HPO_exact_synonym: d.hpo_exac_synonym?.value,
      HPO_related_synonym: d.hpo_related_synonym?.value,
      HPO_seeAlso: d.hpo_seealso?.value,
      HPO_obo_ns: d.hpo_obo_ns?.value,
      MeSH_ID: d.mesh?.value,
      MeSH_label: d.mesh_label?.value,
      MeSH_tree_URI: d.mesh_tree_uri?.value,
      MeSH_concept: d.mesh_concept?.value,
      MeSH_scope_note: d.mesh_scope_note?.value,
      NANDO_ID: d.nando_id?.value,
      NANDO_label: d.nando_label?.value,
      NANDO_label_ja: d.nando_label_jp?.value,
      NANDO_description: d.nando_description?.value,
      NANDO_source: d.nando_source?.value,
      NANDO_altLabel: d.nando_altLabel?.value,
      NANDO_upperClass: d.nando_upper_id?.value,
      NANDO_upperLabel: d.nando_upper_label?.value
           }));
 };

```