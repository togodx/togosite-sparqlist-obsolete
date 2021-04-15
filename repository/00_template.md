# Title of the API

Some description for this API should be written.

SPARQList API can be called with content negotiation via mime type suffix or Accept: header.

```sh
# Default
curl 'http://example.org/sparqlist/api/00_template?param_name=value'

# JSON format
curl 'http://example.org/sparqlist/api/00_template.json?param_name=value'
curl -H 'Accept: application/json' 'http://example.org/sparqlist/api/00_template?param_name=value'

# Plain text format
curl 'http://example.org/sparqlist/api/00_template.text?param_name=value'
curl -H 'Accept: text/plain' 'http://example.org/sparqlist/api/00_template?param_name=value'

# HTML format
curl 'http://example.org/sparqlist/api/00_template.html?param_name=value'
curl -H 'Accept: text/html' 'http://example.org/sparqlist/api/00_template?param_name=value'
```

## Parameters

* `param_name` description
  * default: value
  * example: value2, value3

## Endpoint

http://example.org/sparql

## Query description `var_name`

SPARQL query to be executed at the above SPARQL endpoint.
Any variable can be embedded in the query with a `{{variable}}` format.
The result of query is stored in a variable `var_name` specified in this section header.

```sparql
SELECT * WHERE {
  VALUES ?var { "{{param_name}}" }
  ...
}
```

## Data transformation (optional) `result`

Any number of JavaScript codes and SPARQL queries can be included in the API.

```javascript
({var_name}) => {
  // data transformation function
  return val
}
```

## Output with content negotiation

Content-type negotiation aware JavaScript snippet with `json()`, `text()`, `html()` methods.

```javascript
({
  json({result}) {
    return result
  },
  text({result}) {
    let cols = result.head.vars
    let rows = result.results.bindings
    return cols.join("\t") + "\n" + rows.map(row => cols.map(col => row[col].value).join("\t")).join("\n")
  },
  html({result}) {
    let cols = result.head.vars;
    let rows = result.results.bindings;
    let str = ""
    str += "<table>\n"
    str += "<tr>"
    for (col of cols) {
      str += "<th>" + col + "</th>"
    }
    str += "</tr>\n"
    for (row of rows) {
      str += "<tr>"
      for (col of cols) {
        str += "<td>" + row[col].value + "</td>"
      }
      str += "</tr>\n"
    }
    str += "</table>"
    return str
  }
})
```

For HTML, hbs() method is prepared by SPARQList to embed multiple lines of HTML code to be processed with Handlebars.
To embed TogoStanza visualizing the result of this API, just replace the html() methods above with the following.

```javascript
  html: hbs(`
    <script src="https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@2.2.7/webcomponents-loader.js" crossorigin></script>
    <link rel="import" href="http://togostanza.org/dist/metastanza/htmltable/">
    <togostanza-htmltable sparql_api="http://example.org/sparqlist/api/00_template?param_name={{param_name}}" title="D3 htmltable"></togostanza-htmltable>
  `)
```
