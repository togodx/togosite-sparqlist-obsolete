# Disease detail for metastanza template (ikeda)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 0030432
  * example: 0030432

## `main`

```sparql
PREFIX go: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX hp: <http://identifiers.org/HP/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?hpo ?hpo_id ?hpo_label ?hpo_definition 
       (GROUP_CONCAT(DISTINCT ?hpo_alt_id_s, ",") AS ?hpo_alt_id) 
       (GROUP_CONCAT(DISTINCT ?hpo_dbxref_s, ",") AS ?hpo_dbxref)
       ?hpo_comment
       (GROUP_CONCAT(DISTINCT ?hpo_parent_class_s, ",") AS ?hpo_parent_class)  
       (GROUP_CONCAT(DISTINCT ?hpo_parent_class_label_s, ",") AS ?hpo_parent_class_label)  
       (GROUP_CONCAT(DISTINCT ?hpo_exact_synonym_s, ",")AS ?hpo_exact_synonym)
       (GROUP_CONCAT(DISTINCT ?hpo_related_synonym_s, ",") AS ?hpo_related_synonym)
       ?hpo_obo_ns
       (GROUP_CONCAT(DISTINCT ?hpo_seealso, ",") AS ?hpo_seealso)
WHERE {
  VALUES ?hpo { <http://purl.obolibrary.org/obo/HP_{{id}}> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
    ?hpo rdfs:label ?hpo_label .
    OPTIONAL {?hpo obo:IAO_0000115 ?hpo_definition_temp .}
    OPTIONAL {?hpo go:hasAlternativeId ?hpo_alt_id_s .}
    OPTIONAL {?hpo go:hasDbXref ?hpo_dbxref_s .}
    OPTIONAL {?hpo rdfs:comment ?hpo_comment_temp .}
    OPTIONAL {?hpo rdfs:subClassOf ?hpo_parent_class_s .
              ?hpo_parent_class_s rdfs:label ?hpo_parent_class_label_s . }
    OPTIONAL {?hpo go:hasExactSynonym ?hpo_exact_synonym_s .}
    OPTIONAL {?hpo go:hasOBONamespace ?hpo_obo_ns_temp .}
    OPTIONAL {?hpo go:hasRelatedSynonym ?hpo_related_synonym_s .}
    OPTIONAL {?hpo rdfs:seeAlso ?hpo_seealso .}
    BIND (replace(str(?hpo), 'http://purl.obolibrary.org/obo/HP_', 'HP:') AS ?hpo_id)
    BIND (IF(bound(?hpo_definition_temp), ?hpo_definition_temp, "") AS ?hpo_definition)
    BIND (IF(bound(?hpo_comment_temp), ?hpo_comment_temp, "") AS ?hpo_comment)
    BIND (IF(bound(?hpo_obo_ns_temp), ?hpo_obo_ns_temp, "") AS ?hpo_obo_ns)
  }
}
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "URL ": data.hpo?.value,
    "ID ": data.hpo_id?.value,
    "label": data.hpo_label?.value,
    "definition": data.hpo_definition?.value,
    "altID": data.hpo_alt_id?.value,
    "xrefs": data.hpo_dbxref?.value,
    "comment": data.hpo_comment?.value,
    "subclass_of": data.hpo_parent_class?.value,
    "exact_synonym": data.hpo_exact_synonym?.value,
    "related_synonym": data.hpo_related_synonym?.value,
    "seeAlso": data.hpo_seealso?.value,
    "obo_ns": data.hpo_obo_ns?.value
  };

  const class_ids = main.results.bindings[0].hpo_parent_class.value.split(/,/);
  const class_labels = main.results.bindings[0].hpo_parent_class_label.value.split(/,/);
  if (objs[0]["subclass_of"]) {
    objs[0]["subclass_of"] = "<ul>";
    for (let i=0; i<class_ids.length; i++) {
      objs[0]["subclass_of"] += "<li><a href=\"" + class_ids[i]
                                + "\" target=\"_blank\">" + class_ids[i].replace("http://purl.obolibrary.org/obo/","")
                                + "</a>" + " " +  class_labels[i] + "</li>";
    }
    objs[0]["subclass_of"] += "</ul>";
  }
  if (objs[0].xrefs) objs[0].xrefs = makeList(objs[0].xrefs, ",");
  return objs;

  function makeList(str, sep) {
    return "<ul><li>" + str.replace(new RegExp(sep, 'g'), "</li><li>") + "</li></ul>";
  };
};
```