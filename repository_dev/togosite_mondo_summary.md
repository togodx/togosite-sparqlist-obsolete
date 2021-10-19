# Disease detail for metastanza template (ikeda)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 0004997
  * example: 0004997, 0005854

## `main`

```sparql
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?mondo ?id ?label ?definition
                (GROUP_CONCAT(DISTINCT ?xref, "__") AS ?xrefs)
                (GROUP_CONCAT(DISTINCT ?synonym, "__") AS ?synonyms)
                (GROUP_CONCAT(DISTINCT ?parent_id, "__")  AS ?parent_ids)
                (GROUP_CONCAT(DISTINCT ?parent_label , "__") AS ?parent_labels)
WHERE {
  VALUES ?mondo { <http://purl.obolibrary.org/obo/MONDO_{{id}}> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/mondo> {
    ?mondo oboinowl:id ?id ;
           rdfs:label ?label .
    OPTIONAL { ?mondo obo:IAO_0000115 ?definition . }
    OPTIONAL { ?mondo oboinowl:hasDbXref ?xref . }
    OPTIONAL { ?mondo oboinowl:hasExactSynonym ?synonym . }
    OPTIONAL {
      ?mondo rdfs:subClassOf ?parent_class .
      ?parent_class rdfs:label ?parent_label ;
                    oboinowl:id ?parent_id .
    }
  }
}
```

## `return`

```javascript
({ main, columns }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "URL": data.mondo.value,
    "ID": data.id.value,
    "label": data.label.value,
    "definition": data.definition?.value ?? "",
    "xref": "",
    "subclass_of": "",
    "synonym": ""
  };
  const prefix = "http://purl.obolibrary.org/obo/";
  if (data.parent_ids?.value) {
    const ids = data.parent_ids.value.split("__");
    const labels = data.parent_labels.value.split("__");
    objs[0]["subclass_of"] = makePairList(ids, labels, ids.map((id)=>prefix+id.replace(":", "_")));
  }
  if (data.synonyms?.value) objs[0].synonym = makeList(data.synonyms.value.split("__"));
  if (data.xrefs?.value) objs[0].xref = makeList(data.xrefs.value.split("__"));
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
