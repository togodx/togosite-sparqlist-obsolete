## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: 1
  * example: 1

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX nuc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>

SELECT ?ncbigene ?ncbigene_id ?desc ?location ?gene_symbol ?type_label
       (GROUP_CONCAT(DISTINCT ?others; separator=", ") AS ?other_names)
       (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
       (GROUP_CONCAT(DISTINCT ?synonym; separator=", ") AS ?synonyms)
WHERE {
  VALUES ?ncbigene { ncbigene:{{id}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info> {
    ?ncbigene dct:description ?desc ;
              dct:identifier ?ncbigene_id ;
              hop:typeOfGene ?type_label ;
              nuc:chromosome ?chromosome ;
              nuc:map ?location .
    OPTIONAL {
      ?ncbigene nuc:standard_name ?gene_symbol .
    }
    OPTIONAL {
      ?ncbigene nuc:gene_synonym ?synonym .
    }
    OPTIONAL {
      ?ncbigene dct:alternative ?others .
    }
  }
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human> {
      VALUES ?p { refexo:affyProbeset refexo:refseq }
      ?ncbigene ?p ?gene .
    }
    VALUES ?graph { <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_rnaseq_human_PRJEB2445>
                    <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genechip_human_GSE7307> }
    GRAPH ?graph {
      ?gene refexo:isPositivelySpecificTo ?tissue .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo> {
      ?tissue rdfs:label ?tissue_label .
      FILTER(lang(?tissue_label) = 'en')
    }
  }
}
```

## `return`

```javascript
({ main, id }) => {
  let binding = main.results.bindings[0];
  let objs = [{
    "NCBI Gene ID": binding.ncbigene_id.value,
    "NCBI Gene URL": binding.ncbigene.value,
    "Gene symbol": binding.gene_symbol.value,
    "Synonym": binding.synonyms.value,
    "Description": binding.desc.value,
    "Gene type": binding.type_label.value,
    "Location": binding.location.value,
    "Tissue specificity (RefEx)": binding.tissue_labels.value,
  }];

  if (objs[0]["Tissue specificity (RefEx)"] == "") {
    objs[0]["Tissue specificity (RefEx)"] = "(Low tissue specificity)";
  }
  return objs;
};
```

