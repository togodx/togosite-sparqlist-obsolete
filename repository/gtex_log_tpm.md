# Expression value in tissues (GTEx ver8)（池田）

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `id`
  * default: ENSG00000150773
  * example: ENSG00000150773, ENST00000443779, 1, 5
* `type`
  * default: ensembl_gene
  * example: ensembl_gene, ensembl_transcript, ncbigene, hgnc

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
    case 'ncbigene':
      obj.ncbigene = id;
      break;
  }
  return obj;
}
```

## `main`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ensembl: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX sio:     <http://semanticscience.org/resource/>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?desc ?tpm ?sd
WHERE {
  {{#if idDict.ensg}}
  VALUES ?ensg { ensembl:{{idDict.ensg}} }
  {{/if}}
  {{#if idDict.ncbigene}}
  VALUES ?ncbigene { ncbigene:{{idDict.ncbigene}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid/ncbigene-ensembl_gene> {
    ?ncbigene rdfs:seeAlso ?ensg .
    FILTER(STRSTARTS(STR(?ensg), "http://identifiers.org/ensembl/"))
  }
  {{/if}}
  {{#if idDict.hgnc}}
  VALUES ?hgnc { hgnc:{{idDict.hgnc}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid/hgnc-ensembl_gene> {
    ?hgnc rdfs:seeAlso ?ensg .
    FILTER(STRSTARTS(STR(?ensg), "http://identifiers.org/ensembl/"))
  }
  {{/if}}
  {{#if idDict.enst}}
  VALUES ?enst_input { ensembl:{{idDict.enst}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid/ensembl_gene-ensembl_transcript> {
    ?ensg obo:RO_0002511 ?enst_input .
  }
  {{/if}}

  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?s refexo:isMeasurementOf ?ensg ;
       sio:SIO_000216 ?b_median, ?b_sd ;
       refexo:refexSample ?refexs .
    ?b_median a refexo:logTPMMedian ;
              sio:SIO_000300 ?tpm .
    ?b_sd a refexo:logTPMSD ;
          sio:SIO_000300 ?sd .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dct:description ?desc .
  }
}ORDER BY ?desc
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    desc: elem.desc.value,
    tissueCategory: elem.desc.value.split(" - ")[0],
    logTPM: Number(elem.tpm.value),
    sd: Number(elem.sd.value),
  }));
};
```