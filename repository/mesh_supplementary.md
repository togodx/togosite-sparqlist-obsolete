# MeSH詳細 (Supplemenary Concept Record)(建石)

* Chemicalと分けましたが

# Parameters

* `queryId` (type: MeSH Descriptor UID)
  * example: C536718, C567595, C535297,C571912

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

SELECT ?id ?scr_type ?label ?descriptor ?dlabel

{
SELECT ?id ?scr_type  ?label   
  (GROUP_concat(?descriptor_temp; separator = ", ") as ?descriptor) (Group_concat(?dlabel_temp; separator = ", ") as ?dlabel)
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE 
{ 
 VALUES ?mesh   { {{#each queryArray}} mesh:{{this}} {{/each}} }  
 ?mesh a ?scr_type;
       meshv:preferredMappedTo ?descriptor_temp;
       rdfs:label ?label.
 ?descriptor_temp a meshv:TopicalDescriptor;
             rdfs:label ?dlabel_temp.      
 
 
 BIND (substr(str(?mesh), 28) AS ?id)
 

  } }

GROUP BY ?id
Limit 2
```


## `return`
- 整形
```javascript
({sparql})=>{
  return sparql.results.bindings.map(d=>{ 
    return {
      id: d.id.value, 
      label: d.label.value, 
      descriptor: d.descriptor.value
    };
  }) 
 }
```
