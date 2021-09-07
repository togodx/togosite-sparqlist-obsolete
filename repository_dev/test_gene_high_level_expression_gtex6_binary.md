# Genes expressed in tissues (GTEx ver6)（小野・池田・千葉）

## Description

- Data sources
    - Supplementary Table 1 of [A systematic survey of human tissue-specific gene expression and splicing reveals new opportunities for therapeutic target identification and evaluation; R. Y. Yang et al.; bioRxiv 311563](https://doi.org/10.1101/311563)
    - Mapping from the tissue names to the corresponding UBERON or EFO term is based on [GTEx documentation](https://gtexportal.org/home/samplingSitePage).
- Query
    - Input
        - Ensembl gene ID
    - Output
        - UBERON ID or EFO ID or "low_specificity"

## Endpoint

https://integbio.jp/togosite/sparql


## `main`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX schema: <http://schema.org/>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6_refexsample> {
    ?child refexo:isPositivelySpecificTo ?tissue .
  }
  # 遅い
  BIND(URI(REPLACE(STR(?child), "http://identifiers.org/ensembl/", "http://rdf.ebi.ac.uk/resource/ensembl/")) AS ?ebi_ensg)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ebi_ensg rdfs:label ?child_label .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?tissue dcterms:description ?parent_label ;
            dcterms:identifier ?parent .
  }
}
# ORDER BY ?parent_label
```

## `low_spec`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  BIND("low_specificity" AS ?parent)
  BIND("Low specificity" AS ?parent_label)
  ?child a refexo:GTEx_v6_ts_evaluated_gene .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
    FILTER NOT EXISTS {
      ?child refexo:isPositivelySpecificTo ?t .
    }
  }
  # 遅い
  BIND(URI(REPLACE(STR(?child), "http://identifiers.org/ensembl/", "http://rdf.ebi.ac.uk/resource/ensembl/")) AS ?ebi_ensg)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ebi_ensg rdfs:label ?child_label .
  }
}
```

## `return`

```javascript
({main, low_spec}) => {
  let data = main.results.bindings.concat(low_spec.results.bindings)
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  
  data.map(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
  });
  
  return tree;
};
```
