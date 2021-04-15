# Diseaseカテゴリフィルタ(anatomical system) (フロント開発用 with hasChild flag. moriya をmode対応(3/18三橋))

- Required SPARQLet: disease_mondo_filter
 
 ## testURL
  - [default](http://ep6.dbcls.jp/togoid/sparqlist/api/disease_mondo_filter?categoryIds=0021199&queryIds=&mode=)
  - [catgoryId+queryId+idList](http://ep6.dbcls.jp/togoid/sparqlist/api/disease_mondo_filter?categoryIds=0021199&queryIds=0008903%2C0002691%2C0005260&mode=idList)
  - [catgoryId+queryId+objectList](http://ep6.dbcls.jp/togoid/sparqlist/api/disease_mondo_filter?categoryIds=0021199&queryIds=0008903%2C0002691%2C0005260&mode=objectList)

## Parameters

* `categoryIds` 指定したMONDOノードのリストの下位階層のノード数を返す。
  * default: 0021199 
  * example: 0021199 (anatomical system), 0004992, 0005071  (cancer, nervous system disorder),  Search MONDO at https://monarchinitiative.org/
* `queryIds` 数える対象(MONDO_ ID)リスト。ここで指定されたID群が各内訳に何個ずつ該当するかを返す。
  * example: 0008903,0002691,0005260    (liver cancer, lung cancer,autism (disease))
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList
  
## `httpreq`

```javascript
async ({queryIds, categoryId, categoryIds, mode})=>{
  let url = "https://integbio.jp/togosite/sparqlist/api/disease_mondo_filter"; // localhost:port を叩けると速い
  let options = {
    method: 'POST',
    body	: "categoryId=" + categoryId + "&categoryIds=" + encodeURIComponent(categoryIds) + "&queryIds=" + encodeURIComponent(queryIds) + "&mode=" + mode,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  