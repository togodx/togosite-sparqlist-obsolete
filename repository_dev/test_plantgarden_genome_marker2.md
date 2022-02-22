# PlantGarden taxonomy-genome-chr-marker2 (信定)
- 生物種ごとのmarkerの分類 
- 生物種ーゲノムーchromosomeーmarker　で treeになっている (chromosomeーmarkerの部分は別取り)

## Endpoint
https://plantgarden.jp/sparql

## `main`
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
limit 10000
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
      id: d.chr_id.value,
      label: d.chr.value,
      parent: d.genome_id.value
    })
  }) ;
 main.results.bindings.map(d => {
    tree.push({
      id: d.genome_id.value,
      label: d.genome_label.value,
      parent: d.species_taxid.value
    })
  }) ;
 main.results.bindings.map(d => {
    tree.push({
      id: d.species_taxid.value,
      label: d.species_label.value,
      parent: "root"
    })
  }) ;

 const uniqueTree = Array.from(
  new Map(tree.map((tree2) => [tree2.id, tree2])).values()
) ;

  return uniqueTree;
}
```