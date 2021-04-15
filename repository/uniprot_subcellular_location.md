# uniprot subcellular location (細胞内局在は uniprot_keywords 9998 を使うので使わない)

* 細胞内局在の内訳
  * up:annotation \[ a up:Subcellular_Location_Annotation \] 経由

## Parameters

* `categoryId` (type: uniprot subcellular location annotation)
  * example: 162,191 (Membrane,Nucleus)
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
({categoryId})=>{
  categoryId = categoryId.replace(/,/g, " ");
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/rdf/mirror/uniprot/sparql

## `main`
- メイン SPARQL
  - 内訳返す場合と遺伝子リスト返す場合を handlbars で条件分岐
  - 局在、タンパク質リストでのフィルタリング
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX locat: <http://purl.uniprot.org/locations/>
{{#if mode}}
SELECT DISTINCT ?uniprot
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
WHERE {
{{#if query_array}}
  VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
{{/if}}
{{#if category_array}}
  VALUES ?category { {{#each category_array}} locat:{{this}} {{/each}} }
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:annotation [ a up:Subcellular_Location_Annotation ;
                           up:locatedIn/up:cellularComponent ?category ] .
  ?category skos:prefLabel ?label .
    FILTER(REGEX(STR(?proteome), "UP000005640"))
}
{{#unless mode}}
ORDER BY DESC (?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, main})=>{
  if (mode) return main.results.bindings.map(d=>d.uniprot.value.replace("http://purl.unipriot.org/uniprot/", ""));
  return main.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace("http://purl.uniprot.org/locations/", ""), 
      label: d.label.value,
      count: d.count.value
    };
  });	
}
```