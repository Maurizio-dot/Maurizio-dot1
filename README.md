# ITS project

___

[Methodology](https://maurizio-dot.github.io/Maurizio-dot/) [Project phases](https://maurizio-dot.github.io/Maurizio-dot1/) [Conclusion](https://maurizio-dot.github.io/Maurizio-dot2/) 

___

## Project phases
### 1. Topic identification 
In the initial phase of our project, our primary objectives were to identify a suitable topic and verify the availability of related data within the ArCo Knowledge Graph.
We used the Zero-Shot technique asking ChatGPT to generate a list of lesser-known painters based in Florence. Our hypothesis was that ArCo might have fewer entries concerning works by "minor" artists compared to well-known figures.

<img width="577" alt="Screenshot 2024-06-26 120120" src="https://github.com/Maurizio-dot/Maurizio-dot.github.io/assets/173699843/e37bcd50-6f29-46ed-bd95-bc384db8eec6">
![Cattura 1](https://github.com/Maurizio-dot/Maurizio-dot.github.io/assets/173814358/808ef577-f686-46c3-ba5d-4fa9ca5b4424)

From this list we selected Andrea del Sarto. To verify the existence of properties associated with Andrea del Sarto within ArCo, we formulated the following ASK query:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
ASK { 
?author a arco:HistoricOrArtisticProperty ;
rdfs:label ?label 
FILTER(REGEX(?label,  "Andrea del sarto", "i"))
}
```

The answer is [TRUE](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0AASK+%7B+%0D%0A%3Fauthor+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0Ardfs%3Alabel+%3Flabel+%0D%0AFILTER%28REGEX%28%3Flabel%2C++%22Andrea+del+sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

Using SPARQL, we proceeded to count and sort properties associated with Andrea del Sarto by ordering them in descending order ("ORDER BY DESC"):

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>
PREFIX agent: <https://w3id.org/arco/resource/Agent/>
SELECT DISTINCT ?label COUNT(DISTINCT ?culturalProperty) AS ?n
WHERE {
?culturalProperty a arco:HistoricOrArtisticProperty ;
a-cd:hasAuthor ?author .
?author rdfs:label ?label
FILTER(REGEX(?label, "Andrea del Sarto", "i"))
}
ORDER BY DESC(?n)
LIMIT 100
}
```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0A+PREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0A+PREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0A+PREFIX+agent%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fresource%2FAgent%2F%3E%0D%0A+SELECT+DISTINCT+%3Flabel+COUNT%28DISTINCT+%3FculturalProperty%29+AS+%3Fn%0D%0A+WHERE+%7B%0D%0A+%3FculturalProperty+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0A+a-cd%3AhasAuthor+%3Fauthor+.%0D%0A+%3Fauthor+rdfs%3Alabel+%3Flabel%0D%0A+FILTER%28REGEX%28%3Flabel%2C+%22Andrea+del+Sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0A+ORDER+BY+DESC%28%3Fn%29%0D%0A+LIMIT+100%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).  

With the quantitative data in hand, we need the titles of the entities we want to enrich. Therefore, we asked ChatGPT to formulate a query to identify all paintings associated with Andrea del Sarto including the date, if present:

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

SELECT ?author ?label ?creationDate
WHERE { 
  ?author a arco:HistoricOrArtisticProperty ;
          rdfs:label ?label ;
          arco:creationDate ?creationDate .
  FILTER(REGEX(?label, "Andrea del Sarto", "i"))
}
ORDER BY ?creationDate
```

Since `?creationDate` does not exist in ArCo, we asked ChatGPT to modify the query by replacing it with `?Date`.
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT ?author ?label ?date
WHERE { 
  ?author a arco:HistoricOrArtisticProperty ;
          rdfs:label ?label .
  OPTIONAL { ?author dcterms:date ?date } .
  FILTER(REGEX(?label, "Andrea del Sarto", "i"))
}
ORDER BY ?date
```
We made further modifications to the initial query:
1. Modified `SELECT ?author ?label ?date` to `SELECT DISTINCT ?paintings ?label` to eliminate duplicates and include all paintings with the date, if present;
2. Removed `ORDER BY ?date` as it was irrelevant in this context, and added `LIMIT 20` to narrow down the results.
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?paintings ?label
 WHERE {
?paintings rdfs:label ?label
OPTIONAL { ?author dcterms:date ?date } .
FILTER(REGEX(?label,  "Andrea del Sarto", "i"))
}
LIMIT 20
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+dcterms%3A+%3Chttp%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%3E%0D%0A%0D%0ASELECT+DISTINCT+%3Fpaintings+%3Flabel%0D%0A+WHERE+%7B%0D%0A%3Fpaintings+rdfs%3Alabel+%3Flabel%0D%0AOPTIONAL+%7B+%3Fauthor+dcterms%3Adate+%3Fdate+%7D+.%0D%0AFILTER%28REGEX%28%3Flabel%2C++%22Andrea+del+Sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0ALIMIT+20%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

As a data verification we added to the previous query the `UNION` keyword to merge results from two patterns, one related to works by "Andrea del Sarto" and another related to works with "Cenacolo" in their labels:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?item ?label ?date
WHERE {
  {
	?item rdfs:label ?label .
	?item dcterms:creator ?author .
	?author rdfs:label ?authorLabel .
	FILTER(REGEX(?authorLabel, "Andrea del Sarto", "i")) .
	OPTIONAL { ?item dcterms:date ?date } .
  } UNION {
	?item rdfs:label ?label .
	FILTER(REGEX(?label, "Cenacolo", "i")) .
  }
}
LIMIT 20
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+dcterms%3A+%3Chttp%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%3E%0D%0A%0D%0ASELECT+DISTINCT+%3Fitem+%3Flabel+%3Fdate%0D%0AWHERE+%7B%0D%0A++%7B%0D%0A%09%3Fitem+rdfs%3Alabel+%3Flabel+.%0D%0A%09%3Fitem+dcterms%3Acreator+%3Fauthor+.%0D%0A%09%3Fauthor+rdfs%3Alabel+%3FauthorLabel+.%0D%0A%09FILTER%28REGEX%28%3FauthorLabel%2C+%22Andrea+del+Sarto%22%2C+%22i%22%29%29+.%0D%0A%09OPTIONAL+%7B+%3Fitem+dcterms%3Adate+%3Fdate+%7D+.%0D%0A++%7D+UNION+%7B%0D%0A%09%3Fitem+rdfs%3Alabel+%3Flabel+.%0D%0A%09FILTER%28REGEX%28%3Flabel%2C+%22Cenacolo%22%2C+%22i%22%29%29+.%0D%0A++%7D%0D%0A%7D%0D%0ALIMIT+20%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

From this list, we identified one painting that did not already have a significant amount of information associated with it: [click here to visualise it](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html). This property will serve as the starting point for the second phase of our project.

___

### 2. Identifying missing data properties
The second phase of our project involved searching for the IRIs (Internationalized Resource Identifiers) of the missing properties associated with the selected painting.
In order to enrich the artwork with relevant triples, we employed Gemini using the Zero-Shot CoT technique to provide us with all the necessary information for its description:

<img width="456" alt="Immagine 2024-06-26 141056" src="https://github.com/Maurizio-dot/Maurizio-dot.github.io/assets/173699843/704f68f8-a4c6-46a5-a651-52fcf96d34aa"> 

For each of these properties (author, subjects, type of artwork, material and technique, and location), we identified their respective IRIs using the following queries:
+ For **author**:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX arco: <https://w3id.org/arco/ontology/arco/> 
 SELECT DISTINCT  
?hasAuthor ?label  
WHERE {  
?hasAuthor rdfs:label ?label  
FILTER(?label = "Andrea Del Sarto") 
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E+%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E+%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E+%0D%0A+%0D%0ASELECT+DISTINCT++%0D%0A%3FhasAuthor+%3Flabel++%0D%0AWHERE+%7B++%0D%0A%0D%0A%3FhasAuthor+rdfs%3Alabel+%3Flabel++%0D%0AFILTER%28%3Flabel+%3D+%22Andrea+Del+Sarto%22%29+%0D%0A%7D%0D%0A%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

+ For **subject**:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>  
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>  
PREFIX arco: <https://w3id.org/arco/ontology/arco/>  
 SELECT DISTINCT   
?hasSubject ?label   
WHERE {   
?hasSubject rdfs:label ?label   
FILTER(?label = "Ultima Cena")  
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++%0D%0A+SELECT+DISTINCT+++%0D%0A%3FhasSubject+%3Flabel+++%0D%0AWHERE+%7B+++%0D%0A%3FhasSubject+rdfs%3Alabel+%3Flabel+++%0D%0AFILTER%28%3Flabel+%3D+%22Ultima+Cena%22%29++%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

+ For **title**:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
SELECT DISTINCT 
?hasTitle ?label 
WHERE { 
?hasTitle rdfs:label ?label 
FILTER(?label = "Il Cenacolo")
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0ASELECT+DISTINCT+%0D%0A%3FhasTitle+%3Flabel+%0D%0AWHERE+%7B+%0D%0A%3FhasTitle+rdfs%3Alabel+%3Flabel+%0D%0AFILTER%28%3Flabel+%3D+%22Il+Cenacolo%22%29%0D%0A%7D%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

+ For **cultural property address**:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>   
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>   
PREFIX arco: <https://w3id.org/arco/ontology/arco/>   
 SELECT DISTINCT    
?hasCulturalPropertyAddress?label    
WHERE {    
?hasCulturalPropertyAddress rdfs:label ?label    
FILTER(?label = "Toscana, FI, Firenze")   
} 
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E+++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E+++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E+++%0D%0A+SELECT+DISTINCT++++%0D%0A%3FhasCulturalPropertyAddress%3Flabel++++%0D%0AWHERE+%7B++++%0D%0A%3FhasCulturalPropertyAddress+rdfs%3Alabel+%3Flabel++++%0D%0AFILTER%28%3Flabel+%3D+%22Toscana%2C+FI%2C+Firenze%22%29+++%0D%0A%7D+%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

+ For **cultural institute or site**:
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>    
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>    
PREFIX arco: <https://w3id.org/arco/ontology/arco/>    
 SELECT DISTINCT     
?hasCulturalInstituteOrSite ?label     
WHERE {     
?hasCulturalInstituteOrSite rdfs:label ?label     
FILTER(?label = "Museo del Cenacolo di Andrea del Sarto")    
}  
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++++%0D%0A+SELECT+DISTINCT+++++%0D%0A%3FhasCulturalInstituteOrSite+%3Flabel+++++%0D%0AWHERE+%7B+++++%0D%0A%3FhasCulturalInstituteOrSite+rdfs%3Alabel+%3Flabel+++++%0D%0AFILTER%28%3Flabel+%3D+%22Museo+del+Cenacolo+di+Andrea+del+Sarto%22%29++++%0D%0A%7D++%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on.).

Only for "affresco" we were unable to identify its corresponding property ("Type of Artwork"). Therefore, we directly asked ChatGPT to provide us with the correct property using the Few-Shot Prompting technique:

<img width="482" alt="lol" src="https://github.com/Maurizio-dot/Maurizio-dot.github.io/assets/173699843/32cc0882-c82d-4c82-931a-142704bfef93">

The problem is that the property "hasType" does not exist in ArCo's Denotative Description. The closest property we found is "hasCulturalPropertyType." Therefore, we revised the query, replacing "hasType" with "hasCulturalPropertyType":

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
SELECT DISTINCT 
?hasCulturalPropertyType ?label 
WHERE { 
?hasCulturalPropertyType rdfs:label ?label 
FILTER(?label = "affresco")
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0ASELECT+DISTINCT+%0D%0A%3FhasCulturalPropertyType+%3Flabel+%0D%0AWHERE+%7B+%0D%0A%3FhasCulturalPropertyType+rdfs%3Alabel+%3Flabel+%0D%0AFILTER%28%3Flabel+%3D+%22affresco%22%29%0D%0A%7D%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

In Gemini, the term "affresco" was used to describe both the painting technique and the material used to execute it. However, in ArCo, these aspects are grouped together under a single property, namely: a-dd:hasMaterialorTechnique.
As a final check, we asked ChatGPT using the Generated Knowledge Technique to determine if "pittura a fresco" and "affresco" meant the same thing:

<img width="526" alt="non ce la faccio piu" src="https://github.com/Maurizio-dot/Maurizio-dot.github.io/assets/173699843/ad298b60-d56b-458c-95ba-10ff0a837ba6"> 

Therefore, to search for the IRI corresponding to the word "affresco" or "pittura a fresco", we created the following query for `a-dd:MaterialOrTechnique`:

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

SELECT DISTINCT 
?MaterialorTechnique ?label 
WHERE { 
?MaterialorTechnique rdfs:label ?label 
FILTER(?label = "pittura a fresco")
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0A%0D%0ASELECT+DISTINCT+%0D%0A%3FMaterialorTechnique+%3Flabel+%0D%0AWHERE+%7B+%0D%0A%3FMaterialorTechnique+rdfs%3Alabel+%3Flabel+%0D%0AFILTER%28%3Flabel+%3D+%22pittura+a+fresco%22%29%0D%0A%7D%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

___

### 3. Creating new RDF triples
The third and final phase of the project involves creating triples to link to the entity "Il Cenacolo di Andrea del Sarto, Ultima Cena." We created new sets of triples by linking our [subject](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html) to its respective objects using adequate predicates: 

#### Triple 1
***A-cd:hasAuthor***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: hasAuthor
+ **Object (Andrea del Sarto)**: [https://w3id.org/arco/resource/Agent/11c57796fc5920fdba4b92fd459c0200](https://w3id.org/arco/resource/Agent/11c57796fc5920fdba4b92fd459c0200)

#### Triple 2
***A-cd:hasSubject***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: hasSubject [https://w3id.org/arco/ontology/context-description/hasSubject](https://w3id.org/arco/ontology/context-description/hasSubject)
+ **Object (Ultima Cena)**: [https://w3id.org/arco/resource/Subject/fff38876918a8a316e826d2ccc81f537](https://w3id.org/arco/resource/Subject/fff38876918a8a316e826d2ccc81f537)

#### Triple 3
***A-cd:hasTitle***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: a-cd:hasTitle [https://w3id.org/arco/ontology/context-description/Title](https://w3id.org/arco/ontology/context-description/Title) 
+ **Object (Il Cenacolo)**: [https://w3id.org/arco/resource/Title/0300179928-il-cenacolo](https://w3id.org/arco/resource/Title/0300179928-il-cenacolo)

#### Triple 4
***A-loc:hasCulturalPropertyAddress***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: a-loc:hasCulturalPropertyAddress ([a-loc:hasCulturalPropertyAddress](a-loc:hasCulturalPropertyAddress))
+ **Object (Toscana, FI, Firenze)**: [https://dati.beniculturali.it/lodview-arco/resource/Address/4287e9f9192e011971297d4609f2f41c.html](https://dati.beniculturali.it/lodview-arco/resource/Address/4287e9f9192e011971297d4609f2f41c.html) 

#### Triple 5
***A-loc:hasCulturalInstituteOrSite***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: a-loc:hasCulturalInstituteOrSite ([https://w3id.org/arco/ontology/location/hasCulturalInstituteOrSite](https://w3id.org/arco/ontology/location/hasCulturalInstituteOrSite)) 
+ **Object (Museo del Cenacolo di Andrea del Sarto)**: [https://w3id.org/arco/resource/CulturalInstituteOrSite/610717aa4ebd312c305eb06dcc4d740a](https://w3id.org/arco/resource/CulturalInstituteOrSite/610717aa4ebd312c305eb06dcc4d740a)

#### Triple 6
***A-dd:hasMaterialorTechnique***
+ [**Subject**](https://dati.beniculturali.it/lodview/mibact/eventi/resource/CreativeWork/13596_.html)
+ **Predicate**: a-dd:hasMaterialorTechnique ([https://w3id.org/arco/ontology/denotative-description/MaterialOrTechnique](https://w3id.org/arco/ontology/denotative-description/MaterialOrTechnique))
+ **Object (intonaco / pittura a fresco)**: [https://w3id.org/arco/resource/TechnicalCharacteristic/intonaco-pittura-a-fresco](https://w3id.org/arco/resource/TechnicalCharacteristic/intonaco-pittura-a-fresco) 
