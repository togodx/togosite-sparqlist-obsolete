# UniProtについて、ChEMBL Assayの有無とtargetの内訳を返す（信定）

## Description
 
- Data sources
    - Proteins with or without ChEMBL assay from UniProt
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - Input
        - Existence (1: exists, 0: not exists), UniProt ID
    - Output
        - The number of UniProt entries link to ChEMBL entries.
        - If a UniProt ID is entered, it returns whether ChEMBL entry exists or not.



## Parameters

* `categoryIds`
  * example: 0, 1
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
#PREFIX bao: <http://www.bioassayontology.org/bao#BAO_>
{{#if mode}}
  SELECT DISTINCT ?uniprot
{{else}}
SELECT COUNT(DISTINCT ?uniprot) AS ?count ?assaytype ?conf_score ?conf_label
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
  #VALUES ?bao { bao:0000186 bao:0000187 bao:0000188 bao:0000189 bao:0000190 bao:0000192 } # AC50 CC50 EC50 GI50 IC50 Ki
  VALUES ?conf {9}
  ?chembl a cco:SmallMolecule ;
          cco:hasActivity/cco:hasAssay ?assay.
  ?assay a cco:Assay ;
            cco:targetConfScore ?conf_score ;
           cco:assayType ?assaytype ;
            cco:targetConfDesc ?conf_label ;
            cco:hasTarget/skos:exactMatch [
            cco:taxonomy taxon:9606 ;
            skos:exactMatch ?uniprot
          ] . 
  #?activity bao:0000208 ?bao .
  ?uniprot a cco:UniprotRef .
}
```


## `countHasAssay`

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT COUNT(DISTINCT ?uniprot) AS ?count
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
{{#unless mode}}
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
  ?chembl a cco:SmallMolecule ;
          cco:hasActivity/cco:hasAssay ?assay.
  ?assay a cco:Assay ;
            cco:targetConfScore ?conf_score ;
            cco:targetConfDesc ?conf_label ;
            cco:hasTarget/skos:exactMatch [
            cco:taxonomy taxon:9606 ;
            skos:exactMatch ?uniprot
          ] . 
  ?uniprot a cco:UniprotRef .
{{/unless}}
}
```
           
## `return`

```javascript
({uniprotAll, hasAssay, countHasAssay, queryIds, categoryIds, mode})=>{
  const idVarName = "uniprot";
  const categoryVarName = "conf_score";
  const categoryLabelVarName = "conf_label";
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "without";
  const withoutLabel = "Proteins without ChEMBL assay";

  let categories = {};
  categoryIds.replace(/,/g," ").split(/\s+/).map(d => {
    categories[d] = true;
  })
  //tentatively
    console.log("test")
  //
  if (mode) {
    let hasAssayArray = hasAssay.results.bindings.map(d=>d[idVarName].value.replace(idPrefix, ""));
    let notAssayArray = [];
    if (!categoryIds || categoryIds.match(/without/)) {
      for (let d of uniprotAll.results.bindings) {
        let id = d[idVarName].value.replace(idPrefix, "");
        if (!hasAssayArray.includes(id)) notAssayArray.push(id);
      }
    }

    let obj = [];    
    if (!categoryIds || categoryIds.match(/\d/)) {
      hasAssay.results.bindings.map(d => {
	    if (!categoryIds || categories[d[categoryVarName].value]) {
	      obj.push({
	        id: d[idVarName].value,
            attribute: {categoryId: d[categoryVarName].value, label: d[categoryLabelVarName].value}
          })
        }
      })
    }

    if (!categoryIds || categories[withoutId]) {
      notAssayArray.map(d => {
        obj.push({
          id: d,
          attribute: {categoryId: withoutId, label: withoutLabel}
        })
      })
    }

    if (mode == "objectList") {
      return obj;
    } else if (mode == "idList") {
	  return obj.map(d => d.id);
    }
  }
  
  var countHasAssay = countHasAssay.results.bindings[0].count.value;
  var countRemain = uniprotAll.results.bindings[0].count.value - countHasAssay;
  let obj = [];
  hasAssay.results.bindings.map(d => {
    if (!categoryIds || categories[d[categoryVarName].value]) {
      obj.push({
        categoryId: d[categoryVarName].value, 
        label: d[categoryLabelVarName].value, 
        count: Number(d.count.value)
      })
    }
  })
  if ((!queryIds || Number(countRemain) != 0) && (!categoryIds || categories[withoutId])) {
    obj.push({
      categoryId: withoutId, 
      label: withoutLabel, 
      count: Number(countRemain)
    })
  }
  return obj;
}
```