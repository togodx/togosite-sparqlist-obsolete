# pubchemまたはchemblのIDを入力として化合物の詳細情報を出力（信定）

## Parameters
* `id`
  * example: 15987396, CHEMBL6939
* `type`
  * example: pubchem_compound, chembl_compound
  
## `idDict` returns an id type object

```javascript
({id, type}) => {
  var obj = {};
  switch (type) {
    case 'pubchem_compound':
      obj.pubchem = id;
      break;
    case 'chembl_compound':
      obj.chembl = id;
      break;
  }
  return obj;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `main`

```sparql
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX pubchem_compound: <https://identifiers.org/pubchem.compound/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sio: <http://semanticscience.org/resource/>

SELECT ?pubchem_id ?chembl_id
FROM <http://rdf.integbio.jp/dataset/togosite/togoid>

  {{#if idDict.pubchem}}
WHERE {
  VALUES ?pubchem_id_temp  { pubchem_compound:{{idDict.pubchem}} }
  ?pubchem_id_temp skos:closeMatch ?chembl_id_temp .
  FILTER(CONTAINS(STR(?chembl_id_temp), "chembl" ))
  BIND (strafter(str(?chembl_id_temp), "http://identifiers.org/chembl.compound/") AS ?chembl_id)
  BIND (strafter(str(?pubchem_id_temp), "https://identifiers.org/pubchem.compound/") AS ?pubchem_id)
}
  {{/if}}
   
  {{#if idDict.chembl}}
WHERE {
  VALUES ?chembl_id_temp  { chembl_compound:{{idDict.chembl}} }
  ?chembl_id_temp skos:closeMatch ?pubchem_id_temp .
  FILTER(CONTAINS(STR(?pubchem_id_temp), "pubchem" ))
  BIND (strafter(str(?pubchem_id_temp), "https://identifiers.org/pubchem.compound/") AS ?pubchem_id)
  BIND (strafter(str(?chembl_id_temp), "http://identifiers.org/chembl.compound/") AS ?chembl_id)
}
  {{/if}}
 ```

## `return`
- 整形

```javascript
 ({main})=>{
  return main.results.bindings.map(d=>{ 
    return {
      pubchem_id: d.pubchem_id.value, 
      chembl_id: d.chembl_id.value
    };
  }) 
 }
```