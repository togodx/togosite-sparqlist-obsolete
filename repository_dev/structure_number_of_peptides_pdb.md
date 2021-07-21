# PDBエントリをポリペプチドの数で分類(mode対応版)（井手）

## Description
 
- Data sources
    - Number of peptides contained in one PDB entry
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - Number of peptides, PDB id
    - Output
        - The number of PDB entries included in each peptide number
        - If a PDB id is entered, it returns the number of peptides in each PDB entry.

## Parameters

* `categoryIds` -(type:エントリーに含まれるポリペプチドの数の範囲)
  * example: 1-2 (max:125)
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C
* `mode` 
  * example: idList, objectList

## Endpoint

https://integbio.jp/togosite/sparql

## `input_ragne`
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/\s/g, "");
  if (categoryIds.match(/\d-/) || categoryIds.match(/-\d/)) {
    let range = {begin: 1, end: 125};
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    return range;         
  }
  return false;
}
```

## `input_pdb_entries`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
}
```

## `main`

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <https://www.w3.org/2000/01/rdf-schema#>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

{{#if mode}}
SELECT DISTINCT ?peptides_count ?PDBentry ?title
{{else}}
SELECT ?peptides_count (COUNT(?peptides_count) AS ?count)
{{/if}}
WHERE {
  
     {{#if input_ragne}}
     VALUES (?min ?max) {( {{#each input_ragne}} {{this}} {{/each}} )}
     FILTER(?peptides_count>= ?min)
     FILTER(?peptides_count<= ?max)
     {{/if}}
  {
    SELECT (COUNT(?polypeptide) AS ?peptides_count) ?PDBentry {{#if mode}} ?title {{/if}}
    WHERE {
      {{#if input_pdb_entries}}
      VALUES ?PDBentry { {{#each input_pdb_entries}} pdbr:{{this}} {{/each}} }
      {{/if}}
      ?PDBentry a pdbo:datablock ;
          dc:title ?title ;
          pdbo:has_entity_polyCategory ?polypeptideEntity .
      ?polypeptideEntity pdbo:has_entity_poly ?polypeptide .
      ?polypeptide rdfs:seeAlso ?uniprot_link . #DNAのentryを排除
    }
  }
} 
{{#unless mode}}
ORDER BY (?peptides_count)  
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode == "idList") {
    return Array.from(new Set(
      main.results.bindings.map((d) => d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""))
    ));
  } else if (mode == "objectList") {
    return main.results.bindings.map((d) => ({
      id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
      attribute: {
        categoryId: d.peptides_count.value, 
        label: makeLabel(d.peptides_count.value)
      }
    }));
  } else {
    return main.results.bindings.map((d) => ({
      categoryId: d.peptides_count.value,
      label: makeLabel(d.peptides_count.value),
      count: Number(d.count.value)
    }));
  }

  function makeLabel(number) {
    if (number == 1) {
      return `${number} peptide`;
    } else if (number >= 2) {
      return `${number} peptides`;
    } else {
      return number;
    }
  }
}
```