# PDBエントリをbeta_sheetで分類(ヒトのみ)（井手）

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryId`   -sheet(type:β-sheetの数)
  * example: 1~960
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

## `sheet_number_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## `sheet_number`

```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

{{#if list_flag}}
SELECT distinct (COUNT(?sheet) AS ?sheet_number) ?PDBentry ?title
{{else}}
 SELECT ?sheet_number COUNT(?sheet_number) AS ?count {
       {{#if sheet_number_list}}
    VALUES ?number { {{#each sheet_number_list}} {{this}} {{/each}} }
    FILTER(?sheet_number IN (?number))
    {
    {{/if}}  
 SELECT DISTINCT (COUNT(?sheet) AS ?sheet_number) ?PDBentry ?title 
{{/if}}  
 WHERE{
      {{#if filter_list}}
       VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} }
      {{/if}}
  ?PDBentry  a pdbo:datablock ;
             dc:title ?title ;
             pdbo:has_struct_sheetCategory ?sheet .
  ?sheet     pdbo:has_struct_sheet         ?sheet_each .
  {{#unless list_flag}} 
  {SELECT DISTINCT ?PDBentry {
    ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
   }}
   {{/unless}}
 }
     {{#if sheet_number_list}} } {{/if}}  
{{#unless list_flag}} 
}
order by DESC(?sheet_number) 
{{/unless}}
```

## `results`

```javascript
({list_flag, sheet_number})=>{
   if (list_flag) return sheet_number.results.bindings.map(d=>{ 
     return {
       categoryId: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       label: d.title.value, 
       count: d.sheet_number.value
       };
    });
   var id_val = 0;
   return sheet_number.results.bindings.map(d=>{
     return {
       icategoryI: id_val++, 
       label: d.sheet_number.value + " sheet", 
       count: d.count.value
       };
   });	
}
```