# Capitolo 7: Hindley-Milner e Me

## Quale è il tuo tipo
Se sei nuovo nel mondo funzionale, non passerà molto tempo prima che ti ritroverai immerso nelle firme dei tipi. I tipi sono il meta linguaggio che consente a persone di background diversi di comunicare in modo conciso ed efficace. Per la maggior parte sono scritti con un sistema chiamato "Hindley-Milner", che esamineremo insieme in questo capitolo.

Quando si lavora con funzioni pure, le firme di tipo hanno un potere espressivo a cui la lingua inglese non può reggere il confronto. Queste firme sussurrano all'orecchio i segreti intimi di una funzione. In un'unica linea compatta, espongono comportamento e intenzione. Possiamo derivarne "teoremi liberi". I tipi possono essere dedotti, quindi non sono necessarie annotazioni di tipo esplicite. Possono essere precisi per il caso specifico o lasciati generali e astratti. Non sono solo utili per i controlli in fase di compilazione, ma risultano anche essere la migliore documentazione possibile disponibile. Le firme di tipo svolgono quindi un ruolo importante nella programmazione funzionale, molto più di quanto ci si potrebbe aspettare.

JavaScript è un linguaggio dinamico, ma ciò non significa che evitiamo i tipi. Stiamo ancora lavorando con stringhe, numeri, valori booleani e così via. È solo che non esiste alcuna integrazione a livello di linguaggio, quindi teniamo queste informazioni nelle nostre teste. Non preoccuparti, poiché utilizziamo le firme per la documentazione, possiamo utilizzare i commenti per servire al nostro scopo.

Sono disponibili strumenti di controllo del tipo per JavaScript come [Flow] (https://flow.org/) o il dialetto tipizzato, [TypeScript] (https://www.typescriptlang.org/). Lo scopo di questo libro è quello di fornire gli strumenti per scrivere codice funzionale in modo da restare fedeli al sistema di tipi standard utilizzato nei linguaggi FP.

## Tales from the Cryptic

Dalle pagine polverose dei libri di matematica, attraverso il vasto mare di white paper, tra i post di blog casuali del sabato mattina, fino al codice sorgente stesso, troviamo firme tipo Hindley-Milner. Il sistema è abbastanza semplice, ma richiede una rapida spiegazione e un po' di pratica per assorbire appieno il piccolo linguaggio.

```js
// capitalize :: String -> String
const capitalize = s => toUpperCase(head(s)) + toLowerCase(tail(s));

capitalize('puffo'); // 'Puffo'
```

Qui, `capitalize` prende una `String` e restituisce una `String`. Non importa l'implementazione, è la firma del tipo che ci interessa.

In HM, le funzioni sono scritte come `a -> b` dove `a` e `b` sono variabili per qualsiasi tipo. Quindi le firme di "capitalize" possono essere lette come "una funzione da `String` a `String`". In altre parole, prende una `String` come input e restituisce una` String` come output`

Diamo un'occhiata ad altre firme di funzione:

```js
// strLength :: String -> Number
const strLength = s => s.length;

// join :: String -> [String] -> String
const join = curry((what, xs) => xs.join(what));

// match :: Regex -> String -> [String]
const match = curry((reg, s) => s.match(reg));

// replace :: Regex -> String -> String -> String
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

`strLength` è la stessa idea di prima: prendiamo una `String` e ti restituiamo un `Number`.

Gli altri potrebbero lasciarti perplesso a prima vista. Senza comprendere appieno i dettagli, è sempre possibile visualizzare solo l'ultimo tipo come valore restituito. Quindi per "match" puoi interpretarlo come: Prende una "Regex" e una "String" e ti restituisce "[String]". Ma qui sta succedendo una cosa interessante che vorrei spendere un momento per spiegare se posso.

Per "match" siamo liberi di raggruppare la firma in questo modo:

```js
// match :: Regex -> (String -> [String])
const match = curry((reg, s) => s.match(reg));
```

Ah sì, raggruppare l'ultima parte tra parentesi rivela maggiori informazioni. Ora è vista come una funzione che accetta una `Regex` e ci restituisce una funzione da `String` a `[String]`. A causa del curry, questo è davvero il caso: dagli un `Regex` e recuperiamo una funzione in attesa del suo argomento `String`. Ovviamente non dobbiamo pensarla in questo modo, ma è bene capire perché l'ultimo tipo è quello restituito.

```js
// match :: Regex -> (String -> [String])
// onHoliday :: String -> [String]
const onHoliday = match(/holiday/ig);
```

Ogni argomento fa apparire un tipo nella parte anteriore della firma. `OnHoliday` è `match` che ha già una `Regex`.

```js
// replace :: Regex -> (String -> (String -> String))
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

Come puoi vedere tra le parentesi su `replace`, la notazione extra può diventare un po 'rumorosa e ridondante, quindi le omettiamo semplicemente. Possiamo fornire tutti gli argomenti in una volta se lo scegliamo, quindi è più semplice pensarlo come: `replace` prende una` Regex`, una `String`, un'altra `String` e ti restituisce una `String`.

Alcune ultime cose qui:

```js
// id :: a -> a
const id = x => x;

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));
```

La funzione `id` accetta qualsiasi vecchio tipo `a` e restituisce qualcosa dello stesso tipo `a`. Siamo in grado di utilizzare le variabili nei tipi proprio come nel codice. I nomi di variabili come `a` e `b` sono convenzioni, ma sono arbitrari e possono essere sostituiti con qualsiasi nome si desideri. Se sono la stessa variabile, devono essere dello stesso tipo. Questa è una regola importante quindi ribadiamo: `a -> b` può essere qualsiasi tipo `a` in qualsiasi tipo `b`, ma `a -> a` significa che deve essere lo stesso tipo. Ad esempio, `id` può essere `String -> String` o `Number -> Number`, ma non `String -> Bool`.

Allo stesso modo, `map` usa variabili di tipo, ma questa volta introduciamo `b` che può essere o meno lo stesso tipo di `a`. Possiamo leggerlo come: `map` prende una funzione da qualsiasi tipo `a` allo stesso o diverso tipo `b`, quindi prende un array di `a` e restituisce in un array di `b`.

Si spera che tu sia stato sopraffatto dalla bellezza espressiva in questo modalità di firma. Ci dice letteralmente cosa fa la funzione quasi parola per parola. Viene dara una funzione da `a` a `b`, un array di `a`, e ci fornisce un array di `b`. 

Essere in grado di ragionare sui tipi e sulle loro implicazioni è un'abilità che ti porterà lontano nel mondo funzionale. Non solo documenti, blog, ecc. diventeranno più digeribili, ma la firma stessa ti darà praticamente lezioni sulla sua funzionalità. Ci vuole pratica per acquisire una lettura fluente, ma se lo segui, un mucchio di informazioni diventeranno disponibili senza dover leggere sto maledetto manuale.


Eccone altri solo per vedere se riesci a decifrarli da solo.+

```js
// head :: [a] -> a
const head = xs => xs[0];

// filter :: (a -> Bool) -> [a] -> [a]
const filter = curry((f, xs) => xs.filter(f));

// reduce :: ((b, a) -> b) -> b -> [a] -> b
const reduce = curry((f, x, xs) => xs.reduce(f, x));
```

`reduce` è forse il più espressivo di tutti. È complicato, tuttavia, quindi non sentirti inadeguato se dovessi lottare con esso. Per i curiosi, cercherò di spiegarlo, anche se lavorare da soli sulla firma è molto più istruttivo.

Ehm, qui non va niente... guardando la firma, vediamo che il primo argomento è una funzione che si aspetta `b` e `a` e produce una `b`. Dove prendere questi `a` e `b`? Bene, i seguenti argomenti nella firma sono una `b` e un array di `a` quindi possiamo solo supporre che la `b` e ciascuna di quelle `a` saranno passati. Vediamo anche che il risultato della funzione è una `b` quindi sarà il nostro valore di output. Sapendo cosa fa reduce, possiamo affermare che l'indagine di cui sopra è accurata.

## Restringere le possibilità

Una volta introdotta una variabile di tipo, emerge una curiosa proprietà chiamata *[parametricity](https://en.wikipedia.org/wiki/Parametricity)*. Questa proprietà afferma che una funzione *agirà su tutti i tipi in modo uniforme*. Analizziamo:

```js
// head :: [a] -> a
```

Guardando "head", vediamo che ci vuole `[a]` per `a`. Oltre al tipo concreto `array`, non ha altre informazioni disponibili e, quindi, la sua funzionalità è limitata al solo lavoro sull'array. Cosa potrebbe fare con la variabile `a` se non ne sa niente? In altre parole, `a` dice che non può essere un tipo *specifico*, il che significa che può essere *qualsiasi* tipo, il che ci lascia con una funzione che deve funzionare in modo uniforme per *ogni* tipo concepibile. Questo è ciò che riguarda la *parametricità*. Indovinando l'implementazione, le uniche ipotesi ragionevoli sono che prende il primo, l'ultimo o un elemento da quell'array. Il nome "head" dovrebbe darci una mano.

Eccone un altro:

```js
// reverse :: [a] -> [a]
```

Dalla sola firma del tipo, cosa potrebbe fare `reverse`? Di nuovo, non può fare nulla di specifico per "a". Non può cambiare "a" in un tipo diverso o introdurremo una "b". Può ordinare? Beh, no, non avrebbe abbastanza informazioni per ordinare ogni tipo possibile. Può essere riorganizzato? Sì, suppongo che possa farlo, ma deve farlo esattamente nello stesso modo prevedibile. Un'altra possibilità è che possa decidere di rimuovere o duplicare un elemento. In ogni caso, il punto è che il possibile comportamento è fortemente ridotto dal suo tipo polimorfico.

Questa limitazione delle possibilità ci consente di utilizzare motori di ricerca con firme di tipo come [Hoogle] (https://hoogle.haskell.org/) per trovare una funzione che stiamo cercando. Le informazioni racchiuse strettamente in una firma sono davvero molto potenti.

## Libero come in Teorema

Oltre a dedurre le possibilità di implementazione, questo tipo di ragionamento ci fa guadagnare *teoremi liberi*. Quello che segue sono alcuni esempi casuali di teoremi estratti direttamente da [l'articolo di Wadler sull'argomento](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).


```js
// head :: [a] -> a
compose(f, head) === compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```

Non è necessario alcun codice per ottenere questi teoremi, seguono direttamente dai tipi. Il primo dice che se otteniamo la `head` del nostro array, allora eseguiamo una funzione `f` su di essa, che è equivalente, e incidentalmente, molto più veloce di, se prima eseguiamo `map (f)` su ogni elemento quindi prendiamo la `head` del risultato.

Potresti pensare, beh, è ​​solo buon senso. Ma l'ultima volta che ho controllato, i computer non hanno il buon senso. In effetti, devono disporre di un modo formale per automatizzare questo tipo di ottimizzazioni del codice. La matematica ha un modo di formalizzare l'intuitivo, il che è utile nel terreno rigido della logica del computer.

Il teorema di "filtro" è simile. Dice che se componiamo `f` e `p` per controllare quale dovrebbe essere filtrato, allora applichiamo effettivamente `f` tramite `map`, sarà sempre equivalente a mappare la nostra `f` quindi filtrare il risultato con il predicato `p`.

Questi sono solo due esempi, ma puoi applicare questo ragionamento a qualsiasi firma di tipo polimorfico e sarà sempre valido. In JavaScript sono disponibili alcuni strumenti per dichiarare le regole di riscrittura. Si potrebbe anche farlo tramite la stessa funzione `compose`. La costo è bassa e le possibilità sono infinite.

## Vincoli

Un'ultima cosa da notare è che possiamo vincolare i tipi a un'interfaccia.

```js
// sort :: Ord a => [a] -> [a]
```

Quello che vediamo sul lato sinistro della nostra freccia grossa qui è l'affermazione di un fatto: "a" deve essere un "Ordine". O in altre parole, `a` deve implementare l'interfaccia `Ord`. Cos'è "Ordine" e da dove viene? In un linguaggio tipizzato sarebbe un'interfaccia definita che dice che possiamo ordinare i valori. Questo non solo ci dice di più sulla `a` e su cosa fa la nostra funzione `sort`, ma limita anche il dominio. Chiamiamo queste dichiarazioni di interfaccia *vincoli di tipo*.

```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Qui abbiamo due vincoli: "Eq" e "Show". Quelli assicureranno che possiamo controllare l'uguaglianza della nostra "a" e stampare la differenza se non sono uguali.

Vedremo più esempi di vincoli e l'idea dovrebbe prendere più forma nei capitoli successivi.


## In sintesi

Le firme di tipo Hindley-Milner sono onnipresenti nel mondo funzionale. Sebbene siano semplici da leggere e scrivere, ci vuole tempo per padroneggiare la tecnica di comprensione dei programmi attraverso le sole firme. Aggiungeremo firme di tipo a ciascuna riga di codice da qui in avanti.

[Capitolo 08: Tupperware](ch08-it.md)