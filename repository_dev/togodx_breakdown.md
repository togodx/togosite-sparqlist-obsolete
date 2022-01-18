# TogoDX breakdown SPARQList 22.01.18

- 入れ子 SPARQList の parameters もそのうち修正する
  - categoryIds -> node
  - togokey -> dataset
  - inputIds -> queries
 
## Parameters

* `attribute`
  * example: gene_high_level_expression_refex
* `node`
  * example: v32_40

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
  
  const togoDxAttributes = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/attributes.json";
  let body = "";
  if (node) body = "categoryIds=" + node; // #### 入れ子 SPARQList. 要パラメータ名の整理
  return fetchReq(attribute, options, body).map(d => {
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