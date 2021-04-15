# uniprot go (GOの代わりに uniprot_keywords 9992,9998,9999 を使うので使わない)

- 指定 GO の1階層下の内訳
  - すでに最下層で、下層が無い場合は空配列を返す

## Parameters

* `categoryId` (type: go) (Req.)
  * default: GO_0008150
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function), ... 
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: list

## `query_array`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `category_array`
- UniProt keyword ID を配列に
```javascript
({categoryIOd})=>{
  categoryId = categoryId.replace(/,/g, " ");
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/rdf/mirror/ebi/sparql
- GO を展開していない endpoit

## `target`
- 欲しい GO の階層を取る
  - UniProt エンドポイントのGO階層が展開されてしまってるため
  
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?go
FROM <http://rdf.ebi.ac.uk/dataset/go>
WHERE
{
{{#if mode}}
  VALUES ?go { {{#each category_array}} obo:{{this}} {{/each}} }
{{else}}
  VALUES ?go_list { {{#each category_array}} obo:{{this}} {{/each}} }
  ?go rdfs:subClassOf ?go_list .
{{/if}}
}
```

## `target_go_array`
- 単純な配列に

```javascript
({target}) => {
 return target.results.bindings.map(d => d.go.value.replace("http://purl.obolibrary.org/obo/", ""));
}
```

## Endpoint
https://integbio.jp/rdf/mirror/uniprot/sparql

## `main`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlebars で条件分岐
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
{{#if mode}}
SELECT DISTINCT ?uniprot
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
WHERE {
{{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
{{/if}}
  VALUES ?category { {{#each target_go_array}} obo:{{this}} {{/each}} }
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  { 
    ?uniprot up:classifiedWith ?category .
  } UNION {
    ?uniprot up:classifiedWith/rdfs:subClassOf ?category .
  }
{{#if mode}}
}
{{else}}
  ?category rdfs:label ?label .
}
ORDER BY DESC(?count)
{{/if}}
```

## `return`
- 存在レベル、タンパク質リストでのフィルタリング
```javascript
({mode, main})=>{
  if (mode) return main.results.bindings.map(d=>d.uniprot.value.replace("http://purl.unipriot.org/uniprot/", ""));
  return main.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace("http://purl.obolibrary.org/obo/", ""), 
      label: d.label.value,
      count: d.count.value
    };
  });	
}
```