# TogoGenome genomeのrelease_date （信定）

## Endpoint
http://togogenome.org/sparql

## `main`

```sparql
PREFIX asm: <http://ddbj.nig.ac.jp/ontologies/assembly/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT  ?GCFnumber ?asm_name  ?date ?year
FROM <http://togogenome.org/graph/assembly_report>
WHERE
{
  ?assembly_report a asm:Assembly_Database_Entry ;
                   rdfs:seeAlso ?GCF ;
                   asm:asm_name ?asm_name ;
                   asm:release_date ?date.
  BIND (strafter(str(?GCF), "http://ddbj.nig.ac.jp/ontologies/assembly/") AS ?GCFnumber)
  BIND (strbefore(str(?date), "/") AS ?year)
}
ORDER BY DESC (?year)
limit 100
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
      id: d.GCFnumber.value,
      label: d.asm_name.value,
      leaf: true,
      parent: d.year.value
    })
  // root との親子関係を追加
    if (!edge[d.year.value]) {
      edge[d.year.value] = true;
      tree.push({   
        id: d.year.value,
        label: d.year.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
}
```
