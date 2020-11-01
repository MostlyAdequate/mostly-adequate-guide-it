# Capitolo 4: Currying

## Non posso vivere senza te
Mio padre una volta mi ha spiegato che si puo vivere senza alcune cose finche non le si acquisisce, una volta acquisite sembra impossibile vivere senza. 
Un forno a microonde è una cosa del genere. Smartphone, un altro. Le persone anziane tra noi ricorderanno una vita appagante senza Internet. Per me, il curry è in questa lista.

Il concetto è semplice: puoi chiamare una funzione con meno argomenti di quanto previsto. Restituisce una funzione che accetta gli argomenti rimanenti.

Puoi scegliere di chiamarla tutto in una volta o semplicemente inserire ogni argomento in modo frammentato.

```js
var somma = function(x) {
  return function(y) {
    return x + y;
  };
};

var incrementa = somma(1);
var incrementaDiDieci = add(10);

incrementa(2);
// 3

incrementa(2);
// 12
```

Qui abbiamo creato una funzione `somma` che accetta un argomento e restituisce una funzione. Chiamandola, la funzione restituita ricorda il primo argomento attraverso la `closure`. Chiamarla con entrambi gli argomenti contemporaneamente è un po 'un problema, tuttavia, quindi possiamo usare una speciale funzione di aiuto chiamata `curry` per rendere più facile la definizione e la chiamata di funzioni come questa.

Impostiamo alcune funzioni al curry per il nostro divertimento.

```js
var curry = require('lodash.curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```

Lo schema che ho seguito è semplice, ma importante. Ho posizionato strategicamente i dati su cui stiamo operando (String, Array) come ultimo argomento. Diventerà chiaro il motivo dopo l'uso.

```js
match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["tori_spelling", "tori amos"]);
// ["tori amos"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["tori_spelling", "tori amos"]);
// ["tori amos"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Chocolate Rain");
// 'Ch*c*l*t* R**n'
```

Ciò che è dimostrato qui è la capacità di "precaricare" una funzione con uno o due argomenti per ricevere una nuova funzione che ricordi quegli argomenti.

Ti incoraggio a installare lodash `npm install lodash`, copia il codice sopra e provalo nella sostituzione. Puoi farlo anche in un browser in cui sono disponibili lodash o ramda.

## Più di un gioco di parole / salsa speciale

Il curry è utile per molte cose. Possiamo creare nuove funzioni semplicemente dando alle nostre funzioni di base alcuni argomenti come visto in `hasSpaces`,` findSpaces` e `censored`.

Abbiamo anche la possibilità di trasformare qualsiasi funzione che lavora su singoli elementi in una funzione che lavora su array semplicemente avvolgendola con `map`:

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

Dare a una funzione un numero di argomenti inferiore a quello previsto viene generalmente chiamato *applicazione parziale*. L'applicazione parziale di una funzione può rimuovere molto codice inutile. Immagina come sarebbe la precedente funzione `allTheChildren` con la `map` non gestita mediante curry da lodash (nota che gli argomenti sono in un ordine diverso):

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

In genere non definiamo funzioni che funzionano sugli array, perché possiamo semplicemente chiamare `map (getChildren)` inline. Lo stesso con `sort`,` filter` e altre funzioni di ordine superiore (Funzione di ordine superiore: una funzione che accetta o restituisce una funzione).

Quando abbiamo parlato di *funzioni pure*, abbiamo detto che prendono 1 input per 1 output. Il curry fa esattamente questo: ogni singolo argomento restituisce una nuova funzione in attesa degli argomenti rimanenti. Quello, vecchio mio, è  1 ingresso 1 uscita.

Non importa se l'output è un'altra funzione, si qualifica come pura. Consentiamo più di un argomento alla volta, ma questo può essere visto come una semplice rimozione di `()` per comodità.

## In breve
Il curring è utile e mi piace molto lavorare con le funzioni curried su base giornaliera. È uno strumento che rende la programmazione funzionale meno prolissa e noiosa.

Possiamo creare nuove e utili funzioni al volo semplicemente passando alcuni argomenti e come bonus, abbiamo mantenuto la definizione della funzione matematica nonostante più argomenti.

Acquisiamo un altro strumento essenziale chiamato `compose`.

[Capitolo 5: Programmare per composizione](ch5-it.md)

## Esercizi

Una parola veloce prima di iniziare. Useremo una libreria chiamata *ramda* che applica il curry a ogni funzione per default. In alternativa puoi scegliere di usare *lodash-fp* che fa lo stesso ed è scritto/mantenuto dal creatore di lodash. Entrambi funzioneranno bene ed è una questione di preferenza.

[ramda] (http://ramdajs.com)
[lodash-fp] (https://github.com/lodash/lodash-fp)

Ci sono [unit test] (https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) da eseguire sui tuoi esercizi mentre li scrivi, oppure, per i primi esercizi, puoi semplicemente copiarli e incollarli in un javascript REPL se lo desideri.

Le risposte vengono fornite con il codice nel [repository per questo libro] (https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

```js
var _ = require('ramda');


// Esercizio 1
//==============
// Refactoring per rimuovere tutti gli argomenti applicando parzialmente la funzione

var words = function(str) {
  return _.split(' ', str);
};

// Esercizio 1.a
//==============
// Refactoring per rimuovere tutti gli argomenti applicando parzialmente la funzione.

var sentences = undefined;


// Esercizio 2
//==============
// Refactoring per rimuovere tutti gli argomenti applicando parzialmente le funzioni

var filterQs = function(xs) {
  return _.filter(function(x){ return match(/q/i, x);  }, xs);
};


// Esercizio 3
//==============
// Usa la funzione di supporto _keepHighest per effettuare il refactoring di max in modo da non fare riferimento ad alcuno
// argomenti

// LASCIARE IMMUTATO:
var _keepHighest = function(x,y){ return x >= y ? x : y; };

// RIFATTORIZZARE QUI:
var max = function(xs) {
  return _.reduce(function(acc, x){
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// Bonus 1:
// ============
// avvolgere array slice per essere funzionale e curried.
// //[1,2,3].slice(0, 2)
var slice = undefined;


// Bonus 2:
// ============
// usa slice per definire una funzione "take" che prende n elementi dall'inizio della stringa. Rendilo curried
// Il risultato per "Something" con n = 4 dovrebbe essere "Some"
var take = undefined;
```
