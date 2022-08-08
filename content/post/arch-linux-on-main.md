+++
date = "2014-07-17T16:45:06"
draft = "false"
title = "Arch Linux on Main!"
slug = "arch-linux-on-main"

+++

Salve, 3 giorni fa ho deciso di formattare tutto e installare arch linux sul main pensando di poter finalmente rimpiazzare windows (obbligato ad usarlo per videogiocare) con il mio sistema operativo preferito.

Dopo un veloce `dd if=\arch.iso of=/dev/sdc BS=4M` ero pronto alla installazione. 
Io ho un hard disk da 2T che uso come disco dati e un ssd per il sistema operativo da 128gb. Mai avuti problemi di spazio o altro, con la wiki sul portatile e la penna bootata sul main parto spedito per l'installazione 


### First big step, first big error. 

Mai, mai iniziare un installazione senza prima dare una controllata ai dischi con `lsblk` , potreste far dei danni seri. 
Io per questa volta danni seri non ne ho fatti, ma ci sono andato vicino.
Dopo aver bootato via usb sulla distro di arch per iniziare l'installazione, la prima cosa che ho fatto è stata quella di eliminare la tabella di partizione sul **disco dati** invece che sull'ssd.

`sgdisk --zap-all /dev/sda`

Semplice, pulito, diretto. Peccato che `/dev/sda` fosse l'hard disk da 2 tera e non l'ssd.Primo errore, primo problema.

### Uefi merda.

Dopo aver direzionato l'installazione verso il disco esatto e aver settato tutto, mi ritrovo nella fase pre-reboot e post-installazione, mi manca solo da installare il Boot Loader. Ecco che mi trovo di nuovo in difficoltà (dopo il calvario per avere il dual boot sul portatile) a cercare di installare arch sotto UEFI. Sarà per le precedenti esperienze ma dopo vari tentativi (per lo più errori di scrittura) sono riuscito a far funzionare [gummiboot](http://freedesktop.org/wiki/Software/gummiboot/) e avviare correttamente Arch linux. 

### Ping! 

Internet non va. reboot da penna (non so come mai funziona senza nessun particolare impostazione da pennetta ma da installazione no, ignoranza mia) e per risolvere installo [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager) , dopo un `systemctl enable NetworkManager` tutto funziona a meraviglia.

### Steam on linux

Tutto felice e pimpante per l'installazione avvenuta con successo installo subito un DE (per questa volta [cinnamon](http://cinnamon.linuxmint.com/) nell'attesa di [KDE 5](http://www.kde.org/announcements/plasma5.0/))
e i driver open-source ati, dopo di che mi appresto ad installare Steam, eccitato dall'idea di provare finalmente una esperienza videoludica decente su linux, l'installazione va liscia, apro il terminale e digito velocemente le 5 lettere magiche che evocheranno la macchina a vapore:

`steam`

Black screen, ritorno al Login screen. **WTF?!**
Dopo una ricerca non troppo lunga sul forum di arch trovo la soluzione al mio problema, 

`find ~/.local/share/Steam -name 'libstdc*.so*' -delete`  
`find ~/.local/share/Steam -name 'libgcc_s*.so*' -delete`

Avvio steam. Parte. yeah! 


### Ati e Linux

Che terribile accoppiata. La scheda video con i driver open source crasha random (non dico sotto sforzo, perchè a volte sto solo scrivendo su terminale con nulla sotto in background e crasha, mentre dopo 5 episodi di orange is the new black va liscio come l'olio) e sopratutto i driver catalyst sono un fottuto dito in culo da installare (ancora non ci sono riuscito)  , basta solo guardare [la pagina wiki dei driver](https://wiki.archlinux.org/index.php/Amd_catalyst) per capire che siamo ancora lontani dall'avere un esperienza flawless per quanto riguarda i videogiochi (oppure spendi di più e compri [Nvidia](http://youtu.be/iYWzMvlj2RQ)) .

### Recuperiamo l'hard disk da 2T
Dopo aver risolto ogni problema con calma, mi rimane solo questo, facendo un giro su google (e sulla wiki di Arch, fonte inesauribile di informazioni e soluzioni) trovo un applicazione che fa perfettamente al caso mio: [testDisk](http://www.cgsecurity.org/wiki/TestDisk) , questo programma riesce a ricostruire tabelle di partizioni in pochi secondi e con una semplicità disarmante (e io comunque riesco a svaccare tutto non selezionando partizione / settando GPT invece che MBR e spendendo ore per capire cosa non andasse bene) sono riuscito a ottenere l'hard disk con tutti i dati ancora li dov'erano (alla fin fine ho solo eliminato la tabella, non ho fatto altro) 

### Ok, mi sono divertito, torniamo su Windows 

Dopo aver capito che avere arch con i driver rotti (non mi sono arreso, sto solo riposando per ricominciare a smanettare e risolvere anche questo fastidioso problema) è un po' una sola, ma anche che dopo aver speso tutto questo tempo è un peccato formattare, decido di fare un dual boot. Dopo aver scaricato *legalmente* da torrent una iso di Win 8.1 decido di creare una partizione dall'hard disk di 2T di circa 300 gb e installare li Win 8.1 per poi effettuare Dual Boot. Per ora mi limito a dare la priorità di boot all'hard disk da 2T (non ho ancora un vero e proprio dual boot dato che Gummiboot non mi ha rilevato l'installazione di windows sul disco dati con tabella in MBR) più avanti vedrò se riesco a risolvere e avere un boot manager che mi faccia scegliere sia win 8.1 che Arch.

### Conclusione.

Arch è bello, molto bello. Divertente, potente, ma ancora non puo' rimpiazzare pienamente windows. Almeno non se come me si è accaniti videogiocatori oltre che smanettoni. Un dual boot è la scelta migliore, the best of both worlds. 