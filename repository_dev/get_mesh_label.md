# MeSHコードに対するラベルを取得する （山本）

## Description

## Endpoint

https://integbio.jp/rdf/mesh/sparql

## Parameters

* `mesh_id` ( MeSH Term ID )
  * default: C000657245
  * examples: D000086382, D000086402

## `no_mesh_id`
```javascript
({mesh_id})=>{
  return( mesh_id == "" )
}
```
## `mesh_id`
```javascript
({mesh_id, no_mesh_id})=>{
  if( no_mesh_id ){
    return ""
  } else {
    return "<http://id.nlm.nih.gov/mesh/" + mesh_id + ">"
  }
}
```

## `Query`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT (str(?label) as ?l)
FROM <http://id.nlm.nih.gov/mesh>
WHERE {
  {{mesh_id}} rdfs:label ?label .
}
```

## `Return`

```javascript
({Query})=>{
  if (Query.head.vars[0] == "l"){
     return Query.results.bindings.map(d=>d.l.value)[0]
  } else{
    return Query.head.vars[0]
  }
}