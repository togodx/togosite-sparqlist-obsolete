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

SELECT DISTINCT ?mesh ?mesh_id ?mesh_label ?mesh_tree_uri ?mesh_concept ?mesh_note
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE { 
  VALUES ?mesh { mesh:{{id}} }
  ?mesh rdfs:label ?mesh_label ;
        meshv:identifier ?mesh_id .
  OPTIONAL { ?mesh meshv:treeNumber ?mesh_tree_uri . }
  ?mesh meshv:preferredConcept ?mesh_concept .
  OPTIONAL { ?mesh_concept meshv:scopeNote ?mesh_note . }
}
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "ID": "mesh_id" },
    { "label": "mesh_label" },
    { "tree_URI": "mesh_tree_uri" },
    { "concept": "mesh_concept" },
    { "scope_note": "mesh_note" }
  ];
  return array;
}
```

## `return`

```javascript
({ main, columns }) => {
  return main.results.bindings.map((binding) => {
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
};
```