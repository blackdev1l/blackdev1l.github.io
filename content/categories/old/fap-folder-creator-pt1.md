+++
date = "2014-07-05T18:29:31"
draft = "false"
title = "Fap Folder Creator pt1"
slug = "fap-folder-creator-pt1"

+++

Hello guys, oggi vi parlo di uno [scriptino](https://github.com/blackdev1l/Fap-Folder-Creator) in python che sto scrivendo per imparare il linguaggio, incomincio subito a dire che dopo un anno di c++ scrivere del codice in python è come spegnere il cervello e parlare con una persona. Stupidamente facile e immediato. 
Lo script si appoggia (per ora) principalmente su ~~2~~ 1 libreri~~e~~a: ~~Click e~~ [Haul](https://github.com/vinta/Haul)

 in verità l'idea dello script mi è venuto leggendo una [lista di librerie per python](https://github.com/vinta/awesome-python) trovata su Reddit, precisamente la libreria [Haul](https://github.com/vinta/Haul) mi ha dato l'idea di creare un downloader di immagini automatico dai thread di 4chan, mi sembrava un compito molto semplice e allo stesso tempo utile per imparare il linguaggio. Non ho trovato grosse difficoltà, il linguaggio è abbastanza "libero", le variabili non sono strettamente tipicizzate e non c'è nessuna gestione della memoria, probabilmente meglio se mi fermo qui a descrivere il linguaggio perchè ancora per me è nuovo e non voglio scrivere castronerie. 

Per ora lo script non è interattivo, non prende nessun argomento in input, basta solo avviarlo con `python ffc.py` e di seguito chiederà url e dir.
Principalmente lo script è suddiviso in 3 moduli:

* parsing della pagina e acquisizione dei link immagine
* Download  di ogni immagine
* Watch Thread per eventuali immagini uppate dopo l'avvio dello script fin quando non viene eliminato il thread (404)

### get_images()

il primo punto è abbastanza semplice, anche grazie alla libreria a cui mi sono appoggiato, ho creato una funzione che riceve in input l'url del thread e ritorna la lista di url delle immagini a risoluzione originale (solo quelle e non le thumbnail)

```prettyprint
def get_images(url):
    result = haul.find_images(url, extend=True)

    out = []

    for item in result.image_urls:
        if 'i.4cdn' in item:
            out.append(item[4:])
        else:
            pass
    return out
```


la prima riga della funzione è il parsing vero e proprio dei link attraverso la libreria Haul, dopo di che ho creato una lista vuota in cui poi andranno solo i link delle immagini originali. 
Il ciclo for cerca tra gli url quelli contenenti la sotto-stringa "i.4cdn" (sarebbe il cdn di 4chan in cui sono caricati tutte le immagini dei thread, differente dal t.4cdn dove ci si trovano solo le *thumbnail* delle immagini) e quindi ottenere solo immagini a risoluzione originale e lasciare le thumbnail nella lista temporanea., dopo di che "appendo" l'url nella lista out partendo soltanto dal 4 carattere in poi (il parser non riesce a copiare bene l'inizio dell'url) e in fine ritorno la lista con tutti gli url delle immagini. 

### def_download()

```prettyprint
def download(img_url,filename,dr):
    img_url = 'http://i.'+img_url
    obj = dr+'/'+filename
    print("downloading "+img_url)
    urlretrieve(img_url,obj)
```
Con questa funziione scarico una singola immagine, nel main uso un ciclo for pasando ogni singolo link alla funzione cosìcchè posso usarla in seguito per la funzione watch_thread() . 

La funzione è scritta *male* con parecchio hardcode ma non ho pensato a un modo migliore, magari in futurò farò un refactoring, ma non penso che mi verrà in mente qualcosa di meglio senza dover cambiare libreria o stravolgere anche get_images().
Una funzione molto utile ed estremamente facile da utilizzare è *[urlretrieve()](https://docs.python.org/2/library/urllib.html#urllib.urlretrieve)* la quale mi permette di scaricare qualsiasi oggetto su un sito tramite indirizzo url senza preoccuparmi di altro. 


### watch_thread()
```prettyprint
def watch_thread(time,url,dr):
    print("Watching...")
    urljson = url_to_json(url)


    with contextlib.closing(urllib2.urlopen(urljson)) as j:
        j_obj = json.load(j)

    img_number = j_obj["posts"][0]["images"]

    while True:
        sleep(time)
        try:
            with contextlib.closing(urllib2.urlopen(urljson)) as j:
                j_obj = json.load(j)
        except urllib2.HTTPError:
            print("Thread went 404\n Total image downloaded: " + str(img_number+1))
            exit(3)
        else:
            new_img_number = j_obj["posts"][0]["images"]
            if img_number != new_img_number:
                new_list = get_images(url)
                filename = new_list[-1].split('/')[-1]
                download(new_list[-1],filename,dr)
                img_number = new_img_number
            else:
                pass
    return True
```


Questa è la funzione più "corposa" ma anche  quella che rende lo script "utile" rispetto a un classico down them all. per ora non so se è completamente esente da bug  (simulare una situazione di download passivo di un thread non è difficile ma richiede molto tempo, considerando che i therad su 4chan posso durare giornate o poche ore) . 
Inanzi tutto ho utilizzato un'altra funzione delle librerie standard di python che è quella per il parsing dei file [json](https://docs.python.org/2/library/json.html). Ho deciso di utilizzarla per sfruttare le [api di 4chan](https://github.com/4chan/4chan-API) e monitorare quante immagini ci sono nel thread, se aumentano (e di conseguenza scaricare l'ultima immagine) oppure se il thread è andato 404 (attraverso un try except)

Un bug che ho notato e che non riesco a risolvere è una chiusura forzata della connessione da parte del server, succede dopo circa 10 minuti che il processo è in idle, ho provato a risolvere chiudendo ogni volta le connessioni con un `with contextlib.closing(urllib2.urlopen(urljson)) as j:` ma devo riprovare e vedere se il bug è stato risolto oppure no. Non saprei per cos'altro potrebbe avvenire. 

Lo script è ancora in wip e devo lavorare ancora sulla parte CLI e alla gestione dei file e folder (se la cartella dove si vuole scaricare le immagini non è stata creata lo script si pianta) 