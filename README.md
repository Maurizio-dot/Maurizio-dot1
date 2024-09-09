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

From this list we selected Andrea del Sarto. To verify the existence of properties authored by Andrea del Sarto within ArCo, we formulated the following ASK query:
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

Using SPARQL, we proceeded to count properties associated with Andrea del Sarto to assess how much information Arco held about him.

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

```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0A%0D%0ASELECT+%3Flabel+%28COUNT%28DISTINCT+%3FculturalProperty%29+AS+%3Fn%29%0D%0AWHERE+%7B%0D%0A++%3FculturalProperty+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0A++++++++++++++++%09a-cd%3AhasAuthor+%3Fauthor+.%0D%0A++%3Fauthor+rdfs%3Alabel+%3Flabel+.%0D%0A++FILTER%28REGEX%28%3Flabel%2C+%22Andrea+del+Sarto%22%2C+%22i%22%29%29%0D%0A%7D%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).  

With the quantitative data in hand, we need the titles of the entities we want to enrich. Therefore, we formulated a query to identify all paintings associated with Andrea del Sarto including the date, if present. The OPTIONAL keyword ensures that properties without dates are still included in the results:

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

We proceeded to a more detailed analysis by sorting these properties in descending order (“ORDER BY DESC”) according to the title. Narrowing the result to 50 entries, allowed us to better organize the data and highlight the most relevant artworks at the top of the list.

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>
PREFIX agent: <https://w3id.org/arco/resource/Agent/>
SELECT DISTINCT *
WHERE {
?culturalProperty a arco:HistoricOrArtisticProperty ;
rdfs:label ?title ;
a-cd:hasAuthor agent:e0239e64f8501627798e1a4d013a0f25
}
ORDER BY DESC (?title)
LIMIT 50
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0APREFIX+a-cd%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Fcontext-description%2F%3E%0D%0APREFIX+agent%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fresource%2FAgent%2F%3E%0D%0ASELECT+DISTINCT+*%0D%0AWHERE+%7B%0D%0A%3FculturalProperty+a+arco%3AHistoricOrArtisticProperty+%3B%0D%0Ardfs%3Alabel+%3Ftitle+%3B%0D%0Aa-cd%3AhasAuthor+agent%3Ae0239e64f8501627798e1a4d013a0f25%0D%0A%7D%0D%0AORDER+BY+DESC+%28%3Ftitle%29%0D%0ALIMIT+50%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).


From this list we selected [this artwork](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html). This property will serve as the starting point for the second phase of our project.

___

### Project phases 2 and 3: identifying missing data properties to create new RDF triples
The objectives of the second phase of our project are as follows:
+ **Identifying Missing Data Properties of the Artwork**;
+ **Employing Different Approaches with LLM to Retrieve Information**;
+ **Searching for the IRIs (Internationalized Resource Identifiers) of the Missing Properties**;
+ **Suggesting Improvements by Creating New RDF Triples**.
  
We started the enrichment from the subject of the work, which on ArCo includes several entities under one label: *Ultima cena, Trinità, Santi*. By contrast, we want to associate each of the depicted entities separately in order to provide a more detailed and precise description. For this reason, we asked ChatGPT through the **Zero-Shot CoT** technique to identify key subjects within the artwork and enrich their associations with the cultural property:

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

At this point, we created two new triples by linking our [subject](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html) to its respective objects using the *hasSubject* predicate: 

#### Triple 1
***A-cd:hasSubject***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: hasSubject [https://w3id.org/arco/ontology/context-description/hasSubject](https://w3id.org/arco/ontology/context-description/hasSubject)
+ **Object (Jesus Christ)**: [https://w3id.org/arco/resource/Subject/d767f5be4cd2be535a6d08e9019cca61](https://w3id.org/arco/resource/Subject/d767f5be4cd2be535a6d08e9019cca61​) 
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++%0D%0A%0D%0A+SELECT+DISTINCT+++%0D%0A%3FhasSubject+%3Flabel+++%0D%0AWHERE+%7B+++%0D%0A%3FhasSubject+rdfs%3Alabel+%3Flabel+++%0D%0AFILTER%28%3Flabel+%3D+%22Ges%C3%B9+Cristo%22%29++%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

#### Triple 2
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: hasSubject [https://w3id.org/arco/ontology/context-description/hasSubject](https://w3id.org/arco/ontology/context-description/hasSubject)
+ **Object (Apostoli)**: [https://w3id.org/arco/resource/Subject/cde2df599f8d502cb7d74d2dfed1a9db​](https://w3id.org/arco/resource/Subject/cde2df599f8d502cb7d74d2dfed1a9db​)

Concerning the material or technique of the artwork, in Arco it is classified as «pittura a fresco», however **chiaroscuro** appears to be a feature of this technique. Therefore, we asked ChatGPT to either confirm or deny our hypothesis through the **Generated Knowledge Prompting technique**: 

![immagine](https://github.com/user-attachments/assets/3572c308-6b64-44e3-9525-30849cdb9a56)
![immagine](https://github.com/user-attachments/assets/6ad8e22e-fbe0-4f8f-aab9-08c9ca24cb18)

For greater completeness, we asked Gemini the same question again:

![image](https://github.com/user-attachments/assets/ea4996d5-d976-497a-bfe2-7faf12dd0187)

Asking the same question to Gemini, we noticed different types of responses between the two LLMs:​
+ With the first question, while ChatGPT answered “no”, Gemini answered “yes”​
+ Despite applying a generated knowledge prompting technique, Gemini did not provide us with much information compared to ChatGPT​

Once we know that «chiaroscuro» could be added as a technical feature, we asked ChatGPT through the **Few Shot Prompting** technique to provide us with the correct class in Arco:

![immagine](https://github.com/user-attachments/assets/5346239a-29c9-4d58-9899-cde17c675d85)
![immagine](https://github.com/user-attachments/assets/76386f7f-bec2-4775-a317-135e9f306ea3)

Then, we asked it to transform this information into an RDF triple:

![immagine](https://github.com/user-attachments/assets/e8b7ce0e-6789-4587-92e2-f1ca2987d56b)


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
