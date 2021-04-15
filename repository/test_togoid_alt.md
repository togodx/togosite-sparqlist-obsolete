# test togoid alt (TogoID の代わり)

* source か target はのどちらかに hgnc, uniprot, chembl_compound, mondo のどれかが必要

## Parameters
* `source`
  * default: uniprot
  * example: hgnc, ncbigene, ensembl_gene, ensembl_transcript, uniprot, pdb, chembl_compound, pubchem_compound, mondo
* `target`
  * default: hgnc
  * example: hgnc, ncbigene, ensembl_gene, ensembl_transcript, uniprot, pdb, chembl_compound, pubchem_compound, mondo
* `ids`
  * default: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65

## `run`
```javascript
async ({source, target, ids})=>{
  let fetchReq = async (url, body)=>{
    let options = {
      method: 'POST',
      headers:	 {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    if (body.split(/,/).length > 2000) {
      options.body += "&sparqlet=" + encodeURIComponent(url);
      url = "https://sparql-support.dbcls.jp/rest/api/test_80kb_rec";
    }
    console.log(url);
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }
  
  let uniqueArray = (array)=>{
    return Array.from(new Set(array));
  }
  
  let map = (pair, pair2)=>{
    let medium = {};
    for (let d of pair2) {
      if (!medium[d.source]) medium[d.source] = [];
      medium[d.source].push(d.target);
    }
    let pairHash = {};
    for (let d of pair) {
      if (medium[d.target]) {
        for (let target of medium[d.target]) {
          pairHash[d.source + " " + target] = true;
        }
      }
    }
    return Object.keys(pairHash).map(d=>{
      let a = d.split(" ");
      return {source: a[0], target: a[1]}
    })
  }
  
  // 表記ゆれ修正
  if (source == "chembl") source == "chembl_compound";
  if (target == "chembl") target == "chembl_compound";
  source = source.replace(/\./g, "_");
  target = target.replace(/\./g, "_");

  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/test_togoid_";
  let tmpTarget = target;
  let tmpTarget2 = target;
  let tmpApi = false;
  let pair = [];
  let pair2 = []
  if (source == "uniprot") {
    if (target == "chembl_compound" || target == "pubchem_compound") { tmpTarget = "chembl_target"; tmpApi = "chembl"; }
    if (target == "mondo") { tmpTarget = "ncbigene"; tmpApi = "mondo"; }
    pair = await fetchReq(togoidApi + source, "source=" + source + "&target=" + tmpTarget + "&ids=" + ids);
    if (target != "chembl_compound" && target != "pubchem_compound" && target != "mondo") return pair;
    pair2 = await fetchReq(togoidApi + tmpApi, "source=" + tmpTarget + "&target=" + target + "&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    return map(pair, pair2);
  }
  if (target == "uniprot") {
    if (source != "chembl_compound" && source != "pubchem_compound" && source != "mondo") {
      return await fetchReq(togoidApi + "uniprot", "source=" + source + "&target=" + target + "&ids=" + ids);
    }
    if (source == "chembl_compound" || source == "pubchem_compound") { tmpTarget= "chembl_target"; tmpApi = "chembl"; }
    if (source == "mondo") { tmpTarget = "ncbigene"; tmpApi = "mondo"; }
    pair = await fetchReq(togoidApi + tmpApi, "source=" + source + "&target=" + tmpTarget + "&ids=" + ids);     
    pair2 = await fetchReq(togoidApi + target, "source=" + tmpTarget + "&target=" + target + "&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    return map(pair, pair2);
  }
  if (source == "hgnc") {
    if (target == "ncbigene" || target == "ensembl_gene" || target == "uniprot") {
      pair = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=" + target + "&ids=" + ids);
      return pair;
    }
    if (target == "ensembl_transcript") {
      pair = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ncbigene&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "mondo", "source=ncbigene&target=mondo&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);     
    }
    if (target == "pdb") {
      pair = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=uniprot&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=pdb&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);     
    }
    if (target == "mondo") {
      pair = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ensembl_gene&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "ensembl", "source=ensembl_gene&target=ensembl_transcript&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);     
    }
    if (target == "chembl_compound" || target == "pubchem_compound") {
      pair = await fetchReq(togoidApi + "uniprot", "source=hgnc&target=chembl_target&ids=" + ids);      
      pair2 = await fetchReq(togoidApi + "chembl", "source=chembl_target&target=" + target + "&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
  }
  if (target == "hgnc") {
    if (source == "chembl_compound" || source == "pubchem_compound") {
      pair = await fetchReq(togoidApi + "chembl", "source=" + source + "&target=chembl_target&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=chembl_target&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=uniprot&target=hgnc&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));
      return map(pair, pair2);
    }
    if (source == "pdb") {
      pair = await fetchReq(togoidApi + "uniprot", "source=pdb&target=uniprot&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=uniprot&target=hgnc&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    if (source == "mondo") {
      pair = await fetchReq(togoidApi + "mondo", "source=mondo&target=ncbigene&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=ncbigene&target=hgnc&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    if (source == "ensembl_transcript") {
      pair = await fetchReq(togoidApi + "ensembl", "source=ensembl_transcript&target=ensembl_gene&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=ensembl_gene&target=hgnc&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
	return await fetchReq(togoidApi + "hgnc", "source=" + source + "&target=" + target + "&ids=" + ids);   
  }
  if (source == "chembl_compound") {
    if (target == "pubchem_compound") return await fetchReq(togoidApi + "chembl", "source=chembl_compound&target=pubchem_compound&ids=" + ids);
    pair = await fetchReq(togoidApi + "chembl", "source=chembl_compound&target=chembl_target&ids=" + ids);
    pair2 = await fetchReq(togoidApi + "uniprot", "source=chembl_target&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    pair = map(pair, pair2);
    if (target == "mondo") {
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "mondo", "source=ncbigene&target=mondo&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=" + target + "&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
 	return map(pair, pair2);
  }
  if (target == "chembl_compound") {
    if (source == "pubchem_compound") return await fetchReq(togoidApi + "chembl", "source=pubchem_compound&target=chembl_compound&ids=" + ids);
    if (source == "mondo") {
      pair = await fetchReq(togoidApi + "mondo", "source=mondo&target=ncbigene&ids=" + ids);     
      pair2 = await fetchReq(togoidApi + "uniprot", "source=ncbigene&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=chembl_target&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "chembl", "source=chembl_target&target=chembl_compound&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    pair = await fetchReq(togoidApi + "uniprot", "source=" + source + "&target=uniprot&ids=" + ids);
    pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=chembl_target&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    pair = map(pair, pair2);
    pair2 = await fetchReq(togoidApi + "chembl", "source=chembl_target&target=chembl_compound&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    return map(pair, pair2);
  }
  if (source == "mondo") {
    pair = await fetchReq(togoidApi + "mondo", "source=mondo&target=ncbigene&ids=" + ids);
    if (target == "ncbigene") return pair;
    if (target == "pubchem_compound") {
      pair2 = await fetchReq(togoidApi + "uniprot", "source=ncbigene&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=chembl_target&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "chembl", "source=chembl_target&target=pubchem_compound&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    if (target == "pdb") {
      pair2 = await fetchReq(togoidApi + "uniprot", "source=ncbigene&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=pdb&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
    if (target == "ensembl_gene" || target == "ensembl_transcript") {
      pair2 = await fetchReq(togoidApi + "hgnc", "source=ncbigene&target=hgnc&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ensembl_gene&ids=" + uniqueArray(pair.map(d=>d.target)).join(",")); 
      pair = map(pair, pair2);
      if (target == "ensembl_gene") return pair;
      pair2 = await fetchReq(togoidApi + "ensembl", "source=ensembl_gene&target=ensembl_transcript&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      return map(pair, pair2);
    }
  }
  if (target == "mondo") {
    if (source == "ncbigene") return await fetchReq(togoidApi + "mondo", "source=ncbigene&target=mondo&ids=" + ids);
    if (source == "pdb") {
      pair = await fetchReq(togoidApi + "uniprot", "source=pdb&target=uniprot&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
    }
    if (source == "pubchem_compound") {
      pair = await fetchReq(togoidApi + "chembl", "source=pubchem_compound&target=chembl_target&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=chembl_target&target=uniprot&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "uniprot", "source=uniprot&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
    }
    if (source == "ensembl_gene") {
      pair = await fetchReq(togoidApi + "hgnc", "source=ensembl_gene&target=hgnc&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);     
    }
    if (source == "ensembl_transcipt") {
      pair = await fetchReq(togoidApi + "ensembl", "source=ensembl_transcript&target=ensembl_gene&ids=" + ids);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=ensembl_gene&target=hgnc&ids=" + ids);
      pair = map(pair, pair2);
      pair2 = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2); 
      pair2 = await fetchReq(togoidApi + "hgnc", "source=hgnc&target=ncbigene&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
      pair = map(pair, pair2);
    }
    pair2 = await fetchReq(togoidApi + "mondo", "source=ncbigene&target=mondo&ids=" + uniqueArray(pair.map(d=>d.target)).join(","));     
    return map(pair, pair2);
  }
}
```