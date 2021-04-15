# Gene attributes from Uniprot RDF（井手）

## Endpoint

https://integbio.jp/rdf/mirror/uniprot/sparql

## Parameters
* `uniprot`
  * default: P06493
  * example: P06493
  
## `gene_list`
```javascript
({ uniprot }) => {
  uniprot = uniprot.replace(/\s/g, "");
  if (uniprot) {
    return uniprot.split(",");
  } else {
    return false;
  }
};
```

## `main`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?id ?full_name ?short_name ?mass Count(?citation) AS ?citation_number
WHERE{
  {{#if gene_list}}
  VALUES ?entry { {{#each gene_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  ?entry a core:Protein .
  ?entry core:recommendedName ?rname .
  ?rname core:fullName ?full_name .
  optional{?rname core:shortName ?short_name .}
  ?entry core:sequence/core:mass ?mass .
  optional{?entry core:citation ?citation .}
  BIND(REPLACE(STR(?entry), uniprot:, "") AS ?id)
}

```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    uniprot_id: elem.id.value,
    full_name: elem.full_name.value,
    short_name: elem.short_name.value,
    mass: elem.mass.value,
    citation_number: elem.citation_number.value
  }));
};
```