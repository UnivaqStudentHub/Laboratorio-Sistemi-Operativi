# Introduzione
La shell (bash) mette a disposizione una serie di costrutti e funzionalità che consente di programmare degli script bash in grado di sfruttare a pieno le funzionalità della shell. 

# Variabili
Le variabili permettono di leggere/salvare dei valori all'interno dello script.
<br></br>
Il loro valore può essere letto ponendo il simbolo `$` prima del nome della variabile, mentre l'assegnazione avviene tramite il simbolo `=` **(senza spazi)**.
<br></br>
In bash le variabili non sono tipate.

> Esempio
> ```bash
>test=100
>echo Il valore è $test
>```
> Viene stampato "Il valore è 100".



# Variabili di ambiente
Le variabili di ambiente sono variabili che vengono settate all'avvio del sistema e sono disponibili per tutti i programmi che vengono eseguiti. Esempi di variabili di ambiente sono PATH, HOME, USER, SHELL, PWD, etc.

# Variabili speciali
Le variabili speciali sono variabili che vengono settate automaticamente dal sistema operativo e sono disponibili per tutti i programmi che vengono eseguiti.
* **$0** - Nome del programma
* **$1 - $9** - Argomenti passati al programma
* **$#** - Numero di argomenti passati in input al programma
* **$\*** e **\$@** - Tutti gli argomenti passati al programma
* **$?** - Codice di ritorno dell'ultimo processo eseguito
* **$$** - PID del processo corrente
* **$USER** - Username dell'utente che sta eseguendo il programma
* **$HOSTNAME** - Hostname dell'utente che sta eseguendo il programma
* **$RANDOM** - Numero casuale che viene generato ad ogni lettura
* **$LINENO** - Numero della linea corrente nel file
* **$SECONDS** - Numero di secondi trascorsi dall'avvio dello script


# Scrittura su STDOUT
Per scrivere su STDOUT si può usare il comando **echo**.

> Esempio
>```bash
>echo "Hello World"
>```
> Stampa su STDOUT la stringa "Hello World"

# Lettura da STDIN
Per leggere da STDIN si può utilizzare il comando **read**. 
Usando l'opzione `-p` si può specificare un messaggio (prompt) da mostrare.
> Esempio
>```bash
> read -p "Inserisci il tuo nome: " nome
> echo "Ciao $nome"
>```
> Quando questo script bash viene eseguito, la shell resta in attesa dell'input utente, per poi stampare "Ciao \<nome_inserito\>"

# Stringhe
Le stringhe possono essere scritte in due modi, in maniera "letterale" e con sostituzione di variabili.
* Usando `' '` il testo all'interno della stringa viene interpretato letteralmente.
* Usando `" "` all'interno del testo, vengono sostituite tutte le eventuali variabili presenti con il loro valore.

> Esempio
>```bash
> test=100
> echo 'Il valore è $test'
> echo "Il valore è $test"
>```
> Nel primo caso viene stampato "Il valore è $test".
> <br></br>
> Nel secondo caso invece: "Il valore è 100".

# Command substitution
La command substitution consente di eseguire un comando e salvare il suo output in una variabile.

Ha due sintassi diverse:
>```bash
>$(comando)
>```
> oppure
>```bash
>$( comando )
>```
> o ancora (anche se poco raccomandato)
>```bash
>`comando`
>```
> anche qui è possibile lasciare uno spazio tra i backticks, ma tale sintassi (insieme a l'ultima vista) non viene raccomandata in quanto la concatenazione di comandi risulta più complicata

Quando eseguiamo un comando attraverso una command substitution, 
la bash shell corrente esegue il comando in una subshell, per poi sostituire la command substitution con 
l'output della computazione (si dice anche: fare substitution del risultato). 

Di seguito mostriamo un:

> Esempio
>```bash
>files=$(ls -1)
>echo $files
>echo nesting $(echo $(ls -1)) 
>```
> La prima riga salva nella variabile `files` l'output di `ls -1` (usato nella current working directory).
>
> La seconda riga stampa semplicemente il contenuto della variabile. 
>
>Si noti che l'opzione `-1` consente di stampare in colonna le directory entries listate nella directory corrente, ma la command substitution rimuove tutti i newline presenti.
>
> Infine la terza riga stampa la stringa "nesting" unita all'output di quanto detto sopra.

# Arithmetic expansion
L'arithmetic expansion permette di eseguire operazioni aritmetiche su interi.
<br></br>

In particolare applicando un'artihmetic expansion a una espressione aritmetica, si ha prima la valutazione dell'espressione e poi la substitution del risultato.
<br></br>

Di seguito la notazione:
```bash
$(( operazione ))
```
È necessario che dopo `$(` e `)` non ci siano spazi, in quanto la sintassi `$( (...) )` è relativa al **command grouping**, che vedremo nel prossimo paragrafo.
<br></br>

Di seguito mostriamo un
> Esempio
>```bash
> dieci=10
> undici=$(( $dieci + 1 ))
> echo $undici
>```

# Command grouping
È una funzionalità che consente di raggruppare più comandi, con diverse possibilità per la loro esecuzione.

I comandi soggetti a command grouping possono essere eseguiti in diversi modi:
* nel processo corrente in modo sincrono o asincrono
* in un sottoprocesso in modo sincrono o asincrono

## **<span style="color:red">Command grouping sincrono</span>**

* Per poter eseguire una sequenza di comandi nel processo corrente, si sfrutta la seguente sintassi:

    >```bash
    >{ codice }
    > ```
    > oppure
    >```bash
    >{codice}
    > ```
    > oppure
    >```bash
    >{
    >    codice
    >    ...
    >}
    >```

* Mentre se si vogliono eseguire in un sottoprocesso, si sfrutta la seguente sintassi:

    >```bash
    >( codice )
    >```
    > oppure
    >```bash
    >(codice)
    >```
    > oppure
    >```bash
    >(
    >    codice
    >    ...
    >)
    >```

Si noti che l'esecuzione dei comandi sarà bloccante rispetto la shell in entrambi i casi. 

Nel secondo caso è meno evidente, ma la shell resta in attesa che la sequenza di comandi eseguita nel sottoprocesso venga computata prima di eseguire il resto del codice.

Consideriamo il seguente

> Esempio
>```bash
> (
>    while [ 1 ]
>    do
>       sleep 1
>       echo "Subshell in esecuzione"
>    done    
> ) #Forked process
>
>  echo "Adesso parlo io" #Parent process
>```
> Questo loop infinito non consentirà mai al processo padre di eseguire la sua stampa su STDOUT.

## **<span style="color:red">Command grouping asincrono</span>**
Una soluzione all'esempio mostrato poco fa consiste nello sfruttare il simbolo `&`, la cui semantica verrà approfondita in uno dei paragrafi successivi.

Consideriamo nuovamente il precedente esempio:

> Esempio
>```bash
> (
>    while [ 1 ]
>    do
>       sleep 1
>       echo "Subshell in esecuzione"
>    done    
> )& #The forked process is now executed in background
>
>  echo "Adesso parlo io" #Parent process
>```
> Inserendo il simbolo `&` alla fine della command grouping, si specifica che questi comandi vanno eseguiti in background (e quindi in un altro processo).


# Variabili locali ed export di variabili
Di default le variabili sono locali allo script corrente, per renderle disponibili a tutti (e soli) i processi figli è necessario esportarle.

```bash
export variabile=valore
```

È possibile anche esplicitare che una variabile è locale a un certo script.

```bash
local variabile=valore
```

Si presti particolare attenzione al rapporto tra variabili locali e variabili non locali (non esportate).

> Esempio
>```bash
> variabileTest="buonsalve"
>
> (
>   variabileTest="salve a lei" # sottintesto che sia local
>   echo $variabileTest # restituisce "salve a lei"
> )
>
> echo $variabileTest # restituisce "buonsalve"
>```
> La modifica alla variabile è locale e non impatta la variabile del processo padre.
>
> Se invece esportiamo la variabile:
>```bash
> export variabileTest="buonsalve"
>
> (
>   variabileTest="salve a lei"
>   echo $variabileTest # restituisce "salve a lei"
> )
>
> echo $variabileTest # restituisce "salve a lei"
>```
> Si ha il risultato atteso.

# let
Il comando let permette di eseguire operazioni aritmetiche su variabili (anche in questo caso su interi). 

Permette anche di fare incremento, etc. 

È possibile eseguire più operazioni consecutive (purché separate da spazi) ed è possibile specificarle all'interno delle `" "`.

La sintassi è
```bash
let <codice>
```

Vediamo ora un esempio.

> Esempio
>```bash
> testVar=2
> dieci=10
> let testVar++
> let "testVar = $testVar * 2; testVar = $testVar + 5 * 9" "testVar = $testVar + $dieci"
>```

# expr
Sostanzialmente si comporta come `let`, con la differenza che l'output viene mandato in **STDOUT** e non in una variabile.

Si usa sempre per fare aritmetica su interi.

Di seguito la sintassi:
```bash
expr <codice>
```
A differenza di `let` non c'è bisogno di utilizzare i simboli `" "` per poter scrivere spazi in una espressione.

Vediamo un esempio.
> Esempio
>```bash
> expr 5 + 4
> expr 5 \* 4 # è necessario l'escape del simbolo *
> expr 5+ 4 # sintassi non valida
>```

# Doppia parentesi
Permette di eseguire operazioni aritmetiche su interi (come quelle per let) senza valori di ritorno.

Segue la sintassi del comando:
```bash
(( codice ))
```
E un esempio.
>Esempio
>```bash
> testVar=2
> dieci=10
> (( testVar++ ))
> (( testVar = testVar + dieci ))
> echo $testVar
>```

# Array
Gli array sono una collezione di valori. Possono essere definiti in due modi:
* Usando `array=(valore1 valore2 ...)`
* Usando `array[n]=valore`

```bash
array=(1 2 3 4 5)
echo $array # stampa il primo elemento
echo ${array[0]} # stampa il primo elemento
echo ${array[4]} # stampa il quinto elemento
```

# Sequenze di comandi
I comandi possono essere eseguiti in sequenza e separati da caratteri speciali che ne cambiano il comportamento.

Si possono eseguire anche gruppi di comandi.
* **;** - Esegue i comandi in sequenza, non termina lo script se uno dei comandi fallisce
* **&&** - Esegue i comandi in sequenza, termina lo script se uno dei comandi fallisce
* **||** - Esegue il secondo comando solo se il primo fallisce
* **&** - Esegue il comando in background (in asincrono)
* **|** - Esegue comandi in sequenza, il risultato del primo comando viene passato al secondo
> Sintassi
>```bash
> comando1; comando2
> comando1 && comando2 && comando3
> comando1 || comando2 
> comando
> (
>     codice
>     ...
> ) ? 
> comando1 | comando2
>```

# Lunghezza di una variabile
Per ottenere la lunghezza del contenuto di una variabile si può usare il simbolo `#`.

Il contenuto della variabile verrà trattato come stringa e verrà contato il numero di caratteri.
> Esempio
>```bash
>a=1000
>echo ${#a} //4
>b="Hello World"
>echo ${#b} //11
>```



# test and exit status
Il comando test permette di valutare condizioni.

Ritorna 0 se la condizione è vera, 1 se è falsa.
```bash
test condizione
```
Un altro modo per eseguire il test è usare l'operatore `[` e `]`.
```bash
[ condizione ]
```

# Condizioni
Le condizioni sono espressioni che possono essere valutate come true o false.

Queste sono quelle standard di bash:

![Condizioni 1](/img/conditions_1.png)
![Condizioni 2](/img/conditions_2.png)
![Condizioni 3](/img/conditions_3.png)

Ma esistono anche altre condizioni più "moderne", per poter essere usate bisogna racchiuderle in `[[ condizione ]]` anzichè `[ condizione ]`
![Condizioni 3](/img/conditions_4.jpg)

# if
```bash
if condizione
then
    codice
elif condizione
then
    codice
else
    codice
fi
``` 

# case
```bash
case $variabile in
    valore1)
        codice
        ;;
    valore2)
        codice
        ;;
    *)
        codice default
        ;;
esac
```

# while
Ciclo che esegue il codice finchè la condizione è vera
```bash
while condizione
do
    codice
done
```
# until
Ciclo che esegue il codice finchè la condizione è falsa
```bash
until condizione
do
    codice
done
```
# for
Ciclo che itera una lista
```bash
for variabile in lista
do
    codice
done
```
Ex:
```bash
for file in ./$folder/*
do
    echo $file
done
```

# break [n] & continue [n]
* **break** interrompe il ciclo (più interno se non indicato)
* **continue** interrompe l'iterazione corrente (se non indicata) e passa alla successiva

Si può esplicitare un valore numerico per indicare quale iterazione interrompere o saltare.

```bash
break <numero>
continue <numero>
```

# select
Ciclo che itera una lista all'infinito e permette di scegliere un valore. La variabile indicata sarà quella dove verrà salvato il valore scelto.
```bash
select variabile in lista
do
    codice
done
```
Ex:
```bash
select name in "John" "Paul" "George" "Ringo"
do
    echo "Hello $name"
done
```

# Funzioni
È possibile definire delle funzioni in bash, ma non possono ritornare valori.

È possibile ritornare solo un exit status attraverso il costrutto `return`.
```bash
function nome_funzione {
    <codice>
    return <status_code>
}
```
Non potendo ritornare valori, possiamo usare una variabile globale oppure simulare i moderni linguaggi di programmazione, facendo stampare il risultato della funzione per poi catturarlo tramite command substitution `$()`.

I parametri della funzione sono salvati in `$1`, `$2`, `$3` (se eseguito tramite `$()`)

>Esempio
>```bash
> function somma {
>     echo $(expr $1 + $2)
> }
> result=$(somma 5 4)
> echo $result
>```
Le variabili possono essere dichiarate locali o globali, come abbiamo già visto.

Se dichiarate locali non possono essere usate all'esterno della funzione.


```bash
function somma {
    local a=5
    echo $(expr $a + $2)
}
```
# eval
Permette di eseguire comandi specificati attraverso una stringa.
```bash
eval <codice>
```

Di seguito un esempio.
> Esempio
>```bash
> a="cat file.txt"
> eval $a
>```
> Viene stampato il contenuto del file.

# declare
Permette di dichiarare variabili locali o globali, di impostarne il tipo e altri parametri
* **-a** array
* **-i** intero
* **-r** read only
Il contenuto vero e proprio delle variabili rimane sempre una stringa, ma alcuni comandi, sapendo il tipo, possono eseguire operazioni diverse.

```bash
declare [parametro] <nome_variabile>
```

Di seguito un esempio.

> Esempio
>```bash
> declare -a array
> declare -i intero
> declare -r read_only
> array[0]=1
> intero=5/2 //senza bisogno di expr o let!
>```



# Esempi

Prende tutti i file nella directory che iniziano con "a" o "b" e finiscono con "?", conta all'interno del file quante volte è presente la parola "test", se è maggiore di 2, stampa quanti ne ha trovati, sennò "too few tests"
```bash
for file in ./[ab]*\?.txt; do
        count=$(egrep -c test $file)
        if [ $count -gt 2 ]
        then
                echo "$file: $count"
        else
                echo "$file: too few tests"
        fi
done
```

Prende il primo parametro passato al comando e calcola il numero di Fibonacci su di esso
```bash
function myfunction {
    if [ $1 -gt 1 ]; then
        ((a=$1-1))
        ((b=$1-2))
        temp1=$(myfunction $a)
        temp2=$(myfunction $b)
        ((res=temp1+temp2))
    else
        if [ $1 -eq 1 ]; then
            res=1
        else
            res=0
        fi
    fi
    echo $res
}
res=$(myfunction $1)
echo $res
```
