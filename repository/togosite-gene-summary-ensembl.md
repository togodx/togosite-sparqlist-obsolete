# Gene attributes from Ensembl RDF（池田）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters
* `ensg`
  * default: ENSG00000150773
  * example: ENSG00000150773,ENSG00000006634

## `gene_list`
```javascript
({ ensg }) => {
  ensg = ensg.replace(/\s/g, "");
  if (ensg) {
    return ensg.split(",");
  } else {
    return false;
  }
};
```

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX identifiers: <http://identifiers.org/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?ensg_id ?type_label ?gene_symbol ?location ?ncbigene ?hgnc (GROUP_CONCAT(DISTINCT ?uniprot; separator=",") AS ?uniprots) ?desc
WHERE {
  {{#if gene_list}}
  VALUES ?ensg { {{#each gene_list}} ensembl:{{this}} {{/each}} }
  {{/if}}
    ?ensg obo:RO_0002162 taxon:9606 ;
          dct:identifier ?ensg_id ;
          rdfs:label ?gene_symbol ;
          dc:description ?desc ;
          faldo:location ?loc ;
       	  a ?type .
   	?type rdfs:label ?type_label .
    ?loc rdfs:label ?location .
    OPTIONAL {
      ?ensg rdfs:seeAlso ?hgnc .
      ?hgnc a identifiers:hgnc .
    }
    OPTIONAL {
      ?ensg rdfs:seeAlso ?ncbigene .
      ?ncbigene a identifiers:ncbigene .
    }
    OPTIONAL {
      ?ensg rdfs:seeAlso ?uniprot .
      ?uniprot a identifiers:uniprot .
    }
    FILTER(LANG(?type_label) = 'en')
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    ensg_id: elem.ensg_id.value,
    type_label: elem.type_label.value,
    gene_symbol: elem.gene_symbol.value,
    location: elem.location.value,
    ncbigene: elem.ncbigene.value,
    hgnc: elem.hgnc.value,
    uniprots: elem.uniprots.value.split(','),
    desc: elem.desc.value
  }));
};
```