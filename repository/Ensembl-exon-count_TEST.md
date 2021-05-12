# exon数ごとにトランスクリプトーム数をカウント（鈴木・八塚）テスト中(1,2,,,に改修をもくろみ中ide)

## Endpoint

 https://integbio.jp/rdf/ebi/sparql

## Parameters

* `categoryIds` exon数を"(最小値)-(最大値)"または特定の数字で指定する。カンマ区切りで複数の範囲を指定できる。
  * example:1, 2-10
* `queryIds` 必須パラメータ。デフォルトは空。数える対象の ID リスト。ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。空白文字かコンマ区切りのリスト
  * example: ENST00000589042,ENST00000342175,ENST00000591111,ENST00000604864,ENST00000341594,ENST00000358025,ENST00000460472
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`

- category ID を配列に分割

```javascript
({categoryIds}) => {
  const RANGE = 10;
  const CG_COUNT = 21;
  const LIMIT = 10000;
  let index=0; //これ追加した。
  let cArray = [];
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)){
    var rArray = categoryIds.split(/\s+/);
    rArray.forEach(d=>{
      var minmax = d.split(/-/);
      if (minmax[1]){
        cArray.push({"min":minmax[0], "max":minmax[1]});
      }else{
        cArray.push({"min":minmax[0], "max":minmax[0]});
      }
      index++;
    });
  }else{
    cArray.push(  {"min": 1, "max": 1},
                  {"min": 2, "max": 2},
                  {"min": 3, "max": 3},
                  {"min": 4, "max": 4},
                  {"min": 5, "max": 5},
                  {"min": 6, "max": 6},
                  {"min": 7, "max": 7},
                  {"min": 8, "max": 8},
                  {"min": 9, "max": 9},
                  {"min": 10, "max": 10},
                  {"min": 11, "max": 15},
                  {"min": 16, "max": 20},
                  {"min": 21, "max": 50},
                  {"min": 51, "max": 100},
                  {"min": 101, "max": 200},
                  {"min": 201, "max": 400},
                  {"min": 401, "max": 1000},
                  {"min": 1001,"max": 10000});
  }
  return cArray;
}
```

## `query`
```sparql
# @endpoint https://integbio.jp/rdf/ebi/sparql
PREFIX obo:<http://purl.obolibrary.org/obo/>
PREFIX taxon:<http://identifiers.org/taxonomy/>
PREFIX enst:<http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX terms:<http://rdf.ebi.ac.uk/terms/ensembl/>

{{#if mode}}
SELECT DISTINCT ?ens_id ?bin_id
{{else}}
SELECT ?bin_id (COUNT(DISTINCT ?ens) AS ?count)
{{/if}}       
WHERE {
  VALUES (?bin_id ?min ?max) { {{#each categoryArray}}("{{this.min}}-{{this.max}}" {{this.min}} {{this.max}}) {{/each}} }
  FILTER(?min <= ?exon_count &&  ?exon_count <= ?max){
     SELECT ?ens (COUNT(DISTINCT ?exon) AS ?exon_count)
      WHERE {
        {{#if queryArray}}
          VALUES ?ens { {{#each queryArray}} enst:{{this}}{{/each}} }
        {{/if}}
        VALUES ?func {terms:miRNA terms:lncRNA terms:protein_coding terms:pseudogene}
        ?ens obo:RO_0002162 taxon:9606;
              obo:SO_has_part ?exon;
              a ?func.
    }GROUP BY ?ens
  }
{{#if mode}}
  BIND(REPLACE(STR(?ens), "http://rdf.ebi.ac.uk/resource/ensembl.transcript/", "") AS ?ens_id)
{{/if}}  
}
{{#unless mode}}
GROUP BY ?bin_id
ORDER BY ?bin_id
{{/unless}}
```

## `results`

```javascript
({mode, query})=>{
    if (mode == "idList"){
        return query.results.bindings.map(d=>{
          return d.ens_id.value;
        });
    }else if (mode == "objectList"){
      return query.results.bindings.map(d=>{
        return {
          id: d.ens_id.value, 
          attribute: {
            categoryId: d.bin_id.value, 
            label : d.bin_id.value
          }
        }
      });
    }else{
      let rArray = query.results.bindings.map(d=>{
        var range = d.bin_id.value.split("-");
        return {
          id: d.bin_id.value,
          min: range[0],
          label: makeLabel(range[0], range[1]),
          count: Number(d.count.value)
        }
      });
      rArray.sort(function(a, b){
        var aValue = parseInt(a.min);
        var bValue = parseInt(b.min);
        if(aValue < bValue) return -1;
      	if(aValue > bValue) return 1;
      	return 0;
      });
      
      return rArray.map(d=>{
        return {
          id: d.id,
          label: d.label,
          count: Number(d.count)
        }
      });
      
      function makeLabel(num1, num2) {
         if (num1 == num2) {
           return num1;
         } else {
           return num1 + "-" + num2 ;
         }
      }
  
};
}
```