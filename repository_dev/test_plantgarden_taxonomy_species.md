# PlantGarden taxonomy (信定)

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
select distinct   ?genus  ?family  ?scientific_name  ?species_id
from <http://plantgarden.jp/resource/species>
where {
?s pg_ns:family_name ?family ;
pg_ns:genus_name ?genus ;
pg_ns:scientific_name ?scientific_name ;
pg_ns:common_name ?common_name ;
dcterms:identifier ?species_id .
FILTER (lang(?family) = "ja" )
FILTER (lang(?genus) = "ja" )
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
      id: d.genus.value,
      label: d.genus.value,
      leaf: true,
      parent: d.family.value
    })
      });

  main.results.bindings.map(d => {
    tree.push({
      id: d.family.value,
      label: d.family.value,
      parent: d.species_id.value
    })
  }) ;
 main.results.bindings.map(d => {
    tree.push({
      id: d.species_id.value,
      label: d.scientific_name.value,
      parent: "root"
    })
  }) ;

 const uniqueTree = Array.from(
  new Map(tree.map((tree2) => [tree2.id, tree2])).values()
)
  
  return uniqueTree;
}
```