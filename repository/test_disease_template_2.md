# [WIP] Data source of subject Disease (ohta)

- https://integbio.jp/togosite/sparqlist/disease-mondo
- https://integbio.jp/togosite/sparqlist/Disease-nando
- https://integbio.jp/togosite/sparqlist/disease_hpo_details
- https://integbio.jp/togosite/sparqlist/mesh_descriptor

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 0005043
  * example: 0005043
* `type`
  * default: mondo
  * example: mondo

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

SELECT ?hp ?mesh ?mondo ?nando

{{#if idDict.hp}}
VALUES ?hp { hp:{{idDict.hp}} }
GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
}
{{/if}}

{{#if idDict.mesh}}
VALUES ?mesh { mesh:{{idDict.mesh}} }
GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
}
{{/if}}

{{#if idDict.mondo}}
VALUES ?mondo { mondo:{{idDict.mondo}} }
GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
}
{{/if}}

{{#if idDict.nando}}
VALUES ?nando { nando:{{idDict.nando}} }
GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
}
{{/if}}
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
        obj[key].push(node.value);
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

SELECT DISTINCT
WHERE {
  {{#if togoidDict.hp}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
    ?hpo go:id ?hp.
    ?hpo obo:IAO_0000115 ?definition.
    ?hpo go:hasAlternativeId ?alt_id.
    ?hpo go:hasDbXref ?dbxref.
    ?hpo go:hasExactSynonym ?exac_synonym.
    ?hpo go:hasOBONamespace ?obo_ns.
    ?hpo go:hasRelatedSynonym ?related_synonym.
    ?hpo rdfs:comment ?comment.
    ?hpo rdfs:label ?label.
    OPTIONAL {
      ?hpo rdfs:seeAlso ?hp_seealso.
    }
    ?hpo rdfs:subClassOf ?subclass.
  }
  {{/if}}
    
  {{#if togoidDict.mesh}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/mesh> {
    ?mesh a meshv:TopicalDescriptor ;
          rdfs:label ?label;
          meshv:treeNumber ?tree_uri;
          meshv:preferredConcept ?concept.
    ?concept meshv:scopeNote ?note_temp.

    BIND(IF(bound(?note_temp), ?note_temp,"null") AS ?scope_note) 

    BIND (substr(str(?mesh), 28) AS ?id)
    BIND (substr(str(?tree_uri),28) AS ?tree_temp)
  }
  {{/if}}

  {{#if togoidDict.mondo}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/mondo> {
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
  {{/if}}
    
  {{#if togoidDict.nando}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> {
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
      FILTER(lang(?upper_label)= "en")
    }
  }
  {{/if}}
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    ID: d.id.value
  }));
};
```