# 化合物の詳細情報を出す（信定・山本・建石）

## Parameters

* `id`
  * default: CHEMBL6939
  * example: CHEMBL6939, CHEMBL1201330

## Endpoint

https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX chembl: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX cheminf: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?chembl ?id ?molecular_formula ?type
                (GROUP_CONCAT(DISTINCT ucase(?label); separator = "__") as ?labels)
                ?molecular_weight ?smiles ?inchi ?formula_img
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  VALUES ?chembl  { chembl:{{id}} }
  ?chembl a cco:SmallMolecule ;
          skos:altLabel ?label ;
          cco:substanceType ?type ;
          foaf:depiction  ?formula_img .
  OPTIONAL { ?chembl cheminf:SIO_000008 ?att_1. ?att_1 a cheminf:CHEMINF_000042 ; cheminf:SIO_000300 ?molecular_formula }.
  OPTIONAL { ?chembl cheminf:SIO_000008 ?att_2. ?att_2 a cheminf:CHEMINF_000216 ; cheminf:SIO_000300 ?molecular_weight  }.
  OPTIONAL { ?chembl cheminf:SIO_000008 ?att_3. ?att_3 a cheminf:CHEMINF_000018 ; cheminf:SIO_000300 ?smiles  }.
  OPTIONAL { ?chembl cheminf:SIO_000008 ?att_4. ?att_4 a cheminf:CHEMINF_000113 ; cheminf:SIO_000300 ?inchi }.
  BIND (strafter(str(?chembl), str(chembl:)) AS ?id)
}       
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    URL: data.chembl.value.replace("http://rdf.ebi.ac.uk/resource/chembl/molecule/", "http://identifiers.org/chembl.compound/"),
    ID: data.id.value,
    molecular_formula: data.molecular_formula?.value ?? "",
    type: data.type.value,
    label: makeList(data.labels.value.split("__")),
    molecular_weight: data.molecular_weight?.value ?? "",
    SMILES: data.smiles?.value ?? "",
    InChI: data.inchi?.value ?? "",
    formula_img: data.formula_img.value
  };
  return objs;

  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
};
```