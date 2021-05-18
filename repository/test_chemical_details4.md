# (test4) pubchem, chembl, ChEBIのIDを入力して化合物の詳細情報を出す（信定・山本・建石）

## Parameters

* `id`
  * default:　2244
  * example: 2244, 517068, 15987396, CHEMBL6939, CHEMBL1201330, 15414 (CHEBI), 17232 (CHEBI)
* `type`
  * default: pubchem_compound
  * example: pubchem_compound, chembl_compound, chebi

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
    case 'chebi':
      obj.chebi = id;
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
PREFIX CHEBI: <https://identifiers.org/chebi/CHEBI:>

SELECT ?pubchem_uri, ?chembl_uri,?chebi_uri
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
  {{#if idDict.pubchem}}
  VALUES ?pubchem_id_ent { pubchem_compound:{{idDict.pubchem}} }
    OPTIONAL {  ?pubchem_id_ent skos:closeMatch ?chembl_id_ent .
                FILTER(STRSTARTS(STR(?chembl_id_ent), STR(chembl_compound:)  )) }
    OPTIONAL {  ?pubchem_id_ent skos:closeMatch ?chebi_id_ent .
                FILTER(STRSTARTS(STR(?chebi_id_ent), STR(CHEBI:)  )) }
  {{/if}}
        
  {{#if idDict.chembl}}
  VALUES ?chembl_id_ent { chembl_compound:{{idDict.chembl}} }
    OPTIONAL {   ?chembl_id_ent skos:closeMatch ?pubchem_id_ent .
                 FILTER(STRSTARTS(STR(?pubchem_id_ent), STR(pubchem_compound:) )) }
    OPTIONAL {  ?chembl_id_ent skos:closeMatch ?chebi_id_ent .
                 FILTER(STRSTARTS(STR(?chebi_id_ent), STR(CHEBI:)  )) }
  {{/if}}

  {{#if idDict.chebi}}
  VALUES ?chebi_id_ent { CHEBI:{{idDict.chebi}} }
    OPTIONAL {   ?pubchem_id_ent skos:closeMatch ?chebi_id_ent .
                 FILTER(STRSTARTS(STR(?pubchem_id_ent), STR(pubchem_compound:) )) }
    OPTIONAL {  ?chembl_id_ent skos:closeMatch ?chebi_id_ent .
                FILTER(STRSTARTS(STR(?chembl_id_ent), STR(chembl_compound:)  )) }
  {{/if}}
  BIND(IRI(REPLACE(STR(?chembl_id_ent), STR(chembl_compound:),"http://rdf.ebi.ac.uk/resource/chembl/molecule/")) AS ?chembl_uri)  
  BIND(IRI(REPLACE(STR(?pubchem_id_ent), STR(pubchem_compound:),"http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID")) AS ?pubchem_uri)
  BIND(IRI(REPLACE(STR(?chebi_id_ent), STR(CHEBI:),"http://purl.obolibrary.org/obo/CHEBI_")) AS ?chebi_uri) 
 }}
 ```
 
 ## `togoidDict` returns an id relation object 

```javascript
({togoid}) => {
  return togoid.results.bindings.reduce(
    (obj, elem) => {
      for (const [key, node] of Object.entries(elem)) {
        if (!obj[key]) obj[key] = [];
        if (!obj[key].includes(node.value)) obj[key].push(node.value);
      };
      return obj;
    }, {}
  );
}
```

## `main`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX chembl_molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX cheminf: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
prefix owl: <http://www.w3.org/2002/07/owl#> 
prefix oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
prefix iao: <http://purl.obolibrary.org/obo/IAO_>
prefix ro: <http://purl.obolibrary.org/obo/RO_>
prefix chebi: <http://purl.obolibrary.org/obo/chebi/>
prefix chebi_id: <http://purl.obolibrary.org/obo/CHEBI_>

  SELECT ?pubchem_molecular_formula, ?chembl_molecular_formula, ?chebi_molecular_formula,
         ?pubchem_id, ?chembl_id, 
                      ?chembl_type, 
         ?pubchem_label, (GROUP_concat(?label_temp; separator = ", ") as ?chembl_label), 
         ?pubchem_molecular_weight, ?chembl_molecular_weight, 
         ?pubchem_smiles, ?chembl_smiles, 
         ?pubchem_inchi, ?chembl_inchi, 
         ?pubchem_formula_img , ?chembl_formula_img

FROM <http://rdf.integbio.jp/dataset/togosite/pubchem>
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
 {
   {{#if togoidDict.pubchem_uri }}
{  VALUES ?pubchem_uri  { {{#each togoidDict.pubchem_uri}} <{{this}}> {{/each}} }
 OPTIONAL{ ?pubchem_uri sio:has-attribute
      [ a sio:CHEMINF_000382; sio:has-value ?pubchem_label_temp  ] ,
      [ a sio:CHEMINF_000334; sio:has-value ?pubchem_molecular_weight_temp] ,
      [ a sio:CHEMINF_000335; sio:has-value ?pubchem_molecular_formula_temp ] ,
      [ a sio:CHEMINF_000376; sio:has-value ?pubchem_smiles_temp ] ,
      [ a sio:CHEMINF_000396; sio:has-value ?pubchem_inchi_temp ] .}
      BIND(IF(bound(?pubchem_label_temp), ?pubchem_label_temp,"null") AS ?pubchem_label)
    BIND(IF(bound(?pubchem_molecular_weight_temp), ?pubchem_molecular_weight_temp,"null") AS ?pubchem_molecular_weight)
    BIND(IF(bound(?pubchem_molecular_formula_temp), ?pubchem_molecular_formula_temp,"null") AS ?pubchem_molecular_formula)
    BIND(IF(bound(?pubchem_smiles_temp), ?pubchem_smiles_temp,"null") AS ?pubchem_smiles)
    BIND(IF(bound(?pubchem_inchi_temp), ?pubchem_inchi_temp,"null") AS ?pubchem_inchi)  
    BIND (strafter(str(?pubchem_uri), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/") AS ?pubchem_id)
    BIND(CONCAT("https://pubchem.ncbi.nlm.nih.gov/image/imagefly.cgi?cid=",(SUBSTR(STR(?pubchem_id),4)),"&width=500&height=500") AS ?pubchem_formula_fig)
BIND(CONCAT('<img src="',  ?pubchem_formula_fig,   '"/>') AS ?pubchem_formula_img)
   }  {{/if}}

 {{#if togoidDict.chembl_uri}}                  
{
 VALUES ?chem_id  {  {{#each togoidDict.chembl_uri}} <{{this}}> {{/each}}   }
 ?chem_id a cco:SmallMolecule ;
           skos:altLabel ?label_temp ;
           cco:substanceType ?chembl_type ;
           foaf:depiction  ?chembl_formula_fig .
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_1. ?att_1 a cheminf:CHEMINF_000042 ; cheminf:SIO_000300 ?molecular_formula_temp }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_2. ?att_2 a cheminf:CHEMINF_000216 ; cheminf:SIO_000300 ?molecular_weight_temp  }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_3. ?att_3 a cheminf:CHEMINF_000018 ; cheminf:SIO_000300 ?smiles_temp  }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_4. ?att_4 a cheminf:CHEMINF_000113 ; cheminf:SIO_000300 ?inchi_temp  }.
  BIND(IF(bound(?molecular_formula_temp), ?molecular_formula_temp,"null") AS ?chembl_molecular_formula)
  BIND(IF(bound(?molecular_weight_temp), ?molecular_weight_temp, "null") AS ?chembl_molecular_weight)
  BIND(IF(bound(?smiles_temp), ?smiles_temp,"null") AS ?chembl_smiles)
  BIND(IF(bound(?inchi_temp), ?inchi_temp,"null") AS ?chembl_inchi)
  BIND (strafter(str(?chem_id), "http://rdf.ebi.ac.uk/resource/chembl/molecule/") AS ?chembl_id)
  BIND(CONCAT('<img src="',  ?chembl_formula_fig,   '"/>') AS  ?chembl_formula_img) 
  }
 {{/if}}
{{#if togoidDict.chebi_uri}}                  
{
 VALUES ?chem_id_2  {  {{#each togoidDict.chebi_uri}} <{{this}}> {{/each}}   }
 ?chem_id_2  rdfs:label ?label;
             oboinowl:id ?id.
  optional{ ?chem_id_2 chebi:formula ?chebi_formula_tmp . }
  optional{ ?chem_id_2 chebi:inchi ?chebi_inchi_tmp . }
  optional{ ?chem_id_2 chebi:monoisotopicmass ?chebi_mass_tmp . }
  optional{ ?chem_id_2 chebi:smiles ?chebi_smiles_tmp . }
  optional{ ?chem_id_2 iao:0000115 ?chebi_definition_tmp . }
  optional{ ?chem_id_2 oboinowl:hasAlternativeId ?chebi_altid_tmp . } 
  BIND(IF(bound(?chebi_formula_temp), ?chebi_formula_temp,"null") AS ?chebi_molecular_formula) 
  }
 {{/if}}                     
 }              
```

## `return`

```javascript 
({ main }) => {
  let pubchem_info = main.results.bindings.map((elem) => ({
    cid: elem.pubchem_id.value,
    molecular_formula: elem.pubchem_molecular_formula.value,
    label: elem.pubchem_label.value,
    molecular_weight: elem.pubchem_molecular_weight.value,
    smiles: elem.pubchem_smiles.value,
    inchi: elem.pubchem_inchi.value,
    formula_img: elem.pubchem_formula_img.value,
      }));
  let chembl_info = main.results.bindings.map((elem) => ({
    chembl_id: elem.chembl_id.value,
    molecular_formula: elem.chembl_molecular_formula.value,
    type: elem.chembl_type.value,
    label: elem.chembl_label.value,
    molecular_weight: elem.chembl_molecular_weight.value,
    smiles: elem.chembl_smiles.value,
    inchi: elem.chembl_inchi.value,
    formula_img: elem.chembl_formula_img.value,
      }));
  return {
    pubchem_info: pubchem_info,
    chembl_info: chembl_info,
  };
};
```
