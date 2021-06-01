# hoge

## `test`
```javascript
async ({}) => {
 // let url = "https://integbio.jp/togosite/sparqlist/api/uniprot_keywords_ligand"; // 外
  let url = "http://localhost:3000/api/uniprot_keywords_ligand"; // 内 (localhost)
  return await fetch(url).then(d => d.json());
}
```