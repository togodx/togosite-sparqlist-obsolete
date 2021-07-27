# PDBエントリに含まれる非タンパク質で分類(ヒトのみ)（井手）

## Description
 
- Data sources
    - Non-peptide molecules contained in the 3D structure in the PDB entry
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - Non-peptide molecules, PDB id
    - Output
        - The number of PDB entries included in each non-peptide molecule
        - If a PDB id is entered, it returns the non-peptide molecule in each PDB entry.

## Parameters

* `categoryIds`
  * example: ATP,ZN
* `queryIds` (type: PDB)
  * example: 1NLV,1FJS,1FPC,6BNR
* `mode` 
  * example: idList, objectList

## `input_pdb`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
}
```

## `input_category`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/\s+/, "");
  if (categoryIds) {
    return categoryIds.split(/,/);
  }
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxid: <https://identifiers.org/taxonomy/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdb: <https://rdf.wwpdb.org/pdb/>

{{#if mode}}
SELECT DISTINCT ?pdb ?comp_id ?name
{{else}}
SELECT (COUNT(DISTINCT ?pdb) AS ?count) ?comp_id ?name
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
WHERE {
  {{#if input_category}}
  VALUES ?comp_id { {{#each input_category}} "{{this}}" {{/each}} }
  {{/if}}
  ?pdb a pdbo:datablock ;
      pdbo:has_entityCategory/pdbo:has_entity/rdfs:seeAlso taxid:9606 ;
      pdbo:has_pdbx_entity_nonpolyCategory/pdbo:has_pdbx_entity_nonpoly ?nonpoly .
  ?nonpoly
      pdbo:pdbx_entity_nonpoly.comp_id ?comp_id ;
      pdbo:pdbx_entity_nonpoly.name ?name .
  {{#if input_pdb}} VALUES ?pdb { {{#each input_pdb}} pdb:{{this}} {{/each}} } {{/if}}
}
{{#unless mode}}
order by DESC(?count)
LIMIT 100
{{/unless}} 
```

## `total_count`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdb: <https://rdf.wwpdb.org/pdb/>

SELECT (COUNT(?pdb) AS ?count)
FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
WHERE {
  {{#if input_pdb}} VALUES ?pdb { {{#each input_pdb}} pdb:{{this}} {{/each}} } {{/if}}
  ?pdb a pdbo:datablock ;
      pdbo:has_pdbx_entity_nonpolyCategory/pdbo:has_pdbx_entity_nonpoly ?nonpoly .
  ?nonpoly
      pdbo:pdbx_entity_nonpoly.comp_id ?comp_id ;
      pdbo:pdbx_entity_nonpoly.name ?name .
  {{#if input_category}}
  VALUES ?comp_id { {{#each input_category}} "{{this}}" {{/each}} }
  {{/if}}
  {
    SELECT DISTINCT ?pdb
    WHERE {
      ?pdb pdbo:has_entityCategory/pdbo:has_entity/rdfs:seeAlso taxid:9606 .
    }
  }
}
```

## `results`

```javascript
({ mode, main, total_count }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.pdb.value.replace("https://rdf.wwpdb.org/pdb/", ""))
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({ 
      id: elem.pdb.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
      attribute: {
        categoryId: elem.comp_id.value,
        label: makeLabel(elem)
      }
    }));
  } else {
    const total = total_count.results.bindings[0].count.value;
    let sum = 0;
    let arr = [];
    main.results.bindings.forEach((elem) => {
      arr.push({
        categoryId: elem.comp_id.value,
        label: makeLabel(elem),
        count: Number(elem.count.value)
      });
      sum += Number(elem.count.value);
    });
    if (total - sum > 0) {
      arr.push({
        categoryId: "_other",
        label: "Other molecules",
        count: total - sum
      });
    }
    return arr;
  }

  function makeLabel(elem) {
    let label = capitalize(elem.name.value)
        .replace('(iii)', '(III)').replace('(ii)', '(II)').replace('ix', 'IX')
        .replace('(2r)', '(2R)').replace('(4r)', '(4R)').replace('(4s)', '(4S)').replace('(9z)', '(9Z)').replace('(n-', '(N-')
        .replace('-l-', '-L-').replace(/^Nadh /, 'NADH ').replace(/^Nadph /, 'NADPH ').replace(/ fe$/, ' Fe').replace(/^Fe2\/s2/, 'Fe2/S2');
    if (label.length <= 27) {
      return label;
    } else {
      return `${elem.comp_id.value}: ` + label.substr(0, 27) + '...';
    }
  }
  function capitalize(s) {
    return s.charAt(0).toUpperCase() + s.substring(1).toLowerCase();
  }
}
```