# TogoID dev: route を生成して、TogoID API で変換

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: mondo
* `ids`
  * default: Q3SXY8,O15078,O43924,Q9NPI0,Q9UMX1,Q9H6L2,Q68DC2,Q9BPU9,O60308,O15259,Q68CZ1,Q8N157,Q9P2K1,Q9H799,Q96BN6,O75665,Q7Z4L5,Q96GX1,Q2M1K9,Q9NRR6,Q9BYV8,Q2MV58,Q96Q45,P36405,Q8WXW3,Q9P0N5,Q7Z3E5,Q9UPM9,Q6NUS6,Q8N960,Q1MSJ5,Q2M1P5,O60303,Q9BVV6,Q9NXB0,Q5HYA8
## `route`

```javascript
({source, target}) => {
  let config = {
    database: {
      variant: ["togovar"],
      gene: ["hgnc", "ncbigene", "ensembl_gene", "ensembl_transcript"],
      protein: ["uniprot", "chembl_target"],
      structure: ["pdb"],
      chembl_compound: ["chembl_compound"],
      compound: ["pubchem_compound", "chebi"],
      nando: ["nando"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "hp", "mesh"]
    },
    route: {
      variant: {
        variant: [],
        gene: [],
        protein: ["hgnc", "uniprot"],
        structure: ["hgnc", "uniprot", "pdb"],
        chembl_compound: ["hgnc", "uniprot", "chembl_target", "chembl_compound"],
        compound: ["hgnc", "uniprot", "reactome_reaction", "chebi"],
        nando: ["medgen", "mondo"],
        disease: ["medgen"]
      },
      gene: {
        gene: [],
        protein: ["uniprot"],
        structure: ["uniprot", "pdb"],
        chembl_compound: ["uniprot", "chembl_target", "chembl_compound"],
        compound: ["uniprot", "reactome_reaction", "chebi"],
        nando: ["ncbigene", "medgen", "mondo"],
        disease: ["ncbigene", "medgen"]
      },
      protein: {
        protein: [],
        structure: [],
        chembl_compound: ["chembl_target", "chembl_compound"],
        compound: ["reactome_reaction", "chebi"],
        nando: ["ncbigene", "medgen", "mondo"],
        disease: ["ncbigene", "medgen"]
      },
      structure: {
        structure: [],
        chembl_compound: ["uniprot", "chembl_target", "chembl_compound"],
        compound: ["uniprot", "reactome_reaction", "chebi"],
        nando: ["uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["uniprot", "ncbigene", "medgen"]
      },
      chembl_compound: {
        chembl_compund: [],
        compound: [],
        nando: ["chembl_target", "uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["chembl_target", "uniprot", "ncbigene", "medgen"]
      },
      compound: {
        compound: [],
        nando: ["chebi", "reactome_reaction", "uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["chebi", "reactome_reaction", "uniprot", "ncbigene", "medgen"]
      },
      nando: {
        nando: [],
        disease: ["mondo"]
      },
      disease: {
        disease: ["mondo"]
      }
    }
  };
 
  let sourceSubject, targetSubject;
  for (let subject of Object.keys(config.database)) {
    if (config.database[subject].includes(source)) sourceSubject = subject;
    if (config.database[subject].includes(target)) targetSubject = subject;
  }
  
  let makeRoute = (source, target, route) => {
    // 例外処理
    if (route[0] == "mondo" && source == "hp") route.unshift("medgen");  // hp - medgen - mondo 
    if (route[route.length - 1] == "mondo" && target == "hp") route.push("medgen"); // mondo - medgen - hp
    route.unshift(source);
    route.push(target);
    return route.filter((x, i, self) => self.indexOf(x) === i);
  }
  
  for (let subject_1 of Object.keys(config.route)) {
    for (let subject_2 of Object.keys(config.route[subject_1])) {
      if (subject_1 == sourceSubject && subject_2 == targetSubject) {
        return makeRoute(source, target, config.route[subject_1][subject_2]);
      }
      if (subject_1 == targetSubject && subject_2 == sourceSubject) {
        return makeRoute(source, target, config.route[subject_1][subject_2].reverse());
      }
    }
  }
}
```

## `togoid`
```javascript
async ({ids, route})=>{
  let fetchReq = async (url, body) => {
    let options = {	
      method: 'post',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
   // url += "?" + body;  // fot get
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }
  
  let togoidApi = "https://api.togoid.dbcls.jp.il3c.com/convert";
  let resLimit = 10000; // togoid API limit
  
  let body = "format=json&include=pair";
  body += "&route=" + route.join(",") + "&ids=" + ids;
  let json = await fetchReq(togoidApi, body);
  if (json.total > resLimit) {
    let loop = parseInt(json.total / resLimit) + 1;
    for (let i = 1; i <= loop; i++){
      let tmp = await fetchReq(togoidApi, body + "&offset=" + (resLimit * i));
      json.results = json.results.concat(tmp.results);
    }
  }
  let pair = json.results.map(d=>{
    return {
      source_id: d[0],
      target_id: d[1]
    }
  });
  return Array.from(new Set(pair)); // unique
}
```
