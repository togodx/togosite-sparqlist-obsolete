# TogoDX breakdown SPARQList 22.01.18

- 入れ子 SPARQList の parameters もそのうち修正する
  - categoryIds -> node
  - togokey -> dataset
  - inputIds -> queries
 
## Parameters

* `attribute` (Req.)
  * example: protein_molecular_function_uniprot
* `node`
  * example: GO_0008150

## `primaryIds`
```javascript
async ({attribute, node})=>{
  const fetchReq = async (url, options, body) => {
    if (body) options.body = body;
    /* //==== debug code
    let res = await fetch(url, options);
    console.log(res.status);
    console.log(url + "?" + body);
    return res.json();
    //==== */
    return await fetch(url, options).then(res=>res.json());
  }

  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  
  // attributes.json の order で sort
  // const togoDxAttributes = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/attributes.json";
  // const attributesJson = await fetchReq(togoDxAttributes, {method: "get"});
  // const order = "numerical_desc"; // default
  // if (attributesJson.attributes[attribute].order) order = attributesJson.attributes[attribute].order;

  let body = "";
  if (node) body = "categoryIds=" + node; // #### 入れ子 SPARQList. 要パラメータ名の整理
  let res = await fetchReq(attribute, options, body)
  return res.map(d => {
    let leaf = true;
    if (d.hasChild) leaf = false;
    return {
      node: d.categoryId,
      count: d.count,
      leaf: leaf,
      label: d.label
    }
  })                                            
}
```