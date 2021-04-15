# PDBエントリをalpha_helixで分類(ヒトのみ)(mode対応版)

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryId`  -(type:α-helixの数)
  * example: 1~1397
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C,1PFL
* `mode` 
  * example: idList, objectList
  
## `filter_list`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `helix_number_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```


## `helix_number`

```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

{{#if mode}}
    SELECT distinct ?helix_number ?PDBentry ?title {
    {{#if helix_number_list}}
    VALUES ?number { {{#each helix_number_list}} {{this}} {{/each}} }
    FILTER(?helix_number IN (?number))
    {
    {{/if}}  
    SELECT DISTINCT (COUNT(?helix) AS ?helix_number) ?PDBentry ?title 
                                                              
{{else}}
 SELECT ?helix_number COUNT(?helix_number) AS ?count 
 WHERE{
    {{#if helix_number_list}}
    VALUES ?number { {{#each helix_number_list}} {{this}} {{/each}} }
    FILTER(?helix_number IN (?number))
    {
    {{/if}}  
    SELECT DISTINCT (COUNT(?helix) AS ?helix_number) ?PDBentry ?title 
{{/if}}                                                 
 WHERE {
         {{#if filter_list}}
         VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} }
         {{/if}}
  ?PDBentry  a pdbo:datablock ;
             dc:title ?title ;
             pdbo:has_struct_confCategory   ?helix .
  ?helix     pdbo:has_struct_conf           ?helix_each . 
      {SELECT DISTINCT ?PDBentry {
      ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
      }}
  }
  {{#if helix_number_list}} } {{/if}}      
}
order by DESC(?helix_number) 

```

## `results`

```javascript
({mode, helix_number})=>{
   if (mode == "objectList") return helix_number.results.bindings.map(d=>{ 
     return {
        id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
          attribute: {
          label: d.title.value, 
          count: d.helix_number.value
          }
       };
    });
   if (mode == "idList") return Array.from(new Set(helix_number.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
   var id_val = 0;
   return helix_number.results.bindings.map(d=>{ 
     return {
       categoryId: id_val++, 
       label: d.helix_number.value + " helix", 
       count: d.count.value
       };
   });	
}
```