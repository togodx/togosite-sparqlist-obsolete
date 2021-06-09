# variant consequence　（三橋）

## Description

- Data sources
    -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
    -  Input
        - TogoVar id
    - Output
        -  [Variant consequence calculated with Variant Effect Predictor (VEP)](https://asia.ensembl.org/info/genome/variation/prediction/predicted_data.html#consequences) in terms of [Sequence ontology](http://www.sequenceontology.org/)

## testURL
  - [default](https://integbio.jp/togosite/sparqlist/api/variant_consequence?categoryIds=SO_0001060&queryIds=&mode=)
  - [categoryId+objectList](https://integbio.jp/togosite/sparqlist/api/variant_consequence?categoryIds=SO_0001060&queryIds=&mode=objectList)
  - [catgoryId+queryId+idList](https://integbio.jp/togosite/sparqlist/api/variant_consequence?categoryIds=SO_0001060&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=idList)
  - [catgoryId+queryId+objectList](https://integbio.jp/togosite/sparqlist/api/variant_consequence?categoryIds=SO_0001060&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=objectList)

## Parameters

* `categoryIds` (type: variant consequence)
  * default: 
  * example: SO_0001060 (sequence_variant)  http://www.sequenceontology.org/browser/current_svn/term/SO:0001060
* `queryIds` (type:TogoVar)
  * example: tgv48208871,tgv48208872,tgv48208877,tgv48208884,tgv48208888,tgv48208896,tgv48208897,tgv48208898,tgv48208900,tgv48208904,tgv48208906,tgv48208908,tgv48208938,tgv48208939,tgv48208943,tgv48208945,tgv48208947,tgv48208962,tgv48208965,tgv48208967,tgv48208968,tgv48208970,tgv48208971,tgv48208973,tgv48208977
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

## `isCountAll`
- 全部をカウントする(default)場合は静的JSONをレスポンスする。そのフラグを判定する。

```javascript
({categoryIds,queryIds,mode}) => {
  queryIds = queryIds.replace(/,/g," ");
  if (categoryIds.match(/\S/) || queryIds.match(/\S/) || mode.match(/\S/)) return false;
//  if (queryIds.match(/\S/) || mode.match(/\S/)) return false;
  return true;
}
```

## `queryArray`
- Query togovar_id を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- ユーザが指定したカテゴリ ID リストを配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX cvo:  <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>

{{#if isCountAll}}
SELECT "true" AS ?isCountAll
WHERE {  FILTER(1 = 1) }
{{else}}
{{#if mode}}
SELECT DISTINCT ?tgv_id ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?tgv_id) AS ?count) 
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/so>
WHERE {  
{{#if queryArray}}
  VALUES ?tgv_id { {{#each queryArray}} "{{this}}" {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} obo:{{this}} {{/each}} }     
  {{else}}
  VALUES ?parent { {{#each categoryArray}} obo:{{this}} {{/each}} }
  {{/if}} 
{{/if}}
   ?togovar dct:identifier ?tgv_id.
   ?togovar tgvo:hasConsequence/rdf:type/rdfs:subClassOf* ?category.
 {{#unless  mode}}
   ?category rdfs:subClassOf ?parent.
 {{/unless}}
   ?category rdfs:label ?label.
}
{{#if mode}}
LIMIT 10000
{{else}}  
ORDER BY DESC(?count)
{{/if}}
{{/if}}
```

## `return`
- 整形
```javascript
({mode, data, isCountAll})=>{
  const idVarName = "tgv_id";
  const idPrfix = "";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  
  if (isCountAll == true) {
  return  [
    {"categoryId":"SO_0001627","count":212366571,"label":"Intron variant","hasChild":false},
    {"categoryId":"SO_0001619","count":52390424,"label":"Non coding transcript variant","hasChild":false},
    {"categoryId":"SO_0001628","count":38329053,"label": "Intergenic variant","hasChild":false},
    {"categoryId":"SO_0001621","count":20962310,"label": "NMD transcript variant","hasChild":false},
    {"categoryId":"SO_0001583","count":12385885,"label": "Missense variant","hasChild":false},
    {"categoryId":"SO_0001792","count":7450628,"label": "Non coding transcript exon variant","hasChild":false},
    {"categoryId":"SO_0001819","count":6340518,"label": "Synonymous variant","hasChild":false},
    {"categoryId":"SO_0001624","count":4109311,"label": "3 prime UTR variant","hasChild":false},
    {"categoryId":"SO_0001630","count":2785532,"label": "Splice region variant","hasChild":false},
    {"categoryId":"SO_0001623","count":1546374,"label": "5 prime UTR variant","hasChild":false},
    {"categoryId":"SO_0001589","count":506417,"label": "Frameshift variant","hasChild":false},
    {"categoryId":"SO_0001587","count":402831,"label": "Stop gained","hasChild":false},
    {"categoryId":"SO_0001575","count":210311,"label": "Splice donor variant","hasChild":false},
    {"categoryId":"SO_0001574","count":172987,"label": "Splice acceptor variant","hasChild":false},
    {"categoryId":"SO_0001822","count":143302,"label": "Inframe deletion","hasChild":false},
    {"categoryId":"SO_0001821","count":50924,"label": "Inframe insertion","hasChild":false},
    {"categoryId":"SO_0002012","count":38719,"label": "Start lost","hasChild":false},
    {"categoryId":"SO_0001580","count":18057,"label": "Coding sequence variant","hasChild":false},
    {"categoryId":"SO_0001578","count":16247,"label": "Stop lost","hasChild":false},
    {"categoryId":"SO_0001567","count":7754,"label": "Stop retained variant","hasChild":false},
    {"categoryId":"SO_0001620","count":6871,"label": "Mature miRNA variant","hasChild":false},
    {"categoryId":"SO_0001818","count":3482,"label": "Protein altering variant","hasChild":false},
    {"categoryId":"SO_0001626","count":3443,"label": "Incomplete terminal codon variant","hasChild":false},
    {"categoryId":"SO_0002019","count":585,"label": "Start retained variant","hasChild":false},
    {"categoryId":"SO_0001893","count":18,"label": "Transcript ablation","hasChild":false}
  ];
}

  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix,""),
        uri: d.category.value,
        label : d.label.value.replace(/_/g, " ").charAt(0).toUpperCase() + d.label.value.replace(/_/g, " ").slice(1)   // 先頭の１文字だけを大文字にする。
      }
    }
  });
  
  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix, "")))); // unique

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace(categoryPrefix,""),
      label : d.label.value.replace(/_/g, " ").charAt(0).toUpperCase() + d.label.value.replace(/_/g, " ").slice(1),   // 先頭の１文字だけを大文字にする。
      count: d.count.value,
      hasChild: (Number(d.count.value) > 1 ? true : false)
    };
  });	
}
```