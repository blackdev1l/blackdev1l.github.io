+++
date = "2014-09-21T17:45:38"
draft = "false"
title = "Irc Bot"
slug = "irc-bot"

+++

Salve a tutti! Nelle settimane passate ho scritto un piccolo irc bot insieme a un mio amico in java per un canale tra "amici" appoggiandoci su delle api per il lato network. L'idea generale era implementare i comandi più "usati" da un bot, cioè !quote youtube parser e magari !seen. Dopo di che ~~abbiamo~~ ha pensato di cambiare e rendere il bot modulare (cosa che sta pensando a sviluppare lui data la mia scarsa conoscenza di design pattern e affini) 


### cosa abbiamo implementato fin ora

* !quote e affini
* link parser in regex (rubato da stackoverflow) 
* interfaccia per modularizzare il bot
* caricamento dei settings tramite file xml


### Gestione dei quote

I quote vengono gestiti tramite un listener a parte, essi sono salvati su un mongodb in remoto, mongodb è un database nosql facile da usare e con molto potenziale. Per quello che dobbiamo fare non ci serve nulla di scalabile o chissà cosa in termini di computazioni, alla fine son solo 4 quote. 

### Deployment 

Ancora dobbiamo rilasciare il bot su una vps per renderlo stabile e usabile. Ci manca ancora qualche accortezza e presto potremo usarlo.
