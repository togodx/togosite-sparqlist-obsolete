# PDBエントリをalpha_helixで分類(ヒトのみ)（井手）

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryId`  -(type:α-helixの数)
  * example: 1~1397
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C,1PFL
* `list_flag`
  * example: 1
  
## `filter_list`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `nhelix_number_list`
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

{{#if list_flag}}
SELECT distinct (COUNT(?helix) AS ?helix_number) ?PDBentry ?title
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
     {{#unless list_flag}} 
      {SELECT DISTINCT ?PDBentry {
      ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
      }}
     {{/unless}}
  }
  {{#if helix_number_list}} } {{/if}}      
{{#unless list_flag}} 
}
order by DESC(?helix_number) 
{{/unless}}
```

## `results`

```javascript
({list_flag, helix_number})=>{
   if (list_flag) return helix_number.results.bindings.map(d=>{ 
     return {
       categoryId: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       label: d.title.value, 
       count: d.helix_number.value
       };
    });
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