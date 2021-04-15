# test togoid

* source か target のどちらかは chembl_target

## Parameters
* `source`
  * example: chembl_target
* `target`
  * example: chembl_compound, pubchem_compound
* `ids`
  * example: CHEMBL4295851,CHEMBL3105,CHEMBL3873
  
## `idArray`
```javascript
({ids, source}) => {
  ids = ids.replace(/,/g," ")
  if (ids.match(/[^\s]/) && ids != "undefined") return ids.split(/\s+/).map(d=>source + ":" + d);
  return false;
}
```

## `type`
```javascript
({source, target})=>{
  let obj = {};
  if (source == "chembl_compound" || target == "chembl_compound") obj.chembl_compound = true;
  if (source == "pubchem_compound" || target == "pubchem_compound") obj.pubchem_compound = true;
  
  if (source == "chembl_target") obj.source = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql
## `link`
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX chembl_compound: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX chembl_target: <http://rdf.ebi.ac.uk/resource/chembl/target/>
PREFIX pubchem_compound: <http://pubchem.ncbi.nlm.nih.gov/compound/>
SELECT DISTINCT (REPLACE(STR(?id1_uri), chembl_target:, "") AS ?id1) (REPLACE(STR(?id2_uri), {{#if type.source}}{{target}}{{else}}{{source}}{{/if}}:, "") AS ?id2)
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  {{#if type.source}}
  VALUES ?id1_uri { {{#each idArray}} {{this}} {{/each}} }
  {{else}}
  VALUES ?id2_uri { {{#each idArray}} {{this}} {{/each}} }  
  {{/if}}
  {{#if type.chembl_compound}}
  ?id2_uri a cco:SmallMolecule ;
           cco:hasActivity/cco:hasAssay/cco:hasTarget ?id1_uri .
  {{/if}}
  {{#if type.pubchem_compound}}
   ?chembl a cco:SmallMolecule ;
           cco:hasActivity/cco:hasAssay/cco:hasTarget ?id1_uri ;
           cco:moleculeXref ?id2_uri .
   FILTER (CONTAINS(STR(?id2_uri), "pubchem.ncbi.nlm.nih.gov/compound/"))
   {{/if}}
}
```

## `return`
```javascript
({type, link})=>{
  if (type.source) return link.results.bindings.map(d => {
    return {
      source: d.id1.value,
      target: d.id2.value
    }
  });
  return link.results.bindings.map(d => {
    return {
      source: d.id2.value,
      target: d.id1.value
    }
  });
}
```
