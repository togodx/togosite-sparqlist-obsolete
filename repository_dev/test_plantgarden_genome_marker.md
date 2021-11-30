# PlantGarden taxonomy-genome-chr-marker (信定)

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
```

## `chr_genome`
- 親子関係
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?s ?parent_chr   ?parent_genome_identifier ?parent_genome_label
where {
?s a  pg_ns:Marker ;
pg_ns:genome ?parent_genome ;
pg_ns:chr ?parent_chr .
?parent_genome a  pg_ns:Genome ;
dcterms:identifier ?parent_genome_identifier ;
rdfs:label ?parent_genome_label .
}
```

## `genome_subspecies`
- 親子関係
```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?s ?parent_genome_identifier ?parent_genome_label  ?top_subspecies_identifier  ?top_subspecies_label 
where {
?s a  pg_ns:Marker ;
pg_ns:genome ?parent_genome  .
?parent_genome a  pg_ns:Genome ;
dcterms:identifier ?parent_genome_identifier ;
rdfs:label ?parent_genome_label ;
pg_ns:subspecies ?top_genome_subspecies .
 ?top_genome_subspecies  a   pg_ns:Subspecies  ;
dcterms:identifier ?top_subspecies_identifier ;
rdfs:label ?top_subspecies_label .
FILTER (lang(?top_subspecies_label) = "en" )
}
```
