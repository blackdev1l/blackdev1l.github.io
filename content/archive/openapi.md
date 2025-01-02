---
title: "OpenAPI"
date: 2020-01-18T23:13:51+01:00
draft: false
comments: false
images:
---

Uno dei propositi che mi sono dato per questo 2020 è stato quello di ricominciare a scrivere, scrivere su questo blog, scrivere in generale. 
Dopo 4 anni di assenza da questa piattaforma ricomincio parlando di un *framework* che ho introdotto in azienda e che mi sta dando tante soddisfaziono: Parlo della specifica OpenAPI, conosciuta anche come Swagger.


## Cos'è

Cito direttamente dal sito OpenAPI

> *The OpenAPI Specification: a broadly adopted industry standard for describing modern APIs.*  

Quello che significa questa semplice frase è che tramite un file .yml con regole definite dalla specifica OpenAPI possiamo generare tutto il codice boilerplate per esporre delle api REST sia lato backend, che lato frontend, inclusa la documentazione.

Questo ci torna molto utile perché possiamo astrarci dallo stock che utilizziamo e ci permette di contenere in un unico punto (single source of truth) tutta la conoscenza riguardo il design delle api, non solo: 
Iniziare un progetto con l'approccio __api-first__ ti permette di mettere in chiaro i contratti fin da subito, così da rimuovere incomprensioni tra cliente/fornitore o vari dipartimenti.  

OpenAPI non è solo indirizzato verso gli sviluppatori, ma può’ anche essere  utilizzato da PM/PO  visto che basta la conoscenza della specifica per poter operare sugli endpoint.

Mettere in piedi un endpoint è solo questione poche righe di yml, tramite `OpenAPI-generator` possiamo generare il codice lato client e server in più di 100 linguaggi/framework, rendendo trasparente la parte di infrastruttura dell'api e potendo concentrarci di più sulla parte di business logic. 

## Il bello

Iniziare ad adottare OpenAPI è facile, sia se stiamo parlando di un prodotto legacy, sia se abbiamo intenzione di creare un nuovo servizio. 

Possiamo utilizzarlo a qualsiasi livello: stilare una documentazione sulle api già presenti, introdurne delle nuove e generare il codice solo lato client, o solo lato backend. Mi sono ritrovato a introdurre nuove api su un progetto legacy senza stravolgere nulla, ho solo dovuto generare il codice con la libreria Web che già utilizzavamo, esportando solo le interfacce e utilizzandole con tutto lo stack legacy. 

Il progetto è estremamente in salute, ho trovato un riscontro molto positivo con la community, ho anche potuto contribuire in minima parte e i mantainer riescono a tenere basso il livello di accesso al progetto, con una buona organizzazione considerando che si parla di un code generator che supporta quasi tutti i linguaggi/librerie/framework moderni, diventando di fatto uno standard industriale. 

## Il non così bello 

Nel primo paragrafo, ho menzionato il fatto che OpenAPI ci permette di astrarci dallo stack che utilizziamo, e spesso questo non è sempre un bene, a volte ci troviamo di fronte a dei problemi __comuni__ già risolti dal framework o libreria che utilizziamo, e che per un motivo o per l'altro non possiamo entrare nei binari prestabiliti proprio perchè il generatore di OpenAPI non ci permette di avere i punti di entrata come se li aspetta la libreria, oppure perchè l'aggiungere un livello ulteriore allo stack applicativo porta ad un allungamento dei tempi di rilascio e di modifica all'applicazione, quello che voglio dire è che non tutto è oro quello che luccica, e decidere se adottare un approcio **api-first** dovrà essere valutato anche tenendo conto dei trade-off a cui si dovrà andar incontro.

#### Reference
[Petstore Example](https://editor.swagger.io/)  
[OpenAPI](https://OpenAPI-generator.tech/)
