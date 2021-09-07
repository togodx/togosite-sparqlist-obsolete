# RefEx organ specific expression based on ROKU flag 共有 SPARQLet（守屋）

- 組織特異的発現遺伝子の内訳 (default: high)
## Parameters

* `negatively` (低発現)
  * example: 1

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 全体の遺伝子リスト返す（キャッシュが効く）
  - high, low を handlbars で条件分岐

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?child ?child_label ?parent ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/refexo>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genechip_human_GSE7307>
FROM <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info>
WHERE {
  ?child refexo:affyProbeset ?affy .
{{#if negatively}}
  ?affy refexo:isNegativelySpecificTo ?parent .
{{else}}
  ?affy refexo:isPositivelySpecificTo ?parent .
{{/if}}
  ?parent rdfs:label ?parent_label .
  ?child rdfs:label ?child_label .
  FILTER (LANG (?parent_label) = "en")
}
```

## `return`
```javascript
({data}) => {
  const parentIdPrefix = "http://purl.jp/bio/01/refexo#";
  const childIdPrefix = "http://identifiers.org/ncbigene/";

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
        id: d.parent.value.replace(parentIdPrefix, ""),
        label: d.parent_label.value.replace,
        leaf: false,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(parentIdPrefix, "")
    })
  });
  
  return tree;
};
```
