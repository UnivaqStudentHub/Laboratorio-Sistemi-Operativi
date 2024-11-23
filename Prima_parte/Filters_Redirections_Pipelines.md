# Indice
* [Introduzione](#filters-redirection-e-pipelines)
* [Filtri](#filtri)
* [Pipelines e redirezione](#pipeline-e-redirezione)


# Filters, Redirection e Pipelines
* Il **filtro** è un programma o una subrutine che prende dati in standard input e "scrive" il risultato allo standard output. Parte più importante è che il filtro **NON** modifica il file in input.
* La **redirection** è un meccanismo che permette di cambiare il flusso di dati in ingresso e in uscita di un programma tramite l'utilizzo di `>` e `<`
* Una **pipeline** è una sequenza di comandi collegati tramite pipe `|` che permette di passare il risultato di un comando come input di un altro comando. Le pipelines possono anche essere utilizzate insieme ai filtri.

# Filtri

## **<span style="color:red">head</span>**
`head` è un filtro che permette di visualizzare le prime righe di un file di testo

```bash
head [options] [file_name]
```

**Di default sono visualizzate le prime 10 linee**

### **`head -n`**

```bash
head -n [number] [file_name]
```
Il `number` indica il numero di **righe** che si vogliono visualizzare


### **`head -c`**
```bash
head -c [number] [file_name]
```
Il `number` indica il numero di **bytes** che si vogliono visualizzare ( 1 byte = 1 char )

## **<span style="color:red">tail</span>**
`tail` è un filtro che permette di visualizzare le ultime righe di un file di testo

```bash
tail [options] [file_name]
```
Le opzioni sono le stesse di [head](#head--n), unica differenza è che i bytes e i caratteri verranno conteggiati dalla fine verso l'inizio del file

## **<span style="color:red">sort</span>**
`sort` è un filtro che permette di ordinare testo e file binari ( di default ordina in ordine alfabetico )

**Verrà fatto il sort anche delle linee vuote, che verranno inserite a inizio file**

```bash
sort [options] [file_name]
```
## **<span style="color:red">nl</span>**
`nl` è un filtro che permette di numerare le righe di un file di testo

```bash
nl [options] [file_name]
```

> Esempio:
>```bash
>nl -s '. ' -w 10 persone.txt
>```
>Esegue una formattazione personalizzata <br>
> * Il comando `-w 10` indica che l'output verrà tabulato di 10 <br>
> * Il comando `-s '. '` indica che verrà inserito un punto e uno spazio tra il numero di riga e il testo
> ![](/img/nl.png)

## **<span style="color:red">wc</span>**
`wc` è un filtro che permette di contare le righe, le parole e i caratteri di un file di testo

```bash
wc [options] [path]
```
>Esempio:
><br></br>
>![](/img/wc_base.png)
><br></br>
>L'output di default è così composto: <br></br>
> `linee | parole | caratteri | nome del file`

Tipi di `options` possibili:
* `-c`: visualizza solo il numero di bytes
* `-m`: visualizza solo il numero di caratteri
* `-l`: visualizza solo il numero di righe
* `-w`: visualizza solo il numero di parole

## **<span style="color:red">cut</span>**
`cut` è un filtro che permette di selezionare parti di una riga di testo separate da un delimitatore

```bash
cut [options] [file_name]
```

Prendendo come testo il seguente file:<br>
![](/img/test_originale.png)

Eseguiamo questo comando:
```bash
cut —f <selezione colonne> -d <separatore> test.txt
```
Vediamo cosa succede sostituendo i parametri tra <>:
![](/img/cut_;.png)
<br></br>
Dato che nel file di testo il `;` non è presente, il comando restituisce tutto il file senza nessuna separazione

<br></br>

![](/img/cut_spazio_1.png)
Dato che nel file il testo è separato da `spazi`, il comando restituisce tutti i caratteri per ogni riga fino al primo spazio

<br></br>


![](/img/cut_spazio_1_2.png)
Dato che nel file il testo è separato da `spazi`, il comando restituisce tutti i caratteri per ogni riga fino al secondo spazio


## **<span style="color:red">uniq</span>**
`uniq` è un filtro che permette di rimuovere le righe duplicate di un file di testo

**Attenzione: le righe duplicate devono essere consecutive, cioè non separate da spazi bianchi**

```bash
uniq [file_name]
```

## **<span style="color:red">tac</span>**

`tac` è un filtro che permette di visualizzare il contenuto di un file di testo in ordine inverso

```bash
tac [file_name]
```

![](/img/tac.png)

## **<span style="color:red">diff</span>**
Con `diff` è possibile confrontare due file di testo e visualizzare le differenze linea per linea

```bash
diff [options] [file_name_1] [file_name_2] 
```

![]\(/img/diff.png)

La prima riga va letta in questo modo:<br>

```righe del primo file | lettera | righe del secondo file```

La `lettera` può essere:
* `a`: riga da aggiungere
* `d`: riga da eliminare
* `c`: riga da sostituire

Inoltre:
* `<` indica le righe del primo file
* `>` indica le righe del secondo file
* `---` è il separatore fra files

>Esempio di lettura della prima riga:<br></br>
>`1,7c1,7`<br></br>
> il range di righe *1* - *7* del **primo file** va sostituito con il rispettivo range *1* - *7* del **secondo file** per rendere i due files uguali

**Ricordiamo che i filtri non modificano i files in input!**

## **<span style="color:red">egrep</span>**

```bash
egrep [options] [pattern] [file_name]
```

Il comando **grep** o **egrep** (extended grep) cerca nel file di testo una riga di testo che corrisponde ad uno o più patterns passati come parametro.<br>
**Si usa perlopiù con le [Espressioni Regolari](/Espressioni_regolari.md)**.

> Esempio:
> <br>
> ![]\(/img/egrep1.png)


## **<span style="color:red">sed</span>**

È un comando che consente di cercare e rimpiazzare un'espressione con un'altra. <br></br> Il formato è il seguente:


```bash
sed [command] [file_name]
```

È possibile combinare diversi pattern per rimpiazzare stringhe o per cancellarle, ad esempio:

> Esempio:
> ```bash
> sed -e 's/a/A/g' persone.txt
> ```
> Rimpiazza tutte le occorrenze di a in A in tutto il file.
>
> **NOTA:** **s** sta per sostituisci e **g** per globalmente, senza **g** il comando sostituisce solo la ***prima istanza*** della riga di testo

Vediamo un altro esempio sfruttando il seguente parametro:
* **-e** - se usato prima dell'espressione per rimpiazzare, è utile per combinarne più insieme

> Esempio:
> ```bash
> sed -e 's/a/A/' -e 's/b/B/' persone.txt
> ```
> Rimpiazza la prima occorrenza delle lettere minuscole a e b con le loro lettere maiuscole

È possibile anche cancellare stringhe:
> Esempio:
> ```bash
> sed '/^#/d' persone.txt
> ```
> Elimina tutte le linee che iniziano con #


Come avete notato, è possibile utilizzare questo comando con le espressioni regolari.

Ricordiamo nuovamente che questo filtro, come gli altri, non modificano i files in input.

# File descriptors
 Normalmente sono degli interi non-negativi che il kernel utilizza per identificare i files a cui accede un processo.

 Ogni volta che un file viene creato o aperto, il kernel ritorna un file descriptor che viene usato quando vogliamo leggere o scrivere su quel file.

 Per convenzione, tutte le shell aprono 3 file descriptors ogni volta che un programma viene eseguito. Questi file descriptors di default sono connessi alla shell, ma è possibile redirezionare alcuni di essi verso un file.

 I tre file descriptors aperti dalla shell raffigurano i canali di comunicazione di cui la shell si serve per effettuare operazioni di I/O.

 Questi canali di comunicazione vengono presentati nel paragrafo successivo.


# Pipeline e redirezione

I flussi di dati nella shell si servono di canali di comunicazione. <br></br> Specificatamente, si parla di:

* **STDIN (0)** - è lo standard input

* **STDOUT (1)** - è lo standard output

* **STDERR (2)** - è lo standard error

Lo standard error viene rediretto di default nello standard output:

> Esempio:
> <br>
> ![]\(/img/pipeline1.png)

Per redirezionare manualmente l'input e l'output si utilizzano i seguenti operatori:

* **>** - permette di redirezionare l'output di un programma o di un comando in un file, piuttosto che stamparlo a schermo (STDOUT).

* **<** - permette di specificare l'input di un comando o di un programma

* **>>** - permette di redirezionare l'output come `>` facendo però "append" dell'output nel file destinazione

È possibile concatenare più redirection.

## **<span style="color:red">[n]<&word e [n]>&word</span>**
Permettono di duplicare rispettivamente file descriptors di input e di output.

> Esempio:
> ```bash
> ls -la TestingDirectory > esempio.txt
> ```
> Se **TestingDirectory** non è presente nella working directory corrente, allora il file esempio.txt sarà vuoto.
> Questo perché di default solo lo **STDOUT** viene rediretto su file.
> <br></br>
> Osserviamo la seguente soluzione:
> ```bash
> ls -la TestingDirectory > esempio.txt 2>&1
> ```
> con `2>&1` indichiamo che lo **STDERR** deve diventare una copia del file descriptor di **STDOUT**.
> <br></br>
> In questo modo qualsiasi output che arriva su **STDERR**, viene inviato anche a **STDOUT** e può essere correttamente rediretto verso il file.

Per maggiori dettagli su questi duplicatori potete consultare la documentazione ufficiale di GNU per comprendere meglio il loro funzionamento.

## **<span style="color:red">Pipeline</span>**
Una pipeline è una catena di processi collegati tra loro in modo tale che lo **STDOUT** di un processo sia collegato allo **STDIN** del processo successivo.

Questi processi vengono eseguiti (su bash perlomeno) in sottoprocessi distinti e, anche se la politica di scheduling dovesse favorire l'esecuzione di un processo rispetto a un altro, 
la pipe si occupa di riordinare lo stream dei dati.

L'exit status della pipeline è quello dell'ultimo processo della catena.

La shell resta in attesa della terminazione di tutti i processi della pipeline prima di ridare il controllo alla shell.

> Esempio
> ```bash
> ls -la | egrep '^[d]' | egrep '([0-9]*)$'
>```
> Il seguente esempio filtra tutte le directories della directory corrente e considera solo quelle che hanno un nome che termina con una sequenza (anche nulla) di cifre.

## **<span style="color:red">Pipeline VS Redirection</span>**
Per ciò che riguarda l'output, il loro comportamento è praticamente lo stesso. 
Notiamo però che con le redirection non è possibile inviare direttamente il risultato della computazione
di un programma come input di un altro. Bisogna necessariamente passare per un file d'appoggio.

> Esempio
>```bash
> program1 > output.txt && program2 < output.txt
>```
> Ovviamente con le pipe non si ha questa necessità.
>```bash
> program1 | program2
>```
<br></br>

--------------------
[Go to home](/README.md)
