# Genes expressed in tissues (Bgee) (池田, 守屋)

## Description

- Data sources
    - Bgee latest version: [https://bgee.org/?page=sparql](https://bgee.org/?page=sparql)


## Parameters

* `obo`
  * default: UBERON_0001295

## Endpoint

https://bgee.org/sparql/

## `leaf`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX genex: <http://purl.org/genex#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dcterms: <http://purl.org/dc/terms/>
SELECT DISTINCT ?gene_id ?gene_label ?anat_entity
WHERE {
  [] a orth:Gene ;
     rdfs:label ?gene_label ;
     orth:organism/obo:RO_0002162 <http://purl.uniprot.org/taxonomy/9606> ;
     dcterms:identifier ?gene_id ;
     genex:isExpressedIn obo:{{obo}} .
  ?anat_entity a genex:AnatomicalEntity .
}
```

## `return`

```javascript
({obo, leaf}) => {
  return
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.gene_id.value,
      label: d.gene_label.value,
      parent: obo
    })
  });
};
```