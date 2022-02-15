# plantgarden_mesh_term_pubtator (信定)
- 生物種ごとに存在が報告された化合物をリスト

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
PREFIX ont_pubtator: <http://togodb.org/ontology/pg_pubtator#>
select distinct ?organism ?chemical
from <http://plantgarden.jp/resource/pubtator>
where {
values ?concepts {"Chemical"}
?s a <http://togodb.org/pg_pubtator> ;
rdfs:label ?label ;
ont_pubtator:bio_concepts ?concepts  ;
ont_pubtator:organism ?organism ;
ont_pubtator:term ?chemical .
}
```
## `return`

```javascript
({main}) => {
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  main.results.bindings.map(d => {
    tree.push({
      id: d.chemical.value,
      label: d.chemical.value,
      leaf: true,
      parent: d.organism.value
    })
    
     // root との親子関係を追加
    if (!edge[d.organism.value]) {
      edge[d.organism.value] = true;
      tree.push({   
        id: d.organism.value,
        label: d.organism.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
    

```