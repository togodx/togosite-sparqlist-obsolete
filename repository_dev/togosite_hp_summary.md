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

SELECT ?hpo ?id ?label ?definition
       (GROUP_CONCAT(DISTINCT ?dbxref, ",") AS ?dbxrefs)
       ?comment
       (GROUP_CONCAT(DISTINCT ?parent_class, ",") AS ?parent_classes)
       (GROUP_CONCAT(DISTINCT ?parent_class_label, ",") AS ?parent_class_labels)
       (GROUP_CONCAT(DISTINCT ?exact_synonym, ",")AS ?exact_synonyms)
       (GROUP_CONCAT(DISTINCT ?related_synonym, ",") AS ?related_synonyms)
WHERE {
  VALUES ?hpo { <http://purl.obolibrary.org/obo/HP_{{id}}> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
    ?hpo rdfs:label ?label .
    OPTIONAL { ?hpo obo:IAO_0000115 ?definition . }
    OPTIONAL { ?hpo go:hasDbXref ?dbxref . }
    OPTIONAL { ?hpo rdfs:comment ?comment . }
    OPTIONAL { ?hpo go:hasExactSynonym ?exact_synonym . }
    OPTIONAL { ?hpo go:hasRelatedSynonym ?related_synonym . }
    OPTIONAL { ?hpo rdfs:subClassOf ?parent_class .
               ?parent_class rdfs:label ?parent_class_label . }
    BIND (replace(str(?hpo), 'http://purl.obolibrary.org/obo/HP_', 'HP:') AS ?id)
  }
}
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "URL": data.hpo.value,
    "ID": data.id.value,
    "label": data.label.value,
    "definition": data.definition?.value ?? "",
    "xrefs": data.dbxrefs?.value ?? "",
    "comment": data.comment?.value ?? "",
    "subclass_of": data.parent_classes?.value ?? ""
    //"exact_synonym": data.exact_synonym?.value,
    //"related_synonym": data.related_synonym?.value
  };

  if (objs[0]["subclass_of"]) {
    const class_urls = data.parent_classes.value.split(/,/);
    const class_labels = data.parent_class_labels.value.split(/,/);
    objs[0]["subclass_of"] = "<ul>";
    for (let i=0; i<class_urls.length; i++) {
      objs[0]["subclass_of"] += "<li><a href=\"" + class_urls[i] + "\" target=\"_blank\">"
                                + class_urls[i].replace("http://purl.obolibrary.org/obo/","").replace("_", ":")
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