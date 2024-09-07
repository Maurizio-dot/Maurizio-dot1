# ITS project

___

[Home](https://maurizio-dot.github.io/) [Methodology](https://maurizio-dot.github.io/Maurizio-dot/) [Project phases](https://maurizio-dot.github.io/Maurizio-dot1/) [Conclusion](https://maurizio-dot.github.io/Maurizio-dot2/) 

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
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>
 
ASK {
  ?culturalProperty a arco:HistoricOrArtisticProperty ;
                	a-cd:hasAuthor ?author.
  ?author rdfs:label ?label.
  FILTER(REGEX(?label, "Andrea del Sarto", "i"))
}

```

The answer is [TRUE](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0A+%0D%0AASK+%7B%0D%0A++%3FculturalProperty+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0A++++++++++++++++%09a-cd%3AhasAuthor+%3Fauthor.%0D%0A++%3Fauthor+rdfs%3Alabel+%3Flabel.%0D%0A++FILTER%28REGEX%28%3Flabel%2C+%22Andrea+del+Sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0A&format=auto&timeout=0&signal_void=on).

Using SPARQL, we proceeded to count and sort properties associated with Andrea del Sarto by ordering them in descending order ("ORDER BY DESC"):

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>

SELECT ?label (COUNT(DISTINCT ?culturalProperty) AS ?n)
WHERE {
  ?culturalProperty a arco:HistoricOrArtisticProperty ;
                	a-cd:hasAuthor ?author .
  ?author rdfs:label ?label .
  FILTER(REGEX(?label, "Andrea del Sarto", "i"))
}
ORDER BY DESC(?n)

```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0A%0D%0ASELECT+%3Flabel+%28COUNT%28DISTINCT+%3FculturalProperty%29+AS+%3Fn%29%0D%0AWHERE+%7B%0D%0A++%3FculturalProperty+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0A++++++++++++++++%09a-cd%3AhasAuthor+%3Fauthor+.%0D%0A++%3Fauthor+rdfs%3Alabel+%3Flabel+.%0D%0A++FILTER%28REGEX%28%3Flabel%2C+%22Andrea+del+Sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0AORDER+BY+DESC%28%3Fn%29%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).  

With the quantitative data in hand, we need the titles of the entities we want to enrich. Therefore, we formulated a query to identify all paintings associated with Andrea del Sarto including the date, if present:

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?cp ?label
 WHERE {
  ?cp a arco:HistoricOrArtisticProperty ;
  	rdfs:label ?label .
  OPTIONAL { ?cp dcterms:date ?date } .
  FILTER(LANG(?label) = "it") .
  FILTER(REGEX(?label, "Andrea del sarto", "i"))
}

```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+dcterms%3A+%3Chttp%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%3E%0D%0A%0D%0ASELECT+DISTINCT+%3Fcp+%3Flabel%0D%0A+WHERE+%7B%0D%0A++%3Fcp+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0A++%09rdfs%3Alabel+%3Flabel+.%0D%0A++OPTIONAL+%7B+%3Fcp+dcterms%3Adate+%3Fdate+%7D+.%0D%0A++FILTER%28LANG%28%3Flabel%29+%3D+%22it%22%29+.%0D%0A++FILTER%28REGEX%28%3Flabel%2C+%22Andrea+del+sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

From this list we selected [this artwork](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html). This property will serve as the starting point for the second phase of our project.

**UNION???**

___

### 2. Identifying missing data properties
The second phase of our project involved searching for the IRIs (Internationalized Resource Identifiers) of the missing properties associated with the selected painting.
In order to enrich the artwork with relevant triples, we employed both Gemini and ChatGPT using the Zero-Shot CoT technique to provide us with all the necessary information for its description:

![Gemini](<img width="720" alt="GEMINI1" src="https://github.com/user-attachments/assets/74189669-aa22-40b4-af91-e38c928471a1">)
![ChatGPT](https://github.com/user-attachments/assets/00af153f-bc2a-402d-a00e-f91bfa8a53f6)
![ChatGPT1](https://github.com/user-attachments/assets/1dc6cdf9-efbc-4396-8d9d-1d3aa4b3dd73)

For each of these subjects, we identified their respective IRIs using the following queries:

+ For **Jesus Christ**:

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>  
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>  
PREFIX arco: <https://w3id.org/arco/ontology/arco/>  

 SELECT DISTINCT   
?hasSubject ?label   
WHERE {   
?hasSubject rdfs:label ?label   
FILTER(?label = "Gesù Cristo")  
}

```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++%0D%0A%0D%0A+SELECT+DISTINCT+++%0D%0A%3FhasSubject+%3Flabel+++%0D%0AWHERE+%7B+++%0D%0A%3FhasSubject+rdfs%3Alabel+%3Flabel+++%0D%0AFILTER%28%3Flabel+%3D+%22Ges%C3%B9+Cristo%22%29++%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

+ For **Apostoli**:
  
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>  
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>  
PREFIX arco: <https://w3id.org/arco/ontology/arco/>  
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>
 SELECT DISTINCT   
?hasSubject ?label   
WHERE {   
?hasSubject rdfs:label ?label   
FILTER(?label = "Apostoli")  
}

```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++%0D%0APREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0A+SELECT+DISTINCT+++%0D%0A%3FhasSubject+%3Flabel+++%0D%0AWHERE+%7B+++%0D%0A%3FhasSubject+rdfs%3Alabel+%3Flabel+++%0D%0AFILTER%28%3Flabel+%3D+%22Apostoli%22%29++%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

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

AGGIUNGERE PARTE SU CHIAROSCURO
___

### 3. Creating new RDF triples
The third and final phase of the project involves creating triples to link to the artwork. We created new sets of triples by linking our [subject](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html) to its respective objects using adequate predicates: 

#### Triple 1
***A-cd:hasAuthor***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: hasSubject [https://w3id.org/arco/ontology/context-description/hasSubject](https://w3id.org/arco/ontology/context-description/hasSubject)
+ **Object (Jesus Christ)**: [https://w3id.org/arco/resource/Subject/d767f5be4cd2be535a6d08e9019cca61](https://w3id.org/arco/resource/Subject/d767f5be4cd2be535a6d08e9019cca61​) 

#### Triple 2
***A-cd:hasSubject***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: hasSubject [https://w3id.org/arco/ontology/context-description/hasSubject](https://w3id.org/arco/ontology/context-description/hasSubject)
+ **Object (Apostoli)**: [https://w3id.org/arco/resource/Subject/cde2df599f8d502cb7d74d2dfed1a9db​](https://w3id.org/arco/resource/Subject/cde2df599f8d502cb7d74d2dfed1a9db​)

#### Triple 3
***A-cd:hasTitle***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**:a-dd:includesTechnicalCharacteristic [https://w3id.org/arco/ontology/denotative-description/TechnicalCharacteristic](https://w3id.org/arco/ontology/denotative-description/TechnicalCharacteristic) 
+ **Object (chiaroscuro)**: [https://w3id.org/arco/resource/TechnicalCharacteristic/chiaroscuro](https://w3id.org/arco/resource/TechnicalCharacteristic/chiaroscuro)

#### Triple 4
***Circumstance Type***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: Circumstance Type [https://w3id.org/arco/ontology/context-description/CircumstanceType​](https://w3id.org/arco/ontology/context-description/CircumstanceType​)
+ **Object (Religious Circumstance)**: [https://w3id.org/arco/ontology/context-description/ReligiousCircumstance](https://w3id.org/arco/ontology/context-description/ReligiousCircumstance​) 

#### Triple 5
***a-cd:hasTitle***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: a-cd:hasTitle ([https://w3id.org/arco/ontology/context-description/Title​](https://w3id.org/arco/ontology/context-description/Title​)) 
+ **Object (Il Cenacolo)**: [https://w3id.org/arco/resource/Title/0300158497-il-cenacolo](https://w3id.org/arco/resource/Title/0300158497-il-cenacolo)
