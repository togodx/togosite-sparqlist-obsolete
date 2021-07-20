# Data from user IDs with p-value - 内訳 SPARQList を user IDs でフィルタ

* sparqlet: Property の API (properties.json の 'data')
* primaryKey: Property で対象とするデータベース (properties.json の 'primaryKey')
* categoryIds: 下層を取ってくる場合に使用 (要、UIなどの検討）
* userKey: Upload された ID のデータベース
* userIds: Upload された ID リスト

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/gene_high_level_expression_refex
* `primaryKey` databse of SPARQLet
  * default: ncbigene
* `categoryIds`
  * example:
* `userKey` database of uploaded user IDs
  * default: uniprot
* `userIds` uploaded user IDs
  * default: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65

## `pValueFlag`
```javascript
({primaryKey, sparqlet})=>{
  let obj = {};
  if (primaryKey == "uniprot") {  // || primaryKey == "pdb") {
    obj[primaryKey] = true;
  } else if (sparqlet.match(/gene_biotype_ensembl$/) || sparqlet.match(/Ensembl_gene_type$/)) {
    obj.ensembl_gene_biotype = true;  
  } else if (sparqlet.match(/gene_chromosome_ensembl$/)) {
    obj.ensembl_gene_chromosome = true;  
  } else if (sparqlet.match(/gene_number_of_exons_ensembl$/) || sparqlet.match(/Ensembl-exon-count$/)) {
    obj.ensembl_transcript_exons = true;  
  } else if (sparqlet.match(/gene_number_of_paralogs_homologene$/) || sparqlet.match(/homologene_human_paralog_count$/)
            || sparqlet.match(/gene_evolutionary_conservation_homologene$/) || sparqlet.match(/homologene_category$/)) {
    obj.ncbigene_homologene = true;  
  } else if (sparqlet.match(/gene_\w+_level_expression_refex$/) || sparqlet.match(/refex_specific_\w+_expression$/)) {
    obj.ncbigene_refex = true;  
  } else if (sparqlet.match(/gene_high_level_expression_gtex6$/) || sparqlet.match(/gtex6_tissues$/)) {
    obj.ensembl_gene_gtex6 = true;  
  } else if (sparqlet.match(/gene_transcription_factors_chip_atlas$/) || sparqlet.match(/chip_atlas$/)) {
    obj.ensembl_gene_chip_atlas = true;  
  }
  if (Object.keys(obj).length) return obj;
  obj.nonPValue = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `population`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX taxon_up: <http://purl.uniprot.org/taxonomy/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
{{#if pValueFlag.ensembl_gene_biotype}}
SELECT (COUNT(DISTINCT ?ensg) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?ensg obo:RO_0002162 taxid:9606 ; # in taxon
      a ?type .
  ?enst obo:SO_transcribed_from ?ensg .
  FILTER regex(str(?type), "/terms/ensembl/")
}
{{/if}}
{{#if pValueFlag.ensembl_gene_chromosome}}
SELECT (COUNT(DISTINCT ?ensg) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?enst obo:SO_transcribed_from ?ensg .
  ?ensg obo:RO_0002162 taxid:9606 ; # in taxon
    faldo:location ?ensg_location .
  BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
  VALUES ?chromosome {
    "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
    "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
    "X" "Y" "MT"
  }
}
{{/if}}
{{#if pValueFlag.ensembl_transcript_exons}}
SELECT (COUNT(DISTINCT ?enst) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?enst obo:SO_has_part ?exon ;
      obo:SO_transcribed_from ?ensg .
  ?ensg obo:RO_0002162 taxid:9606 . # in taxon
}
{{/if}}
{{#if pValueFlag.ncbigene_homologene}}
SELECT (COUNT(DISTINCT ?human_gene) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/homologene/data>
WHERE {
  ?human_gene orth:taxon taxid:9606 .
}
{{/if}}
{{#if pValueFlag.ncbigene_refex}}
SELECT (COUNT(?gene) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human>
WHERE {
  ?gene refexo:affyProbeset ?affy .
}
{{/if}}
{{#if pValueFlag.ensembl_gene_gtex6}}
SELECT (COUNT(DISTINCT ?ensg) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6>
WHERE {
  ?ensg a refexo:GTEx_v6_ts_evaluated_gene
}
{{/if}}
{{#if pValueFlag.ensembl_gene_chip_atlas}}
SELECT (COUNT(?ensg) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/chip_atlas_ncbigene_ensembl>
WHERE {
  ?ensg refexo:ncbigene ?ncbigene .
}
{{/if}}
{{#if pValueFlag.uniprot}}
SELECT (COUNT (DISTINCT ?uniprot) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?uniprot a uniprot:Protein ;
         uniprot:organism taxon_up:9606 ;
         uniprot:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
{{/if}}
{{#if pValueFlag.pdb}}
SELECT (COUNT(DISTINCT ?pdb) AS ?total_count)
FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
WHERE {
  ?pdb a pdbo:datablock ;
      pdbo:has_pdbx_entity_nonpolyCategory ?nonpoly ;
      pdbo:has_entityCategory/pdbo:has_entity/rdfs:seeAlso taxid:9606 .
}
{{/if}}
{{#if pValueFlag.nonPValue}}
SELECT ?s
WHERE {
}
{{/if}}  
```

## `distribution`
```javascript
async ({sparqlet, categoryIds, userIds, userKey, primaryKey, pValueFlag, population})=>{
  const dev_stage = await fetch("http://localhost:3000/togosite_dev/sparqlist/api/dev_stage_check").then(res=>res.text());
  let apiurl = "http://localhost:3000/togosite/sparqlist/api/";
  if (dev_stage == "true") apiurl = "http://localhost:3000/togosite_dev/sparqlist/api/";

  const fetchReq = async (url, body) => {
    let options = {	
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    console.log(url);
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }

  const togoidSparqlistSplitter = apiurl + "togoid_sparqlist_splitter";  
  const sparqlistSplitter = apiurl + "sparqlist_splitter";
  const togoidApi = apiurl + "togoid_route_sparql";
  let idLimit = 2000; // split 判定
  if (primaryKey == "chembl_compound") idLimit = 500; // restrict POST response size
  sparqlet = apiurl + sparqlet.split(/\//).slice(-1)[0]; // replace global URL to localhost

  // convert user IDs to primary IDs for SPARQLet
  let queryIds = "";
  if (userKey != primaryKey) {
    let body = "source=" + userKey + "&target=" + primaryKey + "&ids=" +  encodeURIComponent(userIds);
    let togoidPair;
    if (queryIds.split(/,/).length <= idLimit) {
      togoidPair = await fetchReq(togoidApi, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(togoidApi) + "&limit=" + idLimit;
      togoidPair = await fetchReq(togoidSparqlistSplitter, body).map(d=>d.target_id).join(",");
    }
    queryIds = togoidPair.map(d=>d.target_id).join(",");
  } else {
    queryIds = userIds;
  }
  
  if (!queryIds.match(/\w/)) return [];
  
  // get property data
  let distribution = [];
  let body = "queryIds=" + queryIds;
  if (categoryIds) body += "&categoryIds=" + categoryIds;
  if (queryIds.split(/,/).length <= idLimit) distribution = await fetchReq(sparqlet, body);
  body += "&sparqlet=" + encodeURIComponent(sparqlet) + "&limit=" + idLimit;
  distribution = await fetchReq(sparqlistSplitter, body);
  let hit = {};
  for (let d of distribution) {
    hit[d.categoryId] = Number(d.count);
  }
  
  // get unfiltered data
  body = false;
  if (categoryIds) body = "categoryIds=" + categoryIds;
  let originalDistribution = await fetchReq(sparqlet, body);
  for (let i = 0; i < originalDistribution.length; i++) {
    let hit_tmp = 0;
    if (hit[originalDistribution[i].categoryId]) hit_tmp = hit[originalDistribution[i].categoryId];
    originalDistribution[i].hit_count = hit_tmp;
  }

  // without p-value
  if (pValueFlag.nonPValue) return originalDistribution;

  // with pvalue (gene, protein)
  let calcPvalue = (a, b, c, d) => {
    // 不正数値検出
    let maxLimit = 300000; // ensembl_transcript: ~253,000
    // console.log([a,b,c,d].join(","));
    if (a < 0 || b < 0 || c < 0 || d < 0) return false;
    if (a > maxLimit || b > maxLimit || c > maxLimit || d > maxLimit) return false;
    
    let sigDigi = (num, exp) => {
      while (num > 10) {
        num /= 10;
        exp++;
      }
      while (num < 1) {
        num *= 10;
        exp--;
      }
      return [num, exp];
    }
    
    let calcProb = (a, b, c, d) => {
      // prob = num * 10 ** exp;
      let num = 1;
      let exp = 0; 
      for (let i = 1;         i <= a;             i++) { [num, exp] = sigDigi(num / i, exp); }  // 1/a!
      for (let i = b + 1;     i <= a + b;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+b)!/b!
      for (let i = c + 1;     i <= a + c;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+c)!/c!
      for (let i = d + 1;     i <= c + d;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (c+d)!/d!
      for (let i = b + d + 1; i <= a + b + c + d; i++) { [num, exp] = sigDigi(num / i, exp); }  // (b+d)!/(a+b+c+d)!
      return num * 10 ** exp;
    }
    
    let cutoffProb = calcProb(a, b, c, d);
  
    let max = a + b;
    if (max > a + c) max = a + c;
 
    let pValue = 0;
    for (let i = 0; i <= max; i++) {
      let delta = a - i;
      let tmpProb = 1;
      if (b + delta >= 0 && c + delta >= 0 && d - delta >= 0) tmpProb = calcProb(i, b + delta, c + delta, d - delta);
      if (tmpProb <= cutoffProb) pValue += tmpProb;
    }
    if (pValue > 0.9999) pValue = 1; // 有効数字このくらい？
    return pValue;
  }
  
  population = Number(population.results.bindings[0].total_count.value);
  let queries = queryIds.split(/,/).length;
  
  for (let i = 0; i < originalDistribution.length; i++) {
    if (originalDistribution[i].git_count == 0) continue;
    if (originalDistribution[i].hit_count == 1) originalDistribution[i].pValue = 1;
    else originalDistribution[i].pValue = calcPvalue(originalDistribution[i].hit_count - 1, queries - (originalDistribution[i].hit_count - 1), originalDistribution[i].count - (originalDistribution[i].hit_count - 1), population - originalDistribution[i].count - queries);
  }
  return originalDistribution;
}
```