# PDBエントリの結晶化手法で分類（井手）(mode対応版)

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `queryIds` (type: PDB)
  * example: 1G0M,1SFX,2EYI
* `crystallization_method`
  * example: VAPOR DIFFUSION
* `mode` 
  * example: idList, objectList
  
## `filter_list`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/\s/g,"")
  if (queryIds) return queryIds.split(",");
  return false;
}
```

## `crystallization_method_list`
- Filter 用 PDB を配列に
```javascript
({crystallization_method}) => {
  crystallization_method = crystallization_method.replace(/\s/g,"")
  if (crystallization_method) return crystallization_method.split(",");
  return false;
}
```

## `crystal_methods`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

{{#if mode}}
SELECT DISTINCT ?crystal_methods COUNT(?crystal_methods) AS ?count ?PDBentry ?title
{{else}}
SELECT DISTINCT ?crystal_methods COUNT(?crystal_methods) AS ?count
{{/if}}  
WHERE {
      {{#if filter_list}} VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} } {{/if}}
      {{#if crystallization_method_list}} VALUES ?crystal_query { {{#each crystallization_method_list}} "{{this}}" {{/each}} } {{/if}}
          ?PDBentry            rdf:type	                            pdbo:datablock .
          ?PDBentry            dc:title  	                        ?title .
          ?PDBentry            pdbo:has_exptl_crystal_growCategory	?crystal_growCategory .
          ?crystal_growCategory pdbo:has_exptl_crystal_grow	        ?crystal_grow .
          ?crystal_grow         pdbo:exptl_crystal_grow.method	    ?crystal_methods .
          BIND(replace(?crystal_methods, " ","") AS ?crystal_methods_str)
          {{#if crystallization_method_list}}
          FILTER CONTAINS(?crystal_methods_str, ?crystal_query)
          {{/if}}
       }                                   
order by DESC(?count)
```

## `results`

```javascript
({mode, crystal_methods})=>{
   if (mode == "objectList") return crystal_methods.results.bindings.map(d=>{ 
     return {
       categoryId: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       label: d.title.value, 
       count: d.crystal_methods.value
       };
    });
   if (mode == "idList") return Array.from(new Set(crystal_methods.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
   var id_val = 0;
   return crystal_methods.results.bindings.map(d=>{ 
     return {
       categoryId: id_val++, 
       label: d.crystal_methods.value, 
       count: d.count.value
       };
   });	
}
```