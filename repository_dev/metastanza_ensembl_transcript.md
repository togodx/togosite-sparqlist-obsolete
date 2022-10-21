# Transcript attributes from Ensembl RDF（井手・八塚・池田）

ENST ID を受け取って、自分と兄弟の transcript の情報を返す

is_ensg が 1 のときは、入力 ID を ENSG として扱い、紐づいている transcript の情報を返す

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `id`
  * example: ENST00000302586
* `is_ensg` (optional)
  * example: 1

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX dct: <http://purl.org/dc/terms/>

SELECT DISTINCT ?enst_id ?label ?chr_num ?type_name
  (GROUP_CONCAT(DISTINCT ?uniprot_id; separator=",") AS ?uniprot_ids) #(COUNT(DISTINCT ?exon) AS ?exon_count)
  ?begin ?end
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE
{
  {{#if is_ensg}}
    VALUES ?ensg { ensg:{{id}} }
  {{else}}
    VALUES ?input_enst { enst:{{id}} }
    ?input_enst so:transcribed_from ?ensg .
  {{/if}}
  VALUES ?strand { faldo:ReverseStrandPosition faldo:ForwardStrandPosition }
  ?ensg so:part_of ?chr .
  ?enst so:transcribed_from ?ensg ;
        a ?type ;
        rdfs:label ?label ;
        dct:identifier ?enst_id ;
        so:has_part ?exon .
  ?exon faldo:location [
    faldo:begin [
      faldo:position ?begin
    ] ;
    faldo:end [
      faldo:position ?end
    ]
  ] .

  OPTIONAL {
    ?enst rdfs:seeAlso ?uniprot .
    FILTER(CONTAINS(STR(?uniprot), "http://purl.uniprot.org/uniprot/"))
    BIND(STRAFTER(STR(?uniprot), "http://purl.uniprot.org/uniprot/") AS ?uniprot_id)
  }
  FILTER REGEX(?type, "^http://rdf")
  BIND(REPLACE(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/", "") AS ?type_name)
  BIND(STRBEFORE(STRAFTER(STR(?chr), "http://identifiers.org/hco/"), "#") as ?chr_num)
}
ORDER BY ?enst_id
```

## `return`

```javascript
({ main }) => {
  let obj = {};
  let ids = new Set();
  main.results.bindings.forEach((elem) => {
    let exon_length = Math.abs(elem.begin.value - elem.end.value) + 1;
    let enst_id = elem.enst_id.value;
    if (!ids.has(enst_id)) {
      obj[enst_id] = {
        enst_id: enst_id,
        enst_url: "http://identifiers.org/ensembl/" + elem.enst_id.value,
        uniprot_id: elem.uniprot_ids.value.split(",").map((elem)=>("<a href=\"http://identifiers.org/uniprot/"+elem+"\">"+elem+"</a>")).join(", "),
        //uniprot_url: "http://identifiers.org/uniprot/" + elem.uniprot_id?.value,
        type: elem.type_name.value.replace(/_/g, " "),
        label: elem.label.value,
        bp: exon_length,
        exon_count: 1
      };
      ids.add(enst_id);
    } else {
      obj[enst_id].bp += exon_length;
      obj[enst_id].exon_count++;
    }
  });
  return Object.values(obj);
}
```