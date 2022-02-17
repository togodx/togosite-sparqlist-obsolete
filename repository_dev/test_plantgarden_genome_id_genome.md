# plantgarden_genome_id_genome (信定)
- 生物種ごとにゲノム情報をリスト

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
PREFIX pg_ns: <https://plantgardden.jp/ns/>
PREFIX dcterms: <http://purl.org/dc/terms/>
select distinct ?genome_id  ?version   ?subspecies_id  ?subspecies_label ?species_id  ?species_name  ?level 
from <http://plantgarden.jp/resource/genome>
from <http://plantgarden.jp/resource/subspecies>
from <http://plantgarden.jp/resource/species>
where {
?s a pg_ns:Genome ;
pg_ns:subspecies ?subspecies ;
pg_ns:pgid ?subspecies_id ;
pg_ns:species ?species ;
pg_ns:species_id ?species_id ;
dcterms:identifier ?genome_id ;
pg_ns:assembly_version ?version ;
pg_ns:assembly_level  ?level .
?subspecies a pg_ns:Subspecies ;
rdfs:label ?subspecies_label .
?species a pg_ns:Species ;
pg_ns:scientific_name ?species_name .
FILTER (lang(?subspecies_label) = "en" )
}

limit 100
```
## `return`
```javascript
({ main}) => {
  
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
      id: d.genome_id.value,
      label: d.genome_id.value,
      leaf: true,
      parent: d.version.value
    })
      });
 
  main.results.bindings.map(d => {
  tree.push({
      id: d.version.value,
      label: d.version.value,
      parent: d.subspecies_id.value
    })
        });
  
  main.results.bindings.map(d => {
  tree.push({
      id: d.subspecies_id.value,
      label: d.subspecies_label.value,
      parent: d.species_id.value
    })
        });
  
    main.results.bindings.map(d => {
  tree.push({
      id: d.species_id.value,
      label: d.species_name.value,
      parent: d.level.value
    })
        });
  
 main.results.bindings.map(d => {
  tree.push({
      id: d.level.value,
      label: d.level.value,
      parent: "root"
    })
  }) ;
  
  const uniqueTree = tree.filter(
  (element, index, self) => self.findIndex((e) => e.id === element.id) === index
);
  
  return uniqueTree;
}
  ```