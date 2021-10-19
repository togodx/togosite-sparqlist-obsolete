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
       (GROUP_CONCAT(DISTINCT ?dbxref, "__") AS ?dbxrefs)
       ?comment
       (GROUP_CONCAT(DISTINCT ?parent_id, "__") AS ?parent_ids)
       (GROUP_CONCAT(DISTINCT ?parent_label, "__") AS ?parent_labels)
       (GROUP_CONCAT(DISTINCT ?exact_synonym, "__")AS ?exact_synonyms)
       (GROUP_CONCAT(DISTINCT ?related_synonym, "__") AS ?related_synonyms)
WHERE {
  VALUES ?hpo { <http://purl.obolibrary.org/obo/HP_{{id}}> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpo> {
    ?hpo rdfs:label ?label .
    OPTIONAL { ?hpo obo:IAO_0000115 ?definition . }
    OPTIONAL { ?hpo go:hasDbXref ?dbxref . }
    OPTIONAL { ?hpo rdfs:comment ?comment . }
    OPTIONAL { ?hpo go:hasExactSynonym ?exact_synonym . }
    OPTIONAL { ?hpo go:hasRelatedSynonym ?related_synonym . }
    OPTIONAL { 
      ?hpo rdfs:subClassOf ?parent .
      ?parent rdfs:label ?parent_label .
      BIND(REPLACE(STR(?parent), 'http://purl.obolibrary.org/obo/HP_', 'HP:') AS ?parent_id)
    }
    BIND(REPLACE(STR(?hpo), 'http://purl.obolibrary.org/obo/HP_', 'HP:') AS ?id)
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
    "xrefs": "",
    "comment": data.comment?.value ?? "",
    "subclass_of": "",
    "exact_synonym": "",
    "related_synonym": ""
  };
  const prefix = "http://purl.obolibrary.org/obo/";
  if (data.parent_ids?.value) {
    const ids = data.parent_ids.value.split("__");
    const labels = data.parent_labels.value.split("__");
    objs[0]["subclass_of"] = makePairList(ids, labels, ids.map((id)=>prefix+id.replace(":", "_")));
  }
  if (data.exact_synonyms?.value) objs[0].exact_synonym = makeList(data.exact_synonyms.value.split("__"));
  if (data.related_synonyms?.value) objs[0].related_synonym = makeList(data.related_synonyms.value.split("__"));
  if (data.dbxrefs?.value) objs[0].xrefs = makeList(data.dbxrefs.value.split("__"));
  return objs;

  function makeLink(url, text) {
    return "<a href=\"" + url + "\" target=\"_blank\">" + text + "</a>";
  }
  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
  function makePairList(ids, labels, urls) {
    return makeList(ids.map((id, i)=>makeLink(urls[i], id) + " " + labels[i]));
  }
};
```