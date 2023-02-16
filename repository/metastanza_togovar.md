# Variant details and link to TogoVar  (三橋)

## Description

- Data sources
    -  [TogoVar](https://togovar.org/?) (limited to variants with frequency data in Japanese populations)
- Query
    - Input
        - TogoVar id
    - Output
        -  TogoVar id
        -  Variant type (SNV, Insersion, Deletion, InDel, Substitution) 
        -  [HGVS notation](https://varnomen.hgvs.org/bg-material/simple/)
        -  Link to TogoVar. See details about each variant in TogoVar.

## Parameters

* `tgv_id` TogoVar ID
  * default: 
  * example: tgv219804 (autosomal), tgv80912738(chr X), tgv67047924(chr Y), tgv67070461(MT)

## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX obo_in_owl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX m2r: <http://med2rdf.org/ontology/med2rdf#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>

SELECT DISTINCT ?tgv_id ?type ?hgvs ?link_to_togovar
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/so>
WHERE {
    VALUES ?tgv_id { "{{tgv_id}}" }

    ?variation dct:identifier ?tgv_id ;
        a ?type .

    OPTIONAL {
       ?variation tgvo:hasConsequence/tgvo:hgvsg ?hgvs .
    }
  
    BIND(IRI(CONCAT("https://grch38.togovar.org/variant/", ?tgv_id)) AS ?link_to_togovar)
}
```

## `results`
```javascript

({ data }) =>  {
// See https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.40
var chr2nc = {};
chr2nc[1]="NC_000001.11";
chr2nc[2]="NC_000002.12";
chr2nc[3]="NC_000003.12";
chr2nc[4]="NC_000004.12";
chr2nc[5]="NC_000005.10";
chr2nc[6]="NC_000006.12";
chr2nc[7]="NC_000007.14";
chr2nc[8]="NC_000008.11";
chr2nc[9]="NC_000009.12";
chr2nc[10]="NC_000010.11";
chr2nc[11]="NC_000011.10";
chr2nc[12]="NC_000012.12";
chr2nc[13]="NC_000013.11";
chr2nc[14]= "NC_000014.9";
chr2nc[15]="NC_000015.10";
chr2nc[16]="NC_000016.10";
chr2nc[17]="NC_000017.11";
chr2nc[18]="NC_000018.10";
chr2nc[19]="NC_000019.10";
chr2nc[20]="NC_000020.11";
chr2nc[21]="NC_000021.9";
chr2nc[22]="NC_000022.11";
chr2nc['X']="NC_000023.11";
chr2nc['Y']="NC_000024.10";
chr2nc['MT']=" NC_012920.1";   // https://www.ncbi.nlm.nih.gov/nucleotide/NC_012920.1
  
    return data.results.bindings.map((d) => ({
      tgv_id: d.tgv_id .value,
      type_label: d.type.value.replace("http://genome-variation.org/resource#",""),
      hgvs:  d.hgvs.value.replace(/^(\S+):([gm].+)/, function(m, chr, pos_allele){ return chr2nc[chr] + ":" + pos_allele; }),
      url: d.link_to_togovar.value
    }));
 };
```