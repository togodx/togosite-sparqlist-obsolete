# RefEx organ specific 'low' expression 内訳（守屋）

- Required SPARQLet: refex_specific_expression

## Description

- Data sources
    - [Calculations for tissue specificity of genechip human GSE7307](https://doi.org/10.6084/m9.figshare.4028700.v3) from [RefEx \(Reference Expression dataset\)](https://refex.dbcls.jp/)
      - Tissue specificity matrix were constructed consisting of 1 for over-expressed outliers, -1 for under-expressed outliers, and 0 for non-outliers that corresponds to the original gene expression matrix by applying the ROKU method [(Kadota et al., BMC Bioinformatics, 2006, 7:294)](https://doi.org/10.1186/1471-2105-7-294).
        - See details: [http://bioconductor.org/packages/release/bioc/manuals/TCC/man/TCC.pdf](http://bioconductor.org/packages/release/bioc/manuals/TCC/man/TCC.pdf) (page 22-24)    
    - Tissue-specific low-expression genes were those flagged as "-1" as under-expressed outliers only in a single organ.
- Query
    - Input
        - NCBI Gene ID
    - Output
        - [RefEx tissue name classification table of 40 organs](https://doi.org/10.6084/m9.figshare.4028718.v5)
## Parameters

* `categoryIds` (type: refexo 40 tissue)
  * example: v01_40,v19_40,v32_40 (Cerebrum,Skin,Lung)
* `queryIds` (type: ncbigene)
  * example: 1230,2357,101,2219,374,1880,653,695,366,218,1359,221,722,597,1641,576,2173,1949,2246,1463,1183,273,119,2571,1780,1496,1974,2026,2185,2002,605,491,1024
* `mode`
  * example: idList, objectlist
  
## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  let url = "https://integbio.jp/togosite/sparqlist/api/refex_specific_expression"; // localhost:port を叩けると早い
  let options = {
    method: 'POST',
    body	: "negatively=1&categoryIds=" + categoryIds + "&queryIds=" + encodeURIComponent(queryIds) + "&mode=" + mode,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  