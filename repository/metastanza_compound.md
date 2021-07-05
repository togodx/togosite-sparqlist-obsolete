# 化合物の詳細情報を出す（信定・山本・建石）

## Parameters

* `id`
  * default:　2244
  * example: 2244,34756, CHEMBL6939, CHEMBL1201330, 15414 (CHEBI), 17232 (CHEBI)
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

SELECT DISTINCT *
WHERE {
{{#if idDict.pubchem}}
  {
  SERVICE <https://integbio.jp/rdf/pubchem/sparql> {
    SELECT ?pubchem_id ?pubchem_molecular_formula ?pubchem_label ?pubchem_molecular_weight ?pubchem_smiles ?pubchem_inchi ?pubchem_formula_img
    WHERE {
      VALUES ?pubchem_uri  { <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID{{idDict.pubchem}}> }
        OPTIONAL{ ?pubchem_uri sio:has-attribute
          [ a sio:CHEMINF_000382; sio:has-value ?pubchem_label_temp  ] }
        OPTIONAL{ ?pubchem_uri sio:has-attribute
          [ a sio:CHEMINF_000334; sio:has-value ?pubchem_molecular_weight_temp] }
        OPTIONAL{ ?pubchem_uri sio:has-attribute
          [ a sio:CHEMINF_000335; sio:has-value ?pubchem_molecular_formula_temp ] }
        OPTIONAL{ ?pubchem_uri sio:has-attribute
          [ a sio:CHEMINF_000376; sio:has-value ?pubchem_smiles_temp ] }
        OPTIONAL{ ?pubchem_uri sio:has-attribute
          [ a sio:CHEMINF_000396; sio:has-value ?pubchem_inchi_temp ] }
        BIND(IF(bound(?pubchem_label_temp), ?pubchem_label_temp,"null") AS ?pubchem_label)
        BIND(IF(bound(?pubchem_molecular_weight_temp), ?pubchem_molecular_weight_temp,"null") AS ?pubchem_molecular_weight)
        BIND(IF(bound(?pubchem_molecular_formula_temp), ?pubchem_molecular_formula_temp,"null") AS ?pubchem_molecular_formula)
        BIND(IF(bound(?pubchem_smiles_temp), ?pubchem_smiles_temp,"null") AS ?pubchem_smiles)
        BIND(IF(bound(?pubchem_inchi_temp), ?pubchem_inchi_temp,"null") AS ?pubchem_inchi)  
        BIND (strafter(str(?pubchem_uri), "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/") AS ?pubchem_id)
        BIND(CONCAT("https://pubchem.ncbi.nlm.nih.gov/image/imagefly.cgi?cid=",(SUBSTR(STR(?pubchem_id),4)),"&width=500&height=500") AS ?pubchem_formula_fig)
        #BIND(CONCAT('<img src="',  ?pubchem_formula_fig,   '"/>') AS ?pubchem_formula_img)
        BIND( ?pubchem_formula_fig AS ?pubchem_formula_img)
    }}}
  {{/if}}
   
{{#if idDict.chembl}}
{
 SELECT ?chembl_id ?chembl_molecular_formula  ?chembl_type  (GROUP_concat(distinct ucase(?label_temp); separator = ", ") as ?chembl_label) ?chembl_molecular_weight ?chembl_smiles ?chembl_inchi  ?chembl_formula_img
 FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
 WHERE {                                  
 VALUES ?chem_id  { <http://rdf.ebi.ac.uk/resource/chembl/molecule/{{idDict.chembl}}>  }
 ?chem_id a cco:SmallMolecule ;
           skos:altLabel ?label_temp ;
           cco:substanceType ?chembl_type ;
           foaf:depiction  ?chembl_formula_fig .
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_1. ?att_1 a cheminf:CHEMINF_000042 ; cheminf:SIO_000300 ?molecular_formula_temp }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_2. ?att_2 a cheminf:CHEMINF_000216 ; cheminf:SIO_000300 ?molecular_weight_temp  }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_3. ?att_3 a cheminf:CHEMINF_000018 ; cheminf:SIO_000300 ?smiles_temp  }.
  OPTIONAL { ?chem_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_4. ?att_4 a cheminf:CHEMINF_000113 ; cheminf:SIO_000300 ?inchi_temp }.
  BIND(IF(bound(?molecular_formula_temp), ?molecular_formula_temp,"null") AS ?chembl_molecular_formula)
  BIND(IF(bound(?molecular_weight_temp), ?molecular_weight_temp, "null") AS ?chembl_molecular_weight)
  BIND(IF(bound(?smiles_temp), ?smiles_temp,"null") AS ?chembl_smiles)
  BIND(IF(bound(?inchi_temp), ?inchi_temp,"null") AS ?chembl_inchi)
   BIND (strafter(str(?chem_id), "http://rdf.ebi.ac.uk/resource/chembl/molecule/") AS ?chembl_id)
  #BIND(CONCAT('<img src="',  ?chembl_formula_fig,   '"/>') AS  ?chembl_formula_img) 
   BIND(CONCAT(?chembl_formula_fig ) AS  ?chembl_formula_img)
  }}
 {{/if}}

{{#if idDict.chebi}} 
{
 SELECT ?chebi_id ?chebi_molecular_formula  (GROUP_concat(distinct ?chebi_altid; separator = ", ") as ?chebi_altids) (GROUP_concat(distinct ?chebi_label; separator = ", ") as ?chebi_labels) (GROUP_concat(distinct ?chebi_synonym; separator = ", ") as ?chebi_synonym_list) ?chebi_mass ?chebi_smiles ?chebi_inchi
 FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
 WHERE {
 VALUES ?chebi_uri  { <http://purl.obolibrary.org/obo/CHEBI_{{idDict.chebi}}>}
  optional{?chebi_uri  rdfs:label ?chebi_label_temp.}
  optional{?chebi_uri  oboinowl:id ?chebi_id_temp.}
  optional{?chebi_uri oboinowl:hasExactSynonym ?synonym_temp . } 
  optional{ ?chebi_uri chebi:formula ?chebi_formula_temp . }
  optional{ ?chebi_uri chebi:inchi ?chebi_inchi_temp . } 
  optional{ ?chebi_uri chebi:mass ?chebi_mass_temp . }
  optional{ ?chebi_uri chebi:smiles ?chebi_smiles_temp . }
  optional{ ?chebi_uri iao:0000115 ?chebi_definition_temp . }
  optional{ ?chebi_uri oboinowl:hasAlternativeId ?chebi_altid_temp . } 
  BIND(IF(bound(?chebi_label_temp),?chebi_label_temp,"null") as ?chebi_label)
  BIND(IF(bound(?chebi_id_temp),?chebi_id_temp,"null") as ?chebi_id) 
  BIND(IF(bound(?synonym_temp),?synonym_temp,"null") as ?chebi_synonym)
  BIND(IF(bound(?chebi_formula_temp), ?chebi_formula_temp,"null") AS ?chebi_molecular_formula) 
  BIND(IF(bound(?chebi_inchi_temp),?chebi_inchi_temp,"null") as ?chebi_inchi)  
  BIND(IF(bound(?chebi_mass_temp),?chebi_mass_temp,"null") as ?chebi_mass)   
  BIND(IF(bound(?chebi_smiles_temp),?chebi_smiles_temp,"null") as ?chebi_smiles) 
  BIND(IF(bound(?chebi_definition_temp),?chebi_definition_temp,"null") as ?chebi_definition)
  BIND(IF(bound(?chebi_altid_temp),?chebi_altid_temp,"null") as ?chebi_altid) 
  }}
 {{/if}}                     
 }              
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "PubChem_ID": "pubchem_id" },
    { "PubChem_molecular_formula": "pubchem_molecular_formula" },
    { "PubChem_label": "pubchem_label" },
    { "PubChem_molecular_weight": "pubchem_molecular_weight" },
    { "PubChem_smiles": "pubchem_id" },
    { "PubChem_smiles": "pubchem_smiles" },
    { "PubChem_inchi": "pubchem_inchi" },
    { "PubChem_formula_img": "pubchem_formula_img" },
    { "ChEMBL_ID": "chembl_id" },
    { "ChEMBL_molecular_formula": "chembl_molecular_formula" },
    { "ChEMBL_type": "chembl_type" },
    { "ChEMBL_label": "chembl_label" },
    { "ChEMBL_molecular_weight": "chembl_molecular_weight" },
    { "ChEMBL_smiles": "chembl_smiles" },
    { "ChEMBL_inchi": "chembl_inchi" },
    { "ChEMBL_formula_img": "chembl_formula_img" },
    { "ChEBI_ID": "chebi_id" },
    { "ChEBI_molecular_formula": "chebi_molecular_formula" },
    { "ChEBI_label": "chebi_labels" },
    { "ChEBI_synonym": "chebi_synonym_list" },
    { "ChEBI_mass": "chebi_mass" },
    { "ChEBI_smiles": "chebi_smiles" },
    { "ChEBI_inchi": "chebi_inchi" },
    { "ChEBI_altid": "chebi_altids" },
  ];
  return array;
}
```

## `return`

```javascript
({ main, columns }) => {
  return main.results.bindings.map((binding) => {
    const results = columns.map((row) => {
      const obj = {};
      for (const [k, v] of Object.entries(row)) {
        obj[k] = binding[v];
      }
      return obj;
    });

    return results.reduce((obj, elem) => {
      for (const [key, node] of Object.entries(elem)) {
        if (node) {
          obj[key] = node.value;
        }
        return obj;
      };
    }, {});
  });
};
```