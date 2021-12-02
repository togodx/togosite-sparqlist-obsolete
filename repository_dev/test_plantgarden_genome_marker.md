# PlantGarden taxonomy-genome-chr-marker (信定)
- 生物種ごとのmarkerの分類 
- 生物種ーゲノムーchromosomeーmarker　で treeになっている

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `leaf`
- chr と marker のアノテーション関係
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?leaf_marker_id ?leaf_marker_label  ?parent_chr 
where {
?s a  pg_ns:Marker ;
dcterms:identifier ?leaf_marker_id ;
rdfs:label ?leaf_marker_label ;
pg_ns:chr ?parent_chr .
} 
limit 10000
```

## `graph_a`
- 親子関係
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct  ?parent_chr   ?parent_genome_identifier ?parent_genome_label
where {
?s a  pg_ns:Marker ;
pg_ns:genome ?parent_genome ;
pg_ns:chr ?parent_chr .
?parent_genome a  pg_ns:Genome ;
dcterms:identifier ?parent_genome_identifier ;
rdfs:label ?parent_genome_label .
}
limit 10000
```
## `graph_b`
- 親子関係
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?genome_identifier ?genome_label  ?species_taxid  ?species_label 
where {
?s a  pg_ns:Marker ;
pg_ns:genome ?genome ;
 pg_ns:species   ?species  .
?genome a  pg_ns:Genome ;
dcterms:identifier ?genome_identifier ;
rdfs:label ?genome_label .
?species a pg_ns:Species ;
a ?ncbi ;
rdfs:label ?species_label .
FILTER contains (str(?ncbi ), "http://purl.obolibrary.org/obo/NCBITaxon").
BIND (IRI(REPLACE(STR(?ncbi), "http://purl.obolibrary.org/obo/NCBITaxon_","")) AS ?species_taxid)
}
limit 10000
```

## `return`
```javascript
({ leaf, graph_a, graph_b}) => {
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  
  let edge = {};
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.leaf_marker_id.value,
      label: d.leaf_marker_label.value,
      leaf: true,
      parent: d.parent_chr.value
    })
      });
  // 親子関係
  graph_a.results.bindings.map(d => {
    tree.push({
      id: d.parent_chr.value,
      label: d.parent_chr.value,
      parent: d.parent_genome_identifier.value
    })
  }) ;
graph_b.results.bindings.map(d => {
    tree.push({
      id: d.genome_identifier.value,
      label: d.genome_label.value,
      parent: d.species_taxid.value
    })
  }) ;
  
     let subtree = [];
  
graph_b.results.bindings.map(d => {
    subtree.push({
      id: d.species_taxid.value,
      label: d.species_label.value,
      parent: "root"
    })
  }) ;

   const result = subtree.filter((element, index, self) => 
                            self.findIndex(e => 
                                           e.id === element.id &&
                                           e.label === element.label
                                          ) === index
                            );
  
  tree.push(result) ;
  
  return tree;
}

```