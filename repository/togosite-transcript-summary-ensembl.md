# Transcript attributes from Ensembl RDF（井手・八塚）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters
* `enst`
  * default: ENST00000460472
  * example: ENST00000460472

## `enst_id`
```javascript
({ enst }) => {
  return enst.replace(/\s/g, "");
};
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT ?id ?gene_name ?label ?taxon_id ?location ?type_name COUNT(?exon) AS ?exon_count COUNT(?protein) AS ?protein_count
WHERE
{
  VALUES ?entry { enst:{{enst_id}} }
  ?entry a ?type .
  ?entry rdfs:label ?label .
  ?entry biohack:location/rdfs:label ?location .
  ?entry obo:RO_0002162 ?taxon .
  ?entry obo:SO_transcribed_from/dc:description ?gene_name .
  ?entry obo:SO_has_part ?exon .
  ?entry obo:SO_translates_to ?protein .
  FILTER REGEX(?type, "^http://rdf")
  BIND(REPLACE(STR(?entry), enst:, "") AS ?id)
  BIND(REPLACE(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/", "") AS ?type_name)
  BIND(REPLACE(STR(?taxon), taxon:, "") AS ?taxon_id)
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    id: elem.id.value,
    type: elem.type_name.value,
    label: elem.label.value,
    taxonomy: elem.taxon_id.value,
    location: elem.location.value,
    exon_count: elem.exon_count.value,
    protein_count: elem.protein_count.value,
  }));
};
```