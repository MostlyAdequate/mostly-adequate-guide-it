[![cover](images/cover.png)](SUMMARY.md)

## A proposito di questo libro

Questo è un libro sul paradigma funzionale in generale. Useremo il linguaggio di programmazione più diffuso nel mondo: JavaScript. Alcuni potrebbero pensare che questa sia una scelta sbagliata perché va contro la logica del pensiero attuale, che punta maggiormente verso un paradigma imperativo. Tuttavia, credo che sia il modo migliore per imparare la PF (Programmazione Funzionale) per diversi motivi:

 * **Probabilmente lo utilizzi ofni giorno a lavoro.**
	
	Questo ci permette di fare pratica ed applicare le conoscenze acquisite ogni giorno in programmi reali, piuttosto che di notte, o nel fine settimana, in progetti fittizi e con un linguaggio di PF esoterico.


 * **Non dobbiamo imparare niente per poter iniziare con i nostri programmi.**

	In un linguaggio basato puramente sulla PF, non è possibile loggare una variabile o leggere un nodo del DOM senza utilizzare monads. Qui possiamo barare un pochino. E' anche più facile iniziare con questo linguaggio dal momento che utilizza più paradigmi e che potete quindi ripiegare sulle vostre conoscenze per colmare eventuali lacune.


 * **Il linguaggio è pienamente capace di scrivere codice funzionale di prim'ordine.**

    We have all the features we need to mimic a language like Scala or Haskell with the help of a tiny library or two. Object-oriented programming currently dominates the industry, but it's clearly awkward in JavaScript. It's akin to camping off of a highway or tap dancing in galoshes. We have to `bind` all over the place lest `this` change out from under us, we don't have classes(Yet), we have various work arounds for the quirky behavior when the `new` keyword is forgotten, private members are only available via closures. To a lot of us, FP feels more natural anyways.

Detto ciò, i linguaggi funzionali tipizzati, sono senza dubbio il posto migliore per programmare nello stile presentato in questo libro. JavaScript sarà il nostro mezzo di apprendimento, dove applicherai ciò che impari spetta a te. Fortunatamente le interfacce sono matematiche e, come tali, onniprensenti. Ti ritroverai a casa con swiftz, scalaz, haskell, purescript, ed altri ambienti inclini alla matematica.


### Gitbook (per un'esperienza di lettura migliore)

* [Leggilo online](http://drboolean.gitbooks.io/mostly-adequate-guide/)
* [Scarica l'EPUB](https://www.gitbook.com/download/epub/book/drboolean/mostly-adequate-guide)
* [Scarica il Mobi (Kindle)](https://www.gitbook.com/download/mobi/book/drboolean/mostly-adequate-guide)

### Fai da solo

```
git clone https://github.com/DrBoolean/mostly-adequate-guide.git

cd mostly-adequate-guide/
npm install gitbook-cli -g
gitbook init

brew update
brew install Caskroom/cask/calibre

gitbook mobi . ./functional.mobi
```


# Indice dei contenuti

Vedi [SUMMARY.md](SUMMARY-it.md)

### Contribuire

Vedi [CONTRIBUTING.md](CONTRIBUTING-it.md)

### Traduzioni

Vedi [TRANSLATIONS.md](TRANSLATIONS-it.md)


# Obiettivi per il futuro

* **Parte 1** è una guida alle basi. La aggiornerò via via che troverò errori dato che questa è la bozza iniziale. Sentiti libero di dare una mano!
* **Parte 2** tratterà i tipi di classi, come functors e monads, per tutto il percorso percorribile. Spero di spremerli in trasformatori e in un'applicazione pura.
* **Parte 3** inizieremo a percorrere il filo sottile tra la programmazione pratica e quella assurdamente accademica. Vedremo comonads, f-algebras, free monads, yoneda, e altri costruttori categorici.
