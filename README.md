# Shell
I sistemi UNIX mettono a disposizione differenti tipi di Shell, elencati di seguito:
* sh
* bash
* csh
* tcsh
* ksh
* zsh

Per visualizzare il tipo di shell che stiamo utilizzando possiamo eseguire uno dei seguenti comandi nel Terminale: 

```bash
echo $0

echo $SHELL

which $SHELL
```

Per visualizzare le shell disponibili possiamo scrivere il seguente comando: 

```bash
cat /etc/shells
```
Il comando `cat` non è l'unico che consente di visualizzare file, in seguito verranno introdotti altri comandi (come ad esempio `nano`) che oltre a consentire la visualizzazione dei file, ne consentono anche la modifica.

Per cambiare shell basta scrivere il nome della shell che si vuole utilizzare
>Ad esempio: 

```bash
zsh
```

# UNIX file system


Il file system di UNIX è una struttura gerarchica formata da files.

Il file principale **root** è una directory, ossia un file che contiene altri files. 

La directory ***root*** in UNIX viene rappresentata con il simbolo `/`, segue un esempio per chiarire le idee riguardo le directory in generale:


>Analizziamo il Path ```/home/utente```. Ci troviamo nella directory `utente` che è una sottocartella di `home` che a sua volta è una sottocartella di `/`


![](img/fileSystem.jpg)

Per navigare tra le directory si utilizza il comando:
```bash
cd <directory name>
```
Per tornare alla directory precedente si utilizza il comando:
```bash
cd ..
```
Per tornare alla directory principale si utilizza il comando:
```bash
cd ~
```

# Working directory

Esistono due tipologie di directory:
* **Absolute path**: indica la posizione di un file rispetto alla directory principale `/`
* **Relative path**: indica la posizione di un file rispetto alla directory in cui ci troviamo

![](img/workingDirectory.png)

Per cambiare la directory di lavoro si usa la funzione `chdir`, che sarebbe una system call

# Filesystem block
La grandezza dei blocchi rappresenta la quantità di dati che il filesystem usa per leggere e scrivere data

# Comandi

***Per agevolare la scrittura dei comandi, il tasto Tab spesso può essere utilizzato per autocompletare il comando o il nome del file.***
## **man**
Restituisce la documentazione per un determinato comando, si utilizza:
```bash
man [options]
```

Per uscire dalla documentazione si utilizza il tasto `q`, mentre per scorrere la documentazione si possono utilizzare le frecce direzionali.

>**Esempio**:  `man ls`

Di seguito un altro uso interessante di `man`, che permette di visualizzare tutti i comandi disponibili tremite una specifica keyword:
```bash
man -k <keyword>
```

## **ps**
Visualizza i processi in esecuzione con vari gradi di dettaglio a seconda dei parametri specificati nel coman

```bash
ps [options]
```

## **ls**
Mostra il Path relativo alla directory in considerazione

```bash
ls [options] [location]
```

>Esempio: `ls -l`

Questo comando restituisce informazioni dettagliate sulle directory, tra cui data di creazione, l'utente che l'ha creata, se è eseguibile, scrivibile e leggibile.

In seguito vediamo la struttura di output del comando `ls -l`:

![](img/lsLongFormat.png)


 col. 1     | col. 2    | col. 3   |col. 4   |col. 5 | col. 6 | col. 7 | col. 8 | col. 9
:----------:|:---------:| :-------:|:-------:| -----:|-------:|-------:|-------:|--------:
file premission       |    numero di hard links del file o il numero di directory entries contenute  | owner name       |  owner group       |  file size     |     Mese di modifica   |  giorno di modifica      |   anno/ora di modifica |     nome del file o directory o link 

Prima della struttura a tabella è presente una riga che inizia con `total` che indica la dimensione totale dei file contenuti nella directory, visualizzata in numero di blocchi ( 512 bytes per blocco) o in bytes.

All'interno della colonna 1, potrebbero esserci 2 attributi aggiuntivi:
* **`+`**: indica che il file è un file eseguibile
* **`@`**: indica che il file è un link simbolico


**File permissions:**
* r = leggibile
* w = scrivibile
* x = eseguibile

## **nano**
Editor di testo di UNIX, permette la modifica di file di testo da CLI

```bash 
nano [file_name]
```

Una volta eseguito il comando da Terminale, verrà aperto l'editor di testo sul file che si vuole modificare.

Per uscire dall'editor di testo basta premere la combinazione di tasti: **CTRL + X**, se sono state effettuate delle modifiche vi verrà chiesto se volete sovrascrivere il file. 

## **id**
Visualizza l'ID dell'utente corrente
    
```bash
id [option]
```

Differenti opzioni:
* **[login name or user ID]**: visualizza l'ID dell'utente e del gruppo dell'utente specificato
* **-u**: visualizza l'ID dell'utente corrente
* **-un**: visualizza il nome dell'utente corrente
* **-g**: visualizza l'ID del gruppo corrente
* **-gn**: visualizza il nome del gruppo corrente
* **-G**: visualizza tutti gli ID dei gruppi a cui appartiene l'utente corrente
* **-Gn**: visualizza tutti i nomi di tutti i gruppi

>Esempio per la prima opzione: `id <user name>`
![](img/idCommand.png)
# Parametri

I parametri qui sotto elencati sono quelli più utilizzati, per una lista completa si può consultare la documentazione con il comando [man](#man)

* `-l`: mostra una versione dettagliata del comando
* `-r`: esegue il comando in maniera ricorsiva

# Vocabolario

* **Hard Link**: un hard link è un file che punta ad un altro file, in pratica è un alias del file originale. Se si elimina il file originale, il hard link rimane invariato.

* **Path**: una stringa che identifica la posizione di una specifica directory o file.

* **Directory entry**: un file che contiene informazioni su un file o una directory, come ad esempio il nome del file, la data di creazione, la dimensione, ecc.

* **Soft links**: un soft link è un file che contiene il path del file originale, in pratica è un alias del file originale. Se si elimina il file originale, il soft link diventa un file rotto.