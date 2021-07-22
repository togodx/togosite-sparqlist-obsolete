# Open TG-Gates で化合物に応答して発現上昇した遺伝子（池田）

## Description
 
- Data sources
    - [Open TG-Gates RDF](https://integbio.jp/rdf/dataset/open-tggates)

- Query
    - Input
        - Pubchem ID, Affymetrix Probe ID
    - Output
        - The number of genes responsive to a compound

## Parameters

* `categoryIds` (type: Pubchem)
  * example: 178, 1983, 5897
* `queryIds` (type: Affy)
  * example: 1552414_at, 1552767_a_at, 1557712_x_at
* `mode` 
  * example: idList, objectList

## Endpoint

https://integbio.jp/rdf/sparql

## `input_categories`
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/,/g, " ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `input_queries`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
}
```

## `main`

```sparql
PREFIX tgo:     <http://purl.jp/bio/101/opentggates/ontology/>
PREFIX rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:    <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sio:     <http://semanticscience.org/resource/>
PREFIX taxon:   <http://identifiers.org/taxonomy/>
PREFIX pubchem: <http://identifiers.org/pubchem.compound/>
PREFIX probe:   <http://purl.jp/bio/101/opentggates/Probe/>

{{#if mode}}
SELECT DISTINCT ?compound_label ?pubchem ?gene_symbol ?probe_id
{{else}}
SELECT DISTINCT ?compound_label ?pubchem (COUNT(DISTINCT ?gene_symbol) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/open-tggates>
WHERE {
  {{#if input_queries}}
  VALUES ?probe { {{#each input_queries}} probe:{{this}} {{/each}} }
  {{/if}}
  {{#if input_categories}}
  VALUES ?pubchem { {{#each input_categories}} pubchem:{{this}} {{/each}}  }
  {{/if}}
  ?unitset tgo:hasUnitSetProfile ?unitsetprofile .
  ?unitsetprofile sio:000216 [
    a tgo:PValue ;
    sio:SIO_000300 ?pvalue ;
    tgo:probe ?probe
  ] .
  FILTER(?pvalue < 0.01) # FDR would be better
  ?unitset tgo:hasTestSample 
           / tgo:experimentalCondition
           / tgo:exposedCompound ?compound .
  ?compound rdfs:label ?compound_label ;
            rdfs:seeAlso ?pubchem .
  FILTER(STRSTARTS(STR(?pubchem), "http://identifiers.org/pubchem.compound/"))
  ?probe tgo:geneSymbol ?gene_symbol ;
         dcterms:identifier ?probe_id ;
         tgo:taxon taxon:9606 .
} ORDER BY ?compound_label
```

## `return`

```javascript
({ main, mode }) => {
  if (mode == "idList") {
    return Array.from(new Set(
      main.results.bindings.map((d) => d.probe_id.value)
    ));
  } else if (mode == "objectList") {
    return main.results.bindings.map((d) => ({
      id: d.probe_id.value, 
      attribute: {
        categoryId: d.pubchem.value.replace("http://identifiers.org/pubchem.compound/", ""), 
        label: capitalize(d.compound_label.value)
      }
    }));
  } else {
    return main.results.bindings.map((d) => ({
      categoryId: d.pubchem.value.replace("http://identifiers.org/pubchem.compound/", ""), 
      label: capitalize(d.compound_label.value),
      count: Number(d.count.value)
    }));
  }
  
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
}
```