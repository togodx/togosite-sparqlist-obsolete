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
({main})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return main.results.bindings.map(d => {
    const year = d.year.value ;
    const year_id = (year) + "-" + ((year + 1) );
    return {
      id: d.GCFnumber.value,
      label: d.asm_name.value,
      value: Number(d.year.value),
      binId: year + 1,
      binLabel: year_id
    }
  })
}
```
