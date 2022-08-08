+++
date = "2016-01-29T17:37:47"
draft = "false"
title = "Pretty Good Privacy"
slug = "pretty-good-privacy"

+++

Pretty Good Privacy (da ora **PGP**) è un programma e standard crittografico basato su crittografia asimettrica[^n]; è uno dei sistemi crittografici più usati in informatica.
L'implementazione più usata è quella di **GNU**, chiamata **GNUPG** o **gpg**, in genere installata di default sui sistemi GNU/Linux.
Puo' essere usata per crittografare o firmare digitalmente file, cartelle, partizioni e email, quest'ultimo è l'utilizzo più indicato per pgp, ed è quello che affronterò in questo post.

-------------------------------------------

## Creazione

Assicuratevi di avere installato sul vostro sistema il pacchetto pgp.
**importante:** create la chiave su un computer che ritenete protetto e affidabile. 

iniziamo a creare la coppia di chiavi: 
```
gpg --full-gen-key
gpg (GnuPG) 2.1.10; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
```
questo vi  mostrerà una serie di opzioni, premete pure invio o 1, scegliendo l'opzione `(1) RSA and RSA (default)`

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
```

il mio consiglio è di creare sempre una chiave più lunga possibile, essendo il limite ad oggi 4096 bit, usate questa lunghezza. 

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y

```

Si consiglia di tenere la data di validità di un anno, io in genere inserisco due anni, ricordatevi sempre di inserire un promemoria da qualche parte per modificare la chiave e aumentare di altri n anni qualche giorno prima che scada. 

Dopo di che, il programma vi chiederà nome, un commento, e email. Finita questa fase c'è l'inserimento di una **passphase** , molto importante anche questo, inserire una passphrase lunga perchè in caso di furto della chiave segreta  è l'unica cosa che salverebbe le informazioni che avrete crittografato. 
Ora il programma vi chiederà di fare altro, scrivere sulla tastiera o muovere il mouse, fate qualcosa durante questo processo, una volta finito di generare entropia avrete una coppia di chiavi pgp.

ultima cosa: creiamo un certificato di revoca delle chiavi in caso di furto. Questo step è meglio non saltarlo. 

`gpg --output revocation.gpg --gen-revoke <tua@email.it>`


per i più paranoici: In molti consigliano la creazione di sottochiavi per firmare e crittografare, e tenere la coppia di chiavi generati in un posto sicuro, rimuovendole dal keyring del portatile (o fisso).

---------------------

## Utilizzo 

##### keyserver 
Prima cosa, se non è già impostato di default, inserite nella config di gnupg un keyserver da cui prelevare e aggiornare le chiavi pubbliche che man mano inserirete nel vostro keyring. 
`echo 'keyserver hkp://keys.gnupg.net' >> ~/.gnupg/gpg.conf`

non importa quale keyserver scegliate, la chiave si propagherà in ogni keyserver. 

dopo di che, il modo più veloce per distruibre la vostre chiave pubblica è caricandola sul keyserver. 

`gpg --send-key <vostra@email.it>`

ora la vostra chiave pubblica è su un keyserver, chiunque voglia scrivervi o mandare un file crittografato potrà scaricarla dal keyserver e utilizzarla per crittare i documenti o le email. solo la vostra chiave segreta potra decrittare i documenti crittati dalla vostra chiave pubblica. 
##### lista delle chiavi nel keyring 

per mostrare le chiavi pubbliche nel vostro keyring, basta dare il comando 
`gpg --list-key`  
  **attenzione**: mostrerà solo quelle pubbliche. per mostrar anche quelle private bisogna utilizzare il comando `gpg --list-secret-keys`

##### esportare/importare chiavi
* esportare chiave pubblica: 
`gpg --export -a "user name" > pub.asc`  
nota: -a imposta l'esportazione della chiave in ascii armor, utile per poter inviare la chiave pubblica in [formato testuale](https://blackdev1l.com/pgp.txt).  
* esportare chiave privata: 
`gpg --export-secret-key -a "User Name" > private.asc`

* importare chiave pubblica
  * da file: 
`gpg --import public.key`  
  * da keyserver: 
`gpg --recv-key <KEY ID>` la key id si puo' trovare cercando sul sito del keyserver, bisogna fare attenzione a scaricare quella giusta, fare attenzione al fingerprint della chiave e se ci sono firme che confermano che la chiave è autentica. 

##### firma e verifica 
firmare un email o un file con una chiave pgp permette di dimostrare l'autorevolezza del contenuto del messaggio e identificare che quanto inviato/scritto sia stato veramente formulato dal mittente del messaggio/documento. 
mettiamo caso vogliamo firmare un documento senza crittarlo per dimostrare che il documento è stato scritto da noi, abbiamo due possibilità: 
avere la firma *inline*, oppure avere la firma separata al documento. 
mettiamo caso abbiamo un file chiamato hi.txt il cui contenuto è `ciao` utilizzando il comando 
`gpg --clearsign hi.txt`
il risultato sarà 
```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

ciao
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAEBCAAGBQJWq5BQAAoJEIIWH9NMFC2Wu5oQALTtDkdI4nt9IGp5FBmv1OLf
svBfipCPEy4pvlyQ4kfZGoc2A9ywMr6pYR95cAriRGlUT3yBo2XqlXD4l24lD6Bm
FTsXdKkUswhvtaOyTdvbkYrEZuvEUAdvBfIjK53Yn3/eAQU5TBOUWfmN1JEMVgP8
d5/EYY62fRLnys8pbzBGQ9f51iSGNuUQNAe/6aajQayd+RPJheZVdBw0xjNOTx20
wG5b82NuraGBzEAxnoV9uqnUrS8Eq8MiK0UP4Pmc7TmBFzlyPtzqR313He6YpxvD
Bs67awTJyYpiCWKYD9PEWiVPqTdBmWx7FjteT7rjU/6F68sYFOWabNZVuAPLcMT4
Hm2rojkUqlJmfTTrcbBShTLo2P8pLrVrYXoz4POcW/Pq0aY0rG5b4Oo2UZYlJlmP
FfEljDmc1mV0jor/n8GWr0jiNu4VeR5Pkd7hoxIEf0APGVnZAkVav9Kv+Y2zf/PA
auHMxY9Im8CwjfS3Wru8PyGYCxUSG/YPvXONRai5M9/8CQM5Oz5CGhrL4xRbjr1V
RiFA8Ck0uhh2LWf0v5JlnDb9di30Q7UFSzKUeY4DI0idp668mxksLoMEIYqcf39u
PtS2DK7mDQFjrVbBZ4PkEC5aBs3GB1QdKhfa1hzNS6D7QFoGgmb71lETvaHaOgg6
uVW3Kxs78ySS9CeUVJpg
=SefM
-----END PGP SIGNATURE-----
```

mentre con `gpg --output hi.txt.sig --detach-sig hi.txt`
avremo la firma sotto forma di file. 

##### crittare e decrittare
* crittare: `gpg -e file`
* decrittare:  `gpg [-d] file`

----------------------

## Email
La vera forza di gpg sta nell'utilizzarlo per scambiare email. Probabilmente l'utilizzo più diffuso è quello della firma delle email, firmando le email con la propria chiave gpg, come già detto prima, permette di essere sicuri che quella email è stata mandata davvero da voi. Fortunatamente sono stati creati plugins e strumenti grafici che ci aiutano e facilitano notevolmente il lavoro, cosìcchè firmare e criptare un email sarà facile come premere un tasto sullo schermo, senza dover ricorrere continuamente al terminale. 
Probabilmente il più utilizzato sia su windows che su linux è [Enigmail](https://www.enigmail.net/home/index.php), plugin per thunderbird fenomenale, rende l'esperienza di utilizzo indolore. Invece se preferite usare la webmail, conosco [Mailvelope](https://www.mailvelope.com/), un estensione per il browser che permette di utilizzare pgp senza impazzire. 

----------
## L'importanza di utilizzare PGP 
Chiudo questo post con una mia piccola riflessione, che non è solo mia, ma che è di tutti quelli che hanno a cuore la propria privacy, specialmente in questa era dove le nostre informazioni valgono tanto e vengono utilizzate per scopi di lucro o peggio; è importante cercare in tutti i modi di salvare da occhi indiscreti queste informazioni, l'utilizzo di pgp è uno di questi, stiamo lentamente perdendo sempre più potere sui nostri dati personali e sulle informazioni che giornalmente condividiamo su internet. Non bisogna avere qualcosa da nascondere per dover aver a cuore la privacy. 


[^n]: in verità utilizza sia la crittografia simettrica che asimettrica, viene evidenziata la sua natura asimettrica durante l'utilizzo. 