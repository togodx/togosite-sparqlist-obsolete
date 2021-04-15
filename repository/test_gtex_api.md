# GTEx expression median from GTEx API

## Parameters

* `queryIds` (type: Ensembl)
  * default: ENSG00000065613
  * example: ENSG00000000003,ENSG00000065613
  
## `httpreq`

```javascript
async ({queryIds})=>{
  let url = "https://gtexportal.org/rest/v1/reference/gene?gencodeVersion=v26&genomeBuild=GRCh38%2Fhg38&pageSize=250&format=json";
  url = url + "&geneId=" + queryIds;
  let result = await fetch(url).then(res=>res.json());
  let versionedId = result.gene[0].gencodeId;

  url = "https://gtexportal.org/rest/v1/expression/medianGeneExpression?datasetId=gtex_v8&tissueSiteDetailId=Adipose_Subcutaneous%2CAdipose_Visceral_Omentum%2CAdrenal_Gland%2CArtery_Aorta%2CArtery_Coronary%2CArtery_Tibial%2CBladder%2CBrain_Amygdala%2CBrain_Anterior_cingulate_cortex_BA24%2CBrain_Caudate_basal_ganglia%2CBrain_Cerebellar_Hemisphere%2CBrain_Cerebellum%2CBrain_Cortex%2CBrain_Frontal_Cortex_BA9%2CBrain_Hippocampus%2CBrain_Hypothalamus%2CBrain_Nucleus_accumbens_basal_ganglia%2CBrain_Putamen_basal_ganglia%2CBrain_Spinal_cord_cervical_c-1%2CBrain_Substantia_nigra%2CBreast_Mammary_Tissue%2CCells_Cultured_fibroblasts%2CCells_EBV-transformed_lymphocytes%2CCells_Transformed_fibroblasts%2CCervix_Ectocervix%2CCervix_Endocervix%2CColon_Sigmoid%2CColon_Transverse%2CEsophagus_Gastroesophageal_Junction%2CEsophagus_Mucosa%2CEsophagus_Muscularis%2CFallopian_Tube%2CHeart_Atrial_Appendage%2CHeart_Left_Ventricle%2CKidney_Cortex%2CKidney_Medulla%2CLiver%2CLung%2CMinor_Salivary_Gland%2CMuscle_Skeletal%2CNerve_Tibial%2COvary%2CPancreas%2CPituitary%2CProstate%2CSkin_Not_Sun_Exposed_Suprapubic%2CSkin_Sun_Exposed_Lower_leg%2CSmall_Intestine_Terminal_Ileum%2CSpleen%2CStomach%2CTestis%2CThyroid%2CUterus%2CVagina%2CWhole_Blood&hcluster=false&format=json";
  url = url + "&gencodeId=" + versionedId;
  result = await fetch(url).then(res=>res.json());
  let data = result.medianGeneExpression.map((elem) => ({
    tissueSiteDetailId: elem.tissueSiteDetailId,
    median: elem.median,
    tissueCategory: elem.tissueSiteDetailId.split("_")[0]
  }))
  return data;
}
```