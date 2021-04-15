# exon数ごとにトランスクリプトーム数をカウント(フロント開発用 moriya)

## Parameters

* `categoryId` デフォルトは空でも良い。SPARQLクエリに必要な場合はルートノードの ID が初期値。階層がある場合は中間階層のノードの ID も受け付け、指定ノードの一つ下位階層の内訳を返す。階層も無くクエリにも必要ないならオプショナルパラメータ
  * example: 20-
* `queryIds` 必須パラメータ。デフォルトは空。数える対象の ID リスト。ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。空白文字かコンマ区切りのリスト
  * example: ENST00000589042,ENST00000342175,ENST00000591111,ENST00000604864,ENST00000341594,ENST00000358025,ENST00000460472

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint

 https://integbio.jp/rdf/ebi/sparql
 
## `exon_count`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
SELECT ?exon_count (COUNT (DISTINCT ?enst) AS ?count)
WHERE{
  {
    SELECT ?enst (COUNT (DISTINCT ?exon) AS ?exon_count)
    WHERE {
      {{#if queryArray}}
      VALUES ?enst { {{#each queryArray}} enst:{{this}} {{/each}} }
      {{/if}}
  	  ?enst obo:RO_0002162 taxon:9606 ;
            obo:SO_has_part ?exon;
            a ?func .
      FILTER (?func = <http://rdf.ebi.ac.uk/terms/ensembl/miRNA> || ?func = <http://rdf.ebi.ac.uk/terms/ensembl/lncRNA> ||
              ?func = <http://rdf.ebi.ac.uk/terms/ensembl/protein_coding> || ?func = <http://rdf.ebi.ac.uk/terms/ensembl/pseudogene>)
    } GROUP BY ?enst
  }
}
ORDER BY ?exon_count
```

## `results`

```javascript
({exon_count, categoryId})=>{
  let limit_1 = 20;
  let limit_2 = 100;
  let bin_2 = 10;
  if (!categoryId) {
    let res = [];
    for (let d of exon_count.results.bindings) {
      let num = Number(d.exon_count.value);
      if (num < limit_1) res.push( { categoryId: d.exon_count.value, label: d.exon_count.value, count: Number(d.count.value)} );
      else if (num >= limit_1 && res.length < limit_1) res.push( { categoryId: limit_1 + "-", label: limit_1 + "-", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else if (categoryId == limit_1 + "-") {
    let res = [];
    for (let d of exon_count.results.bindings) {
      let num = Number(d.exon_count.value);
      if (num < limit_1) continue;
      if (num < limit_2 && res.length <= (num - limit_1) / bin_2) res.push( { categoryId: d.exon_count.value + "-" + (Number(d.exon_count.value) + 9), label:  d.exon_count.value + "-" + (Number(d.exon_count.value) + 9), count: Number(d.count.value), hasChild: true} );
      else if (num >= limit_2 && res.length <= (limit_2 - limit_1) / bin_2) res.push( { categoryId: limit_2 + "-", label: limit_2 + "-", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else {
    let range = categoryId.split(/-/);
    let res = [];
    for (let d of exon_count.results.bindings) {
      let num = Number(d.exon_count.value);
      if (num < Number(range[0]) || (range[1] && num > Number(range[1]))) continue;
      res.push( { categoryId: d.exon_count.value, label:  d.exon_count.value, count: Number(d.count.value)} );
    }
    return res;
  }
}
```