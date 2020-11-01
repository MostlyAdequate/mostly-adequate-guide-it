# Capitolo 3: Pura Felicità con Funzioni Pure

## Oh, per essere di nuovo puro

Una cosa che dobbiamo avere ben chiara è il concetto di funzione pura.

>Una funzione pura è una funzione che, introdotto lo stesso ingresso, ritorna sempre lo stesso risultato e non ha nessun effetto indesiderato visibile.

Prendiamo ad esempio `slice` e `splice`. Sono due funzioni che fanno esattamente la stessa cosa, ma bada bene, in un modo totalmente differente. Diciamo che `slice` è *pura* perché per ogni ingresso, ritorna sempre lo stesso risultato, garantito. `splice`, tuttavia, modifica permanentemente l'array che le viene passato, restituendolo cambiato. Il che è un effetto visibile.

```js
var xs = [1,2,3,4,5];

// pura
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]


// impura
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []
```

Nella programmazione funzione, evitiamo le funzioni poco pratiche come `splice` che *mutano* i dati. I dati non dovrebbero mai essere modificati, ed è ciò che cerchiamo di ottenere con le nostre funzioni affidabili che ritornano sempre lo stesso risultato, non come nelle funzioni che creano confusione come `splice`.

Vediamo un altro esempio.

```js
// impura
var minimo = 21;

var controllaEta = function(eta) {
  return eta >= minimo;
};



// pura
var controllaEta = function(eta) {
  var minimo = 21;
  return eta >= minimo;
};
```

Nella porzione impura, `controllaEta` dipende dalla variabile mutabile `minimo` per determinare il risultato. In altre parole, dipende dallo stato del sistema che è deludente perché aumenta il carico cognitivo introducendo un ambiente esterno.

Potrebbe non essere così evidente in questo esempio, ma questo affidamento sullo stato è una delle maggiori cause della complessità del sistema (http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf). Questa `controllaEta` potrebbe restituire risultati differenti in base a fattori esterni all'ingresso, che non solo non la rende pura, ma impegna anche le nostre menti ogni volta che stiamo ragionando sul funzionamento del programma.

La sua forma pura, d'altra parte, è completamente autosufficiente. Possiamo anche rendere `minimo` immutabile, il che conserva la purezza non modificando mai lo stato. Per fare ciò, dobbiamo creare un oggetto da congelare.

```js
var statoImmutabile = Object.freeze({
  minimo: 21
});
```

## Gli effetti collaterali possono includere ...

Diamo un'occhiata più a questi "effetti collaterali" (Side Effects) per migliorare la nostra intuizione. Allora, qual è questo indubbiamente nefasto * effetto collaterale * menzionato nella definizione di *funzione pura*? Ci riferiremo a * effetto * come qualsiasi cosa che si verifica nella nostra computazione diverso dal calcolo di un risultato.

Non c'è niente di intrinsecamente negativo nei side effects e li useremo ovunque nei capitoli a venire. È quella parte * laterale * che porta la connotazione negativa. L'acqua da sola non è un'incubatrice di larve intrinseca, è la parte *stagnante* che produce gli sciami, e ti assicuro che gli effetti *collaterali* sono un terreno fertile simile nei tuoi programmi.

> Un *effetto collaterale* è un cambiamento dello stato del sistema o *interazione* con il mondo esterno che si verifica durante il calcolo di un risultato.

Gli effetti collaterali possono includere, ma non sono limitati a

  * interagire con il file system
  * inserire un record in un database
  * effettuare una chiamata http
  * mutazioni
  * stampa sullo schermo / registrazione
  * ottenere l'input dell'utente
  * interrogare il DOM
  * accesso allo stato del sistema

E l'elenco potrebbe continuare all'infinito. Qualsiasi interazione con il mondo al di fuori di una funzione è un effetto collaterale, che è un fatto che potrebbe indurti a sospettare che la programmazione funzionale sia inutile senza di essi. La filosofia della programmazione funzionale postula che gli effetti collaterali siano una causa primaria di comportamenti scorretti.

Non è che ci sia vietato usarli, piuttosto vogliamo contenerli e gestirli in modo controllato. Impareremo come farlo quando arriveremo a funtori e monadi nei capitoli successivi, ma per ora proviamo a mantenere queste funzioni insidiose separate da quelle pure.

Gli effetti collaterali squalificano una funzione dall'essere *pura*. E ha senso: le funzioni pure, per definizione, devono sempre restituire lo stesso output dato lo stesso input, cosa che non è possibile garantire quando si tratta di argomenti al di fuori della nostra funzione locale.

Diamo uno sguardo più da vicino al motivo per cui insistiamo sullo stesso output per input. Mettiti il grembiulino, daremo un'occhiata a un po 'di matematica di terza media.

## Matematica da terza media

Da mathisfun.com:

> Una funzione è una relazione speciale tra i valori:
> Ciascuno dei suoi valori di input restituisce esattamente un valore di output.

In altre parole, è solo una relazione tra due valori: l'input e l'output. Sebbene ogni input abbia esattamente un output, tale output non deve necessariamente essere univoco per input. Di seguito viene mostrato un diagramma di una funzione perfettamente valida da `x` a `y`;

<img src="images/function-sets.gif" />(http://www.mathsisfun.com/sets/function.html)

Al contrario, il diagramma seguente mostra una relazione che *non* è una funzione poiché l'input `5` punta a diversi output:

<img src="images/relation-not-function.gif" />(http://www.mathsisfun.com/sets/function.html)

Functions can be described as a set of pairs with the position (input, output): `[(1,2), (3,6), (5,10)]` (It appears this function doubles its input).

Le funzioni possono essere descritte come un insieme di coppie con posizione (input, output): `[(1,2), (3,6), (5,10)]` (Sembra che questa funzione raddoppi il suo input).

O forse una tabella:
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

O anche come un grafico con `x` come input e `y` come output:

<img src="images/fn_graph.png" width="300" height="300" />


Non sono necessari dettagli di implementazione se l'input determina l'output. Poiché le funzioni sono semplicemente mappature di input su output, è possibile semplicemente annotare i valori letterali degli oggetti ed eseguirli con `[]` invece di `()`.

```js
var toLowerCase = {"A":"a", "B": "b", "C": "c", "D": "d", "E": "e", "D": "d"};

toLowerCase["C"];
//=> "c"

var isPrime = {1:false, 2: true, 3: true, 4: false, 5: true, 6:false};

isPrime[3];
//=> true
```

Naturalmente, invece di scrivere a mano le cose, potresti voler calcolare il risultato, ma questo illustra un modo diverso di pensare alle funzioni. (Potresti pensare "e le funzioni con più argomenti?". In effetti, questo presenta un piccolo inconveniente quando si pensa in termini di matematica. Per ora, possiamo raggrupparle in un array o semplicemente pensare agli `arguments` oggetto come input. Quando impareremo a *currying*, vedremo come possiamo modellare direttamente la definizione matematica di una funzione.)

Ecco la drammatica rivelazione: le funzioni pure *sono* funzioni matematiche e sono l'essenza della programmazione funzionale. La programmazione con questi angioletti può fornire enormi vantaggi. Diamo un'occhiata ad alcuni motivi per cui siamo disposti a fare di tutto per preservare la purezza.

## Il caso della purezza
 
### Cacheable

Per i principianti, le funzioni pure possono sempre essere memorizzate nella cache tramite input. Questo viene in genere fatto utilizzando una tecnica chiamata memoizzazione:

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // restituisce il valore in cache per l'input 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // restituisce la cache per l'input 5
//=> 25
```

Ecco un'implementazione semplificata, sebbene siano disponibili molte versioni più robuste.

```js
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
```

Qualcosa da notare è che puoi trasformare alcune funzioni impure in funzioni pure ritardando la valutazione:

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

La cosa interessante qui è che in realtà non effettuiamo la chiamata http: restituiamo invece una funzione che lo farà quando viene chiamata. Questa funzione è pura perché restituirà sempre lo stesso output dato lo stesso input: la funzione che effettuerà la particolare chiamata http dati ʻurl` e `params`.

La nostra funzione `memoize` funziona bene, sebbene non memorizzi nella cache i risultati della chiamata http, piuttosto memorizza nella cache la funzione generata.

Questo non è ancora molto utile, ma presto impareremo alcuni trucchi che lo renderanno tale. Il punto è che possiamo memorizzare nella cache ogni funzione, non importa quanto distruttiva possa sembrare.

### Portabile / Auto-documentazione

Pure functions are completely self contained. Everything the function needs is handed to it on a silver platter. Ponder this for a moment... How might this be beneficial? For starters, a function's dependencies are explicit and therefore easier to see and understand - no funny business going on under the hood.

Le funzioni pure sono completamente autonome. Tutto ciò di cui ha bisogno la funzione viene consegnato su un piatto d'argento. Rifletti su questo per un momento... Come questa caratteristica potrebbe essere utile? Per i principianti, le dipendenze di una funzione sono esplicite e quindi più facili da vedere e da capire: non ci sono magie.

```js
//impure
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

//pure
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};
```

L'esempio qui dimostra che la funzione pura deve essere onesta riguardo alle sue dipendenze e, come tale, dirci esattamente cosa sta facendo. Solo dalla sua firma (signature), sappiamo che userà un `Db`, `Email` e `attrs` che dovrebbero essere a dir poco significativi.

Impareremo come rendere funzioni come questa, pure, senza limitarsi a rimandare la valutazione, ma il punto dovrebbe essere chiaro che la forma pura è molto più istruttiva della sua subdola controparte impura che dipende da Dio sa cosa.

Un'altra cosa da notare è che siamo costretti a "iniettare" dipendenze o passarle come argomenti, il che rende la nostra app molto più flessibile perché abbiamo parametrizzato il nostro database o client di posta o quello che hai (non preoccuparti, vedremo un modo per renderlo meno noioso di quanto sembri). Se dovessimo scegliere di utilizzare un Db diverso, dobbiamo solo chiamare la nostra funzione con esso. Se dovessimo trovarci a scrivere una nuova applicazione in cui vorremmo riutilizzare questa funzione affidabile, diamo semplicemente a questa funzione qualsiasi tipo di "Db" ed "Email" che abbiamo in quel momento.

In un'impostazione JavaScript, la portabilità potrebbe significare serializzare e inviare funzioni su un socket. Potrebbe significare eseguire tutto il codice della nostra app nei web worker. La portabilità è un tratto potente.

Contrariamente ai metodi e alle procedure "tipici" della programmazione imperativa, radicati in profondità nel loro ambiente tramite lo stato, le dipendenze e gli effetti disponibili, le funzioni pure possono essere eseguite ovunque vogliamo.

Quando è stata l'ultima volta che hai copiato un metodo in una nuova app? Una delle mie citazioni preferite proviene dal creatore di Erlang, Joe Armstrong: "Il problema con i linguaggi orientati agli oggetti è che hanno tutto questo ambiente implicito che si portano dietro. Volevi una banana ma quello che hai ottenuto è stato un gorilla con banana ... e l'intera giungla ".

### Testabile

Successivamente, arriviamo a realizzare che le funzioni pure rendono i test molto più facili. Non dobbiamo simulare un gateway di pagamento che rispecchi quello "reale" o impostare e affermare lo stato dopo ogni test. Diamo semplicemente input alla funzione e dichiariamo output.

In effetti, troviamo che la comunità funzionale apre la strada a nuovi strumenti di test che possono far unire le nostre funzioni con l'input generato e affermare che le proprietà mantengono l'output. Va oltre lo scopo di questo libro, ma ti incoraggio vivamente a cercare e provare *Quickcheck*, uno strumento di test su misura per un ambiente puramente funzionale.

### Ragionevole

Molti credono che la più grande vittoria quando si lavora con funzioni pure sia la *trasparenza referenziale*. Un blocco di codice è referenzialmente trasparente quando può essere sostituito con il suo valore valutato senza modificare il comportamento del programma.

Poiché le funzioni pure restituiscono sempre lo stesso output dato lo stesso input, possiamo fare affidamento su di esse per restituire sempre gli stessi risultati e quindi preservare la trasparenza referenziale. Vediamo un esempio.


```js

var Immutable = require("immutable");

var decrementHP = function(player) {
  return player.set("hp", player.get("hp")-1);
};

var isSameTeam = function(player1, player2) {
  return player1.get("team") === player2.get("team");
};

var punch = function(player, target) {
  if (isSameTeam(player, target)) {
    return target;
  } else {
    return decrementHP(target);
  }
};

var jobe = Immutable.Map({name:"Jobe", hp:20, team: "red"});
var michael = Immutable.Map({name:"Michael", hp:20, team: "green"});

punch(jobe, michael);
//=> Immutable.Map({name:"Michael", hp:19, team: "green"})
```

`decrementHP`, `isSameTeam` e `punch` are all pure and therefore referentially transparent. We can use a technique called *equational reasoning* wherein one substitutes "equals for equals" to reason about code. It's a bit like manually evaluating the code without taking into account the quirks of programmatic evaluation. Using referential transparency, let's play with this code a bit.

sono tutte funzione pure e quindi referenzialmente trasparenti. Possiamo usare una tecnica chiamata *ragionamento equazionale* (equational reasoning) in cui si sostituisce "uguale a uguale" al ragionamento sul codice. È un po 'come valutare manualmente il codice senza tenere conto delle stranezze della valutazione programmatica. Usando la trasparenza referenziale, giochiamo un po 'con questo codice.

renderemo inline `isSameTeam`.

```js
var punch = function(player, target) {
  if (player.get("team") === target.get("team")) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Poiché i nostri dati sono immutabili, possiamo semplicemente sostituire i team con il loro valore effettivo

```js
var punch = function(player, target) {
  if ("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Vediamo che in questo caso è falso, quindi possiamo rimuovere l'intero ramo if

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

E se incorporiamo `decrementHP`, vediamo che, in questo caso, il pugno (punch) diventa una chiamata per decrementare "hp" di 1.

```js
var punch = function(player, target) {
  return target.set("hp", target.get("hp")-1);
};
```

Questa capacità di ragionare sul codice è eccezionale per il refactoring e la comprensione del codice in generale. In effetti, abbiamo utilizzato questa tecnica per eseguire il refactoring del nostro programma stormo di gabbiani. Abbiamo utilizzato il ragionamento equazionale per sfruttare le proprietà di addizione e moltiplicazione. In effetti, utilizzeremo queste tecniche in tutto il libro.

### Parallel Code

Infine, ed ecco il colpo di grazia, possiamo eseguire qualsiasi funzione pura in parallelo poiché non necessita di accesso alla memoria condivisa e non può, per definizione, avere una race condition a causa di qualche effetto collaterale.

Questo è molto possibile in un ambiente js lato server con thread così come nel browser con i web worker, sebbene la cultura attuale sembri evitarlo a causa della complessità quando si tratta di funzioni impure.

## In sintesi

Abbiamo visto cosa sono le funzioni pure e perché noi, come programmatori funzionali, crediamo che siano l'abito da sera del gatto. Da questo punto in poi, ci sforzeremo di scrivere tutte le nostre funzioni in modo puro. Avremo bisogno di alcuni strumenti extra per aiutarci a farlo, ma nel frattempo proveremo a separare le funzioni impure dal resto del codice puro.

Scrivere programmi con funzioni pure è un po 'laborioso senza alcuni strumenti extra nella nostra cintura. Dobbiamo destreggiarci tra i dati passando argomenti dappertutto, ci è vietato usare lo stato, per non parlare degli effetti. Come si fa a scrivere questi programmi masochisti? Acquisiamo un nuovo strumento chiamato curry.

[Chapter 4: Currying](ch4.md)
