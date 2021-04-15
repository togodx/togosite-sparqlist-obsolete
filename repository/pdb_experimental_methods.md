# PDBエントリを実験手法で分類（井手）

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters

* `categoryId` -(type:実験手法)
  * example: X-RAY_DIFFRACTION, SOLUTION_NMR, ELECTRON_MICROSCOPY, NEUTRON_DIFFRACTION, ELECTRON_CRYSTALLOGRAPHY, SOLID-STATE_NMR, SOLUTION_SCATTERING, FIBER_DIFFRACTION, POWDER_DIFFRACTION, EPR, THEORETICAL_MODEL, INFRARED_SPECTROSCOPY, FLUORESCENCE_TRANSFER
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C
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

## `methods_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
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

SELECT distinct ?method_id ?methods count(?methods) AS ?count
   WHERE {
          ?PDBentry       rdf:type	               pdbo:datablock .
          ?PDBentry       dc:title  	           ?title .
          ?PDBentry       pdbo:has_exptlCategory   ?exptlCategory .
          ?exptlCategory  rdf:type                 pdbo:exptlCategory .
          ?exptlCategory  pdbo:has_exptl	       ?exptl .
          ?exptl          pdbo:exptl.method	       ?methods .
          BIND(REPLACE(?methods, " ", "_") AS ?method_id)
}
order by DESC(?count)
```

## `results`

```javascript
({methods_list, count_methods})=>{
  var r = [];
   count_methods.results.bindings.forEach(d=>{ 
     var mid = d.method_id.value;
     if(methods_list){
       if (methods_list.includes(mid)){
         r.push({categoryId: mid, label: d.methods.value, count: Number(d.count.value)});
       }
     }else{
       r.push({categoryId: mid, label: d.methods.value, count: Number(d.count.value)});
     }
   });
  return r;
}
```
