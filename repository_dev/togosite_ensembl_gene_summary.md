## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: ENSG00000150773
  * example: ENSG00000150773

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX idt_ensg: <http://identifiers.org/ensembl/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>

SELECT DISTINCT ?ensg_id ?idt_ensg ?gene_symbol ?desc ?type_label ?location (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
WHERE {
  VALUES ?ensg { ensg:{{id}} }
  VALUES ?idt_ensg { idt_ensg:{{id}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ensg obo:RO_0002162 taxon:9606 ;
             dc:identifier ?ensg_id ;
             rdfs:label ?gene_symbol ;
             dc:description ?desc ;
             faldo:location ?loc ;
             a ?type .
    FILTER(STRSTARTS(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/"))
    BIND(REPLACE(STRAFTER(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/"), "_", " ") as ?type_label)
    ?loc rdfs:label ?location .
  }
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
      ?idt_ensg refexo:isPositivelySpecificTo ?tissue .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
      VALUES ?name {"cell type" "tissue"}
      ?refexs schema:additionalProperty [
        schema:name ?name ;
        schema:value ?tissue_label ;
        schema:valueReference ?tissue
      ] .
    }
  }
}
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "Ensembl ID": "ensg_id" },
    { "Ensembl URL": "idt_ensg" },
    { "Gene symbol": "gene_symbol" },
    { "Description": "desc" },
    { "Location": "location" },
    { "Tissue specificity": "tissue_labels"},
    { "Type": "type_label" }
  ];
  return array;
}
```

## `return`

```javascript
({ main, columns }) => {
  return main.results.bindings.map((binding) => {
    const results = columns.map((row) => {
      const obj = {};
      for (const [k, v] of Object.entries(row)) {
        obj[k] = binding[v];
      }
      return obj;
    });

    return results.reduce((obj, elem) => {
      for (const [key, node] of Object.entries(elem)) {
        if (node) {
          obj[key] = node.value;
          if (key == "Tissue specificity" && obj[key] == "") {
            obj[key] = "(Low tissue specificity)";
          }
        }
        return obj;
      };
    }, {});
  });
};
```
