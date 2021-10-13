# [WIP] Disease detail for metastanza template (ohta)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 0004997
  * example: MONDO_0004997(w/ nando:2200051, mesh:D002804, HP_0030432), MONDO_0005854(w/nando:1200278,2200430, mesh:D008947)

## `main`

```sparql
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?mondo ?mondo_id ?mondo_label ?mondo_definition
                (GROUP_CONCAT(DISTINCT ?related, ", ") AS ?mondo_related)
                (GROUP_CONCAT(DISTINCT ?synonym, ", ") AS ?mondo_synonym) 
                (GROUP_CONCAT(DISTINCT ?upper_class_s,",")  AS ?mondo_upper_class)
                (GROUP_CONCAT(DISTINCT ?upper_label , ",") AS ?mondo_upper_label)
WHERE { 
  VALUES ?mondo { <http://purl.obolibrary.org/obo/MONDO_{{id}}> }
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
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "ID": "mondo_id" },
    { "URL": "mondo" },
    { "label": "mondo_label" },
    { "definition": "mondo_definition" },
    { "related DB": "mondo_related" },
    { "synonym": "mondo_synonym" },
    { "upper_class": "mondo_upper_class" },
    { "upper_label": "mondo_upper_label" }
  ];
  return array;
}
```

## `return`

```javascript
({ main, columns }) => {
  const objs = main.results.bindings.map((binding) => {
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
  const class_ids = main.results.bindings[0].mondo_upper_class.value.split(/,/);
  const class_labels = main.results.bindings[0].mondo_upper_label.value.split(/,/);
  if (objs[0]["upper_class"]) {
    objs[0]["upper_class"] = "<ul>";
    for (let i=0; i<=class_ids.length; i++) {
      objs[0]["upper_class"] += "<li>" + class_ids[i] + " " +  class_labels[i] + "</li>";
    }
    objs[0]["upper_class"] += "</ul>;"
  }
  return objs;
};
```