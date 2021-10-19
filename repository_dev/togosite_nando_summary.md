# Disease detail for metastanza template (ikeda)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 2200051
  * example: 1200278

## `main`

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?nando ?id ?label ?label_jp ?description ?source
                (GROUP_CONCAT(DISTINCT ?altLabel, "__") AS ?altLabels) ?parent_label ?parent_id
                (GROUP_CONCAT(DISTINCT ?mondo_id, "__") AS ?mondo_ids)
                (GROUP_CONCAT(DISTINCT ?mondo_label, "__") AS ?mondo_labels)
FROM <http://rdf.integbio.jp/dataset/togosite/nando>
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  VALUES ?nando { <http://nanbyodata.jp/ontology/NANDO_{{id}}> }
  ?nando dcterms:identifier ?id ;
         rdfs:label ?label .
  FILTER(lang(?label) = "en")
  ?nando rdfs:label ?label_jp .
  FILTER(lang(?label_jp) = "ja")
  OPTIONAL { ?nando dcterms:description ?description . }
  OPTIONAL { ?nando dcterms:source ?source . }
  OPTIONAL { ?nando skos:altLabel ?altLabel . }
  OPTIONAL {
    ?nando skos:closeMatch ?mondo . 
    ?mondo oboinowl:id ?mondo_id ;
           rdfs:label ?mondo_label .
  }
  OPTIONAL {
    ?nando rdfs:subClassOf ?parent .
    ?parent rdfs:label ?parent_label ;
            dcterms:identifier ?parent_id .
    FILTER(lang(?parent_label) = "en")
  }
}
```

## `return`

```javascript
({ main, columns }) => {
 const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "URL": data.nando.value,
    "ID": data.id.value,
    "label": data.label.value,
    "label_ja": data.label_jp.value,
    "description": data.description?.value ?? "",
    "source": data.source?.value ?? "",
    "altLabel": "",
    "MONDO_related": "",
    "subclass_of": ""
  };
  const prefix = "http://nanbyodata.jp/ontology/";
  if (data.parent_id?.value)
    objs[0]["subclass_of"] = makeLink(prefix+data.parent_id.value.replace(":", "_"),
                                      data.parent_id.value) + " " + data.parent_label.value;
  if (data.altLabels?.value) objs[0]["altLabel"] = makeList(data.altLabels.value.split("__"));
  if (data.mondo_ids?.value) {
    const ids = data.mondo_ids.value.split("__");
    const labels = data.mondo_labels.value.split("__");
    objs[0]["MONDO_related"] = makePairList(ids, labels, ids.map((id)=>"http://purl.obolibrary.org/obo/"+id.replace(":", "_")));
  }
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
