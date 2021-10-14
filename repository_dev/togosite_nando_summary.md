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

SELECT DISTINCT ?nando ?nando_id ?nando_label ?nando_label_jp ?nando_description ?nando_source 
                (GROUP_CONCAT(DISTINCT ?nando_altLabel, ",")AS ?nando_altLabels) ?nando_parent_label ?nando_parent_id
                (GROUP_CONCAT(DISTINCT ?nando_mondo, ",") AS ?nando_mondos)
WHERE { 
  VALUES ?nando { <http://nanbyodata.jp/ontology/NANDO_{{id}}> }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> {
    ?nando dcterms:identifier ?nando_id ;
           rdfs:label ?nando_label .
    FILTER(lang(?nando_label) = "en")
    ?nando rdfs:label ?nando_label_jp .
    FILTER(lang(?nando_label_jp) = "ja")
    OPTIONAL{?nando dcterms:description ?nando_description .}
    OPTIONAL{?nando skos:closeMatch ?nando_mondo .}
    OPTIONAL{?nando dcterms:source ?nando_source .}
    OPTIONAL{?nando skos:altLabel ?nando_altLabel .}
    OPTIONAL{
      ?nando rdfs:subClassOf ?nando_parent .
      ?nando_parent rdfs:label ?nando_parent_label ;
                    dcterms:identifier ?nando_parent_id .
      FILTER(lang(?nando_parent_label) = "en") 
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
    "URL": data.nando?.value,
    "ID": data.nando_id?.value,
    "label": data.nando_label?.value,
    "label_ja": data.nando_label_jp?.value,
    "description": data.nando_description?.value,
    "source": data.nando_source?.value,
    "altLabel": data.nando_altLabel?.value,
    "MONDO_related": data.nando_mondo?.value,
    "subclass_of": data.nando_parent_id?.value
  };
  const class_ids = main.results.bindings[0].nando_parent_id.value.split(/,/);
  const class_labels = main.results.bindings[0].nando_parent_label.value.split(/,/);
  if (objs[0]["subclass_of"]) {
    objs[0]["subclass_of"] = "<ul>";
    for (let i=0; i<class_ids.length; i++) {
      objs[0]["subclass_of"] += "<li><a href=\"http://nanbyodata.jp/ontology/" + class_ids[i].replace("_", ":") + "\" target=\"_blank\">"
                                + class_ids[i].replace("_", ":")
                                + "</a>" + " " +  class_labels[i] + "</li>";
    }
    objs[0]["subclass_of"] += "</ul>";
  }
  return objs;
  function makeList(str, sep) {
    return "<ul><li>" + str.replace(new RegExp(sep, 'g'), "</li><li>") + "</li></ul>";
  };
};
```