# ChIP-Atlas の転写因子を GO MF で分類(池田)

uniprot GO 共有 SPARQLet を流用

- 指定 GO の1階層下の内訳
  - すでに最下層で、下層が無い場合は空配列を返す

## Parameters

* `categoryIds` (type: go) (Req.)
  * default: GO_0008150
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function), ... 
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `queryArray`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- UniProt keyword ID を配列に
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) {
    let array = categoryIds.split(/\s+/);
    let categoryArray = [];
    for (let id of array) {
      if (!id.match(/^wo_GO_\d+/)) categoryArray.push(id);
    }
    if (categoryArray.length == 0) return false;
    return categoryArray;
  } 
  return false;
}
```

## `withoutId`
```javascript
({categoryIds})=>{
  if (categoryIds == "GO_0008150" || categoryIds == "GO_0005575" || categoryIds == "GO_0003674" ) return categoryIds;
  if (categoryIds.match(/wo_GO_\d+/)) return categoryIds.match(/wo_(GO_\d+)/)[1];
  return false;
}
```

## `category_top_flag`
```javascript
({categoryIds})=>{
  if (categoryIds == "GO_0008150" || categoryIds == "GO_0005575" || categoryIds == "GO_0003674") return true;
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `targetGo`
- 欲しい GO の階層を取る
  - UniProt エンドポイントのGO階層が展開されてしまってるため
  
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?go (SAMPLE(?child_category) AS ?child)
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE
{
{{#if mode}}
  {{#if category_top_flag}}
  VALUES ?go_list { {{#each categoryArray}} obo:{{this}} {{/each}} }
  ?go rdfs:subClassOf ?go_list .
  {{else}}
  VALUES ?go { {{#each categoryArray}} obo:{{this}} {{/each}} }
  {{/if}}
{{else}}
  VALUES ?go_list { {{#each categoryArray}} obo:{{this}} {{/each}} }
  ?go rdfs:subClassOf ?go_list .
{{/if}}
  OPTIONAL {
    ?child_category rdfs:subClassOf ?go .
    # taxon:9606 ^up:organism/up:classifiedWith/rdfs:subClassOf ?child_category . # 重い
  }
}
```

## `targetGoArray`
- 単純な配列に

```javascript
({targetGo}) => {
 return targetGo.results.bindings.map(d => d.go.value.replace("http://purl.obolibrary.org/obo/", ""));
}
```

## Endpoint
https://integbio.jp/togosite/sparql
 
## `withAnnotation`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlebars で条件分岐
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
{{#if mode}}
SELECT DISTINCT ?tf_ensg ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?tf_ensg) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
FROM <http://rdf.integbio.jp/dataset/togosite/chip_atlas>
FROM <http://rdf.integbio.jp/dataset/togosite/togoid/ensembl_gene-uniprot>
WHERE {
{{#if categoryArray}}
  {{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
  {{else}}
  ?tf_ensg obo:RO_0002428 ?target .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid/ensembl_gene-uniprot> {
    ?tf_ensg obo:RO_0002205 ?uniprot .
  }
  {{/if}}
  VALUES ?category { {{#each targetGoArray}} obo:{{this}} {{/each}} }
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  ?uniprot up:classifiedWith/rdfs:subClassOf* ?category .
  ?category rdfs:label ?label .
{{/if}}
}
{{#unless mode}}
ORDER BY DESC(?count)
{{/unless}}
```

- あるGOカテゴリを持たないUniProtを１つのSPARQLで取ろうとするとメモリオーバーするので変則的

## `allUniProt`
- UniProts without GO annotation 
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?uniprot
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
{{#if withoutId}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
{{/if}}
}
```

## `withGoUniProt`
- UniProts without GO annotation 
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?uniprot
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
{{#if withoutId}}
  {
    SELECT DISTINCT ?go
    FROM <http://rdf.integbio.jp/dataset/togosite/go>
    WHERE {
      ?go rdfs:subClassOf+ obo:{{withoutId}} .
    }
  }
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  ?uniprot up:classifiedWith ?go .
{{/if}}
}
```

## `withoutAnnotation`
```javascript
({mode, queryArray, allUniProt, withGoUniProt, withoutId}) => {
  if (!withoutId) return {results: {bindings: []}};
  let withGo = {};
  for (let d of withGoUniProt.results.bindings) {
    withGo[d.uniprot.value] = true;
  }
  let query = {};
  if (queryArray) {
    for (let d of queryArray) {
      query["http://purl.uniprot.org/uniprot/" + d] = true;
    }
  }
  let bindings = [];
  if (mode == "objectList") {
    for (let d of allUniProt.results.bindings) {
      if (!withGo[d.uniprot.value] && (!queryArray || (queryArray && query[d.uniprot.value]))) {
        bindings.push({
          uniprot: {value: d.uniprot.value},
          category: {value: "wo_" + withoutId},
          label: {value: "without annotation"}
        });
      }
    }
    return {results: {bindings: bindings}};
  }
  for (let d of allUniProt.results.bindings) {
    if (!withGo[d.uniprot.value] && (!queryArray || (queryArray && query[d.uniprot.value]))) {
      bindings.push({uniprot: {value: d.uniprot.value}})
    }
  }
  if (mode == "idList") return {results: {bindings: bindings}};
  let count = bindings.length;
  bindings = [{
          count: {value: count},
          category: {value: "wo_" + withoutId},
          label: {value: "without annotation"}
        }];
  return {results: {bindings: bindings}};
}
```

## `return`
- 存在レベル、タンパク質リストでのフィルタリング
```javascript
({mode, categoryArray, withoutId, withAnnotation, withoutAnnotation, targetGo}) => {
  const idVar = "uniprot";
  const idPrfix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  let data = [];
  if (categoryArray) data = withAnnotation.results.bindings;
  if (withoutId) data = data.concat(withoutAnnotation.results.bindings);
  if (mode == "objectList") return data.map(d => {
    return {
      id: d[idVar].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix, ""), 
        uri: d.category.value,
        label : d.label.value
      }
    }
  });
  if (mode == "idList") return data.map(d => d[idVar].value.replace(idPrfix, ""));
  let hasChild = {};
  for (let d of targetGo.results.bindings) {
    if (d.child) hasChild[d.go.value] = true;
  }
  return data.map(d => { 
    let obj = {
      categoryId: d.category.value.replace(categoryPrefix, ""), 
      label: d.label.value,
      count: Number(d.count.value)
    }
    if (hasChild[d.category.value]) obj.hasChild = true;
    return obj;
  });	
}
```