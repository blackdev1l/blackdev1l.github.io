+++
date = "2014-11-02T22:15:08"
draft = "false"
title = "Rails Rumble"
slug = "rails-rumble"

+++

Salve a tutti, qualche settimana fa ho partecipato all'edizione di quest'anno della [Rails Rumble](http://railsrumble.com/) , è stato emozionante ed estremamente istruttivo. Ma partiamo dall'inizio.

##Cos'è:

Rails Rumble è un [hackathon](http://en.wikipedia.org/wiki/Hackathon) online in cui si forma un team di "creativi" per finire in un prestabilito lasso di tempo un progetto. 
Per la Rails Rumble il tema principale è stato l'utilizzo del web framework [Ruby on Rails](http://rubyonrails.org/) , col quale in 48 ore bisognava produrre una web app / sito web. Il limite dei team era di 4 persone.

ho trovato il team su reddit, precisamente in /r/ruby (o /r/rails?) , 2 americani e un inglese, più o meno tutti sullo stesso livello di abilità, essendo io alle prime armi con Rails all'inizio della gara ho deciso di occuparmi del lato backend della web app. 

##Cosa abbiamo fatto:

###World Spotlight

>A new way to explore the world we live in, making it feel just a little bit smaller as you explore countries on a world map stage. Data collected from Youtube, Flickr, Twitter and more is used to present a small snapshot of a countries current moment in digital space.

L'idea era quella di dare uno scorcio sul "Mondo", offrendo notizie varie su un preciso paese selezionato attraverso una mappa, una cosa molto semplice e d'impatto.
Il mio lavoro era quindi quello di fornire le informazioni sotto forma di dati, queste informazioni erano principalmente 4: Trends di twitter, News locali, informazioni sul meteo e della valuta, video più visitati su youtube nel paese scelto e, in più, una immagine rappresentante il paese come background per la mappa. Per fare ciò avevo bisogno di prendere questi dati dai siti che maggiormente coprivano i corrispettivi task e fare lo [scraping](http://en.wikipedia.org/wiki/Web_scraping) delle informazione in base  al paese scelto. Siccome nel team non ero l'unico a lavorare sul backend scelsi di lavorare sulle immagini in background, le news e twitter.


**NB: I seguenti pezzi di codice sono stati scritti tra le 2 di notte e le 8 del mattino, di fretta e con solo la voglia di avere qualcosa di funzionante, saranno tutte quasi sicuramente soluzioni inefficienti, ma con solo 48h di tempo non potevo permettermi di pensare alla soluzione ottimale e/o ottimizare il codice.**

###Immagine in background
prima di tutto ho creato una classe base così da poter snellire un pelino il codice, tutte le future classi ereditano da questa classe:

```prettyprint lang-ruby
class Scraper
   attr_accessor :url, :location

   def initialize(name)
   	@location = name
   end

  def scrapeUrl(url)
  	doc = Nokogiri::HTML(open(url))
  	return doc
  end

  def scrapeObj(url)
  	doc = Nokogiri::XML(url)
  	return doc
  end
end
```

Per l'immagine in background mi sono affidato a flickr e le sue api, volevo che per ogni paese ci fosse la possibilità di avere un immagine a "tema", per fare ciò ho deciso di utilizzare una [gem](http://guides.rubygems.org/what-is-a-gem/) estremamente utile per quanto riguarda lo scraping su ruby: [nokogiri](http://www.nokogiri.org). 
nokogiri permette il [parsing](http://en.wikipedia.org/wiki/Parsing) delle informazioni di qualsiasi documento html e xml, permettendomi così di usufruire delle api di flickr a pieno. L'idea era di prendere una foto attraverso la ricerca con il nome del paese come parola chiave e possibilmente una foto ad alta risoluzione (e possibilmente avere la
risoluzione originale).
```prettyprint lang-ruby
class FlickrScraper < Scraper
	def getImageUrl()
		string = ""
		response = open('https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key=apikey&privacy_filter=1&safe_search=1&content_type=1&tags='+location+'&per_page=100&page=1&extras=original_format&format=rest').read
		doc = scrapeObj(response)
		res = doc.xpath("//photo")
		res.each do |rs|
			id = rs[:id]
			puts "id is #{id}"
			sizeGET = open('https://api.flickr.com/services/rest/?method=flickr.photos.getSizes&api_key=apikey&&photo_id='+id+'&format=rest')
			docSize = scrapeObj(sizeGET)
			docSizePath = docSize.xpath("//size")
			puts docSizePath.count
			docSizePath.each do |dsp|
				if(dsp[:label] == 'Original')
					string = dsp[:source]
					puts "string is #{string}"
					return string
				end
			end
		end
		puts "Image not found"
		return 1
	end
end
```

Questa classe ha un unica funzione che attraverso le api di flickr ottiene una serie di link alle immagini, essa ritorna il link alla prima immagine che trova con il formato originale disponibile.
essendo di solito il formato originale molto grande, abbiamo pensato di inserire un video di nuvole che si muovono nell'homepage mentre si aspetta che l'utente scelga il paese, e per non far vedere il caricamente abbiamo inserito un  fade effect, cosìcchè da non mostrare gradualmente la foto ma solo dopo averla caricata completamente, stessa cosa da una foto all'altra.
Da notare il doppio ciclo uno dentro l'altro che nel worst case porterebbe a un costo di ben O(n^n) , ora non ho studiato molto statistica, ma prima di avere un worst case del genere con il database fotografico di flickr penso che ne passerà di tempo :) 

###News
Per quanto riguarda le news, considerando che google avesse chiuso l'accesso alle sue api per google news, decidemmo di utilizzare un [motore di ricerca](http://www.faroo.com) semi sconosciuto, ma che con le sue api immediate e di facile utilizzo ha dato una grande mano nel concludere questo task.

```prettyprint lang-ruby
class NewsScraper < Scraper
	def getNews()
		doc = scrapeUrl("http://www.faroo.com/api?q=#{location}&start=1&length=5&l=en&src=news&f=xml&key=lBbetYupJAk2n8scJmiKTVDlNrw_")
		return doc
	end
    
	def getCapital()
		doc = scrapeUrl("http://en.wikipedia.org/wiki/"+location)
		#print(doc)
		doc.xpath("//Infobox") do |node|
			print(node)
		end
	end
end
```
la funzione su cui ho lavorato io è la prima, con un semplice scrape delle api ho ottenuto un oggetto contenente link, titolo, una breve descrizione e persino un immagine raffigurativa per l'articolo. Fantastico vero?

###Twitter
Non sempre le cose vanno come vorresti, con questo task è stato esattamente così. All'inizio si pensava a voler fare una lista dei 10 top trends della nazione selezionata, ma tra infiniti errori e il tempo che inesorabilmente volgeva allo scadere abbiamo optato per una lineare scelta dei top tweets 

```prettyprint lang-ruby
class TwitterScraper < Scraper
	include Twitter::Autolink
	attr_accessor :client

	def initialize(name)
		@location = name
		@client = Twitter::REST::Client.new do |config|
			config.consumer_key        = "consumer_key"
			config.consumer_secret     = "consumer_secret"
			config.access_token        = "access_token"
			config.access_token_secret = "access_token_secret"
		end
	end

	def parseJson()
		json = File.read(Rails::root+"lib/assets/trends_available.json")
		return JSON.parse(json)
	end

	def getTrends()
		puts "location is #{@location} , looking for WOEID"
		countryCode = ""
		locationAvailable = parseJson()
		locationAvailable.each do |loc|
			if(loc["name"].capitalize.to_s == @location.to_s)
				puts "found!"
				puts loc["name"]
				puts loc["woeid"]
				countryCode = loc["woeid"]
			end
		end
		puts "code is #{countryCode}"
		trends = client.trends(countryCode)
		trends.each do |tr|
			puts tr.name
		end
		return trends
	end
	
	def getTweets()
    puts "waiting on twitter"
    return @client.search(@location, :result_type => "recent").take(6).collect
	end

	def getCountryCode()
		res = []
		countries = parseJson()
		countries.each do |country|
			res << country["countryCode"]
		end
		return res
	end
end
```

Classe un pelino più *corposa* rispetto alle altre, purtroppo il rate limit delle api di twitter è stato un problema non indifferente, che ci ha portati a tardare non poco  il completamento di questo task. La funzione getTrends funzionava solo per alcuni paesi, per altri no, non abbiamo indagato molto sul motivo e avevamo deciso di usare la funzione getTweets come "toppa" se i trends non caricavano. 

##Com'è andata: 



dopo aver passato l'ultima ora a cercare di sistemare il lato twitter dell'app il tempo stava per scadere, ce l'avevamo fatta giusto in tempo, mancavano 10 minuti, quando:
>[10/20/2014 12:49:37 AM] Cameron C.: fuck  
>[10/20/2014 12:49:40 AM] Cameron C.: I have conflicts  
>[10/20/2014 12:49:41 AM] Cameron C.: hang on  

![Conflicts everywhere](http://www.quickmeme.com/img/02/023019c6c3b84097ee6df4a38f58c16d9da2b8954988f074aad24a6cb4861fd5.jpg)

>[10/20/2014 12:53:15 AM] Mike R.: guy you have 2 minutes to fix conflicts or im pushing with old api  


![sweatyman](http://replygif.net/i/1209.gif)
>[10/20/2014 12:56:54 AM] Mike R.: oh my god  
>[10/20/2014 12:56:57 AM] Mike R.: conflicts out my ass

i minuti scadevano e l'unica cosa che potevamo fare era un rollback e lasciare solamente i tweets, senza trends (sigh, tutto lavoro sprecato)

>[10/20/2014 12:57:50 AM] Mike R.: im pushing old api

Fu l'unica cosa che potemmo fare. il deploy fu fatto alle 23:59 UTC, un minuto in più (o forse meno) e saremmo stati squalificati. 
##Post Deploy

Il sito sembrava  andare bene, tutto funzionava come doveva, eccetto twitter. Il problema non era il codice perchè sia in locale che su box esterne le api funzionavano e  il sito mostrava i top 10 tweets. Quindi cosa poteva essere? Probabilmente heroku o il nostro sito era blacklistato da twitter, cosa che ci ha sicuramente penalizzato tantissimo al momento del judging. 

Alla fine non abbiamo vinto, ma è stato estremamente divertente e ho imparato tantissimo, il team è ancora ben saldo e si pensa già al prossimo hackathon online a cui partecipare, mi è piaciuto moltissimo lavorare in gruppo e trovo che loro siano delle persone fantastiche, 
Se vi ho incuriosito un pochino, vi lascio il link al sito che abbiamo creato in 48 ore per la Rails Rumble, un commento col vostro giudizio mi renderebbe ancora più felice! 

http://worldspotlight.r14.railsrumble.com/
