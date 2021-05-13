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

SELECT DISTINCT *
#?ensg_id ?ncbigene_id ?hgnc_id ?type_label ?desc ?location ?gene_symbol (GROUP_CONCAT(DISTINCT ?uniprot_id; separator=",") AS ?uniprots) (GROUP_CONCAT(DISTINCT ?enst_id; separator=",") AS ?ensts)
WHERE {
  {{#if idDict.ensg}}
    {
      SELECT ?ensg ?ensg_id ?gene_symbol ?desc ?location (GROUP_CONCAT(DISTINCT ?tissue_label; separator=",") AS ?tissue_labels)
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
        GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
          ?ensg refexo:isPositivelySpecificTo ?tissue .
        }
        {
          GRAPH <http://rdf.integbio.jp/dataset/togosite/efo> {
            ?tissue rdfs:label ?tissue_label .
          }
        }
        UNION
        {
          GRAPH <http://rdf.integbio.jp/dataset/togosite/uberon> {
            ?tissue rdfs:label ?tissue_label .
          }
        }
      }
    }
  {{/if}}
  {{#if idDict.ncbigene}}
    {
      SELECT *
      WHERE {
        VALUES ?ncbigene { ncbigene:{{idDict.ncbigene}} }
        BIND(STRAFTER(STR(?ncbigene), "http://identifiers.org/ncbigene/") AS ?ncbigene_id)
        GRAPH <> {
          ?ncbigene ?p ?o .
        }
      }
    }

  {{/if}}
  {{#if idDict.hgnc}}
    {
      SELECT ?hgnc ?hgnc_id ?gene_symbol ?desc ?loc ?pubmed
      WHERE {
        VALUES ?hgnc { hgnc:{{idDict.hgnc}} }
        GRAPH <http://rdf.integbio.jp/dataset/togosite/hgnc> {
          ?hgnc dct:identifier ?hgnc_id ;
                rdfs:label ?gene_symbol ;
                dct:description ?desc ;
                dct:references ?pubmed ;
                obo:so_part_of ?loc .
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
    { "Tissue": "tissue_labels"}
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
        }
        return obj;
      };
    }, {});
  });
};
```

