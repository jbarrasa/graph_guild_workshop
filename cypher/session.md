Explore source data `/data/*.csv`

## Building the graph
Go to data importer `https://data-importer.neo4j.io/`

In data importer map and load data 

Import model in `other/northwind-with-mappings.json`


## Querying the graph in browser

### Subgraph for a given customer
```
match subgraph = (c1:Customer { companyName: 'Frankenversand'})-[:places_order]->(o:Order)-[:includes]->(:LineItem)-[:references]->(p:Product)-[:in_category]->(:Category)
return subgraph
```

### Products by customer
```
match (c1:Customer { companyName: 'Frankenversand'})-[:places_order]->(o:Order)-[:includes]->(:LineItem)-[:references]->(p:Product)
return p.productName, count(o) as ordercount
order by ordercount desc
```

### Customer overlap similarity by product
```
match (c1:Customer)-[:places_order]->(o:Order)-[:includes]->(:LineItem)-[:references]->(p:Product),
      (c2:Customer)-[:places_order]->(:Order)-[:includes]->(:LineItem)-[:references]->(p)
return c1.companyName, c2.companyName, count(distinct p) as prod_overlap order by prod_overlap desc
```


## Visual exploration in Bloom 

Create perspective

Take the previous query and generate a search phrase

Run algorithms in Bloom


## Additional topics

### Serialising the graph as RDF

`:get /rdf/neo4j/describe/3136?format=RDF/XML`

Other serialisation formats (Turtle, N-Triples, JSON-LD...)



### Import RDF data to enrich the graph 

`CREATE CONSTRAINT n10s_unique_uri FOR (r:Resource) REQUIRE r.uri IS UNIQUE`

`call n10s.graphconfig.init({ handleVocabUris: "IGNORE"})`

```call n10s.rdf.import.inline('

@prefix sch: <http://schema.org/> .
@prefix n4ind: <http://neo4j.individuals/> .

n4ind:p_1234 a sch:Product;
  sch:productID "101";
  sch:unitPrice 200;
  sch:unitsOnOrder "0"^^<http://www.w3.org/2001/XMLSchema#long>;
  sch:productName "Some Expensive Rioja";
  sch:reorderLevel "10"^^<http://www.w3.org/2001/XMLSchema#long>;
  sch:discontinued false;
  sch:unitsInStock "150"^^<http://www.w3.org/2001/XMLSchema#long>;
  sch:quantityPerUnit "6 - 1L bottles";
  sch:in_category n4ind:c_2345 .

n4ind:c_2345 a sch:Category;
  sch:categoryID "12";
  sch:categoryName "Fine Wines";
  sch:description "Top end wines and sparklings".

','Turtle')

```


### Create a data importer model from an OWL ontology

Use the `rdf/northwind-onto.ttl` ontology created with Protege (https://protege.stanford.edu/)

Generating data-importer model from ontology
`call n10s.experimental.export.dimodel.fetch("..path to file.../northwind-onto.ttl","Turtle")`

