# RefEx organ specific 'high' expression 内訳（守屋）

- Required SPARQLet: refex_specific_expression

## Description

- Data sources
    - [Calculations for tissue specificity of genechip human GSE7307](https://doi.org/10.6084/m9.figshare.4028700.v3) from [RefEx \(Reference Expression dataset\)](https://refex.dbcls.jp/)
- Input/Output
    - Input
        - NCBI Gene ID
    - Output
        - [RefEx tissue name classification table of 40 organs](https://doi.org/10.6084/m9.figshare.4028718.v5)
- Supplementary information
	- The tissue-specific highly expressed genes provided in this attribute are data available from [RefEx](https://refex.dbcls.jp) and are calculated for 34 organ samples in the original gene expression data ([GSE7307](https://www.ncbi.nlm.nih.gov /geo/query/acc.cgi?acc=GSE7307)). Tissue specificity of each gene was calculated using the ROKU method [(Kadota et al., BMC Bioinformatics, 2006, 7:294)](https://doi.org/10.1186/1471-2105-7-294), with 1 for high expression outliers, -1 for low expression outliers, and 0 for non-outliers. Tissue-specific highly expressed genes provided here refer to genes flagged as "1" as outliers that are highly expressed only in a single organ.
	- ここで提供している組織特異的な高発現遺伝子は、オリジナルの遺伝子発現データ([GSE7307](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE7307))に含まれる34臓器のサンプルに対して、ROKU法[(Kadota et al., BMC Bioinformatics, 2006, 7:294)](https://doi.org/10.1186/1471-2105-7-294)を用いて、高発現の外れ値に1、低発現の外れ値に-1、非外れ値に0からなる組織特異性行列を計算した結果です。ROKU法の技術的詳細は[こちらをご覧ください(22-24ページ) ](http://bioconductor.org/packages/release/bioc/manuals/TCC/man/TCC.pdf) 。ここで提供している組織特異的な高発現遺伝子とは、単一の器官でのみ高発現している外れ値として「1」のフラグが立てられた遺伝子を示します。

## Parameters

* `categoryIds` (type: refexo 40 tissue)
  * example: v01_40,v19_40,v32_40 (Cerebrum,Skin,Lung)
* `queryIds` (type: ncbigene)
  * example: 1230,2357,101,2219,374,1880,653,695,366,218,1359,221,722,597,1641,576,2173,1949,2246,1463,1183,273,119,2571,1780,1496,1974,2026,2185,2002,605,491,1024
* `mode`
  * example: idList, objectList
  
## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  const url = "refex_specific_expression"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body	: "categoryIds=" + categoryIds + "&queryIds=" + encodeURIComponent(queryIds) + "&mode=" + mode,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  