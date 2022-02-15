# plantgarden_genome_id_genome (信定)
- 生物種ごとにゲノム情報をリスト

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
PREFIX pg_ns: <https://plantgardden.jp/ns/>
PREFIX dcterms: <http://purl.org/dc/terms/>
select distinct ?genome_id  ?version   ?subspecies_id  ?species_id  ?level 
from <http://plantgarden.jp/resource/genome>
where {
?s a pg_ns:Genome ;
pg_ns:pgid ?subspecies_id ;
pg_ns:species_id ?species_id ;
dcterms:identifier ?genome_id ;
pg_ns:assembly_version ?version ;
pg_ns:assembly_level  ?level .
}
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
  
  
  
  ```