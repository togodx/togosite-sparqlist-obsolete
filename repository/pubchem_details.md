# pubchem compoundの詳細情報を出す（信定・山本）

## Parameters

* `pubchem_compound` (type: pubchem_compound)
  * example: 2244, 517284, 11403845, 15987396, 34756

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({pubchem_compound}) => {
  pubchem_compound = pubchem_compound.replace(/,/g," ")
  if (pubchem_compound.match(/[^\s]/)) return pubchem_compound.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `sparql`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>
PREFIX dcterms: <http://purl.org/dc/terms/>
SELECT
   distinct ?pubchem_id (GROUP_concat(?chembl3; separator = ", ") as ?chembl_id)   (GROUP_concat(?chebi3; separator = ", ") as ?chebi_id) ?label ?molecular_formula ?molecular_weight ?smiles ?inchi (CONCAT("https://pubchem.ncbi.nlm.nih.gov/compound/",(SUBSTR(STR(?id),49)),"#section=2D-Structure&fullscreen=true") AS ?formula_figure)
{
SELECT
  DISTINCT ?pubchem_id ?chembl3 ?chebi3 ?label ?molecular_formula ?molecular_weight ?smiles ?inchi (CONCAT("https://pubchem.ncbi.nlm.nih.gov/compound/",(SUBSTR(STR(?id),49)),"#section=2D-Structure&fullscreen=true") AS ?formula_figure)
FROM <http://rdf.integbio.jp/dataset/togosite/pubchem>
WHERE {
  VALUES ?cid  { {{#each queryArray}} compound:CID{{this}} {{/each}} }
 ?cid sio:has-attribute
      [ a sio:CHEMINF_000382; sio:has-value ?label  ] ,
      [ a sio:CHEMINF_000334; sio:has-value ?molecular_weight] ,
      [ a sio:CHEMINF_000335; sio:has-value ?molecular_formula ] ,
      [ a sio:CHEMINF_000376; sio:has-value ?smiles ] ,
      [ a sio:CHEMINF_000396; sio:has-value ?inchi ] .
  OPTIONAL {[ sio:is-attribute-of ?cid ; a sio:CHEMINF_000412 ; sio:has-value ?chembl ] . 
            BIND (UCASE(?chembl) AS ?chembl2)}
  OPTIONAL { [] sio:is-attribute-of ?cid ;
     a sio:CHEMINF_000407 ; # ChEBI identifier
     sio:has-value ?chebi .
    BIND (UCASE(?chebi) AS ?chebi2)}
    BIND (strafter(str(?cid), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID") AS ?pubchem_id)
    BIND(IF(bound(?chembl2), ?chembl2,"-") AS ?chembl3)
    BIND(IF(bound(?chebi2), ?chebi2,"-") AS ?chebi3)
  # https://pubchem.ncbi.nlm.nih.gov/compound/5861095#section=2D-Structure&fullscreen=true
} }
```
## `return`
- 整形

```javascript
 ({sparql})=>{
  return sparql.results.bindings.map(d=>{ 
    return {
      pubchem_compound_id: d.pubchem_id.value,
      chembl_id: d.chembl_id.value,
      chebi_id: d.chebi_id.value,
      compound_label: d.label.value, 
      molecular_formula: d.molecular_formula.value,
      molecular_weight: d.molecular_weight.value,
     　smiles: d.smiles.value,
      inchi: d.inchi.value,
      image: d.formula_figure.value
    };
  }) 
 }
```