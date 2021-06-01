# hoge

## `test`
```javascript
async ({}) => {
 // let url = "https://integbio.jp/togosite/sparqlist/api/uniprot_keywords_ligand";
  //let url = "http://localhost:3000/api/uniprot_keywords_ligand";
  let url = "http://sparqlist-togosite-2:3000/api/uniprot_keywords_ligand";
  return await fetch(url).then(d => d.json());
}
```