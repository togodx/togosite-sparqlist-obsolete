# PlantGarden taxonomy-genome-chr-marker2 (信定)
- 生物種ごとのmarkerの分類 
- 生物種ーゲノムーchromosomeーmarker　で treeになっている

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`

```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?marker_id ?marker_label ?chr_id  
from <http://plantgarden.jp/resource/pg_marker>
from <http://plantgarden.jp/resource/genome>
where {
?s a  pg_ns:Marker ;
dcterms:identifier ?marker_id ;
rdfs:label ?marker_label ;
pg_ns:genome ?genome ;
pg_ns:chr ?chr  .
?genome a  pg_ns:Genome ;
dcterms:identifier ?genome_id .
BIND ( CONCAT(?genome_id, ".", ?chr) as ?chr_id)
} 
limit 50000
```
## `main2`

```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?chr_id   ?chr   ?genome_id   ?genome_label   ?species_taxid   ?species_label
from <http://plantgarden.jp/resource/pg_marker>
from <http://plantgarden.jp/resource/genome>
from <http://plantgarden.jp/resource/species>
where {
?s a  pg_ns:Marker ;
pg_ns:genome ?genome ;
pg_ns:chr ?chr ;
 pg_ns:species   ?species  .
?genome a  pg_ns:Genome ;
dcterms:identifier ?genome_id ;
rdfs:label ?genome_label .
?species a pg_ns:Species ;
a ?ncbi ;
rdfs:label ?species_label .
FILTER contains (str(?ncbi ), "http://purl.obolibrary.org/obo/NCBITaxon").
BIND (IRI(REPLACE(STR(?ncbi), "http://purl.obolibrary.org/obo/NCBITaxon_","")) AS ?species_taxid)
BIND ( CONCAT(?genome_id, ".", ?chr) as ?chr_id)
} 
limit 50000
```

## `return`
```javascript
({main, main2}) => {
  
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
      id: d.marker_id.value,
      label: d.marker_label.value,
      leaf: true,
      parent: d.chr_id.value
    })
      });

  let tree2 = [];
  
  main2.results.bindings.map(d => {
    tree2.push({
      id: d.chr_id.value,
      label: d.chr.value,
      parent: d.genome_id.value
    })
  }) ;
 main2.results.bindings.map(d => {
    tree2.push({
      id: d.genome_id.value,
      label: d.genome_label.value,
      parent: d.species_taxid.value
    })
  }) ;
 main2.results.bindings.map(d => {
    tree2.push({
      id: d.species_taxid.value,
      label: d.species_label.value,
      parent: "root"
    })
  }) ;

 const uniqueTree1 = Array.from(
  new Map(tree.map((tree3) => [tree3.id, tree3])).values()
)
 
 const uniqueTree2 = Array.from(
  new Map(tree2.map((tree4) => [tree4.id, tree4])).values()
)
 
 var uniqueTree = uniqueTree1.concat(uniqueTree2) ;
  
  return uniqueTree;
}
```