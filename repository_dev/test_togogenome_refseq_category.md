# TogoGenomeのGenomeをRefSeqのcategoryに分ける（信定） 

## Endpoint
http://togogenome.org/sparql

## `main`

```sparql
PREFIX asm: <http://ddbj.nig.ac.jp/ontologies/assembly/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT  ?GCFnumber  ?asm_name  ?category
FROM <http://togogenome.org/graph/assembly_report>
WHERE
{
  ?assembly_report a asm:Assembly_Database_Entry ;
                   rdfs:seeAlso ?GCF ;
                   asm:asm_name ?asm_name ;
                   asm:refseq_category ?category.
  BIND (strafter(str(?GCF), "http://ddbj.nig.ac.jp/ontologies/assembly/") AS ?GCFnumber)
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
   
    let category_id = d.category.value;
    if (category_id  == "reference genome") category_id= "1";
    else if (category_id  == "representative genome") category_id = "2";
    else if (category_id  == "na") category_id = "3";
    
    tree.push({
      id: d.GCFnumber.value,
      label: d.asm_name.value,
      leaf: true,
      parent: category_id
    })
  // root との親子関係を追加
    if (!edge[d.category.value]) {
      edge[d.category.value] = true;
      tree.push({   
        id: category_id,
        label: d.category.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```
