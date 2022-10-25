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
       (GROUP_CONCAT(DISTINCT ?others; separator="__") AS ?other_names)
       (GROUP_CONCAT(DISTINCT ?p_tissue_label; separator=", ") AS ?p_tissue_labels)
       (GROUP_CONCAT(DISTINCT ?n_tissue_label; separator=", ") AS ?n_tissue_labels)
       (GROUP_CONCAT(DISTINCT ?synonym; separator=", ") AS ?synonyms)
WHERE {
  VALUES ?ncbigene { ncbigene:{{id}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info> {
    ?ncbigene dct:description ?desc ;
              dct:identifier ?ncbigene_id ;
              hop:typeOfGene ?type_label ;
              nuc:chromosome ?chromosome ;
              nuc:map ?location ;
              rdfs:label ?gene_symbol .
    OPTIONAL {
      ?ncbigene nuc:gene_synonym ?synonym .
    }
    OPTIONAL {
      ?ncbigene dct:alternative ?others .
    }
  }
  VALUES ?p { refexo:affyProbeset refexo:refseq }
  VALUES ?graph { <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_rnaseq_human_PRJEB2445>
                  <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genechip_human_GSE7307> }
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human> {
      ?ncbigene ?p ?gene .
    }
    GRAPH ?graph {
      ?gene refexo:isPositivelySpecificTo ?p_tissue .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo> {
      ?p_tissue rdfs:label ?p_tissue_label .
      FILTER(lang(?p_tissue_label) = 'en')
    }
  }
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human> {
      ?ncbigene ?p ?gene .
    }
    GRAPH ?graph {
      ?gene refexo:isNegativelySpecificTo ?n_tissue .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo> {
      ?n_tissue rdfs:label ?n_tissue_label .
      FILTER(lang(?n_tissue_label) = 'en')
    }
  }
}
```

## `go`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?go ?go_category ?go_label ?go_id
WHERE {
  VALUES ?ncbigene { ncbigene:{{id}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info> {
    ?ncbigene hop:hasGO ?go .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/go> {
    ?go oboinowl:hasOBONamespace ?go_category ;
        oboinowl:id ?go_id ;
        rdfs:label ?go_label .
  }
}
ORDER BY ?go
```

## `return`

```javascript
({ main, id, go }) => {
  let data = main.results.bindings[0];
  let objs = [{
    "NCBI_Gene_ID": data.ncbigene_id.value,
    "NCBI_Gene_URL": data.ncbigene.value,
    "Gene_symbol": data.gene_symbol.value,
    "Synonym": data.synonyms.value,
    "Description": data.desc.value,
    "Gene_type": data.type_label.value,
    "Location": data.location.value,
    "Other_names": "",
    "Biological_process": "",
    "Cellular_component": "",
    "Molecular_function": "",
    "Tissue-specific_high_expression_(RefEx)": "",
    "Tissue-specific_low_expression_(RefEx)": "",
  }];

  if (data.p_tissue_labels?.value)
    objs[0]["Tissue-specific_high_expression_(RefEx)"] += makeList(data.p_tissue_labels.value.split(", "));
  if (data.n_tissue_labels?.value)
    objs[0]["Tissue-specific_low_expression_(RefEx)"] += makeList(data.n_tissue_labels.value.split(", "));

  const go_categories = ["Biological_process", "Cellular_component", "Molecular_function"];
  go_categories.forEach((category => {
    if(go.results.bindings.length > 0) {
      let gos = go.results.bindings.filter((x => x.go_category.value==category.toLowerCase()));
      if (gos.length > 0) {
        let ids = gos.map((x => x.go_id.value));
        let urls = gos.map((x => x.go.value));
        let labels = gos.map((x => x.go_label.value));
        objs[0][category] = makePairList(ids, labels, urls);
      }
    }
  }));
  if (data.other_names?.value) objs[0].Other_names = makeList(data.other_names.value.split("__"));
  return objs;

  function makeLink(url, text) {
    return "<a href=\"" + url + "\" target=\"_blank\">" + text + "</a>";
  }
  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
  function makePairList(ids, labels, urls) {
    return makeList(ids.map((id, i)=>makeLink(urls[i], id) + " " + labels[i]));
  }
};
```
