# Glycan information from GlyCosmos RDF （池田）

## Endpoint

https://ts.glycosmos.org/sparql

## Parameters
* `id`
  * default: G14606UO
  * example: G14606UO

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>
PREFIX glycan2: <http://rdf.glycoinfo.org/glycan/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?gtc_1 ?mass ?sbsmpt ?wurcs_label ?iupac (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
WHERE {
  VALUES ?gtc_1 { struct:{{id}} }
  VALUES ?gtc_2 { glycan2:{{id}} }
  ?wurcs mass:WURCSMassCalculator ?mass ;
         rdfs:seeAlso ?gtc_1 ;
         rdfs:label ?wurcs_label ;
         a ?sbsmpt .
  OPTIONAL {
  [] glycan:has_glycan ?gtc_2 ;
     a <http://purl.jp/bio/4/id/200906013374193296> ;
     glycan:has_tissue ?tissue .
  ?tissue rdfs:label ?tissue_label .
  }
  OPTIONAL {
    ?wurcs rdfs:label / ^glycan:has_sequence / ^glycan:has_glycosequence / skos:altLabel ?iupac .
  }
}GROUP BY ?gtc_1 ?mass ?sbsmpt ?wurcs_label ?iupac
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    GlyTouCan_ID: elem.gtc_1.value.replace("https://glytoucan.org/Structures/Glycans/", ""),
    GlyTouCan_URL: elem.gtc_1.value.replace("https://glytoucan.org/Structures/Glycans/", "https://glycosmos.org/glycans/show?gtc_id="),
    IUPAC: elem.iupac?.value ?? "",
    WURCS: elem.wurcs_label.value,
    mass: elem.mass.value,
    Subsumption: elem.sbsmpt.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", "").replace(/_/g, " "),
    tissue: elem.tissue_labels?.value ?? ""
  }))
}
```