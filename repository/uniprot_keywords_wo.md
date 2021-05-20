# uniprot keywords 内訳 with not annotated proteins 共有SPARQLet（守屋）

* UniProt keyword の内訳
  * ?uniprot up:classifiedWith [keyword ID]
  * [keyword ID] rdfs:subClassOF+ [10 keyword types]
    * 9990: Technical term (1階層 (階層なし))
    * 9991: PTM (最大3階層)
    * 9992: Molecular function (最大5階層)
    * 9993: Ligand (最大5階層)
    * 9994: Domain (最大2階層)
    * 9995: Disease (最大3階層)
    * (9996: Developmental stage (最大3階層) human 無し)
    * 9997: Coding sequence diversity (1階層 (階層なし))
    * 9998: Cellular component (最大4階層)
    * 9999: Biological process (最大8階層)

## Parameters

* `categoryIds` (type: UniProt keyword ID) (Req.)
  * default: 9992
  * example: 9992 (Molecular function), 9998 (Cellular component), 9999 (Biological process), 472 (Membrane)
* `queryIds` (type: UniProt)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `query_array`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/) && queryIds != "undefined") return queryIds.split(/\s+/);
  return false;
}
```

## `category_array`
- UniProt keyword ID を配列に
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) {
    let array = categoryIds.split(/\s+/);
    let categoryArray = [];
    for (let id of array) {
      if (!id.match(/^wo/)) categoryArray.push(id);
    }
    if (categoryArray.length == 0) return false;
    return categoryArray;
  } 
  return false;
}
```

## `category_top_flag`
```javascript
({categoryIds})=>{
  if (categoryIds.match(/^999\d$/)) return true;
  return false;
}
```

## `without_id`
```javascript
({categoryIds})=>{
  if (categoryIds.match(/^999\d$/)) return categoryIds.match(/^(999\d)$/)[1];
  if (categoryIds.match(/wo999\d$/)) return categoryIds.match(/wo(999\d)/)[1];
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `withAnnotation`
- メイン SPARQL
  - 内訳返す場合と遺伝子リスト返す場合を handlbars で条件分岐
  - keyword、タンパク質リストでのフィルタリング
  - ?category a up:Concept でタイプで絞り込まないと遅くなる
  - OPTIONAL で category の下階層、 child_category の有無を確認
    - 有無の確認なので SELECT では SAMPLE で１つだけ出す
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
{{#if mode}}
SELECT DISTINCT ?uniprot ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count) (SAMPLE(?child_category) AS ?child)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/keywords>
WHERE {
{{#if category_array}}
  {{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
  {{/if}}
  {{#if mode}}
    {{#if category_top_flag}}
  VALUES ?parent { {{#each category_array}} keywords:{{this}} {{/each}} }
    {{else}}
  VALUES ?category { {{#each category_array}} keywords:{{this}} {{/each}} }   
    {{/if}}
  {{else}}
  VALUES ?parent { {{#each category_array}} keywords:{{this}} {{/each}} }
  {{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith/rdfs:subClassOf* ?category .
  {{#unless mode}}
  ?category a up:Concept ;
            rdfs:subClassOf ?parent .
  {{/unless}}
  {{#if category_top_flag}}
  ?category a up:Concept ;
            rdfs:subClassOf ?parent .
  {{/if}}
  ?category skos:prefLabel ?label .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  OPTIONAL {
    ?child_category rdfs:subClassOf ?category .
    # taxon:9606 ^up:organism/up:classifiedWith/rdfs:subClassOf ?child_category . # 重い
  }
{{/if}}
}
{{#unless mode}}
ORDER BY DESC (?count)
{{/unless}}
```

## `withoutAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
{{#if mode}}
SELECT DISTINCT ?uniprot ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count) (SAMPLE(?child_category) AS ?child)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/keywords>
WHERE {
{{#if without_id}}
  {{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
  {{/if}}
  VALUES ?parent { keywords:{{without_id}} }
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  MINUS {
    ?uniprot up:classifiedWith/rdfs:subClassOf* ?parent .
  }
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  BIND("wo{{without_id}}" AS ?category)
  BIND("without annotation" AS ?label)
{{/if}}
}
```

## `return`
- 整形
- hasChild に下階層の有無を Boolean で
```javascript
({mode, category_array, withAnnotation, withoutAnnotation, without_id})=>{
  if (!category_array) withAnnotation.results.bindings = [];
  let data = withAnnotation.results.bindings;
  if (without_id) data.concat(withoutAnnotation.results.bindings)
  const idPrfix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/keywords/";
  if (mode == "objectList") return data.map(d=>{
    return {
      id: d.uniprot.value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix, ""), 
        uri: d.category.value,
        label : d.label.value
      }
    }
  });
  if (mode == "idList") return Array.from(new Set(data.map(d=>d.uniprot.value.replace(idPrfix, "")))); // unique

  return data.map(d=>{ 
    return {
      categoryId: d.category.value.replace(categoryPrefix, ""), 
      label: d.label.value,
      count: Number(d.count.value),
      hasChild: Boolean(d.child)
    };
  });	
}
```