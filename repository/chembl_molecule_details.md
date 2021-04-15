# chembl compoundの詳細情報を出す（信定）

## Parameters

* `queryId` (type: chembl_compound)
  * example: CHEMBL6939, CHEMBL102358

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryId}) => {
  queryId = queryId.replace(/,/g," ")
  if (queryId.match(/[^\s]/)) return queryId.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `sparql`

```sparql
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
prefix foaf: <http://xmlns.com/foaf/0.1/>
prefix chembl_molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
prefix cheminf: <http://semanticscience.org/resource/>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?id ?type  ?label ?molecular_formula ?molecular_weight ?smiles ?inchi ?formula_figure
{SELECT ?id  ?type  ?molecular_formula ?molecular_weight ?smiles ?inchi ?formula_figure
(GROUP_concat(?label_temp; separator = ", ") as ?label)
WHERE 
{
  VALUES ?chembl_id  { {{#each queryArray}} chembl_molecule:{{this}} {{/each}} }
 ?chembl_id a cco:SmallMolecule ;
           skos:altLabel ?label_temp ;
           cco:substanceType ?type ;
           foaf:depiction ?formula_figure .
  OPTIONAL { ?chembl_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_1. ?att_1 a cheminf:CHEMINF_000042 ; cheminf:SIO_000300 ?molecular_formula_temp }.
  OPTIONAL { ?chembl_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_2. ?att_2 a cheminf:CHEMINF_000216 ; cheminf:SIO_000300 ?molecular_weight_temp  }.
  OPTIONAL { ?chembl_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_3. ?att_3 a cheminf:CHEMINF_000018 ; cheminf:SIO_000300 ?smiles_temp  }.
  OPTIONAL { ?chembl_id a cco:SmallMolecule ; cheminf:SIO_000008 ?att_4. ?att_4 a cheminf:CHEMINF_000113 ; cheminf:SIO_000300 ?inchi_temp  }.
  BIND(IF(bound(?molecular_formula_temp), ?molecular_formula_temp,"null") AS ?molecular_formula)
  BIND(IF(bound(?molecular_weight_temp), ?molecular_weight_temp, "null") AS ?molecular_weight)
  BIND(IF(bound(?smiles_temp), ?smiles_temp,"null") AS ?smiles)
  BIND(IF(bound(?inchi_temp), ?inchi_temp,"null") AS ?inchi)
  BIND (strafter(str(?chembl_id), "http://rdf.ebi.ac.uk/resource/chembl/molecule/") AS ?id)
  } }
 GROUP BY ?id    
```
## `return`
- 整形

```javascript
({sparql})=>{
  return sparql.results.bindings.map(d=>{ 
    return {
      id: d.id.value, 
     type: d.type.value, 
      label: d.label.value, 
      molecular_formula: d.molecular_formula.value,
      molecular_weight: d.molecular_weight.value,
     　smiles: d.smiles.value,
      inchi: d.inchi.value,
      image: d.formula_figure.value
    };
  }) 
 }
```


