# 化合物の詳細情報を出す（信定・山本・建石）

## Parameters

* `id`
  * default:　2244
  * example: 2244, 34756

## Endpoint

https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>

SELECT DISTINCT ?pubchem ?id ?molecular_formula ?label ?molecular_weight ?smiles ?inchi ?formula_img
WHERE {
  VALUES ?pubchem  { compound:CID{{id}} }
    OPTIONAL { ?pubchem sio:has-attribute
      [ a sio:CHEMINF_000382; sio:has-value ?label  ] }
    OPTIONAL { ?pubchem sio:has-attribute
      [ a sio:CHEMINF_000334; sio:has-value ?molecular_weight] }
    OPTIONAL { ?pubchem sio:has-attribute
      [ a sio:CHEMINF_000335; sio:has-value ?molecular_formula ] }
    OPTIONAL { ?pubchem sio:has-attribute
      [ a sio:CHEMINF_000376; sio:has-value ?smiles ] }
    OPTIONAL { ?pubchem sio:has-attribute
      [ a sio:CHEMINF_000396; sio:has-value ?inchi ] }
    BIND (strafter(str(?pubchem), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/") AS ?id)
    BIND(CONCAT("https://pubchem.ncbi.nlm.nih.gov/image/imagefly.cgi?cid=",(SUBSTR(STR(?id),4)),"&width=500&height=500") AS ?formula_img)
}
            
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    URL: data.pubchem.value,
    ID: data.id.value,
    molecular_formula: data.molecular_formula?.value ?? "",
    label: data.label?.value ?? "",
    molecular_weight: data.molecular_weight?.value ?? "",
    SMILES: data.smiles?.value ?? "",
    InChI: data.inchi?.value ?? "",
    formula_img: data.formula_img.value
  };
  return objs;
};
```