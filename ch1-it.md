# Capitolo 1: Che cosa stiamo facendo?

## Introduzione

Ciao, sono il professore Franklin Risby, piacere di conoscerti. Passeremo un po' di tempo insieme dato che dovrò insegnarti un po' di cose sulla programmazione funzionale. Ma ho già parlato abbastanza di me, tu che mi dici? Spero tu abbia già un po' di esperienza con il linguaggio JavaScript, con la programmazione orientata agli oggetti, e che tu creda di essere un programmatore. Non hai bisogno di un dottorato in Entomologia, devi solo sapere come scovare ed uccidere alcuni scaragaggi (bugs).

Non darò per scontato alcuna conoscenza di programmazione funzionale perché entrambi sappiamo cosa succede a farlo, ma mi aspetterò che ti sia già imbattuto in alcune situazioni sfavorevoli come lavorare con stati mutabili (mutable states), effetti collaterali privi di restrizioni (side effects), ed il design privo di principi (unprincipled design). Ora che abbiamo fatto le duvute introduzioni, iniziamo.

Lo scopo di questo capitolo è quello di farti capire cosa cerchiamo quando scriviamo programmi funzionali. Dobbiamo avere un'idea su ciò che rende un programma *funzionale*, altrimenti ci ritroveremmo in un mare di scarabocchi senza senso, cercando di evitare a tutti i costi gli oggetti in modo molto maldestro.
Abbiamo bisogno di un bersaglio verso cui puntare con il nostro codice, una sorta di bussola celeste per quando le acque si fanno impetuose.

Oggi esistono alcuni principi generali di programmazione, vari acronimi e mantra che ci guidano attraverso i tunnel bui di ogni applicazione: DRY (don't repeat yourself - non ripeterti), YAGNI (ya ain't gonna need it - non ne hai bisogno), basso accoppiamento alta coesione (loose coupling high cohesion), principio di minima sorpresa (principle of least surprise), responsabilità singola (single responsibility), e così via.

Non voglio insistere elencando ogni singola linea guida che ho sentito negli anni... il punto è che reggono in un ambiente funzionale, anche se toccano minimamente il nostro obiettivo. Quello che mi piacerebbe farti capire per ora, prima di addentrarci oltre, è la nostra intenzione quando ci metteremo alla tastiera; il nostro Xanadu funzionale.

<!--BREAK-->

## Un breve incontro

Cominciamo con un tocco di follia. Ecco un'applicazione sui gabbiani. Quando degli stormi si uniscono, formano stormi più grandi e quando si riproducono aumentano per il numero di gabbiani con cui si stanno riproducendo. Ora, si badi bene, questo non è destinato ad essere un buon codice orientato agli oggetti, è qui per mettere in evidenza i pericoli del nostro approccio moderno, basato sull'assegnazione. Ecco:

```js
var Stormo = function(n) {
  this.gabbiani = n;
};

Stormo.prototype.unione = function(altroStormo) {
  this.gabbiani += altroStormo.gabbiani;
  return this;
};

Stormo.prototype.riproduzione = function(altroStormo) {
  this.gabbiani = this.gabbiani * altroStormo.gabbiani;
  return this;
};

var stormo_a = new Stormo(4);
var stormo_b = new Stormo(2);
var stormo_c = new Stormo(0);

var result = stormo_a.unione(stormo_c)
    .riproduzione(stormo_b).unione(stormo_a.riproduzione(stormo_b)).gabbiani;
//=> 32
```

Chi sulla terra vorrebbe lavorare con un orribile abomino del genere? E' irragionevolmente difficile tener traccia degli stati.
E, santo cielo, il risultato è anche sbagliato! Doveva essere `16`, ma `stormo_a` viene alterato permanentemente durante il processo.`stormo_a`. Questa è anarchia nell'I.T.! Questa è aritmetica selvaggia degli animali!

Se non riesci a comprendere questo programma, va bene, nemmeno io ci riesco. Il punto è che lo stato di un programma ed i valori mutabili sono difficili da seguire, anche in un esempio così breve.

Proviamo di nuovo con un approccio pià funzionale:

```js
var unione = function(stormo_x, stormo_y) { return stormo_x + stormo_y };
var riproduzione = function(stormo_x, stormo_y) { return stormo_x * stormo_y };

var stormo_a = 4;
var stormo_b = 2;
var stormo_c = 0;

var risultato = unione(
  riproduzione(stormo_b, unione(stormo_a, stormo_c)), riproduzione(stormo_a, stormo_b)
);
//=>16
```

Bene, abbiamo ottenuta il giusto risultato. In meno codice. L'annidamento di funzione crea un po' di confusione...(rimedieremo a questa situazione nel capitolo 5). Il nuovo codice è migliore, ma cerchiamo di scavare più a fondo. Ci sono dei benefici nel chiamare le cose con il loro nome. Se avessimo fatto così, ci saremmo resi conto che stavamo lavorando con una semplice addizione (`unione`) ed una moltiplicazione (`riproduzione`).

Non c'è veramente niente di speciale riguardo a queste due funzioni, eccetto il loro nome. Rinominiamole quindi per mostrare la loro vera identità.

```js
var somma = function(x, y) { return x + y };
var moltiplica = function(x, y) { return x * y };

var storm_a = 4;
var storm_b = 2;
var storm_c = 0;

var risultato = somma(
  moltiplica(stormo_b, somma(stormo_a, stormo_c)), moltipica(stormo_a, stormo_b)
);
//=>16
```
E con questo, acquisiamo la conoscenza degli antichi:

```js
// proprietà associativa
somma(somma(x, y), z) == somma(x, somma(y, z));

// proprietà commutativa
somma(x, y) == somma(y, x);

// identità
somma(x, 0) == x;

// proprietà distributiva
moltiplica(x, somma(y,z)) == somma(moltiplica(x, y), moltiplica(x, z));
```

Ah certo, queste vecchie e fedeli proprietà matematice ci torneranno utili. Non preoccuparti se non le ricordate a memoria. Per molti di noi ci è voluto un po' per assimilare certe informazioni. Vediamo se possiamo usare queste proprietà per semplificare il nostro programma sui gabbiani.

```js
// Linea originale
somma(moltiplica(stormo_b, somma(stormo_a, stormo_c)), moltiplica(stormo_a, stormo_b));

// Applichiamo la proprità di identità per rimuovere una somma di troppo
// (somma(stormo_a, stormo_c) == stormo_a)
somma(moltiplica(stormo_b, stormo_a), moltiplica(stormo_a, stormo_b));

// Applichiamo la proprità distributiva per ottenere il nostro risultato
moltiplica(stormo_b, somma(stormo_a, stormo_a));
```

Ottimo! Non abbiamo avuto bisogno di scrivere neppure una riga di codice personalizzato oltre la nostra funzione di chiamata. Abbiamo incluso le definizioni di `somma` e `moltiplica` per completezza, ma non c'era veramente bisogno di scriverle - sicuramente abbiamo la possibilità di effettuare `somme` e `moltiplicazioni` grazie a qualche libreria scritta da qualcuno prima di noi.

Forse starai pensando "che razza di persona strana mette un esempio così matematico per iniziare". Oppure "i programmi reali non sono così semplici e non possono essere spiegati in questo modo". Ho scelto questo esempio perché molti di noi già conoscono l'addizione e la moltiplicazione, quindi è semplice vedere come qui la matematica può esserci di aiuto.

Non disperarti, durante questo libro, vedremo una spruzzatina di teoria delle categorie, insiemi e lambda calcolo per scrivere esempi dal mondo reale con la stessa semplicità e gli stessi risultati del nostro esempio sugli stormi di gabbiani. Non devi essere un matematico, ti sembrerà di utilizzare un normale framework o delle api.

Potresti rimanere sorpreso nell'apprendere che possiamo scrivere intere applicazioni, di uso quotidiano, sulla falsa riga delle linee di codice dell'esempio precedente. Programmi che abbiano proprietà orecchiabili. Programmi concisi, ma facili da comprendere. Programmi che non reinventino la ruota ogni volta. L'illegalità è una cosa buona se sei un criminale, ma in questo libro, vogliamo riconoscere e risptettare le leggi della matematica.

Vogliamo usare la teoria dove ogni pezzo tende ad incastrarsi così educatamente. Vogliamo rappresentare i nostri problemi specifici in termini generici, in pezzetti componibili e sfruttare le loro proprietà a nostro unico beneficio. Ci vorrà un po' più di disciplina rispetto al normale approccio "va bene tutto" della programmazione imperativa (vedremo la definizione precisa di programmazione imperativa più tardi nel libro, ma per ora considerala come tutto ciò che non è programmazione funzionale), ma il risultato di lavorare in una struttura matematica con dei principi vi stupirà.

Abbiamo visto un barlume della nostra stella polare funzionale, ma ci sono alcuni concetti concreti da cogliere prima di iniziare vermante il nostro viaggio.

[Capitolo 2: Funzioni di Prima Classe](ch2-it.md)
