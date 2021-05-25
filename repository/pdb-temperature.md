# PDBエントリを結晶化時の温度で分類（井手）

## Description
 
- Data sources
    - The temperature of crystal formation in the PDB entry
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - temperature (K), PDB id
    - Output
        - The number of PDB entries included in the temperature range
        - If a PDB id is entered, it returns the temperature of crystal formation in each PDB entry.


## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds` -温度範囲(単位K)
  * example: 273.15-300.0
* `queryIds` (type: PDB)
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

## `temperature_range`
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/\s/g, "");
  if (categoryIds.match(/\d-/) || categoryIds.match(/-\d/)) {
    let range = {begin: 99, end: 350};                                       //false
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    return range;         
  }
  return false;
}
```

## `temperature`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

{{#if mode}}
  SELECT ?temperature ?temperature_str ?PDBentry ?title
{{else}}
  SELECT ?temperature COUNT(?temperature) AS ?count #?temperature_str
{{/if}}
         WHERE {
         {{#if filter_list}}
           VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} }
         {{/if}}
         {{#if temperature_range}} VALUES (?temperature_MIN ?temperature_MAX)  {( {{#each temperature_range}} {{this}} {{/each}} )} {{/if}}
          ?PDBentry            rdf:type	                            pdbo:datablock .
          ?PDBentry            dc:title  	                        ?title .
          ?PDBentry            pdbo:has_exptl_crystal_growCategory	?crystal_growCategory .
          ?crystal_growCategory pdbo:has_exptl_crystal_grow	        ?crystal_grow .
          ?crystal_grow         pdbo:exptl_crystal_grow.temp	    ?temperature_str .
          {{#if temperature_range}} FILTER(xsd:float(?temperature_str) <= xsd:float(?temperature_MAX))
                                    FILTER(xsd:float(?temperature_str) >= xsd:float(?temperature_MIN)) {{/if}}          
          BIND(Round(10*(xsd:decimal(?temperature_str))/10) AS ?temperature)
      }
group by ?temperature
order by ?temperature  
```

## `results`

```javascript
({mode, temperature})=>{
   if (mode == "objectList") return temperature.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
         attribute: {
         categoryIds: d.temperature_str.value, 
         label: d.temperature_str.value +" K"
         }
       };
    });
   if (mode == "idList") return Array.from(new Set(temperature.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
   return temperature.results.bindings.map(d=>{
     return {
       categoryIds: d.temperature.value, 
       label: d.temperature.value +" K", 
       count: Number(d.count.value)
       };
   });	
}
```
