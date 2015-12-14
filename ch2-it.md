# Capitolo 2: Funzioni di Prima Classe

## Un breve ripasso
Quando diciamo che le funzioni sono di "prima classe", intendiamo dire che si comportano come chiunque altro... quindi come le normali classi (professore?). Possiamo trattare le funzioni come ogni altro tipo di dato e non c'è niente di particolarmente speciale in loro - possono essere salvate in array, passate come parametri, assegnate a variabili, quello che vi pare.

Questo è il corso base di JavaScript, ma vale la pena menzionare come una rapida ricerca di codice su github possa mostrare il fraintendimento collettivo, o forse una diffusa ignoranza riguardo al concetto. Dobbiamo proseguire con un esempio finto? Dobbiamo.

```js
var ciao = function(nome){
  return "Ciao " + nome;
};

var saluti = function(nome) {
  return ciao(nome);
};
```

Qui, la funzione che contiene `ciao` in `saluti` è completamente ridondante. Perché? Perché le funzioni possono essere *chiamate* in JavaScript. Quando `ciao` ha le `()` alla fine viene eseguita e ritorna un valore. Quando invece non le ha, ritorna semplicemente la funzione contenuta nella variabile. Giusto per esser chiari, diamo un'occhiata:


```js
ciao;
// function(nome){
//  return "Ciao " + nome
// }

ciao("jonas");
// "Ciao jonas"
```

Finché `saluti` non fa nient'altro che chiamare `ciao` con gli stessi identici argomenti, possiamo scrivere semplicemente:

```js
var saluti = ciao;


saluti("times");
// "Ciao times"
```

In altre parole, `ciao` è già una funzione che si aspetta un parametro, perché racchiuderla in un'altra funzione che richiama semplicemente `ciao` con lo stesso parametro? Non ha veramente senso. Sarebbe come indossare il vostro giaccone pesante nel cuore di Luglio solo per scoppiare di caldo e chiedere un ghiacciolo.

E' odiosamente prolisso, e come nell'esempio, una cattiva pratica racchiudere una funzione con un'altra al solo scopo di ritardarne la valutazione. (Vedremo perché tra un momento, ma ha a che fare con la manutenzionze.)

Comprendere chiaramente tutto ciò è cruciale prima di passare oltre, esaminiamo quindi qualche altro esempio divertente estratto dai moduli npm.

```js
// ignaro
var ottieniDatiServer = function(callback){
  return chiamataAjax(function(json){
    return callback(json);
  });
};

// illuminato
var ottieniDatiServer = chiamataAjax;
```

Il mondo è pieno di codice ajax proprio come questo. Di seguito la ragione per cui gli esempi sono equivalenti:

```js
// questa linea
return chiamataAjax(function(json){
  return callback(json);
});

// è la stessa di questa
return chiamataAjax(callback);

// quindi riscriviamo getServerStuff
var ottieniDatiServer = function(callback){
  return chiamataAjax(callback);
};

// ...che è equivalente a questo
var ottieniDatiServer = chiamataAjax; // <-- guarda mamma, niente parentesi ()
```

E questo, gente, è ciò che si trova in giro. Ancora un esempio e capirai perché sono così insistente.

```js
var BlogController = (function() {
  var indice = function(articoli) {
    return Views.index(articoli);
  };

  var mostra = function(articolo) {
    return Views.mostra(articolo);
  };

  var crea = function(parametri) {
    return Db.crea(parametri);
  };

  var aggiorna = function(articolo, parametri) {
    return Db.aggiorna(articolo, parametri);
  };

  var distruggi = function(articolo) {
    return Db.distruggi(articolo);
  };

  return {
    indice: indice, mostra: mostra, crea: crea, aggiorna: aggiorna, distruggi: distruggi
  };
})();
```

Questo controller è inutiler per il 99%. Potremmo riscriverlo così:

```js
var BlogController = {
  indice: Views.indice,
  mostra: Views.mostra,
  crea: Db.crea,
  aggiorna: Db.aggiorna,
  distruggi: Db.distruggi
};
```

...oppure rottamarlo del tutto dato che non fa nient'altro che raggruppare View e Db insieme.

## Perché preferire le funzioni di prima classe?

Bene, approfondiamo il motivo per cui preferire le funzioni di prima classe. Come abbiamo visto negli esempi `ottieniDatiServer` e `BlogController`, è facile aggiungere strati di riferimento indiretto che non hanno alcun valore reale e che aumentano soltanto la quantità di codice in cui cercare e da manutenere.

In aggiunta, se una funzione che stiamo racchiudendo inutilmente cambia, dobbiamo cambiare anche la funzione che la racchiude.

```js
httpGet('/articolo/2', function(json){
  return visualizzaArticolo(json);
});
```

Se `httpGet` dovesse cambiare per inviare un possibile errore `err`, avremmo bisogno di tornare indietro a cambiare la ogni chiamata ad `httpGet`.

```js
// ricerca tutte le chiamate a httpGet indietro nell'applicazione e passa esplicitamente err.
httpGet('/articolo/2', function(json, err){
  return visualizzaArticolo(json, err);
});
```

Se l'avessimo scritta come una funzione di prima classe, non avrebbe necessitato di alcuna modifica:

```js
// visualizzaPost viene richiamata da dentro httpGet con ogni argomento desiderato
httpGet('/post/2', visualizzaPost);
```

Oltre alla rimozione di funzioni inutili, dobbiamo nominare e referenziare i parametri. I nomi sono un po' un problema, come puoi vedere. Abbiamo potenziali termini impropri - soprattutto mano a mano che il codice invecchia e le richieste cambiano.

Avere più nomi per gli stessi concetti è una fonte comune di confusione nei progetti. C'è inoltre la questione del codice generico. Per esempio, queste due funzioni fanno esattamente la stessa cosa, ma una è decisamente più generica e riutilizzabile:

```js
// specifica per il nostro blog attuale
var articoliValidi = function(articoli) {
  return articoli.filter(function(articolo){
    return articolo !== null && articolo !== undefined;
  });
};

// di gran lunga più rilevante per progetti futuri
var compatta = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

Utilizzando una denominazione specifica, ci siamo apparentemente legati a dati specifici (in questo caso gli `articoli`). Questo ogni tanto accade e porta a dover reinventare cose già fatte.

Devo dire che, proprio come nel codice orientato agli oggetti, è necessario essere consapevoli che l'operatore `this` è pronto ad azzannarvi alla giugulare. Se una funzione sottostante usa `this` e lo chiamiamo di prima classe, siamo soggetti all'ira di questa astrazione.

```js
var fs = require('fs');

// pauroso
fs.readFile('venerdi_pazzo.txt', Db.save);

// un po' meno
fs.readFile('venerdi_pazzo.txt', Db.save.bind(Db));

```

Dopo aver effettuato il bind su se stesso, `Db` è libero di accedere al codice spazzatura del suo prototipo. Evito di usare `this` come un pannolino sporco. Non ce n'è davvero alcun bisogno nella programmazione funzionale. Tuttavia, quando ci si interfaccia con le altre librerie, dobbiamo adeguarci al pazzo mondo che ci circonda.

Alcuni sosterranno che `this` è necessario per la velocità. Se siete il tipico micro-ottimizzatore, per favore chiudete questo libro. Se non riuscite ad ottenere indietro i soldi, forse lo potete scambiare per qualcosa di più intricato.

E con questo, siamo pronti ad andare avanti.

[Capitolo 3: Felicità Pura con Funzioni Pure](ch3-it.md)
