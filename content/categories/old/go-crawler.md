+++
date = "2015-02-09T02:17:51"
draft = "false"
title = "Go Crawler"
slug = "go-crawler"

+++

Iniziamo l'anno con un progettino semplice semplice per impratichirsi con
[golang](http://golang.org/) , un linguaggio creato da google che permette un esperienza estremamente flawless durante la scrittura del codice e una velocità impressionante, tant'è che molte aziende stanno migrando i propri servizi riscrivendoli in go. 

Quello che avevo in mente era creare un sito usando [Polymer](https://www.polymer-project.org/) come frontend e go in backend, che dato un URL parsi tutti i link che trovi all'interno e lo mostri sotto forma di grafo (precisamente albero) , decidendo quantità di nodi,~~ e anche quanto andare in profondità durante il crawl degli URL.~~ 

Il progetto lo trovate su [github](https://github.com/blackdev1l/go-crawler). 

# Analisi del problema

Il problema non è dei più difficili, ho suddiviso il progetto in 2 grandi sotto-problemi

* Front-end
* Back-end

L'idea era quella di avere un webserver in go che attraverso un HTTP REQUEST  ricevesse l'url e ritornasse in response sottoforma di json il grafo da poi visualizzare attraverso la libreria [sigma.js](http://sigmajs.org/) .

## Front-end

Polymer è un framework scritto da google che attraverso web components permette di creare bellissime UI con l'utilizzo di elementi predefiniti  e la creazione di nuovi. 
Ah, e tutto in Material Design. 


inanzitutto bisogna scaricare in locale Polymer, io ho usato bower, il quale mi permette di gestire anche eventuali dipendenze senza uscir matto, dopo di che importare gli elementi nel file .html 
```prettyprint lang-html
    <!-- 2. Load the component using an HTML Import -->
    <link rel="import" href="bower_components/paper-input/paper-input.html">
    <link rel="import" href="bower_components/core-header-panel/core-header-panel.html">
    <link rel="import" href="bower_components/core-toolbar/core-toolbar.html">
    <link rel="import" href="bower_components/paper-button/paper-button.html">
    <link rel="import" href="bower_components/ajax-form/ajax-form.html">
    <link rel="import" href="bower_components/paper-slider/paper-slider.html">
```

dopo aver dato un occhiata alla documentazione (potevi fare di meglio google, ma va bene così) ho creato una bar fissata in top, un paper-input (per l'inserimento dell'url) , e due paper-slider per la quantità dei nodi e profondità del grafo. 

La documentazione consiglia di utilizzare l'element [ajax-form](http://garstasio.github.io/ajax-form/components/ajax-form/) per l'invio dei dati attraverso paper-input, è molto facile da utilizzare e attraverso essa sono riuscito con semplicitià a inviare la request, ricevere e utilizzare il response. 

```prettyprint lang-html
<form is="ajax-form" action="send" method="post" enctype="application/json">
    <paper-input name="url" label="Crawl this url" floatingLabel></paper-input>
    <div id="button">
        <paper-button raised id="submitButton">Submit</paper-button>
    </div>
    <paper-slider id="slider" pin min="5" max="50" name="number"></paper-slider>
</form>
```

Per quanto riguarda la parte in java script 
```prettyprint lang-js 
  <script>
  var s = new sigma(document.getElementById('container'));
  document.getElementById('submitButton').addEventListener('click', function() {
  document.forms[0].submit();
  });

  document.forms[0].addEventListener('submitted', function(event) {
    var data = JSON.parse(event.detail.responseText);
    console.log(data);
    s.graph.read(data)
    s.refresh();
    s.graph.clear()
   });

</script>
```


Così ogni volta che premo submit manda i dati, attende che vengono parsati dal server scritto in go, al trigger del "submitted" (quando i dati vengono ricevuti) la libreria sigma si aggiorna con il grafo ricevuto in formato json. Si puo' quindi inserire l'url o modificare i nodi e clickare submit senza dover aggiornare la pagina o creare qualche tipo di blocco. 
Ma ora guardiamo cosa c'è sotto il "cofano". 


## Back-end

Scrivere web server in go grazie alle sue librerie std è davvero una favola. Ci si impiega letteralmente poche righe di codice per poter avere un semplice webserver che mostra un Hello World. 
direttamente dalla documentazione ufficiale: 

```
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

Fantastico vero? Ho suddiviso in 3 file il progetto

* handlers.go 
* graph.go
* main.go

Il primo gestisce le richieste, solo 2, quella per il root path e quindi static files e la richiesta /send che viene usata per ricevere l'url e il numero dei nodi e ritornare un json con tutti i link trovati da quell'url. 

Intanto guardiamo Main.go 

```
package main

import (
	"github.com/gorilla/mux"
	"log"
	"math/rand"
	"net/http"
	"time"
)

func main() {

	r := mux.NewRouter()
	r.HandleFunc("/", Index)
	r.HandleFunc("/send", Send)
	r.PathPrefix("/").Handler(http.FileServer(http.Dir("./www/")))
	http.Handle("/", r)

	log.Fatal(http.ListenAndServe(":8080", r))

}
```

Per questo progettino, ho voluto provare delle librerie esterne, precisamente il mux del framework gorilla. 
la riga `r.PathPrefix("/").Handler(http.FileServer(http.Dir("./www/")))` ci permette di servire i file statici dentro la cartella `./www/`. 

In graph.go Ho inserito le strutture dati del grafo. Ho preferito spezzarle in 3 perchè mi veniva comodo in seguito poter distinguerle, anche se non era necessario. 

le 3 strutture dati sono le seguenti: 

```
type Edge struct {
	Id     string `json:"id"`
	Source string `json:"source"`
	Target string `json:"target"`
}

type Graph struct {
	Url   string  `json:"-"`
	Child []*Node `json:"nodes,omitempty"`
	Edges []*Edge `json:"edges,omitempty"`
}

type Node struct {
	Id    string `json:"id,omitempty"`
	X     int    `json:"x"`
	Y     int    `json:"y"`
	Size  int    `json:"size"`
	Label string `json:"label,omitempty"`
}
```

quelle che vedete a destra delle variabili, sono tag, permettono di identificare durante il Marshal alcune utili proprietà. Esse mi permettono di Esternalizzare le variabili all'interno della struttura dati ma allo stesso tempo avere nel json dopo il marshal tutte le variabili in minuscolo, il tag `omitempty` come da nome, permette di omettere dal json le variabili dichiarate nil o 0. 
il tag `-` invece, omette ogni caso dal marshal quella variabile.

Ci sarebbe molto altro da dire riguardo alle implementazioni, ma preferisco lasciare a voi il gusto di leggere il codice e imparare questo fantastico linguaggio. 


## Resoconto

Golang è bellissimo. Ti permette di scrivere codice in modo semplice e allo stesso tempo ordinato, con prestazioni elevatissime (ricordiamo che è un linguaggio compilato) , le librerie standard sono ottime, e ci sono [tool](https://golang.org/doc/cmd) di cui non vi ho parlato che vi semplificano di molto la vita.

Buon coding. 