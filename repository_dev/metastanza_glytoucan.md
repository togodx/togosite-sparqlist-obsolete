# Glycan information from GlyCosmos RDF （池田）

## Endpoint

https://ts.glycosmos.org/sparql

## Parameters
* `id`
  * default: G14606UO
  * example: G14606UO, G00035MO

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX glycan2: <http://rdf.glycoinfo.org/glycan/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?gtc ?mass ?sbsmpt ?wurcs_label ?iupac (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
WHERE {
  VALUES ?gtc { glycan2:{{id}} }
  ?wurcs mass:WURCSMassCalculator ?mass ;
         dcterms:source ?gtc ;
         rdfs:label ?wurcs_label ;
         a ?sbsmpt .
  OPTIONAL {
    ?gtc rdfs:seeAlso [
      glycan:has_tissue ?tissue ;
      glycan:has_taxon / dcterms:identifier "9606"
    ] .
    ?tissue rdfs:label ?tissue_label .
  }
  OPTIONAL {
    ?gtc glycan:has_glycosequence ?gsq .
    ?gsq glycan:in_carbohydrate_format glycan:carbohydrate_format_iupac_condensed ;
         glycan:has_sequence ?iupac .
  }
}GROUP BY ?gtc ?mass ?sbsmpt ?wurcs_label ?iupac
```

## `return`

```javascript
({ main, id }) => {
  const data = main.results.bindings[0];
  const objs = main.results.bindings.map((elem) => ({
    ID: id,
    url: "https://glycosmos.org/glycans/show/" + id,
    IUPAC: elem.iupac?.value ?? "",
    WURCS: elem.wurcs_label.value,
    mass: elem.mass.value,
    subsumption: elem.sbsmpt.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", "").replace(/_/g, " "),
    tissue: "",
  }));
  if (data.tissue_labels?.value) {
    objs[0].tissue = makeList(data.tissue_labels.value.split(", ").sort());
  }
  return [objs[0]];

  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
}
```