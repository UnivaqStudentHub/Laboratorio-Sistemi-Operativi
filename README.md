# INDICE

* [Shell](#shell)
* [Da dove prende i comandi il terminale?](#da-dove-prende-i-comandi-il-terminale)
* [UNIX file system](#unix-file-system)
* [Working directory](#working-directory)
* [Filesystem block](#filesystem-block)
* [Wildcards](/Wildcards.md)
* [Espressioni regolari](/Espressioni_regolari.md)
* [Comandi Shell](/comandi_shell.md)
* [Comandi per scrittura su file](/Comandi_files.md)
* [Filters, Redirection e Pipelines](/Filters_Redirections_Pipelines.md)
* [Shell in generale e bash](/Shell.md)
* [Parametri](#parametri)
* [Vocabolario](#vocabolario)
* [Riassunto comandi](/Comandi.md)

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

Per visualizzare le shell disponibili nel sistema che stiamo utilizzando, possiamo utilizzare il seguente comando: 

```bash
cat /etc/shells
```
Il comando `cat` non è l'unico che consente di visualizzare file, in seguito verranno introdotti altri comandi (come ad esempio `nano`) che oltre a consentire la visualizzazione dei file, ne consentono anche la modifica.

Per cambiare shell basta scrivere il nome della shell che si vuole utilizzare
>Ad esempio: 
>```bash
>zsh
>```
><br>

<br>

# Da dove prende i comandi il terminale?
Quando scriviamo un comando nel terminale, il sistema operativo cerca l'implementazione del comando all'interno di un set predefinito di *directories*, specificate all'interno della variabile `$PATH`.

# UNIX file system


Il file system di UNIX è una struttura gerarchica formata da files.

Il file principale **root** è una directory, ossia un file che contiene altri files. 

La directory ***root*** in UNIX viene rappresentata con il simbolo `/`, segue un esempio per chiarire le idee riguardo le directory in generale:


>Analizziamo il Path ```/home/utente```. Ci troviamo nella directory `utente` che è una sottocartella di `home` che a sua volta è una sottocartella di `/`
>![](img/fileSystem.jpg)

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

Per cambiare la directory di lavoro si utilizza una system-call, ossia la funzione `chdir`.

Le ***directories*** sono dei files che contengono `directory entries`, ossia dei files che contengono altri files; la dimensione che viene visualizzata è quella usata per immagazzinare le *meta information* (directory entries) per quella directory.<br>
Per visualizzare la dimensione su **disco** usare il comando [`du`](#du)

# Filesystem block
* `Che cos'è un blocco?` È una sequenza di bytes / bit

* `La dimensione di un blocco è l'unità di lavoro del filesystem`; la sua importanza è rilevante dato che viene impiegata dal filesystem per le operazioni di lettura/scrittura dei dati.

* La dimensione minima che un blocco può avere su un disco fisso è di 16 bytes. 

* Aumentare la dimensione di un blocco consente di migliorare le prestazioni di Input/Output sui file di grandi dimensioni, in quanto le operazioni di lettura/scrittura vengono concentrate per un periodo di tempo maggiore su un singolo blocco prima di spostarsi al successivo!

* **Problema**: i file di piccole dimensioni come riportato di seguito, occupano più spazio di quanto previsto, questo perché i blocchi che li compongono sono più grandi!

  Di seguito una immagine esplicativa! Il file dovrebbe pesare 1KB ma sul disco pesa 4KB!




  ![](img/block.png)


# Parametri

I parametri qui sotto elencati sono quelli più utilizzati, per una lista completa si può consultare la documentazione con il comando [man](#man)

* `-l`: mostra una versione dettagliata del comando
* `-r`: esegue il comando in maniera ricorsiva

# Vocabolario

* **Soft links**: Chiamati anche link simbolici, sono dei file che puntano al file vero e proprio e conservano dentro di loro il nome del file a cui puntano ( in Windows lo puoi pensare come un file "Collegamento")


* **Hard Link**: è l'equivalente del file che si trova nell'HDD. Sono reference o puntatori verso punti nell'hdd. <br> La caratteristica principale dell'`Hard Link` che lo differisce dal `Soft Link` è il fatto che tramite la cancellazione del file originale non andremo a intaccare l'`Hard Link`, mentre invece il `Soft Link` diventerebbe inutilizzabile.





* **Path**: una stringa che identifica la posizione di una specifica directory o file.

* **Directory entry**: un file che contiene informazioni su un file o una directory, come ad esempio il nome del file, la data di creazione, la dimensione, ecc.

[def]: img/ls-s.png
[def2]: £
