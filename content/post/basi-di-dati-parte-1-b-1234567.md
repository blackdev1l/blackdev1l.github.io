+++
date = "2014-10-23T16:58:38"
draft = "false"
title = "Basi di dati parte 1: proprietà"
slug = "basi-di-dati-parte-1-b-1234567"

+++

###Nozioni

**Istanza di entità:** Cosa o persona che esiste, della quale vogliamo registrare  gli attributi e che puo' essere chiaramente indentificata in modo da poterla distringuere dalle altre.  
**Istanza di relazione (associazione):** Fatto che descrive un'azione e che stabilisce legami tra istanza di entità.

**Classi:** Insieme di *istanze* considerate dello stesso *tipo*.  
**Proprietà (o attributi):** fatti che descrivono le caratteristiche delle istanze di
entità e le caratteristiche delle istanze di associazione.  
**Aggregazione:** Permette di definire il tipo delle istanze e delle classi con *aggregazioni* di proprietà comuni.

****

###Vincoli di Integrità

* ogni istanza è un elemento in qualche classe definita
* gli elementi di una classe sono dello stesso tipo, da diversi tra loro
* Una classe contiene tutti e soli gli elementi che rappresentano entità dello stesso tipo

****

###Modello E-R
**Simbolo grafico per rappresentare una entità**
![Modello Entità](http://i.imgur.com/7Em2h3a.png)

**Simbolo grafico per rappresentare un'associazione**

![Modello Associazione](/content/images/2014/Oct/Untitled-drawing--1-.png)
****
###Proprietà

* scalare (ad un solo valore)
![scalare](/content/images/2014/Oct/propiert-.png)
* Multipla (sono ammessi *n* valori)
![](/content/images/2014/Oct/multiple.png)
* Composta
![](/content/images/2014/Oct/multiple--2--1.png)
* Multipla composta
![](/content/images/2014/Oct/multiplecomposta.png)
* Opzionale
![](/content/images/2014/Oct/optional.png) Alternativa alla proprietà opzionale è quella **Totale**, per la quale bisognerà specificare separatamente dello schema l'obbligatorietà della presenza di valore.
* Costante     
I valori non possono essere cambiati, in alternativa la properietà è **modificabile**    
* calcolata   
Il valore è calcolato con un algoritmo, in alternativa si dice **esplicita**
* Unica  
il valore è diverso per ogni istanza, in alternativa si dice **generica**
* Temporale
 * modifiche che sono memorizzate
 *  periodicamente (snapshot)
 *  registrando le variazioni ed il tempo di viariazione
 *  altro
* Chiave  
![](/content/images/2014/Oct/chiave.png)
identifica in maniera univoca la singola istanza di **entità**
