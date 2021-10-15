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

SELECT DISTINCT ?mesh ?mesh_id ?mesh_label ?mesh_parent_id ?mesh_parent_label ?mesh_preferred_concept ?mesh_preferred_concept_label ?mesh_note
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE { 
  VALUES ?mesh { mesh:{{id}} }
  ?mesh rdfs:label ?mesh_label ;
        meshv:identifier ?mesh_id .
  OPTIONAL {
    ?mesh meshv:treeNumber / meshv:parentTreeNumber / ^meshv:treeNumber ?parent .
    ?parent rdfs:label ?mesh_parent_label ;
            meshv:identifier ?mesh_parent_id .
  }
  ?mesh meshv:preferredConcept ?mesh_preferred_concept .
  ?mesh_preferred_concept rdfs:label ?mesh_preferred_concept_label .
  OPTIONAL {
    ?mesh_preferred_concept meshv:scopeNote ?mesh_note . 
  }
}
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    "ID": data.mesh_id.value,
    "URL": data.mesh.value,
    "label": data.mesh_label.value,
    "scope_note": "",
    "parent": ""
  };
  if (data.mesh_parent_id?.value)
    objs[0].parent = "<a href=\"http://id.nlm.nih.gov/mesh/" + data.mesh_parent_id.value + "\" target=\"_blank\">"
                     + data.mesh_parent_id.value + "</a> " + data.mesh_parent_label.value;
  if (data.mesh_note?.value)
    objs[0].scope_note = data.mesh_note.value;
  objs[0].preferred_concept = "<a href=\"" + data.mesh_preferred_concept.value + "\" target=\"_blank\">"
                              + data.mesh_preferred_concept.value.replace("http://id.nlm.nih.gov/mesh/", "")
                              + "</a> " + data.mesh_preferred_concept_label.value;
  return objs;
};
```