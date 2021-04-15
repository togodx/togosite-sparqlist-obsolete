# NCBI Gene Summary（千葉）

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `ncbigene`
  * default: 100
  * example: 100

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
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX nuc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT ?id ?hgnc_symbol ?description ?type_of_gene ?chromosome ?map_location (GROUP_CONCAT(DISTINCT ?synonym; separator="|") AS ?synonyms) (GROUP_CONCAT(DISTINCT ?others; separator="|") AS ?other_descriptions)
(GROUP_CONCAT(DISTINCT ?tissue_label; separator=",") AS ?tissue_labels)
(GROUP_CONCAT(DISTINCT ?gtex_v6_tissue_label; separator=",") AS ?gtex_v6_tissue_labels)
?homologene_branch_label
?paralog_count
?human_paralog_ids
WHERE {
  {{#if gene_list}}
  VALUES ?human_gene { {{#each gene_list}} ncbigene:{{this}} {{/each}} }
  {{/if}}
  ?human_gene dct:description ?description ;
      dct:identifier ?id ;
      hop:typeOfGene ?type_of_gene ;
      nuc:chromosome ?chromosome ;
      nuc:map ?map_location .
  OPTIONAL {
    ?human_gene nuc:standard_name ?hgnc_symbol .
  }
  OPTIONAL {
    ?human_gene nuc:gene_synonym ?synonym .
  }
  OPTIONAL {
    ?human_gene dct:alternative ?others .
  }

  OPTIONAL{
    ?human_gene refexo:affyProbeset/refexo:isPositivelySpecificTo ?tissue .
    ?tissue rdfs:label ?tissue_label .
    FILTER(lang(?tissue_label) = 'en')
  }

  OPTIONAL {
    GRAPH <https://refex.dbcls.jp/rdf/tissue_specific_genes_gtex_v6> {
      ?ensg refexo:isPositivelySpecificTo ?gtex_v6_tissue .
    }
    ?ensg refexo:ncbigene ?human_gene .
    ?gtex_v6_tissue rdfs:label ?gtex_v6_tissue_label .
  }

  OPTIONAL{
    {
      SELECT ?human_gene (max(?time) as ?branch_time_mya)
      WHERE {
        ?grp orth:inDataset homologene: ;
            orth:hasHomologousMember ?human_gene, ?gene .
        ?gene orth:taxon ?taxid .
        ?taxid hop:branchTimeMya ?time .
      }
    }
    ?branch a hop:HomoloGeneBranch ;
        rdfs:label ?homologene_branch_label ;
        hop:timeMya ?branch_time_mya .
  }

  OPTIONAL {
    {
      SELECT ?human_gene (COUNT(DISTINCT ?human_paralog) AS ?paralog_count)
      WHERE {
        ?grp orth:inDataset homologene: ;
            orth:hasHomologousMember ?human_gene, ?human_paralog .
        ?human_paralog orth:taxon taxid:9606 .
      }
    }
  }

  OPTIONAL {
    {
      SELECT ?human_gene (GROUP_CONCAT(DISTINCT ?human_paralog_id; separator="|") AS ?human_paralog_ids)
      WHERE {
        ?grp orth:inDataset homologene: ;
            orth:hasHomologousMember ?human_gene, ?human_paralog .
        ?human_paralog orth:taxon taxid:9606 ;
            dct:identifier ?human_paralog_id
        FILTER (?human_gene != ?human_paralog)
      }
    }
  }

}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    id: elem.id.value,
    hgnc_symbol: getValue(elem.hgnc_symbol),
    synonyms: elem.synonyms.value,
    description: elem.description.value,
    other_descriptions: elem.other_descriptions.value,
    refex_tissue_labels: elem.tissue_labels?.value,
    gtex_v6_tissue_labels: getValue(elem.gtex_v6_tissue_labels),
    homologene_branch_label: getValue(elem.homologene_branch_label),
    type_of_gene: elem.type_of_gene.value,
    human_paralog_count: getNumber(elem.paralog_count),
    humman_paralog_ncbigene_ids: getValue(elem.human_paralog_ids),
    chromosome: elem.chromosome.value,
    map_location: elem.map_location.value,
  }));
  function getValue(obj) {
    if (obj) {
      return obj.value;
    } else {
      return '';
    }
  }
  function getNumber(obj) {
    if (obj == null) {
      return undefined;
    } else {
      return Number(obj.value);
    }
  }
};
```