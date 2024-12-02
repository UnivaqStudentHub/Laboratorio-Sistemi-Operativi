# Introduzione
In questa sezione introduciamo **make**, un tool per la generazione di eseguibili e altri files non sorgente di un programma, partendo dai sorgenti del programma stesso.

Sfrutteremo questa utility per automatizzare la compilazione dei sorgenti C.

# Make vs Bash scripting
Per quanto l'automatizzazione della compilazione dei sorgenti C sia possibile anche attraverso l'utilizzo di script bash, ricompilare ogni volta tutti i sorgenti è molto dispendioso in termini di risorse. 

Nel caso in cui si modifichino soltanto alcuni sorgenti, non è necessario che i sorgenti non modificati vengano ricompilati nuovamente.

Di fatto l'utility **make**, controlla l'ultima volta che il sorgente è stato modificato, e in base a questo criterio decide se ricompilare o meno il file.

# Il makefile e la sua struttura
Per poter utilizzare **make** bisogna definire un **makefile**, un file di testo che consente di definire dipendenze tra files per la generazione di uno o più files target e/o l'esecuzione di comandi shell.

Fatto ciò è possibile usare **make** semplicemente scrivendolo nel terminale (purché make sia installato nella vostra macchina).
Ciò che fa questo comando è cercare all'interno della cartella corrente un file denominato **Makefile** o **makefile** in modo tale da leggerlo e effettuare le operazioni in esso indicate.

Un **makefile** contiene delle regole definite nella seguente forma:

```make
targets: prerequisites
   recipe / command
```
Dove la prima riga viene detta **dependency line**, mentre la seconda: **shell line**.

Vedremo tra qualche paragrafo come sia possibile specificare più **shell lines**, al momento vi basti sapere che per inserirne una bisogna inserire una tabulazione a inizio riga (come potete vedere dalla struttura in alto).

## Regole del makefile
Definendo una regola si crea una dipendenza tra i target files e i prerequisiti.

Nel momento in cui uno dei prerequisiti di una regola viene aggiornato, vengono rieseguiti i comandi associati a quella regola.

Consideriamo il seguente
> Esempio
>```make
>ex0.exe: ex0.o
>   gcc ex0.o -o ex0.exe
>
>ex0.o: ex0.c
>   gcc -c ex0.c
>```
> Abbiamo definito due regole:
> - la prima si occupa di buildare il file eseguibile a partire dal file oggetto **ex0.o**;
> - la seconda è necessaria affinché la prima possa essere eseguita: si genera quindi il file oggetto a partire dal sorgente **ex0.c**.
>
> Se dovessimo modificare il file **ex0.c**, allora verrebbero rieseguite nuovamente le azioni relative le due regole.
>
> L'opzione `-o <name>`, indica che l'output della compilazione deve essere un file con uno nome specifico.
>
> Mentre l'opzione `-c` del compilatore **gcc**, serve a specificare che la fase di compilazione deve limitarsi ai primi tre step: *preprocess, compile and assemble*.

Quindi se dei files target hanno come dipendenze dei files generati da altre regole, **make** controlla ricorsivamente se quelle dipendenze sono aggiornate prima di creare un file target.

Tornando allo scorso esempio... Se non modificassi in alcun modo le varie dipendenze, otterrei sul terminale:
```
make: 'ex0.exe' is up to date.
```
Altrimenti sul terminale verrebbero riportati i comandi eseguiti insieme ad eventuali comandi shell indicati nella regola.

## Albero delle dipendenze
Dato che **make** analizza le regole in modo top-down, quindi è buona prassi riportare le regole all'interno del makefile in ordine decrescente di dipendenza, 
partendo prima dalla regola che crea il file finale e poi specificando tutte le regole che portano alla creazione dei files richiesti per la sua creazione.

Consideriamo il seguente
> Esempio
>```make
>ex1.exe: ex1.o ex1a.o ex1b.o
>   gcc ex1.o ex1a.o ex1b.o -o ex1.exe
>
>ex1.o: ex1.c
>   gcc -c ex1.c
>
>ex1a.o: ex1a.c
>   gcc -c ex1a.c
>
>ex1b.o: ex1b.c
>   gcc -c ex1b.c
>```
> In questo caso `ex1.exe` per poter essere creato necessita di altri 3 files: `ex1.o`, `ex1a.o` ed `ex1b.o`.
> Abbiamo definito quindi 3 regole per poter generare questi files oggetto a partire dai loro sorgenti.

L'albero delle dipendenze dello scorso esempio sarà quindi:

>![](/img/dependency_tree.jpg)

## Componenti del makefile
Abbiamo già introdotto alcuni dei componenti del makefile, tra cui:
- Regole
- Dependency lines
- Shell lines

Introduciamo ora ulteriori componenti e ne approfondiamo alcuni:
- [Commenti](#commenti)
- [Shell lines](#shell-lines)
- [Macros](#macros)
- [Inference rules](#inference-rules)

### Commenti
È possibile inserire commenti a singola riga all'interno del makefile, la sintassi è la seguente: `#<text>`

Consideriamo il seguente
> Esempio
>```make
># Questo è un makefile per la generazione dell'eseguibile ex2.exe
>ex2.exe: ex2.o 
>  gcc ex2.o -o ex2.exe # Compiliamo il file ex2.o e come output otteniamo ex2.exe 
> # Fine regola
>```
> È possibile inserire commenti anche sulla dependency line.

### Shell lines
Fino ad ora abbiamo visto regole con una sola **shell line**, ma è possibile inserirne di più.

> Sintassi
>```make
>targets: prerequisites
>  recipe/command
>  ...
>  recipe/command
>```

Ogni volta che una **shell line** viene eseguita, **make** controlla l'*exit status* del comando.<br>Se l'*exit status* è 0 allora l'esecuzione è andata a buon fine, altrimenti **make** sospende l'esecuzione e ritorna un errore.

È possibile modificare quest'ultimo comportamento semplicemente anteponendo un `-` prima del comando.
</br>

Consideriamo un
> Esempio
>```make
> ex3: ex3.o
>  echo "now generating the executable file ex3"
>  - gcc ex3.o -o ex3
>```
> Si noti che in questo caso abbiamo omesso l'estensione per via della peculiarità dei sistemi UNIX-like di non dedurre il tipo di file dall'estensione ma dal suo contenuto.
> Ovviamente su sistemi Windows bisogna inserire l'estensione `.exe`.
>
> Abbiamo omesso la regola di generazione del file oggetto in quanto **make** la esegue implicitamente (provate voi stessi sulla vostra macchina).

### Macros
Specificare una **macro** significa inserire un **alias** nel makefile.

È possibile dichiarare una **macro** in questo modo:
>```make
><macro_name> = $(<other_macro>) # Dichiarazione attraverso un'altra macro
>
><macro_name> = <text>
>```
> Oppure una combinazione delle due.

Per poter espandere una macro si utilizza la seguente sintassi: `$(<macro_name>)`.

Consideriamo il seguente
> Esempio
>```make
>PROJ = /home/src/ex4
>INCL = -I $(PROJ)
>
> ex4.exe: ex4.o
>  gcc $(INCL) ex4.o -o ex4.exe
>```
> L'opzione `-I <path>` serve a specificare al compilatore C dove prendere gli header files inclusi nel file sorgente C.
> 
> Si noti che un cambiamento nell'header file non causerà la ricompilazione del file oggetto, lasciando quindi invariato il file eseguibile. **Make** stamperà su terminale la stringa:
>
>```
> make: 'ex4.exe' is up to date.
>```
> Per risolvere questo problema basterà inserire come dipendenza l'header file in questione.

Anche se meno usato, è possibile specificare una macro da terminale:
>```
> make <macro_name> = <text>
> 
> make <macro_name> = <other_macro>
>```
> Oppure una combinazione delle due.

Consideriamo un
> Esempio
>```bash
> make PROJ = /home/other_src/ex4
>```
> Tale dichiarazione sovrascriverà quella definita nel makefile dell'esempio precedente.

### Inference rules
In progetti di vaste dimensioni si cerca di generalizzare il più possibile il **Makefile**, in modo tale da renderlo il più conciso possibile.

Quindi le **inference rules**  sono regole in cui possono essere utilizzate alcune macros speciali (dette **automatic variables**) al fine di rendere il **Makefile** più breve.

Introduciamo alcune delle macros più usate:
- **$@** indica il nome del target;
- **$>** indica il primo prerequisito della regola;
- **$^** indica tutti i prerequisiti della regola.

E una **pattern rule**: `%`, che fa match con qualsiasi stringa.

Queste macros (inclusa la pattern rule) vengono automaticamente espanse nel momento in cui eseguiamo l'utility make.

Consideriamo il seguente
> Esempio
>```make
>PROJ = /home/src/ex4
>INCL = -I $(PROJ)
>
> %.exe: %.o 
>  gcc $(INCL) $> -o $@
>```
> In questo modo creiamo una dipendenza tra files `.exe` e `.o` dello stesso nome.
> Ricordiamo che **make** creerà automaticamente anche le regole per la creazione dei files oggetto.
> 
> Così possiamo compilare più sorgenti C con una sola regola. 

Vediamo un altro
> Esempio
>```make
>PROJ = /home/src/ex4
>INCL = -I $(PROJ)
>OTHER_RESOURCE = ex4a.o
>
> %.exe: %.o $(OTHER_RESOURCE)
>  gcc $(INCL) $^ -o $@
>```
> In questo caso abbiamo aggiunto un'ulteriore dipendenza e specificato che `ex4a.o` deve essere utilizzato per creare quegli eseguibili.
> <br>Notate infatti che abbiamo sostituito `$>` con `$^`.</br>

Questi sono soltanto esempi di base, tra poco affronteremo anche esempi più dettagliati e analizzeremo che cosa viene stampato sul terminale ogni volta che eseguiamo l'utility make.

Intanto si pensi al makefile del kernel di un sistema operativo. Per quanto esso possa essere generico, non sarà praticamente mai possibile sfruttare una sola regola per la compilazione dell'intero kernel: non solo perché **make** ne potrebbe introdurre di implicite, ma anche perché i files dipenderanno da gruppi di files possibilmente distinti.