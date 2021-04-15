# Expression value in tissues (GTEx ver8)（池田）

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `queryIds` (type: ensembl_gene)
  * default: ENSG00000002587
  * example: ENSG00000000005,ENSG00000002587,ENSG00000003989

## `input_genes`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
};
```

## `main`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ensembl: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX sio:     <http://semanticscience.org/resource/>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?desc ?tpm ?sd
WHERE {
  {{#if input_genes}}
  VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
  {{/if}}

  GRAPH <https://refex.dbcls.jp/rdf/gtex_v8_sum> {
    ?s refexo:isMeasurementOf ?ensg ;
       sio:SIO_000216 ?b_median, ?b_sd ;
       refexo:refexSample ?refexs .
    ?b_median a refexo:logTPMMedian ;
              sio:SIO_000300 ?tpm .
    ?b_sd a refexo:logTPMSD ;
          sio:SIO_000300 ?sd .
  }
  GRAPH <https://refex.dbcls.jp/rdf/refexsample_gtex_v8_sum> {
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