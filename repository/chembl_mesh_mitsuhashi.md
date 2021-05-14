# chembl mesh（山本, 守屋）→三橋改

* Top レベル ('C') はノードじゃ無いため URI もラベルもないので objectList 出力の Attribute は取れるもので最上位を出力

# Parameters

* `categoryIds` (type: mesh tree number)
  * default: C
  * example: C03.335
* `queryIds` (type: ChEMBL compound)
  * example: CHEMBL491473,CHEMBL313006,CHEMBL1201191,CHEMBL1201190,CHEMBL1201866,CHEMBL254316,CHEMBL1336,CHEMBL1200485,CHEMBL398435,CHEMBL298817,CHEMBL1201834,CHEMBL48361,CHEMBL560592,CHEMBL502182,CHEMBL27759,CHEMBL1088752,CHEMBL272980,CHEMBL377559,CHEMBL447955,CHEMBL271227
* `mode`
  * example: idList, objectList

## `top`
- Top レベルかどうかのチェック
```javascript
({categoryIds})=>{
  if (categoryIds.match(/^[A-Z]$/)) return true;
  return false;
}
```

## `categoryArray`
- category ID を配列に分割
```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql
* https://integbio.jp/rdf/mirror/mesh/sparql

## `targetMesh`
- mesh D番号 と目的 tree 階層の対応表
  - Top レベルだけ例外処理
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>
SELECT DISTINCT ?mesh ?tree ?label (SAMPLE(?child_tree) AS ?child)
#FROM <http://integbio.jp/rdf/mirror/ontology/mesh>
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
{{#if top}}
  ?tree a meshv:TreeNumber .
  MINUS { 
    ?tree meshv:parentTreeNumber ?parent .
  }
  FILTER (CONTAINS(STR(?tree),"mesh/{{categoryIds}}"))
{{else}}
  {{#if mode}}
  VALUES ?tree { {{#each categoryArray}} tree:{{this}} {{/each}} }
  {{else}}
  VALUES ?parent { {{#each categoryArray}} tree:{{this}} {{/each}} }
  ?tree meshv:parentTreeNumber ?parent .
  {{/if}}
{{/if}}
   ?tree ^meshv:treeNumber/rdfs:label ?label .
   ?mesh meshv:treeNumber/meshv:parentTreeNumber* ?tree .
   OPTIONAL {
     ?child_tree meshv:parentTreeNumber ?tree .
   }
}
```

## Endpoint
https://integbio.jp/togosite/sparql
* https://integbio.jp/rdf/mirror/ebi/sparql

## `chemblList`
- ChEMBL molecule - mesh D番号の対応リスト
  - 変数無いので結果がキャッシュに入れば早い
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
SELECT DISTINCT ?molecule ?mesh
#FROM <http://rdf.ebi.ac.uk/dataset/chembl>
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  ?molecule cco:hasDrugIndication [
    a cco:DrugIndication ;
    cco:hasMesh ?mesh
  ] .
}
```

## `count`
```javascript
({mode, targetMesh, chemblList, queryIds})=>{
  queryIds = queryIds.replace(/,/g, " ");
  let chemblFilter = [];
  if (queryIds.match(/[^\s]/)) chemblFilter = queryIds.split(/\s+/); 
  let meshList = targetMesh.results.bindings.map(d=>d.mesh.value.replace("http://id.nlm.nih.gov/mesh/", ""));
  let mesh2id = {};
  let id2label = {};
  let id2child = {};
  for (let d of targetMesh.results.bindings) {
    let mesh = d.mesh.value.replace("http://id.nlm.nih.gov/mesh/", "");
    let id = d.tree.value.replace("http://id.nlm.nih.gov/mesh/", ""); // tree number
    if (!mesh2id[mesh]) mesh2id[mesh] = [id];
    mesh2id[mesh].push(id);
    if (!id2label[id]) id2label[id] = d.label.value;
    id2child[id] = Boolean(d.child);
  }
  let id2chembl = {};
  let filteredList = [];
  for (let d of chemblList.results.bindings) {
    let mesh = d.mesh.value.replace("http://identifiers.org/mesh/", "");
    let chembl = d.molecule.value.replace("http://rdf.ebi.ac.uk/resource/chembl/molecule/", "");
    if (meshList.includes(mesh) && (chemblFilter.length == 0 || chemblFilter.includes(chembl))) {
      for (let id of mesh2id[mesh]) {
        if (!id2chembl[id]) id2chembl[id] = [];
        id2chembl[id].push(chembl);　// tree number と chembl 対応
        filteredList.push({
          id: chembl,
          attribute: {
            categoryId: id,
            uri: "http://id.nlm.nih.gov/mesh/" + id,
            labdl: id2label[id]
          }
        })
      }
    }
  }
  let id2count = {};
  for (let id of Object.keys(id2chembl)) {
    id2count[id] = Array.from(new Set(id2chembl[id])).length;  // chembl を unique してカウント
  }
  if (mode == "objectList") return filteredList;
  if (mode == "idList") return Array.from(new Set(filteredList.map(d=>d.id))); // chembl を unique
  
  return Object.keys(id2count).sort((a,b)=>{
    if (id2count[a] < id2count[b]) return 1;
    if (id2count[a] > id2count[b]) return -1;
    return 0;
  }).map(id=>{
    return {
      categoryId: id,
      label: id2label[id],
      count: id2count[id],
      hasChild: id2child[id]
    }
  });                        
}
```