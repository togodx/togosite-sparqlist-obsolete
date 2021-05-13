# uniprot pdb-existence from uniprot（守屋）

- PDB エントリー(立体構造)の有無の内訳を返す

## Parameters

* `categoryIds` (type: PDB entry existence)
  * example: 1 (exists), 0 (not exists)
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `query_array`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `withTarget`
- メイン SPARQL（PDB 有り）
  - 内訳返す場合と遺伝子リスト返す場合を handlebars で条件分岐
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://purl.uniprot.org/database/>
{{#if mode}}
SELECT DISTINCT ?uniprot
{{else}}
SELECT (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
{{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           rdfs:seeAlso/up:database db:PDB .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `withoutTarget`
- メイン SPARQL（PDB 無し）
  - 同時に取ると遅いので有る無しでわけた

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://purl.uniprot.org/database/>
{{#if mode}}
SELECT DISTINCT ?uniprot
{{else}}
SELECT (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
{{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  MINUS {
    ?uniprot rdfs:seeAlso/up:database db:PDB . 
  }
}
```

## `return`
- 集計

```javascript
({categoryIds, mode, withTarget, withoutTarget})=>{
  const targetVarName = "uniprot";
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withId = "1";
  const withoutId = "0";
  const withLabel = "Proteins with structure data";
  const withoutLabel = "Proteins without structure data";
  if (mode) {
    if (mode == "objectList") {
      if (categoryIds == withId) return withTarget.results.bindings.map(d=>{
        return {
          id: d[targetVarName].value.replace(idPrefix, ""),
          attribute: {categoryId: withId, label: withLabel}
        }
      });
      if (categoryIds == withoutId) return withoutTarget.results.bindings.map(d=>{
        return {
          id: d[targetVarName].value.replace(idPrefix, ""),
          attribute: {categoryId: withoutId, label: withoutLabel}
        }
      });
      return withTarget.results.bindings.map(d=>{
        return {
          id: d[targetVarName].value.replace(idPrefix, ""),
          attribute: {categoryId: withId, label: withLabel}
        }
      }).concat(withoutTarget.results.bindings.map(d=>{
        return {
         id: d[targetVarName].value.replace(idPrefix, ""),
          attribute: {categoryId: withoutId, label: withoutLabel}
        }
      }));
    }
    if (mode == "idList") {
      if (categoryIds == withId) return withTarget.results.bindings.map(d=>d[targetVarName].value.replace(idPrefix, ""));
      if (categoryIds == withoutId) return withoutTarget.results.bindings.map(d=>d[targetVarName].value.replace(idPrefix, ""));
      return withTarget.results.bindings.map(d=>d[targetVarName].value.replace(idPrefix, "")).concat(withoutTarget.results.bindings.map(d=>d[targetVarName].value.replace(idPrefix, "")));  
    }
  }
  return [
    {
      categoryId: withId,
      label: withLabel,
      count: Number(withTarget.results.bindings[0].count.value)
    },
    {
      categoryId: withoutId,
      label: withoutLabel,
      count: Number(withoutTarget.results.bindings[0].count.value)
    }
  ];
}
```