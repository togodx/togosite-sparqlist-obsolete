# PDBエントリを実験手法で分類(mode対応版)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds` -(type:実験手法)
  * example: X-RAY_DIFFRACTION, SOLUTION_NMR, ELECTRON_MICROSCOPY, NEUTRON_DIFFRACTION, ELECTRON_CRYSTALLOGRAPHY, SOLID-STATE_NMR, SOLUTION_SCATTERING, FIBER_DIFFRACTION, POWDER_DIFFRACTION, EPR, THEORETICAL_MODEL, INFRARED_SPECTROSCOPY, FLUORESCENCE_TRANSFER
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C
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

## `methods_list`
- categoryIds を配列に
```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `count_methods`
```sparql

PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

{{#if mode}}
SELECT DISTINCT ?methods_id ?methods ?PDBentry                     
{{else}}
SELECT DISTINCT ?methods_id ?methods count(?methods) AS ?count                         
{{/if}}
   WHERE {
          {{#if filter_list}}
          VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} }
          {{/if}}
          ?PDBentry       rdf:type	               pdbo:datablock .
          ?PDBentry       dc:title  	           ?title .
          ?PDBentry       pdbo:has_exptlCategory   ?exptlCategory .
          ?exptlCategory  rdf:type                 pdbo:exptlCategory .
          ?exptlCategory  pdbo:has_exptl	       ?exptl .
          ?exptl          pdbo:exptl.method	       ?methods .
          BIND(REPLACE(?methods, " ", "_") AS ?methods_id)
             {{#if methods_list}}
             VALUES ?methods_query { {{#each methods_list}} "{{this}}" {{/each}} }
             FILTER(?methods_id IN (?methods_query))
             {{/if}} 
         }

order by DESC(?count)
```



## `results`

```javascript
({mode, methods_list, count_methods})=>{
            if (mode == "objectList") return count_methods.results.bindings.map(d=>{ 
                return {
                id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
                attribute: {
                            categoryIds: d.methods_id.value, 
                            label: capitalize(d.methods.value)
                            }
                };
              });
  if (mode == "idList") return Array.from(new Set(count_methods.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique 
   return count_methods.results.bindings.map(d=>{
     return {
       categoryIds: d.methods_id.value, 
       label: capitalize(d.methods.value),
       count: Number(d.count.value)
       };
   });	  
  function capitalize(label) {
    const s = label.split(' ').map((word) => {
      if (word === 'NMR' || word === 'EPR') {
        return word;
      } else {
        return word.toLowerCase();
      }
    }).join(' ');
    return s.charAt(0).toUpperCase() + s.substring(1);
  }
}
```