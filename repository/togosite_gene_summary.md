## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `id`
  * default: ENSG00000150773
  * example: ENSG00000150773
* `type`
  * default: ensembl_gene
  * example: ensembl_gene

## `idDict` returns an id type object

```javascript
({id, type}) => {
  var obj = {};
  switch (type) {
    case 'ensembl_gene':
      obj.ensg = id;
      break;
    case 'ensembl_transcript':
      obj.enst = id;
      break;
    case 'hgnc':
      obj.hgnc = id;
      break;
    case 'ncbigene':
      obj.ncbigene = id;
      break;
  }
  return obj;
}
```

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX identifiers: <http://identifiers.org/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX schema: <http://schema.org/>
PREFIX nuc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>

SELECT DISTINCT *
WHERE {
  {{#if idDict.ensg}}
    {
      SELECT ?ensg ?ensg_id ?gene_symbol ?desc ?location (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
      WHERE {
        VALUES ?ensg { ensembl:{{idDict.ensg}} }
        BIND(STRAFTER(STR(?ensg), "http://identifiers.org/ensembl/") AS ?ensg_id)
        GRAPH <http://rdf.ebi.ac.uk/dataset/ensembl/102/homo_sapiens> {
          ?ebiensg obo:RO_0002162 taxon:9606 ;
                   dc:identifier ?ensg_id ;
                   rdfs:label ?gene_symbol ;
                   dc:description ?desc ;
                   faldo:location ?loc ;
       	           a ?type .
   	      FILTER(STRSTARTS(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/"))
          BIND(STRAFTER(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/") as ?type_label)
          ?loc rdfs:label ?location .
        }
        OPTIONAL {
          GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
            ?ensg refexo:isPositivelySpecificTo ?tissue .
          }
          GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
            ?refexs schema:additionalProperty ?b1 .
            VALUES ?name {"cell type" "tissue"}
            ?b1 schema:name ?name ;
                schema:value ?tissue_label ;
                schema:valueReference ?tissue .
          }
        }
#        {
#          GRAPH <http://rdf.integbio.jp/dataset/togosite/efo> {
#            ?tissue rdfs:label ?tissue_label .
#          }
#        }
#        UNION
#        {
#          GRAPH <http://rdf.integbio.jp/dataset/togosite/uberon> {
#            ?tissue rdfs:label ?tissue_label .
#          }
#        }
      }
    }
  {{/if}}
  {{#if idDict.ncbigene}}
    {
      SELECT ?ncbigene ?ncbigene_id ?desc ?location ?gene_symbol ?type_of_gene
                       (GROUP_CONCAT(DISTINCT ?others; separator=", ") AS ?other_names)
                       (GROUP_CONCAT(DISTINCT ?tissue_label; separator=", ") AS ?tissue_labels)
                       (GROUP_CONCAT(DISTINCT ?synonym; separator=", ") AS ?synonyms)
      WHERE {
        VALUES ?ncbigene { ncbigene:{{idDict.ncbigene}} }
        GRAPH <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info> {
          ?ncbigene dct:description ?desc ;
                    dct:identifier ?ncbigene_id ;
                    hop:typeOfGene ?type_of_gene ;
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
    }

  {{/if}}
  {{#if idDict.hgnc}}
    {
      SELECT ?hgnc ?hgnc_id ?gene_symbol ?desc ?location
      WHERE {
        VALUES ?hgnc { hgnc:{{idDict.hgnc}} }
        GRAPH <http://rdf.integbio.jp/dataset/togosite/hgnc> {
          ?hgnc dct:identifier ?hgnc_id ;
                rdfs:label ?gene_symbol ;
                dct:description ?desc ;
                obo:so_part_of ?location .
        }
      }
    }

  {{/if}}

}
```

## `columns` columns and their order to show

```javascript
() => {
  const array = [
    { "Ensembl ID": "ensg_id" },
    { "Ensembl URL": "ensg" },
    { "HGNC ID": "hgnc_id" },
    { "HGNC URL": "hgnc" },
    { "NCBI Gene ID": "ncbigene_id" },
    { "NCBI Gene URL": "ncbigene" },
    { "Gene symbol": "gene_symbol" },
    { "Description": "desc" },
    { "Location": "location" },
    { "Tissue specificity": "tissue_labels"},
    { "Synonym": "synonyms" },
    { "Type": "type_of_gene" },
    { "Others": "other_names" }
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

