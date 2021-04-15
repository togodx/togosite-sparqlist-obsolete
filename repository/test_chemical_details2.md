# (test) pubchem, chemblのIDを入力して化合物の詳細情報を出す（信定・山本）

## Parameters

* `id`
  * default:　15987396
  * example: 15987396, CHEMBL6939
* `type`
  * default: pubchem_compound
  * example: pubchem_compound, chembl_compound

## Endpoint

https://integbio.jp/togosite/sparql

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
 
## `togoid` resolve id relations 
 
 ```sparql
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX pubchem_compound: <https://identifiers.org/pubchem.compound/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT ?pubchem_uri, ?chembl_uri

WHERE {
      GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
{{#if idDict.pubchem}}
VALUES ?pubchem_id_ent { pubchem_compound:{{idDict.pubchem}} }
    ?pubchem_id_ent skos:closeMatch ?chembl_id_ent .
  FILTER(STRSTARTS(STR(?chembl_id_ent), STR(chembl_compound:)  ))
{{/if}}
{{#if idDict.chembl}}
VALUES ?chembl_id_ent { chembl_compound:{{idDict.chembl}} }
     ?chembl_id_ent skos:closeMatch ?pubchem_id_ent .
  FILTER(STRSTARTS(STR(?pubchem_id_ent), STR(pubchem_compound:) ))
{{/if}}
  BIND(IRI(REPLACE(STR(?chembl_id_ent), "http://identifiers.org/chembl.compound/","http://rdf.ebi.ac.uk/resource/chembl/molecule/")) AS ?chembl_uri)  
  BIND(IRI(REPLACE(STR(?pubchem_id_ent), "https://identifiers.org/pubchem.compound/","http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID")) AS ?pubchem_uri)
 }}
 ```
 
 ## `togoidDict` returns an id relation object 

- Create a object from the rows of sparql query result (`Array.reduce`)
  - loop for columns of a row (`for Object.entries`)

```javascript
({togoid}) => {
  return togoid.results.bindings.reduce( );
}
```

 ## ## `main` 
```sparql
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX pubchem_compound: <https://identifiers.org/pubchem.compound/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>
PREFIX dcterms: <http://purl.org/dc/terms/>
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
prefix foaf: <http://xmlns.com/foaf/0.1/>
prefix chembl_molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
prefix cheminf: <http://semanticscience.org/resource/>
prefix skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT *

WHERE {
  {{#if togoidDict.pubchem}}
  VALUES ?pubchem_uri { {{#each togoidDict.pubchem}} <{{this}}> {{/each}} }    
 GRAPH  <http://rdf.integbio.jp/dataset/togosite/pubchem>
{OPTIONAL {
  ?pubchem_uri obo:has-role pubchemv:FDAApprovedDrugs ;
      	sio:has-attribute
      [ a sio:CHEMINF_000382; sio:has-value ?pubchem_label_temp ] .}
  BIND(IF(bound(?pubchem_label_temp), ?pubchem_label_temp,"null") AS ?pubchem_label)
}
 BIND (strafter(str(?pubchem_uri), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/") AS ?pubchem_id)
 {{/if}}
    
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chembl>
 {?chembl_uri a cco:SmallMolecule ;
           skos:altLabel ?label_temp ;
           foaf:depiction ?formula_figure .
  OPTIONAL { ?chembl_uri a cco:SmallMolecule ; cheminf:SIO_000008 ?att_1. ?att_1 a cheminf:CHEMINF_000042 ; cheminf:SIO_000300 ?molecular_formula_temp }.
  OPTIONAL { ?chembl_uri a cco:SmallMolecule ; cheminf:SIO_000008 ?att_2. ?att_2 a cheminf:CHEMINF_000216 ; cheminf:SIO_000300 ?molecular_weight_temp  }.
  OPTIONAL { ?chembl_uri a cco:SmallMolecule ; cheminf:SIO_000008 ?att_3. ?att_3 a cheminf:CHEMINF_000018 ; cheminf:SIO_000300 ?smiles_temp  }.
  OPTIONAL { ?chembl_uri a cco:SmallMolecule ; cheminf:SIO_000008 ?att_4. ?att_4 a cheminf:CHEMINF_000113 ; cheminf:SIO_000300 ?inchi_temp  }.
  BIND(IF(bound(?molecular_formula_temp), ?molecular_formula_temp,"null") AS ?molecular_formula)
  BIND(IF(bound(?molecular_weight_temp), ?molecular_weight_temp, "null") AS ?molecular_weight)
  BIND(IF(bound(?smiles_temp), ?smiles_temp,"null") AS ?smiles)
  BIND(IF(bound(?inchi_temp), ?inchi_temp,"null") AS ?inchi)
  BIND (strafter(str(?chembl_uri), "http://rdf.ebi.ac.uk/resource/chembl/molecule/") AS ?chembl_id)
  BIND (strafter(str(?pubchem_uri), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/") AS ?pubchem_id)
  }
}}

  
```
## `return`
- 整形

```javascript
 ({sparql})=>{
  return sparql.results.bindings.map((elm)=>({ 
      pubchem_id: elm.pubchem_id.value, 
      chembl_id: elm.chembl_id.value, 
      pubchem_label: elm.pubchem_label.value, 
      chembl_label: elm.chembl_label.value, 
      molecular_formula: elm.molecular_formula.value,
      molecular_weight: elm.molecular_weight.value,
     　smiles: elm.smiles.value,
      inchi: elm.inchi.value,
      image: elm.formula_figure.value
    })); 
 }
```
