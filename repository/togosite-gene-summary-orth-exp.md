# Gene attributes

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `ncbigene`
  * default: 100
  * example: 100,101

## `gene_list`
```javascript
({ ncbigene }) => {
  ncbigene = ncbigene.replace(/\s/g, "");
  if (ncbigene) {
    return ncbigene.split(",");
  } else {
    return false;
  }
};
```

## `main`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?human_gene ?branch_time_mya (GROUP_CONCAT(DISTINCT ?tissue_label; separator=",") AS ?tissue_labels) (GROUP_CONCAT(DISTINCT ?gtex_v6_tissue_label; separator=",") AS ?gtex_v6_tissue_labels)
WHERE {
  {{#if gene_list}}
  VALUES ?human_gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
  {{/if}}
  {
    SELECT ?human_gene (max(?time) as ?branch_time_mya)
    WHERE {
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene, ?gene .
      ?human_gene orth:taxon taxid:9606 .
      ?gene orth:taxon ?taxid .
      ?taxid hop:branchTimeMya ?time .
    }
  }
  ?human_gene refexo:affyProbeset/refexo:isPositivelySpecificTo ?tissue .
  ?tissue rdfs:label ?tissue_label .
  FILTER(lang(?tissue_label) = 'en')

  OPTIONAL {
    GRAPH <https://refex.dbcls.jp/rdf/tissue_specific_genes_gtex_v6> {
      ?ensg refexo:isPositivelySpecificTo ?gtex_v6_tissue .
    }
    ?ensg refexo:ncbigene ?human_gene .
  }
  ?gtex_v6_tissue rdfs:label ?gtex_v6_tissue_label .
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    human_gene: elem.human_gene.value,
    branch_time_mya: elem.branch_time_mya.value,
    tissue_labels: elem.tissue_labels.value,
    gtex_v6_tissue_labels: elem.gtex_v6_tissue_labels,
  }));
};
```