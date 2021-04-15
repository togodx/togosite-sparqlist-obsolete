# PubChem で分子量によりフィルタ（FDA Approved Drugs） (mode対応修正途中）
## Todo
- max, minが指定されていない場合の区分分け
- max,minのペアを複数指定

## Parameters

* `maxWeight`  最大値 空なら無制限
  * default: 
  * example: 1000
* `minWeight` 最小値 空なら無制限
  * default: 
  * example: 100
* `queryIds` 必須パラメータ。デフォルトは空。Pubchem Compound ID (番号のみ）のリスト、なければ FDA Approved Drugs であるもの全部
  * example: 2793, 6037, 4724, 37632, 4823, 5217, 54120, 42008, 10235, 17676, 5523, 4116
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList
   
## Endpoint

https://integbio.jp/rdf/pubchem/sparql


## `maxW`
- 数値かどうか判定
```javascript
({maxWeight}) => {
  if (isNaN(maxWeight)) {
  	return false;
  } else {
    return Number(maxWeight)
  }
}
```

## `minW`
- 数値かどうか判定
```javascript
({minWeight}) => {
  if (isNaN(minWeight)) {
  	return false;
  } else {
    return Number(minWeight)
  }
}
```

## `queryArray`
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `molecularWeight`

```sparql
## https://integbio.jp/rdf/pubchem/sparql
## FDA approved drugs -> molecular weight


PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>

PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT distinct ?mw ?cid 
WHERE {
  {{#if queryArray}}
    VALUES ?cid { {{#each queryArray}} compound:{{this}} {{/each}} }
  {{else}}
    ?cid      obo:has-role pubchemv:FDAApprovedDrugs .
  {{/if}} 
    ?cid      sio:has-attribute ?attr.  
    ?attr a sio:CHEMINF_000334 . 
    ?attr sio:has-unit ?unit;
          sio:has-value ?mw.
  {{#if maxW}} FILTER(?mw <= {{maxW}}) {{/if}}  
  {{#if minW}} FILTER(?mw >= {{minW}}) {{/if}}
} 
```


## `return`
```javascript
({molecularWeight,maxW,minW,mode})=>{
  const prefix="http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID"
  var list = molecularWeight.results.bindings;
  var r = [];
  var tmp = 0;
  var maxS = maxW;
  var minS = minW;
  if (!maxW){
    maxW=Infinity;
    maxS="∞";
  }
  if (!minW) {
    minW=0;
  }
  
  if(mode==="idList") {
    for(var i = 0; i < list.length; i++){	
      var c = list[i].cid.value.replace(prefix,""); 
      var m = list[i].mw.value
      if (minW <= m && m <= maxW) {
        r.push(c);
      }
    }
    return r;
  } else if (mode==="objectList") {
     for(var i = 0; i < list.length; i++){	
      var c = list[i].cid.value.replace(prefix,""); 
      var m = list[i].mw.value
      var s = minS+"-"+maxS
      if (minW <= m && m <= maxW) {
        r.push(
            {
              id: c,
    		  attribute: {
      			categoryId: s,
      			label: s,
      			molecularWeight: m
              }
              
    		}
        );
      }
    }   
    return r;
  } else {  
    var s = minS+"-"+maxS
    for(var i = 0; i < list.length; i++){	
      var m = list[i].mw.value
      if (minW <= m && m <= maxW) {
        tmp++;
      }  
    }
    return (
      { 
        categoryId:s,
        count:tmp
      }
    )
  }
};	
```

