# Expression value in tissues (GTEx ver8)（池田）

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `id`
  * example: ENSG00000150773

## `main`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX ensembl: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX sio:     <http://semanticscience.org/resource/>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?desc ?tpm ?sd
WHERE {
  VALUES ?ensg { ensembl:{{id}} }

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