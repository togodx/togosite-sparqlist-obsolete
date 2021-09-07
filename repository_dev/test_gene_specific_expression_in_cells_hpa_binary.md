# Genes specifically expressed in cells (HPA)（小野・池田・千葉）

## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpa_cell_specificity> {
    ?child refexo:isPositivelySpecificTo ?parent .
  }
  # 遅い
  BIND(URI(REPLACE(STR(?child), "http://identifiers.org/ensembl/", "http://rdf.ebi.ac.uk/resource/ensembl/")) AS ?ebi_ensg)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ebi_ensg rdfs:label ?child_label .
  }
  ?parent rdfs:label ?parent_label .
}
LIMIT 100
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://identifiers.org/ensembl/";

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