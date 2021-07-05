# Genes on chromosomes (片山、池田)

## Description

- Data sources
    - Ensembl human release 102: [http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index](http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index)
- Input/Output
    -  Input
        - Ensembl gene ID
    - Output
        - Chromosome number
 - Supplementary information
      - The chromosome on which each human gene is located.
      - ヒトの各遺伝子が位置している染色体の番号を示します。

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds`
  * defintion: Chromosome number
  * example: 1, 2, 22, X, Y, MT
* `queryIds` (ensembl_gene)
  * example: ENSG00000130234
* `mode` (type: string)
  * example: idList, objectList
  
## `query_id_list`

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/\s/g, "");
  if (queryIds) {
    return queryIds.split(",");
  } else {
    return false;
  }
};
```

## `category_id_list`

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g, " ")
  if (categoryIds.match(/[^\s]/)) {
    return categoryIds.split(/\s+/);
  } else {
    return false;
  }
}
```

## Query `sparql_results`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX taxonomy: <http://identifiers.org/taxonomy/>

{{#if mode}}
SELECT DISTINCT ?ensg_id ?chromosome
{{else}}
SELECT DISTINCT (COUNT(DISTINCT ?ensg_id) AS ?count) ?chromosome
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  VALUES ?ensg_taxonomy { taxonomy:9606 }
  {{#if query_id_list}}
    VALUES ?EnsemblGene { {{#each query_id_list}} ensg:{{this}} {{/each}} }
  {{/if}}
  ?EnsemblGene dc:identifier ?ensg_id ;
    faldo:location ?ensg_location ;
    obo:RO_0002162 ?ensg_taxonomy .
  ?EnsemblTranscript obo:SO_transcribed_from ?EnsemblGene .
  BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
  {{#if category_id_list}}
  VALUES ?chromosome { {{#each category_id_list}} "{{this}}" {{/each}} }
  {{else}}
  VALUES ?chromosome {"1" "2" "3" "4" "5" "6" "7" "8" "9" "10" "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22" "X" "Y" "MT"}
  {{/if}}
}
```

## Output

```javascript
({mode, sparql_results}) => {
  results = sparql_results.results.bindings;
  switch (mode) {
    case 'objectList':
      return results.map(d => {
        return {
          id: d.ensg_id.value,
          attribute: {
            categoryId: d.chromosome.value,
            uri: "",
            label: "chr" + d.chromosome.value
          }
        };
      });
      break;
    case 'idList':
      return Array.from(new Set(results.map(d => d.ensg_id.value))); // unique
      break;
    default:
      return results.map(d => {
        return {
          categoryId: d.chromosome.value,
          label: chrNameToLabel(d.chromosome.value),
          count: Number(d.count.value)
        };
      }).sort((a, b) => chrLabelToNumber(a.label) < chrLabelToNumber(b.label) ? -1 : 1);
  };

  function chrNameToLabel(chr) {
    if(chr === "MT") {
      return "chrM";
    } else {
      return "chr" + chr;
    }
  };

  function chrLabelToNumber(chr) {
    var chrNumber = Number(chr.replace("chr", ""))
    if(chrNumber){
      return chrNumber;
    } else {
      switch(chr){
        case "chrX":
          return 23;
        case "chrY":
          return 24;
        case "chrM":
          return 25;
      }
    }
  };
};
```