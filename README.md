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
The objectives of these phases are as follows:
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
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E++%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E++%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E++%0D%0A%0D%0A+SELECT+DISTINCT+++%0D%0A%3FhasSubject+%3Flabel+++%0D%0AWHERE+%7B+++%0D%0A%3FhasSubject+rdfs%3Alabel+%3Flabel+++%0D%0AFILTER%28%3Flabel+%3D+%22Ges%C3%B9+Cristo%22%29++%0D%0A%7D%0D%0A%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on). 

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

![gem 1](https://github.com/user-attachments/assets/5077e88e-b578-42fc-9f58-fb472d3fabf6)


Asking the same question to Gemini, we noticed different types of responses between the two LLMs:​
+ As far as the first question is concerned, while ChatGPT answered “no”, Gemini answered “yes”​
+ Despite applying a generated knowledge prompting technique, Gemini did not provide us with much information compared to ChatGPT​

Once we know that «chiaroscuro» could be added as a technical feature, we asked ChatGPT through the **Few Shot Prompting** technique to provide us with the correct class in Arco:

![immagine](https://github.com/user-attachments/assets/5346239a-29c9-4d58-9899-cde17c675d85)
![chatgpt2](https://github.com/user-attachments/assets/a856d995-6fa4-4c77-8d81-8219733e3151)

Then, we asked it to transform this information into an RDF triple:

![Immagine 2024-09-09 170649](https://github.com/user-attachments/assets/e7e8da23-b0c2-4112-953d-20fffccc97d5)

We used the same technique on Gemini to compare the two answers: 

![Immagine 2024-09-09 170756](https://github.com/user-attachments/assets/36147f11-d692-434a-93fa-7a162936cf8a)

We asked to transform this information into an RDF triple using Arco ontology, however, Gemini did not include the class for "chiaroscuro" in its initial description but added it only after it was requested a second time:

![Immagine 2024-09-09 170938](https://github.com/user-attachments/assets/39c3652c-aa59-4035-8ab0-a7c6a22288b1)


In both cases, ChatGPT and Gemini gave us the incorrect class for "chiaroscuro". The solution was checking Arco's denotative description and correcting the class with **a-dd:includesTechnicalCharacteristic**

+ For **chiaroscuro**:
  
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
SELECT DISTINCT
?MaterialorTechnique ?label
WHERE {
?MaterialorTechnique rdfs:label ?label
FILTER(?label = "chiaroscuro")
}

```

[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0ASELECT+DISTINCT%0D%0A%3FMaterialorTechnique+%3Flabel%0D%0AWHERE+%7B%0D%0A%3FMaterialorTechnique+rdfs%3Alabel+%3Flabel%0D%0AFILTER%28%3Flabel+%3D+%22chiaroscuro%22%29%0D%0A%7D%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

#### Triple 3
***a-dd:includesTechnicalCharacteristic***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: a-dd:includesTechnicalCharacteristic [https://w3id.org/arco/ontology/denotative-description/TechnicalCharacteristic](https://w3id.org/arco/ontology/denotative-description/TechnicalCharacteristic) 
+ **Object (chiaroscuro)**: [https://w3id.org/arco/resource/TechnicalCharacteristic/chiaroscuro](https://w3id.org/arco/resource/TechnicalCharacteristic/chiaroscuro)

Since our artwork in Arco is titled as *Ultima Cena, Trinità, Santi*, we wanted to know whether there was a specific title to describe all of them. Furthermore the class for «title» was missing within the page.​ We asked ChatGPT through the Zero-Shot technique to provide us with this information: 

![Immagine 2024-09-09 171109](https://github.com/user-attachments/assets/34cb4f95-e2a8-45a5-884b-e9e9faf0faf7)

At this point, we wondered whether The Last Supper was the title attributed by the author himself or whether it had been attributed posthumously so that it could be associated with the right predicate: 

<img width="307" alt="Immagine7" src="https://github.com/user-attachments/assets/bf1ec9c5-ecee-4a01-9211-a1c956b351f9">



<img width="307" alt="Immagine7" src="https://github.com/user-attachments/assets/7da005fd-3521-438d-8757-0ccf6369b3b7">


This means that both *Ultima Cena* and *Il Cenacolo* are valid as titles​. So, the next step consists in formulating a query that looks simultaneously for the IRIs of both titles:

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

SELECT DISTINCT 
?hasTitle ?label 
WHERE { 
  { 
    ?hasTitle rdfs:label ?label .
    FILTER(?label = "Il Cenacolo")
  } 
  UNION 
  { 
    ?hasTitle rdfs:label ?label .
    FILTER(?label = "Ultima Cena")
  }
}
```
[Results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0APREFIX+arco%3A+%3Chttps%3A%2F%2Fw3id.org%2Farco%2Fontology%2Farco%2F%3E%0D%0A%0D%0ASELECT+DISTINCT+%0D%0A%3FhasTitle+%3Flabel+%0D%0AWHERE+%7B+%0D%0A++%7B+%0D%0A++++%3FhasTitle+rdfs%3Alabel+%3Flabel+.%0D%0A++++FILTER%28%3Flabel+%3D+%22Il+Cenacolo%22%29%0D%0A++%7D+%0D%0A++UNION+%0D%0A++%7B+%0D%0A++++%3FhasTitle+rdfs%3Alabel+%3Flabel+.%0D%0A++++FILTER%28%3Flabel+%3D+%22Ultima+Cena%22%29%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0A&format=text%2Fhtml&timeout=0&signal_void=on).

#### Triple 4
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: a-cd:hasTitle ([https://w3id.org/arco/ontology/context-description/Title​](https://w3id.org/arco/ontology/context-description/Title​)) 
+ **Object (Ultima Cena)**: [https://w3id.org/arco/resource/Title/0800437344-ultima-cena](https://w3id.org/arco/resource/Title/0800437344-ultima-cena)


#### Triple 5
***a-cd:hasTitle***
+ [**Subject**](https://dati.beniculturali.it/lodview-arco/resource/HistoricOrArtisticProperty/0900281487-0.html)
+ **Predicate**: a-cd:hasTitle ([https://w3id.org/arco/ontology/context-description/Title​](https://w3id.org/arco/ontology/context-description/Title​)) 
+ **Object (Il Cenacolo)**: [https://w3id.org/arco/resource/Title/0300158497-il-cenacolo](https://w3id.org/arco/resource/Title/0300158497-il-cenacolo)
