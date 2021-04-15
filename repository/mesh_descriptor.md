# MeSH詳細 (Topical Descriptor)(建石)

* 入力： MeSH UID

# Parameters

* `queryId` (type: MeSH Descriptor UID)
  * example: D003139, D007251, D045169, D018352, D000086382

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryId}) => {
  queryId = queryId.replace(/,/g," ")
  if (queryId.match(/[^\s]/)) return queryId.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql


## `sparql`
- mesh UID → Tree Number(s)、Label、Scope Note

```sparql
## endpoint https://integbio.jp/togosite/sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?id ?tree_numbers ?label ?scope_note 
{ SELECT ?id  ?label  ?scope_note
  (GROUP_concat(?tree_temp; separator = ", ") as ?tree_numbers)
  WHERE 
  {
    VALUES ?mesh  { {{#each queryArray}} mesh:{{this}} {{/each}} }  
    ?mesh a meshv:TopicalDescriptor ;
          rdfs:label ?label;
          meshv:treeNumber ?tree_uri;
          meshv:preferredConcept ?concept.
    ?concept meshv:scopeNote ?note_temp.

    BIND(IF(bound(?note_temp), ?note_temp,"null") AS ?scope_note) 

    BIND (substr(str(?mesh), 28) AS ?id)
    BIND (substr(str(?tree_uri),28) AS ?tree_temp)

  } 
}
GROUP BY ?id
```


## `return`
- 整形
```javascript
({sparql})=>{
  return sparql.results.bindings.map(d=>{ 
    return {
      id: d.id.value, 
      tree_numbers: d.tree_numbers.value, 
      label: d.label.value, 
      scope_note: d.scope_note.value
    };
  }) 
 }
```
