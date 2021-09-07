# Tissues where a gene is not expressed （池田）

## Description

- Data sources
    - [GTEx version 8](https://gtexportal.org/home/datasets)
- Query
    -  Input
        - Ensembl Gene ID
    - Output
        - RefEx Sample ID for tissues

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
# Endpoint: https://integbio.jp/togosite/sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?parent_label ?parent ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 [
             a refexo:logTPMMax ;
             sio:SIO_000300 0
             #sio:SIO_000300 ?logtpmmax
           ] ;
           refexo:isMeasurementOf ?child ;
           refexo:refexSample ?refexs .
    #FILTER(?logtpmmax >= 0 && ?logtpmmax < 1)
  }
  
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dcterms:description ?parent_label ;
            dcterms:identifier ?parent .
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?child a ?type ;
           rdfs:label ?child_label .
    VALUES ?type { enso:lncRNA obo:SO_0001217 obo:SO_0000336 enso:TEC }
    MINUS { ?child a enso:rRNA_pseudogene }
  }

}LIMIT 100#ORDER BY ?parent_label
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  
  data.results.bindings.map(d => {
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
