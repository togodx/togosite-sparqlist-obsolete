# plantgarden_pubtator_gene (信定)

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
prefix ont_pubtator: <http://togodb.org/ontology/pg_pubtator#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?pubtatorid  ?taxid ?organism   ?pmid     ?term    ?label  
from <http://plantgarden.jp/resource/pubtator>
where {
values ?concept {"Gene"}   
?s a  <http://togodb.org/pg_pubtator> ;
ont_pubtator:id ?pubtatorid ;  
rdfs:label ?label ;
ont_pubtator:bio_concepts ?concept ;    
ont_pubtator:organism  ?organism ;       
ont_pubtator:pmid ?pmid ;  
ont_pubtator:taxonomy_id  ?taxid ;     
ont_pubtator:term ?term .
} 
```

## `return`
```javascript
({main}) => {
  
    // leafをまとめる
    let pmid　= d.pmid.value ;
   let term = d.term.value ;
  let terms = term + '(pubmed_id: ' + pmid + ')' ;
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  
  let edge = {};
 main.results.bindings.map(d => {
    tree.push({
      id: d.label.value,
      label: d.label.value,
      leaf: true,
      parent: d.pubtatorid.value
    })
   
  
  tree.push({   
        id: d.pubtatorid.value,
        label: d.terms.value,
        leaf: false,
        parent: d.taxid.value
      })

      // root との親子関係を追加
    if (!edge[d.taxid.value]) {
      edge[d.taxid.value] = true;
      tree.push({   
        id: d.taxid.value,
        label: d.organism.value,
        leaf: false,
        parent: "root"
      })
    }
  });

  return tree;
};
```