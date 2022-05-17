# TogoGenome から各生物種のゲノムサイズ （信定）

## Endpoint
http://togogenome.org/sparql

## `genome_size`
```sparql
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX stats: <http://togogenome.org/stats/>
SELECT distinct ?tax ?tax_id ?genome_size 
FROM <http://togogenome.org/graph/stats>
where{
 #values ?genome_size { 439609253}
  ?tax stats:sequence_length ?genome_size .
  FILTER contains (str(?tax), "taxonomy")
  BIND (strafter(str(?tax), "http://identifiers.org/taxonomy/") AS ?tax_id)
}
limit 10
```
## `query`
```javascript
({genome_size}) => {
  
let tree = [] ;
genome_size.results.bindings.map(d => {
  tree.push ( d.tax_id.value ) ;
}) ;

 (tree) =>  { 
   tree = tree.replace(/,/g," ")
  if (tree.match(/[^\s]/)) return tree.split(/\s+/);
  return false;
  };
}

```

