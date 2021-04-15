# PDBエントリをbeta_sheetで分類(ヒトのみ)（井手）(mode対応版)

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryId`   -sheet(type:β-sheetの数)
  * example: 1~960
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

{{#if mode}}
 SELECT DISTINCT ?sheet_number ?PDBentry ?title {
    {{#if sheet_number_list}}
    VALUES ?number { {{#each sheet_number_list}} {{this}} {{/each}} }
    FILTER(?sheet_number IN (?number))
    {
    {{/if}}
    SELECT DISTINCT COUNT(?sheet) AS ?sheet_number ?PDBentry ?title
 
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

  {SELECT DISTINCT ?PDBentry {
    ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
   }}

 }
     {{#if sheet_number_list}} } {{/if}}  

}
order by DESC(?sheet_number) 

```

## `results`

```javascript
({mode, sheet_number})=>{
   if (mode == "objectList") return sheet_number.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
          attribute: {    
          label: d.title.value, 
          count: d.sheet_number.value
          }
       };
    });
   if (mode == "idList") return Array.from(new Set(sheet_number.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
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