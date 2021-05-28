# ChEMBLでtargetの有無の内訳を返す（信定）

## Parameters

* `categoryIds`
  * example: 0, 1
* `queryIds` (type: chembl_compound)
  * example: CHEMBL1071, CHEMBL1200945, CHEMBL1274, CHEMBL204021, CHEMBL2111108, CHEMBL286398, CHEMBL252164, CHEMBL304087, CHEMBL452461, CHEMBL134702, CHEMBL279433, CHEMBL591429, CHEMBL101285, CHEMBL101360, CHEMBL101454, CHEMBL101609, CHEMBL101721, CHEMBL101775, CHEMBL101993, CHEMBL102307
* `mode`
  * example: idList, objectList

## `queryArray`
- Filter 用 chembl_compoundを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `uniprotAll`

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
{{#if mode}}
  SELECT DISTINCT ?uniprot
{{else}}
SELECT COUNT(DISTINCT ?uniprot) AS ?count
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `hasAssay`

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
{{#if mode}}
  SELECT DISTINCT ?uniprot
{{else}}
SELECT COUNT(DISTINCT ?uniprot) AS ?count
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
  ?chembl a cco:SmallMolecule ;
          cco:hasActivity/cco:hasAssay/cco:hasTarget/skos:exactMatch [
            cco:taxonomy taxon:9606 ;
            skos:exactMatch ?uniprot
          ] . 
  ?uniprot a cco:UniprotRef .
}
```

## `return`

```javascript
({uniprotAll, hasAssay, queryIds, categoryIds, mode})=>{
  const idVarName = "uniprot";
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withId = "1";
  const withoutId = "0";
  const withLabel = "Proteins with ChEMBL assay";
  const withoutLabel = "Proteins without ChEMBL assay";
  if (mode) {
    let hasAssayArray = hasAssay.results.bindings.map(d=>d[idVarName].value.replace(idPrefix, ""));
    let notAssayArray = [];
    if (!categoryIds || categoryIds.match(/0/)) {
      for (let d of uniprotAll.results.bindings) {
        let id = d[idVarName].value.replace(idPrefix, "");
        if (!hasAssayArray.includes(id)) notAssayArray.push(id);
      }
    }
    if (mode == "objectList") {
      if (categoryIds == withId) return hasAssayArray.map(d=>{
        return {
          id: d,
          attribute: {categoryId: withId, label: withLabel}
        }
      });
      if (categoryIds == withoutId) return notAssayArray.map(d=>{
        return {
          id: d,
          attribute: {categoryId: withoutId, label: withoutLabel}
        }
      });
      return hasAssayArray.map(d=>{
        return {
          id: d,
          attribute: {categoryId: withId, label: withLabel}
        }
      }).concat(notAssayArray.map(d=>{
        return {
         id: d,
          attribute: {categoryId: withoutId, label: withoutLabel}
        }
      }));
    }
    if (mode == "idList") {
      if (categoryIds == withId) return hasAssayArray;
      if (categoryIds == withoutId) return notAssayArray;
      return hasAssayArray.concat(notAssayArray);  
    }
  }
      
  var countHasAssay = hasAssay.results.bindings[0].count.value;
  var countRemain = uniprotAll.results.bindings[0].count.value - countHasAssay;
  let obj = [];
  if (!queryIds || Number(countHasAssay) != 0) {
    obj.push({
      categoryId: withId, 
      label: withLabel, 
      count: Number(countHasAssay)
    });
  }
  if (!queryIds || Number(countRemain) != 0) {
    obj.push({
      categoryId: withoutId, 
      label: withoutLabel, 
      count: Number(countRemain)
    });
  }
  return obj;
};	
```