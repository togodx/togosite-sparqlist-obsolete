# uniprotのエントリを膜貫通回数で分類 (井手, 守屋)

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The number of transmembrane site

## Parameters
* `categoryIds` (type: 膜貫通部位数)
  * example: 4-7 -5 20-
* `queryIds` (type: uniprot)
  * example: Q5VV42,Q9BSA9,Q12884,P04233,O15162,O00322,P16070,O75844,Q9BXK5,Q12983,P08195,Q496J9,P63027,P51681,P58335,Q9Y5U4,P12830,P08581,Q96NB2,O75746,Q9Y548
* `mode`
  * example: idList, objectList

## `queryArray`

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `filter`
```javascript
({mode, categoryIds})=>{
  if (!mode || !categoryIds) return "";
  else if (categoryIds.match(/^[^-]+$/)) {
    let filters = categoryIds.split(/,/).map(d=>{ return "?target_num = " + d });
　  return "FILTER( " + filters.join(" || ") + " )";
  } else {
    let filters = [];
    if (categoryIds.match(/^[\d\.]+-/)) filters.push("?target_num >= " + categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) filters.push("?target_num <= " + categoryIds.match(/-([\d\.]+)$/)[1]);
    return "FILTER( " + filters.join(" && ") + " )";
  }
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `withTarget`

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT ?target_num (COUNT(DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {
	SELECT ?uniprot (COUNT(DISTINCT ?annotation) AS ?target_num)
	WHERE {
        {{#if queryArray}}
        VALUES ?uniprot { {{#each queryArray}} upid:{{this}} {{/each}} }
        {{/if}}        
	  	?uniprot a up:Protein ;
  		         up:organism taxon:9606 ;
  		         up:proteome ?proteome ;
  		         up:annotation ?annotation .       
  		?annotation rdf:type up:Transmembrane_Annotation .
  	FILTER(REGEX(STR(?proteome), "UP000005640"))
    }
  }
  {{filter}}
}    
{{#unless mode}}                      
ORDER BY ?target_num
{{/unless}}
```

## `zero_check`
```javascript
({categoryIds})=>{
  if (!categoryIds || categoryIds.match(/^0*-\d/) || categoryIds.split(/,/).includes("0")) return true;
  return false;
}
```

## `withoutTarget`
- 膜貫通部位を持たないタンパク質の数
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT (COUNT(DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
{{#if zero_check}}
  {{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} upid:{{this}} {{/each}} }
  {{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  MINUS {
    ?uniprot up:annotation [ a up:Transmembrane_Annotation ] .
  }
  BIND ("0" AS ?target_num)
{{/if}}
}
```

## `results`

```javascript
({mode, queryIds, categoryIds, withTarget, withoutTarget})=>{
  // renge
  let range = {begin: 0, end: Infinity};
  let cids = false;
  if (categoryIds) {
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    if (categoryIds.match(/^[^-]+$/)) {
      cids = categoryIds.split(/,/).reduce((obj, a)=>{
        obj[Number(a)] = true;
        return obj;
      }, {});
    }
  }
  
  if (mode) {
    const idVarName = "uniprot";
    const idPrefix = "http://purl.uniprot.org/uniprot/";
    // add without-binding-site-data
    if (withoutTarget.results.bindings[0] && withoutTarget.results.bindings[0][idVarName]) withTarget.results.bindings = withTarget.results.bindings.concat(withoutTarget.results.bindings);
    let filteredData = [];
    if (categoryIds) {
      for (let d of withTarget.results.bindings) {
        const num = Number(d.target_num.value);
        if ((!cids && range.begin <= num && num <= range.end)
            || (cids[num])) {
          filteredData.push(d);
        }
      }
    } else filteredData = withTarget.results.bindings;
    if (mode == "objectList") return filteredData.map(d=>{
      return {
        id: d[idVarName].value.replace(idPrefix, ""),
        attribute: {categoryId: d.target_num.value, label: d.target_num.value}
      }
    });
    if (mode == "idList") return filteredData.map(d=>d[idVarName].value.replace(idPrefix, ""));
  }

  if (!queryIds || withoutTarget.results.bindings[0].count.value != 0) {
    withTarget.results.bindings.unshift( {count: {value: withoutTarget.results.bindings[0].count.value}, target_num: {value: "0"}}  ); // カウント 0 を追加
  }
  let value = 0;
  let res = [];
  for (let d of withTarget.results.bindings) {
    const num = Number(d.target_num.value);
    if (value < num) {
      // fill missing value by 0
      for (let emptyValue = value; emptyValue < num; emptyValue++) {
        if ((!queryIds) 
            && ((!categoryIds) || ((!cids && range.begin <= emptyValue && emptyValue <= range.end) || (cids[emptyValue])))) {
        	res.push( { categoryId: emptyValue.toString(), label: emptyValue.toString(), count: 0} );
        }
      }
    }
    value = num + 1;
    if ((!categoryIds)
      || ((!cids && range.begin <= num && num <= range.end) || (cids[num]))) {
      res.push( { categoryId: d.target_num.value, label: d.target_num.value, count: Number(d.count.value)} );
    }
  }       
  return res;
}
```