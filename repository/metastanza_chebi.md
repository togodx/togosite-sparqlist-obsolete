# 化合物の詳細情報を出す（信定・山本・建石）

## Parameters

* `id`
  * default: 15414
  * example: 15414, 17232

## Endpoint

https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX iao: <http://purl.obolibrary.org/obo/IAO_>
PREFIX chebi: <http://purl.obolibrary.org/obo/chebi/>

SELECT DISTINCT ?chebi ?molecular_formula ?id
                (GROUP_concat(distinct ?altid; separator = ", ") as ?altids)
                (GROUP_concat(distinct ?label; separator = "__") as ?labels)
                (GROUP_concat(distinct ?synonym; separator = "__") as ?synonyms)
                ?mass ?smiles ?inchi ?definition
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE {
  VALUES ?chebi  { obo:CHEBI_{{id}} }
  ?chebi rdfs:label ?label ;
         oboinowl:id ?id .
  OPTIONAL { ?chebi oboinowl:hasExactSynonym ?synonym . }
  OPTIONAL { ?chebi chebi:formula ?molecular_formula . }
  OPTIONAL { ?chebi chebi:inchi ?inchi . }
  OPTIONAL { ?chebi chebi:mass ?mass . }
  OPTIONAL { ?chebi chebi:smiles ?smiles . }
  OPTIONAL { ?chebi iao:0000115 ?definition . }
  OPTIONAL { ?chebi oboinowl:hasAlternativeId ?altid . }
}
```

## `return`

```javascript
({ main }) => {
  const objs = [];
  const data = main.results.bindings[0];
  objs[0] = {
    URL: data.chebi.value,
    ID: data.id.value,
    molecular_formula: data.molecular_formula?.value ?? "",
    label: makeList(data.labels.value.split("__")),
    synonym: "",
    definition: data.definition?.value ?? "",
    mass: data.mass?.value ?? "",
    SMILES: data.smiles?.value ?? "",
    InChI: data.inchi?.value ?? "",
    secondary_ID: data.altids?.value ?? ""
  };
  if (data.synonyms?.value)
    objs[0].synonym = makeList(data.synonyms.value.split("__"));
  return objs;

  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
};
```
