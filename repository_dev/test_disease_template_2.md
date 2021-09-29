# [WIP] Disease detail for metastanza template (ohta)

- https://integbio.jp/togosite/sparqlist/disease-mondo
- https://integbio.jp/togosite/sparqlist/Disease-nando
- https://integbio.jp/togosite/sparqlist/disease_hpo_details
- https://integbio.jp/togosite/sparqlist/mesh_descriptor

## Endpoint description


- Data sources
  - NANDO: https://github.com/aidrd/nando
  - Mondo: https://mondo.monarchinitiative.org
  - HP: https://hpo.jax.org
  - MeSH: https://www.ncbi.nlm.nih.gov/mesh


## Test pattern

* [type=mondo(MONDO_0004997)で対応する全ての値が1つずつ存在する](https://integbio.jp/togosite/sparqlist/api/test_disease_template_3_mitsuhashi?id=0004997&type=mondo)
* [type=nando(nando:2200051)で対応する全ての値が1つずつ存在する](https://integbio.jp/togosite/sparqlist/api/test_disease_template_3_mitsuhashi?id=2200051&type=nando)
* [type=mesh( mesh:D002804)で対応する全ての値が1つずつ存在する](https://integbio.jp/togosite/sparqlist/api/test_disease_template_3_mitsuhashi?id=D002804&type=mesh)
* [type=hp( HP_0030432)で対応する全ての値が1つずつ存在する](https://integbio.jp/togosite/sparqlist/api/test_disease_template_3_mitsuhashi?id=0030432&type=hp)
* [type=mondo(MONDO_0005854)でNANDOが２つ(1200278,2200430)存在する](https://integbio.jp/togosite/sparqlist/api/test_disease_template_3_mitsuhashi?id=0005854&type=mondo)

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

## `main`

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX go: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX hp: <http://identifiers.org/HP/>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT *
WHERE {    
{{#if idDict.hp}}
  {
    SELECT ?hpo_id ?hpo_label ?hpo_definition 
           (GROUP_CONCAT(DISTINCT ?hpo_alt_id_s, ",") AS ?hpo_alt_id) 
           (GROUP_CONCAT(DISTINCT ?hpo_dbxref_s, ",") AS ?hpo_dbxref)
           ?hpo_comment 
           (GROUP_CONCAT(DISTINCT ?hpo_subclass_s, ",") AS ?hpo_subclass)  
           (GROUP_CONCAT(DISTINCT ?hpo_exact_synonym_s, ",")AS ?hpo_exact_synonym)
           (GROUP_CONCAT(DISTINCT ?hpo_related_synonym_s, ",") AS ?hpo_related_synonym)
           ?hpo_obo_ns
           (GROUP_CONCAT(DISTINCT ?hpo_seealso, ",") AS ?hpo_seealso)
    WHERE {
      VALUES ?hpo { <http://purl.obolibrary.org/obo/HP_{{idDict.hp}}> }
      GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
        ?hpo rdfs:label ?hpo_label.
      OPTIONAL {?hpo obo:IAO_0000115 ?hpo_definition_temp.}
      OPTIONAL {?hpo go:hasAlternativeId ?hpo_alt_id_s.}
      OPTIONAL {?hpo go:hasDbXref ?hpo_dbxref_s.}
      OPTIONAL {?hpo rdfs:comment ?hpo_comment_temp.}
      OPTIONAL {?hpo rdfs:subClassOf ?hpo_subclass_s.}
      OPTIONAL {?hpo go:hasExactSynonym ?hpo_exact_synonym_s.}
      OPTIONAL {?hpo go:hasOBONamespace ?hpo_obo_ns_temp.}
      OPTIONAL {?hpo go:hasRelatedSynonym ?hpo_related_synonym_s.}
      OPTIONAL {?hpo rdfs:seeAlso ?hpo_seealso.}
        BIND (replace(str(?hpo), 'http://purl.obolibrary.org/obo/HP_', 'HP:') AS ?hpo_id)
        BIND (IF(bound(?hpo_definition_temp), ?hpo_definition_temp, "") AS ?hpo_definition)
        BIND (IF(bound(?hpo_comment_temp), ?hpo_comment_temp, "") AS ?hpo_comment)
        BIND (IF(bound(?hpo_obo_ns_temp), ?hpo_obo_ns_temp, "") AS ?hpo_obo_ns)
    }
   }
  }
{{/if}}
{{#if idDict.mesh}}
  {
    SELECT ?mesh_id ?mesh_label ?mesh_tree_uri ?mesh_concept ?mesh_scope_note
    FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
    WHERE { 
      VALUES ?mesh { <http://id.nlm.nih.gov/mesh/{{idDict.mesh}}> }
      VALUES ?type { meshv:TopicalDescriptor meshv:SCR_Disease }
      ?mesh a ?type.
      ?mesh rdfs:label ?mesh_label.
      OPTIONAL { ?mesh meshv:treeNumber ?mesh_tree_uri. }
      ?mesh meshv:preferredConcept ?mesh_concept.
      OPTIONAL { ?mesh_concept meshv:scopeNote ?mesh_note_temp. }

      BIND(IF(bound(?mesh_note_temp), ?mesh_note_temp,"null") AS ?mesh_scope_note) 
      BIND (substr(str(?mesh), 28) AS ?mesh_id)
      BIND (substr(str(?mesh_tree_uri),28) AS ?mesh_tree_temp)
    }
  }
{{/if}}

{{#if idDict.mondo}}
  {
    SELECT DISTINCT ?mondo_id ?mondo_label ?mondo_definition
                    (GROUP_CONCAT(DISTINCT ?related, ", ") AS ?mondo_related)
                    (GROUP_CONCAT(DISTINCT ?synonym, ", ") AS ?mondo_synonym) 
                    (GROUP_CONCAT(DISTINCT ?upper_class_s,",")  AS ?mondo_upper_class)
                    (GROUP_CONCAT(DISTINCT ?upper_label , ",") AS ?mondo_upper_label)
    WHERE { 
     VALUES ?mondo { <http://purl.obolibrary.org/obo/MONDO_{{idDict.mondo}}> }
     GRAPH <http://rdf.integbio.jp/dataset/togosite/mondo> {
      ?mondo oboinowl:id ?mondo_id ;
         rdfs:label ?mondo_label .
      OPTIONAL {?mondo obo:IAO_0000115 ?mondo_definition_temp .}
      OPTIONAL {?mondo oboinowl:hasDbXref ?related .}
      OPTIONAL {?mondo oboinowl:hasExactSynonym ?synonym .}
      OPTIONAL {?mondo rdfs:subClassOf ?upper_class .
                ?upper_class rdfs:label ?upper_label.}
      BIND(REPLACE(STR(?upper_class), "http://purl.obolibrary.org/obo/MONDO_","MONDO:") AS ?upper_class_s)
      BIND(IF(bound(?mondo_definition_temp), ?mondo_definition_temp, "") AS ?mondo_definition)
    }
   }
  }
{{/if}}
    
{{#if idDict.nando}}
  {
    SELECT DISTINCT ?nando ?nando_id ?nando_label  ?nando_label_jp ?nando_description ?nando_source 
                    (GROUP_CONCAT(DISTINCT ?nando_altLabel_s, ",")AS ?nando_altLabel) ?nando_upper_label ?nando_upper_id
                    (GROUP_CONCAT(DISTINCT ?nando_mondo_s, ",") AS ?nando_mondo)
                    
    WHERE { 
     VALUES ?nando { <http://nanbyodata.jp/ontology/NANDO_{{idDict.nando}}> }
     GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> {
      ?nando dcterms:identifier ?nando_id;
             rdfs:label ?nando_label.
      FILTER(lang(?nando_label)= "en")
      ?nando rdfs:label ?nando_label_jp.
      FILTER(lang(?nando_label_jp)= "ja")
      OPTIONAL{?nando dcterms:description ?nando_description_temp.}
      OPTIONAL{?nando skos:closeMatch ?nando_mondo_s.}
      OPTIONAL{?nando dcterms:source ?nando_source_temp.}
      OPTIONAL{?nando skos:altLabel ?nando_altLabel_s.}
      OPTIONAL{?nando rdfs:subClassOf ?nando_upper.
               ?nando_upper rdfs:label ?nando_upper_label.
               ?nando_upper dcterms:identifier ?nando_upper_id.
        FILTER(lang(?nando_upper_label)= "en") }
     
      BIND(IF(bound(?nando_description_temp), ?nando_description_temp,"") AS ?nando_description)
      BIND(IF(bound(?nando_source_temp), ?nando_source_temp, "") AS ?nando_source)
    }
   }
  }
{{/if}}
}
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "Mondo_ID": "mondo_id" },
    { "Mondo_label": "mondo_label" },
    { "Mondo_definition": "mondo_definition" },
    { "Mondo_relatedDB": "mondo_related" },
    { "Mondo_synonym": "mondo_synonym" },
    { "Mondo_upperClass": "mondo_upper_class" },
    { "Mondo_upperLabel": "mondo_upper_label" },
    { "HPO_ID ": "hpo_id" },
    { "HPO_label": "hpo_label" },
    { "HPO_definition": "hpo_definition" },
    { "HPO_altID": "hpo_alt_id" },
    { "HPO_relatedDB": "hpo_dbxref" },
    { "HPO_comment": "hpo_comment" },
    { "HPO_upperClass": "hpo_subclass" },
    { "HPO_exact_synonym": "hpo_exact_synonym" },
    { "HPO_related_synonym": "hpo_related_synonym" },
    { "HPO_seeAlso": "hpo_seealso" },
    { "HPO_obo_ns": "hpo_obo_ns" },
    { "MeSH_ID": "mesh_id" },
    { "MeSH_label": "mesh_label" },
    { "MeSH_tree_URI": "mesh_tree_uri" },
    { "MeSH_concept": "mesh_concept" },
    { "MeSH_scope_note": "mesh_scope_note" },
    { "NANDO_ID": "nando_id" },
    { "NANDO_label": "nando_label" },
    { "NANDO_label_ja": "nando_label_jp" },
    { "NANDO_description": "nando_description" },
    { "NANDO_source": "nando_source" },
    { "NANDO_altLabel": "nando_altLabel" },
    { "NANDO_MONDO_related": "nando_mondo"},
    { "NANDO_upperClass": "nando_upper_id" },
    { "NANDO_upperLabel": "nando_upper_label" }
  ];
  return array;
}
```

## `return`

```javascript
({ main, columns }) => {
  return main.results.bindings.map((binding) => {
    const results = columns.map((row) => {
      const obj = {};
      for (const [k, v] of Object.entries(row)) {
        obj[k] = binding[v];
      }
      return obj;
    });

    return results.reduce((obj, elem) => {
      for (const [key, node] of Object.entries(elem)) {
        if (node) {
          obj[key] = node.value;
        }
        return obj;
      };
    }, {});
  });
};
```