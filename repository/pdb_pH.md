# PDBエントリを結晶化時のpHで分類（井手）

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryIds`
  * example: 6.8-7.1
* `queryIds` -(type: PDB)
  * example: 1G0M,1SFX,2EYI
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

## `pH_range`
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/\s/g, "");
  if (categoryIds.match(/\d-/) || categoryIds.match(/-\d/)) {
    let range = {begin: 0, end: 13};                                       //false
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    return range;         
  }
  return false;
}
```

## `pH`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

{{#if mode}}
  SELECT ?pH ?pH_str ?PDBentry ?title
{{else}}
  SELECT ?pH COUNT(?pH) AS ?count 
{{/if}}  
    WHERE {
      {{#if filter_list}} VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} } {{/if}}
      {{#if pH_range}}    VALUES (?pH_MIN ?pH_MAX) {( {{#each pH_range}}  {{this}}  {{/each}} )} {{/if}}  
     ?PDBentry             rdf:type	                            pdbo:datablock .
     ?PDBentry             dc:title  	                        ?title .
     ?PDBentry             pdbo:has_exptl_crystal_growCategory	?crystal_growCategory .
     ?crystal_growCategory pdbo:has_exptl_crystal_grow	        ?crystal_grow .
     ?crystal_grow         pdbo:exptl_crystal_grow.pH	        ?pH_str .
     {{#if pH_range}} FILTER(xsd:float(?pH_str) <= xsd:float(?pH_MAX))    
                      FILTER(xsd:float(?pH_str) >= xsd:float(?pH_MIN)) {{/if}}          
     BIND(Round(10*(xsd:decimal(?pH_str))/10) AS ?pH)         
     }
order by ?pH  
```

## `results`

```javascript
({mode, pH})=>{
   if (mode == "objectList") return pH.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
         attribute: {
         categoryId: d.pH_str.value, 
         label: "pH " + d.pH_str.value
         }
       };
    });
   if (mode == "idList") return Array.from(new Set(pH.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
    return pH.results.bindings.map(d=>{
     return {
       categoryId: d.pH.value, 
       label: "pH " + d.pH.value, 
       count: Number(d.count.value)
       };
   });	
}
```



