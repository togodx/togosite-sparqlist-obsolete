# Forked: Data source of subject Gene (ohta)

Forked from Gene attributes from Ensembl RDF

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: ENSG00000150773
  * example: ENSG00000150773
* `type`
  * default: ensembl_gene
  * example: ensembl_gene

## `idDict` returns an id type object

```javascript
({id, type}) => {
  var obj = {};
  switch (type) {
    case 'ensembl_gene':
      obj.ensg = id;
      break;
    case 'ensembl_transcript':
      obj.enst = id;
      break;
    case 'ncbigene':
      obj.ncbigene = id;
      break;
  }
  return obj;
}
```

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX identifiers: <http://identifiers.org/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?ensg_id ?ncbigene_id ?hgnc_id ?type_label ?desc ?location ?gene_symbol (GROUP_CONCAT(DISTINCT ?uniprot_id; separator=",") AS ?uniprots) (GROUP_CONCAT(DISTINCT ?enst_id; separator=",") AS ?ensts)
WHERE {
  {{#if idDict.ensg}}
  VALUES ?ensg { ensembl:{{idDict.ensg}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?ensg rdfs:seeAlso ?ncbigene, ?hgnc .
    ?ensg obo:RO_0002511 ?enst .
    FILTER(STRSTARTS(STR(?ncbigene), "http://identifiers.org/ncbigene/"))
    FILTER(STRSTARTS(STR(?hgnc), "http://identifiers.org/hgnc/"))
  }
  {{/if}}
  {{#if idDict.ncbigene}}
  VALUES ?ncbigene { ncbigene:{{idDict.ncbigene}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?ncbigene rdfs:seeAlso ?ensg .
    ?hgnc rdfs:seeAlso ?ncbigene .
    ?ensg obo:RO_0002511 ?enst .
    FILTER(STRSTARTS(STR(?ensg), "http://identifiers.org/ensembl/"))
    FILTER(STRSTARTS(STR(?hgnc), "http://identifiers.org/hgnc/"))
  }
  {{/if}}
  {{#if idDict.hgnc}}
  VALUES ?hgnc { hgnc:{{idDict.hgnc}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?hgnc rdfs:seeAlso ?ensg .
    ?hgnc rdfs:seeAlso ?ncbigene .
    ?ensg obo:RO_0002511 ?enst .
    FILTER(STRSTARTS(STR(?ensg), "http://identifiers.org/ensembl/"))
    FILTER(STRSTARTS(STR(?ncbigene), "http://identifiers.org/ncbigene/"))
  }
  {{/if}}
  {{#if idDict.enst}}
  VALUES ?enst_input { ensembl:{{idDict.enst}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?ensg obo:RO_0002511 ?enst_input, ?enst .
    ?hgnc rdfs:seeAlso ?ensg .
    ?ensg rdfs:seeAlso ?ncbigene .
    FILTER(STRSTARTS(STR(?hgnc), "http://identifiers.org/hgnc/"))
    FILTER(STRSTARTS(STR(?ncbigene), "http://identifiers.org/ncbigene/"))
  }
  {{/if}}

  BIND(STRAFTER(STR(?ensg), "http://identifiers.org/ensembl/") AS ?ensg_id)
  BIND(STRAFTER(STR(?enst), "http://identifiers.org/ensembl/") AS ?enst_id)
  BIND(STRAFTER(STR(?ncbigene), "http://identifiers.org/ncbigene/") AS ?ncbigene_id)
  BIND(STRAFTER(STR(?hgnc), "http://identifiers.org/hgnc/") AS ?hgnc_id)
  #?ensg dct:identifier ?ensg_id .
  #?enst dct:identifier ?enst_id .
  #?ncbigene dct:identifier ?ncbigene_id .
  #?hgnc dct:identifier ?hgnc_id .
  GRAPH <http://rdf.ebi.ac.uk/dataset/ensembl/102/homo_sapiens> {
    ?ebiensg obo:RO_0002162 taxon:9606 ;
          dc:identifier ?ensg_id ;
          rdfs:label ?gene_symbol ;
          dc:description ?desc ;
          faldo:location ?loc ;
       	  a ?type .
   	FILTER(STRSTARTS(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/"))
    BIND(STRAFTER(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/") as ?type_label)
    ?loc rdfs:label ?location .
    OPTIONAL {
      ?ebiensg rdfs:seeAlso ?uniprot .
      ?uniprot a identifiers:uniprot .
      BIND(STRAFTER(STR(?uniprot), "http://identifiers.org/uniprot/") AS ?uniprot_id)
    }
  }
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    gene_symbol: elem.gene_symbol.value,
    ensg_id: elem.ensg_id.value,
    hgnc_id: elem.hgnc_id.value,
    ncbigene_id: elem.ncbigene_id.value,
    ensts: elem.ensts.value.split(','),
    uniprots: elem.uniprots.value.split(','),
    type_label: elem.type_label.value,
    desc: elem.desc.value,
    location: elem.location.value,
  }));
};
```