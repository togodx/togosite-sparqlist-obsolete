# Disease detail for metastanza template (ikeda)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: D002804
  * example: D008947

## `main`

```sparql
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?mesh ?id ?label ?preferred_concept_id ?preferred_concept_label
                (GROUP_CONCAT(DISTINCT ?concept_id, "__") AS ?concept_ids)
                (GROUP_CONCAT(DISTINCT ?concept_label, "__") AS ?concept_labels)
                (GROUP_CONCAT(DISTINCT ?broader_id, "__") AS ?broader_ids)
                (GROUP_CONCAT(DISTINCT ?broader_label, "__") AS ?broader_labels)
                ?note
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  VALUES ?mesh { mesh:{{id}} }
  ?mesh meshv:identifier ?id ;
        rdfs:label ?label .
  ?mesh meshv:preferredConcept ?preferred_concept .
  ?preferred_concept meshv:identifier ?preferred_concept_id ;
                          rdfs:label ?preferred_concept_label .
  OPTIONAL {
    ?preferred_concept meshv:scopeNote ?note .
  }
  OPTIONAL {
    ?mesh meshv:broaderDescriptor ?broader .
    ?broader meshv:identifier ?broader_id ;
                 rdfs:label ?broader_label .
  }
  OPTIONAL {
    ?mesh meshv:scopeNote ?note .
  }
  OPTIONAL {
    ?mesh meshv:concept ?concept .
    ?concept meshv:identifier ?concept_id ;
                  rdfs:label ?concept_label .
  }
}
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "ID": data.id.value,
    "URL": data.mesh.value,
    "label": data.label.value,
    "concept": "",
    "scope_note": data.note?.value ?? "",
    "broader_descriptor": ""
  };
  const prefix = "http://id.nlm.nih.gov/mesh/";
  if (data.broader_ids?.value) {
    const ids = data.broader_ids.value.split("__");
    const labels = data.broader_labels.value.split("__");
    objs[0].broader_descriptor = makePairList(ids, labels, ids.map((id)=>prefix+id));
  }
  if (data.concept_ids?.value) {
    const ids = data.concept_ids.value.split("__");
    const labels = data.concept_labels.value.split("__");
    objs[0].concept = makePairList(ids, labels, ids.map((id)=>prefix+id));
  }
  objs[0].preferred_concept = makeLink(prefix + data.preferred_concept_id.value, data.preferred_concept_id.value)
                              + " " + data.preferred_concept_label.value;
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
