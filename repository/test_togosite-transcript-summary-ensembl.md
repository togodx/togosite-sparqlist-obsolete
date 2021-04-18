# Transcript attributes from Ensembl RDF（井手・八塚・池田）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters
* `ensg`
  * default: ENSG00000171097
  * example: ENSG00000171097

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX ensembl: <http://identifiers.org/ensembl/>

SELECT ?enst_id ?gene_name ?label ?location ?type_name ?uniprot_id COUNT(?exon) AS ?exon_count
WHERE
{
  VALUES ?ensg { ensg:{{ensg}} }
  ?entry a ?type .
  ?entry rdfs:label ?label .
  ?entry biohack:location/rdfs:label ?location .
  ?entry obo:SO_transcribed_from ?ensg .
  ?ensg dc:description ?gene_name .
  ?entry obo:SO_has_part ?exon .
  OPTIONAL {
    ?entry obo:SO_translates_to ?ensp .
    ?ensp rdfs:seeAlso ?uniprot .
    FILTER(CONTAINS(STR(?uniprot), "http://identifiers.org/uniprot/"))
    BIND(STRAFTER(STR(?uniprot), "http://identifiers.org/uniprot/") AS ?uniprot_id)
  }
  FILTER REGEX(?type, "^http://rdf")
  BIND(REPLACE(STR(?entry), enst:, "") AS ?enst_id)
  BIND(REPLACE(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/", "") AS ?type_name)
}ORDER BY ?enst_id
```

## `return`

```javascript
({ main }) => {
  return {
  "head": {
    "vars": [
      "enst_id",
      "enst_url",
      "uniprot_id",
      "uniprot_url",
      "type",
      "label",
      "location",
      "exon_count"
    ],
    "labels": [
      "ENST ID",
      null,
      "Type",
      null,
      "Uniprot ID",
      "Name",
      "Location",
      "# exon"
    ],
    "order": [
      0,
      -1,
      2,
      -1,
      1,
      3,
      4,
      5
    ],
    "href": [
      "enst_url",
      null,
      "uniprot_url",
      null,
      null,
      null,
      null
    ],
    "rowspan": [
      false,
      false,
      false,
      false,
      false,
      true,
      false
    ]
  },
  "body": main.results.bindings.map((elem) => ({
    enst_id: {
      "type": "literal",
      "value": elem.enst_id.value
    },
    enst_url: {
      "type": "url",
      "value": "http://identifiers.org/ensembl/" + elem.enst_id.value
    },
    uniprot_id: {
      "type": "literal",
      "value": elem.uniprot_id?.value
    },
    uniprot_url: {
      "type": "url",
      "value": "http://identifiers.org/uniprot/" + elem.uniprot_id?.value
    },
    type: {
      "type": "literal",
      "value": elem.type_name.value.replace("_", " ")
    },
    label: {
      "type": "literal",
      "value": elem.label.value
    },
    location: {
      "type": "literal",
      "value": elem.location.value
    },
    exon_count: {
      "type": "literal",
      "value": elem.exon_count.value
    },
  }))
 };
}
```