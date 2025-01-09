# Introduzione
Nella prima parte di questa sezione ci focalizzeremo nello studio della struttura che c'è dietro i files, per poi spostarci verso i tipi di files e infine su alcune funzioni I/O unbuffered per la loro gestione.

Durante la seconda parte tratteremo invece files e directories, introducendo le strutture che i sistemi UNIX-like sfruttano per poter gestire la condivisione di files tra processi, aprendo una piccola parentesi su hard links e soft links, per poi concludere la seconda parte trattando più in dettaglio i file descriptors e la loro gestione.

Infine tratteremo lo Standard I/O (buffered I/O).

<br></br>

# Note importanti
Molti degli esempi proposti in questa sezione fanno uso di utilities e librerie che potete trovare direttamente in una cartella del team del docente del corso.

Per comodità non riportiamo tutti i dettagli relativi il funzionamento di questa grande libreria, potete trovare tutto sulle slides del docente del corso.

Tutti i meriti per la realizzazione di questo materiale vanno al docente e ai creatori del libro.

<br></br>

# File information structure
Lo standard POSIX ha definito nell'header file `sys/stat.h` una funzione `stat()` che restituisce dettagli riguardanti il file specificato in input.

Il prototipo della funzione è il seguente:
>```C
>int stat (const char *restrict pathname, struct stat *restrict buf);
>```
> La funzione prende in input un **pathname** e restituisce informazioni riguardo il file specificato all'interno di **buf**.
>
> La parola chiave `restrict` viene utilizzata per indicare al compilatore quali reference a puntatore possono essere ottimizzate, indicando quindi che quel puntatore è l'unico che verrà utilizzato per accedere al dato puntato.

Il secondo parametro del prototipo della funzione ha come tipo un puntatore a una struttura denominata `stat`, anch'essa introdotta da POSIX e definita in questo modo:

>```C
> struct stat {
>   dev_t       st_dev;     /* ID del dispositivo contenente il file */ 
>   ino_t       st_ino;     /* numero dell'i-node */ 
>   mode_t      st_mode;    /* codifica il tipo del file e i suoi permessi */ 
>   nlink_t     st_nlink;   /* numero di hard links */ 
>   uid_t       st_uid;     /* user ID del proprietario */ 
>   gid_t       st_gid;     /* group ID del proprietario */ 
>   dev_t       st_rdev;    /* ID del dispositivo (se il file è speciale) */ 
>   off_t       st_size;    /* dimensione totale, in byte */ 
>   blksize_t   st_blksize; /* dimensione dei blocchi di I/O del filesystem */ 
>   blkcnt_t    st_blocks;  /* numero di blocchi assegnati */ 
>   time_t      st_atime;   /* tempo dell'ultimo accesso */ 
>   time_t      st_mtime;   /* tempo dell'ultima modifica */ 
>   time_t      st_ctime;   /* tempo dell'ultimo cambiamento */
> };
>```

Tratteremo in dettaglio alcuni di questi attributi in questa e nelle prossime sezioni.

<br></br>

## Introduzione al tipo *mode_t*
Come già specificato nello scorso paragrafo, l'attributo di tipo **mode_t** codifica il tipo del file specificato e i suoi permessi.

### File types
Durante la prima parte del corso abbiamo introdotto soltanto due tipi di files:
- `d` per indicare le directories
- `-`  per indicare i files

Vediamoli nel dettaglio introducendone anche di nuovi.

#### Regular file (-)
Viene indicato con l'hyphen (trattino) un qualsiasi file binario o di testo.

> Osservazione:  il kernel non fa distinzione tra i due, a meno che il file non sia un binario eseguibile. In quel caso gli eseguibili sono conformi ad un formato standard, così da consentire al kernel di caricare il programma in memoria.

#### Directory file (d)
Viene indicato con la lettera **d** un qualsiasi file che contiene directory entries.

> Introdurremo rigorosamente il concetto di directory entry quando tratteremo le strutture dati che il kernel utilizza per gestire i files.
> Al momento vi basti ricordare che sono meta-informazioni riguardanti le directory a cui puntano. Vi rimandiamo a [questa](/Prima_parte/Introduzione_prima_parte.md#pathname-e-working-directory) sezione per un piccolo ripasso.

#### Block special [device] file (b)
Vengono indicati con la lettera **b** quei files che forniscono accesso I/O buffered in unità di dimensione fissa a dispositivi.

> Ad esempio: 
> - **/dev/disk0**
>
> Dove `disk0` è un file che rappresenta un dispositivo di archiviazione di massa.
>
> In pratica è un'astrazione di quel dispositivo di archiviazione a livello di sistema operativo.

#### Character special [device] file (c)
Vengono indicati con la lettera **c** quei files che forniscono accesso I/O unbuffered in unità di dimensione variabile a dispositivi.

> Ad esempio: 
> - **/dev/fd/0**
> - **/dev/fd/1**
> - **/dev/fd/2**
> - **/dev/tty**
>
> Dove 0,1 e 2 rappresentano rispettivamente STDIN, STDOUT, STDERR. Mentre tty rappresenta il terminale.

#### FIFO (p)
È un tipo di file utilizzato per IPC (interprocess communication), vengono anche dette **named pipes**.

> Le introdurremo nella prossima sezione.
#### Socket (s)
È un tipo di file utilizzato per la comunicazione tra processi attraverso la rete.

#### Symbolic link (l)
È un tipo di file che punta ad un altro file.

> Tra qualche paragrafo vedremo la differenza tra hard e soft (symbolic) links.

### Permessi di accesso ai files
Per ciò che riguarda i permessi dei files, approfondiremo più in dettaglio nella prossima sezione questa questione.
Per un piccolo ripasso sui permessi dei files, potete andare [qui](/Prima_parte/Comandi_bash.md#ls).

Al momento forniamo soltanto dei flags che sfrutteremo più avanti per la creazione di files:


| st_mode mask                  | Significato                                |
|-------------------------------|--------------------------------------------|
| S_IRUSR<br>S_IWUSR<br>S_IXUSR | user-read<br>user-write<br>user-execute    |
| S_IRGRP<br>S_IWGRP<br>S_IXGRP | group-read<br>group-write<br>group-execute |
| S_IROTH<br>S_IWOTH<br>S_IXOTH | other-read<br>other-write<br>other-execute |


### Bit mask per *mode_t*
Dato che l'attributo **st_mode** della struttura **stat** codifica due informazioni diverse, attraverso l'utilizzo della bitmask `S_IFMT` è possibile recuperare le due informazioni separatamente.

Se vogliamo recuperare il tipo del file, basta mettere in **bitwise AND** l'attributo **st_mode** e la bit mask, come mostrato di seguito:
```mystat.st_mode & S_IFMT```.

Mentre se si vogliono recuperare i permessi di quel file, basta negare i bit della bitmask: ```mystat.st_mode & ~S_IFMT```.

### Macros per determinare il tipo di un file
Un metodo alternativo e più veloce per poter determinare il tipo di un file prevede l'utilizzo di diverse macros: ognuna di queste prende in input l'attributo **st_mode** e restituisce un valore booleano.

Queste macros vengono implementate proprio attraverso la bit mask vista nello scorso paragrafo.

Di seguito la lista delle macros contenute nell'header file `sys/stat.h`:

| Macro        | Tipo del file           |
|--------------|-------------------------|
| S_ISREG()    | regular file            |
| S_ISDIR()    | directory file          |
| S_ISCHR()    | character special file  |
| S_ISBLK()    | block special file      |
| S_ISFIFO()   | pipe or FIFO            |
| S_ISLNK()    | symbolic link           |
| S_ISSOCK()   | socket                  |

Lo standard POSIX consente anche di rappresentare code di messaggi e semafori come files. Le loro macros differiscono leggermente rispetto a quelle viste precedentemente: prendono in input direttamente un puntatore alla struttura **stat**.

Le riportiamo di seguito:

| Macro        | Tipo del file           |
|--------------|-------------------------|
| S_TYPEISMQ()    | message queue           |
| S_TYPEISSEM()    | semaphore          |
| S_TYPEISSHM()    | shared memory object  |

<br></br>

# Funzioni I/O unbuffered
Le funzioni che stiamo per introdurre fanno uso dei file descriptors per effettuare operazioni di I/O su file, prima di proseguire vi consigliamo di ripassare questo concetto in [questa](/Prima_parte/Filters_Redirections_Pipelines.md#file-descriptors) sezione.

## creat
Seppure questa funzione sia ormai stata totalmente sostituita dalla funzione `open` (che introdurremo nel prossimo paragrafo), la introduciamo in modo tale da trattare progressivamente queste funzioni.

La funzione consente di creare un nuovo file restituendo:
- il file descriptor aperto in sola scrittura se tutto è andato a buon fine;
- -1 in caso di errore.

Il prototipo della funzione è il seguente:
>```C
> #include <fcntl.h> // dove fcntl sta per "file control"
>
> int creat(const char *path, mode_t mode);
>```
> Come potete notare, la funzione prende in input un path e un parametro di tipo *mode_t*, quindi consente di specificare sia il nome del file da creare sia i suoi permessi / il suo tipo.

## open
Questa funzione consente di aprire un file, restituendo:
- il file descriptor del file, che è il più piccolo inutilizzato nel sistema;
- -1 in caso di errore.

Il suo prototipo è il seguente:
```C
 #include <fcntl.h> // dove fcntl sta per "file control"

 int open(const char *path, int oflag, ..., /* mode_t mode */);
```

dove:
- `path` rappresenta il percorso del file da aprire;
- `oflag` è un insieme di flags definite nell'header file `fcntl.h`, combinate tra loro sfruttando l'operatore OR-bitwise `|`. (si dice che anche le flags sono messe in **ORing** tra loro).

Non tutti i flags possono essere messi in **ORing** tra loro, di fatto è possibile specificare solo uno dei seguenti nel parametro oflag:

| Flag        | Cosa fa        |
|--------------|-------------------------|
| O_RDONLY    | apre il file in sola lettura   |
| O_WRONLY    | apre il file in sola scrittura    |
| O_RDWR    | apre il file in lettura e scrittura  |
| O_EXEC    | apre il file per la sola esecuzione  |
| O_SEARCH    | apre il file per la sola ricerca (si applica alle directories) |

Per semplicità abbiamo parlato di "apertura di file", ma in realtà il kernel associa al processo chiamante un file descriptor aperto nelle modalità specificate nella tabella di sopra.

### Creare files con *open*
Inizialmente non era possibile creare files con questa funzione, per questo venne realizzata la funzione `creat`.

In seguito la funzione `open` venne aggiornata e **creat** venne dichiarata **deprecated**.

Per poter creare dei files con *open*, bisogna sfruttare l'oflag `O_CREAT`, che crea il file se non è già esistente. Per sfruttare questo oflag però, bisogna specificare l'ultimo parametro del prototipo della funzione, che normalmente è facoltativo:

- `mode` specifica il tipo del file ed i suoi permessi.

> Quindi la funzione:
>```C
> open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);
>```
> è del tutto equivalente a:
>```C
> creat(path, mode);
>```

dove `O_TRUNC` indica che se il file esiste ed è stato aperto con successo per lettura / scrittura, tronca la sua lunghezza a 0 (ossia la dimensione del file viene ridotta a 0 bytes cancellando ciò che conteneva in precedenza).

Consideriamo il seguente
> Esempio
>```C
> open("testfile.txt", O_WRONLY | O_CREAT | O_EXCL, S_IRUSER | S_IWUSER | S_IRGRP | S_IROTH);
>```
> In questo caso invece di specificare l'opzione `O_TRUNC`, abbiamo sfruttato `O_EXCL` che genera un errore se il file specificato è già esistente.
>
> Nel caso in cui avessimo lasciato l'altra opzione, avremmo potuto sovrascrivere files, mentre in questo caso non è possibile.
>
> I flags in ORing specificati nel parametro mode, indicano la concessione dei permessi di:
> - lettura e scrittura agli utenti;
> - lettura ai gruppi;
> - lettura agli others.
> 
> Che avevamo già visto in precedenza [qui](#permessi-di-accesso-ai-files).

## openat
La funzione è quasi del tutto analoga a **open**, infatti restituisce:
- il file descriptor del file, che è il più piccolo inutilizzato nel sistema;
- -1 in caso di errore.

L'unica cosa che distingue le due funzioni è il parametro `fd`.


La sintassi è la seguente:
```C
#include <fcntl.h>

int openat(int fd, const char *path, int oflag, ..., /*mode_t mode*/);
```

Ci sono tre possibilità:
- Se `path` specifica un **pathname assoluto**, allora il parametro `fd` viene ignorato: la funzione si comporta esattamente come **open**;
- `path` specifica un **pathname relativo** ed `fd` è un file descriptor che specifica il punto di partenza (directory) da cui valutare il path. Il parametro `fd` può essere ottenuto effettuando una **open** sulla directory da cui verrà valutato il path relativo.
- `path` specifica un **pathname relativo** e `fd` ha il valore speciale `AT_FDCWD`. In questo caso il pathname viene valutato partendo dalla Current Working Directory e la funzione si comporta esattamente come **open**.

Consideriamo ora qualche esempio:

> Supponiamo di voler effettuare operazioni su un file di testo di nome `textfile.txt` contenuto all'interno della directory `documents`.
>
>Prendiamo prima il file descriptor della directory `documents`, aprendola con `open`:
>```C
>int directoryFD = open("/documents/", O_RDONLY); 
>```
>Adesso prendiamo il file descriptor del file di testo, aprendo anch'esso:
>```C
>int textFileFD = openat(directoryFD, "textfile.txt", O_RDWR); 
>```
> Siamo rientrati nel secondo caso: `directoryFD` contiene il file descriptor della directory da cui valutare il path relativo `textfile.txt`.
>
> Vedremo nei prossimi paragrafi delle funzioni che ci consentiranno di effettuare operazioni di lettura / scrittura su file (unbuffered).


## read
Attraverso questa funzione è possibile leggere i dati di un file aperto. 

La funzione tenta di leggere `nbytes` dal file associato al file descriptor `fd`, memorizzandoli nel buffer puntato da `*buf`e restituisce:
- il **numero di bytes letti** se la lettura è andata a buon fine;
- **0** se viene letto un End of File;
- **-1** in caso di errore.

Il prototipo della funzione nella sua definizione classica è il seguente:
```C
int read(int fd, char *buf, unsigned nbytes);
```

Un'altra definizione di questa funzione passa per l'header file `unistd.h`:
```C
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t nbytes);
```

## write
Attraverso questa funzione è possibile scrivere dati su un file aperto.

La funzione tenta di scrivere `nbytes` dal buffer puntato da `*buf` nel file associato al file descriptor `fd` e restituisce:
- il **numero di bytes scritti** se la scrittura è andata a buon fine;
- **-1** in caso di errore (molto spesso causati da spazio insufficiente su disco).

Il prototipo della funzione è il seguente:
```C
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t nbytes);
```

Di solito il numero di bytes ritornati dalla funzione coincide proprio con il parametro *nbytes*.

## Esempio di I/O unbuffered
Per poter seguire meglio questo esempio, vi consigliamo come indicato [qui](#note-importanti), di scaricare i files contenuti nella cartella src del team del corso.

In questo caso tratteremo i contenuti della cartella `chap1-intro`.

Mettiamo insieme le due funzioni viste poco fa nel seguente esempio:

> **Lettura da STDIN e scrittura su STDOUT** (mycat.c)
> 
>```C
>#include "apue.h"
>
>#define BUFFER_SIZE 4096
>
>// il metodo main è stato scritto secondo l'ISO C Standard
>int 
>main(void)
>{
>   int n;
>   char buffer[BUFFER_SIZE];
>
>   while ((n = read(STDIN_FILENO, buffer, BUFFER_SIZE)) > 0) {
>       if (write(STDOUT_FILENO, buffer, n) != n) err_sys("write_error"); 
>   }
>
>   if (n < 0) err_sys("read error");
>
>   exit(0); // Successfull program termination
>} 
>```
> `apue.h` è un header file distribuito dal libro di testo adottato dal corso e contiene diverse dichiarazioni di macros e funzioni, oltre l'inclusione di diverse librerie, tra cui `unistd.h` e `fcntl.h`.
>
> Infatti le macros `STDIN_FILENO`, `STDOUT_FILENO` e `STDERR_FILENO` sono contenute in `unistd.h` e riferiscono direttamente i files `0`, `1` e `2` contenuti in `/dev/fd`.
>
> Notate che siccome le funzioni sono unbuffered, per effettuare operazioni di lettura o scrittura dobbiamo specificare noi un buffer di una certa dimensione. In questo caso abbiamo specificato un buffer di 4096 bytes.
>
> Il programma legge da STDIN fino a quando non incontra un End of File, in quel caso `read` ritornerà 0 e terminerà l'esecuzione. Altrimenti stampa il contenuto del buffer su STDOUT.
>
> In questo esempio avrete notato che non abbiamo aperto STDIN e STDOUT manualmente, infatti supponiamo che siano già stati aperti dalla shell.
>
> Non c'è bisogno di forzare la chiusura dei due file descriptors, in quanto verranno chiusi manualmente quando il processo terminerà.
>
> Eseguiamo quindi questo programma dalla shell:
> ![](/img/es_unbuffered_io_1.png)
> Ovviamente per terminare il programma utilizziamo il più comune segnale di interruzione CTRL+C, in quanto non abbiamo modo di scrivere esplicitamente un End of File.
>
> Proviamo invece a far leggere il seguente file di testo al programma
>```
> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod temporincidunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrum exercitationem ullamco laboriosam, nisi ut aliquid ex ea commodi consequatur. Duis aute irure reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint obcaecat cupiditat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
>```
> E vediamo cosa accade sul terminale:
> ![](/img/es_unbuffered_io_2.png)
> Come vedete in questo caso l'esecuzione è terminata perché alla fine del file di testo la `read` ha incontrato un End of File.
>
> Ovviamente è possibile redirezionare lo STDOUT verso un file.
>
> Per concludere vi consigliamo di provare a modificare la `BUFFER_SIZE` del file sorgente e a ricompilare attraverso il makefile. Se provate con buffer molto piccoli riuscirete a vedere le stringhe comparire più lentamente su terminale, proprio perché il buffer riesce a contenere meno bytes.

## Current file offset
Ogni file aperto ha un `current file offset` associato ad esso, ossia un intero non-negativo che rappresenta la posizione corrente all'interno di un file, misurata in byte dall'inizio del file.

Indica quindi quanti byte sono stati saltati dall'inizio del file per arrivare alla posizione attuale.

Di default quando un file viene aperto, il suo offset viene inizializzato a 0 (a meno che non si espliciti l'oflag `O_APPEND`).

Le funzioni `read` e `write` effettuano le loro operazioni a partire dal byte successivo il current file offset, aggiornandolo durante la lettura / scrittura del file.

In altri termini possiamo dire che il current file offset rappresenta il puntatore all'interno di un file.

### lseek
Questa system call consente di cambiare il current file offset, permettendo quindi di leggere / scrivere su file da punti diversi dello stesso.

Dato un file associato al file descriptor `fd`, la funzione sposta il current file offset di `offset` bytes da `whence` e restituisce:
- il **nuovo file offset** se l'operazione è andata a buon fine;
- **-1** in caso di errore.

La sintassi è la seguente:
```C
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```
dove il parametro `offset` viene interpretato diversamente a seconda del valore di `whence`:
- se `whence` ha il valore `SEEK_SET`, il current file offset del file viene impostato al valore di `offset` bytes dall'inizio del file;
- se `whence` ha il valore `SEEK_CUR`, il current file offset del file viene impostato a `current_file_offset + offset` (in questo caso *offset* può essere sia positivo sia negativo);
- se `whence` ha il valore `SEEK_END`, il current file offset del file viene impostato a `file_size + offset` (in questo caso *offset* può essere sia positivo sia negativo).

Quindi si ha modo di spostare il current file offset a partire da tutte le posizioni del file.

> Notate che effettuando un seek di 0 bytes, è possibile ottenere il current file offset:
>```C
>off_t current_position;
>current_position = lseek(fd, 0, SEEK_CUR);
>```

## Esempio riassuntivo
Presentiamo passo per passo un esempio riassuntivo che sfrutta le funzioni `open`, `read` ed `lseek`. Questo programma leggerà un file di testo presente nella current working directory ed effettuerà delle stampe spostando di volta in volta il current file offset nel file.

Il contenuto del file di testo è il seguente:
```
This is a test file that will be used 
to demonstrate the use of lseek, open, and read.
```

> Consideriamo il seguente sorgente C:
>```C
>#include "apue.h"
>#include <fcntl.h>
>
>#define BUFFER_SIZE 4096
>
>int 
>main()
>{
>	int fd = 0;
>	char buffer[BUFFER_SIZE];
>
>	fd = open("testfile.txt", O_RDONLY);
>
>	if (fd < 0) {
>		printf("\n Something went wrong, here's the fd returned %d \n", fd);
>		return 1;
>	}
>
>	if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>	printf("%s\n", buffer);
>	memset(buffer, 0, sizeof(buffer));
>
>	if (lseek(fd, 3, SEEK_SET) < 0) return 1;
>	if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>	printf("%s\n", buffer);
>	memset(buffer, 0, sizeof(buffer));
>
>	if (lseek(fd, -20, SEEK_END) < 0) return 1;
>	if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>	printf("%s\n", buffer);
>
>	if (lseek(fd, -30, SEEK_CUR) < 0) return 1;
>	int currentOffset = lseek(fd, 0, SEEK_CUR);
>	printf("%d\n", currentOffset);
>
>	return 0;
>}
>```
> Oltre all'inclusione delle due librerie, dichiariamo la buffer size del buffer che sfrutteremo per le operazioni di lettura. In questo caso adottiamo un buffer di 4096 bytes.
>```C
>#include "apue.h"
>#include <fcntl.h>
>
>#define BUFFER_SIZE 4096
>```
>
> Nel seguente snippet invece, dichiariamo le variabili che andremo ad utilizzare:
> - un intero che conterrà il file descriptor del file che apriremo con `open`;
> - un array di caratteri che sfrutteremo come buffer.
>```C
>int fd = 0;
>char buffer[BUFFER_SIZE];
>```
> Ora che è tutto pronto, possiamo cominciare aprendo il file di testo `testfile.txt`:
>```C
>fd = open("testfile.txt", O_RDONLY);
>
>if (fd < 0) {
>	printf("\n Something went wrong, here's the fd returned %d \n", fd);
>	return 1;
>}
>```
> Ci basta aprirlo in sola lettura, ma se avessimo voluto effettuare delle operazioni di scrittura su di esso sarebbe bastato esplicitare `O_RDWR`, magari in ORing con `O_APPEND`.
>
> Passiamo finalmente al succo del discorso:
>```C
>if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>printf("%s\n", buffer);
>memset(buffer, 0, sizeof(buffer));
>```
> Per prima cosa effettuiamo una read sul file descriptor associato a `testfile.txt`, ciò che verrà letto verrà memorizzato all'interno di `buffer`. Dato che la funzione read restituisce il numero di bytes letti, se restituisce un numero negativo si è verificato un errore.
>
> Procediamo quindi a stampare il contenuto del buffer e a resettare il buffer.
>
> L'output di questa computazione sarà:
>```
>This is a test file that will be used 
>to demonstrate the use of lseek, open, and read.
>```
>
> Bene, ora proviamo a modificare il current file offset:
>```C
>if (lseek(fd, 3, SEEK_SET) < 0) return 1;
>if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>printf("%s\n", buffer);
>memset(buffer, 0, sizeof(buffer));
>```
> In questo caso abbiamo impostato il current file offset al valore 3, quindi il puntatore nel file si trova al terzo byte dall'inizio del file. La funzione lseek ritorna il current file offset, quindi se dovesse ritornare un valore negativo, si verificherebbe un errore.
>
> L'output di questa computazione sarà:
>```
>s is a test file that will be used 
>to demonstrate the use of lseek, open, and read.
>```
>
> Consideriamo un altro caso:
>```C
>if (lseek(fd, -20, SEEK_END) < 0) return 1;
>if (read(fd, buffer, BUFFER_SIZE) < 0) return 1;
>printf("%s\n", buffer);
>```
> Notate come in questo caso abbiamo esplicitato un valore negativo per l'offset. Infatti attraverso il parametro SEEK_END, l'offset viene determinato sommando alla file size l'offset. In altri termini: il puntatore nel file viene spostato all'End of File, ciò implica che spostando il puntatore di -20 bytes, verranno letti gli ultimi 20 bytes del file.
>
> L'output di questa computazione sarà:
>```
> ek, open, and read.
>```
>
> Infine:
>```C
>if (lseek(fd, -30, SEEK_CUR) < 0) return 1;
>int currentOffset = lseek(fd, 0, SEEK_CUR);
>printf("%d\n", currentOffset);
>```
> Attenzione: ogni volta che effettuate una read, il puntatore viene spostato. In questo caso si trovava all'End of File, quindi spostandolo di 30 bytes indietro, si troverà al byte `58`, che è anche l'intero che verrà stampato.
>
> Vi consigliamo di esercitarvi autonomamente con la scrittura su file

## Altre funzioni per ottenere informazioni su files
### fstat
Questa funzione restituisce dettagli riguardanti files già aperti, il prototipo è il seguente:

>```C
>int fstat(int fd, struct stat *buf);
>```
> Anche in questo caso, le informazioni relative il file associato al file descriptor *fd* vengono memorizzate nel buffer puntato da **buf*.

### lstat
È l'equivalente di [stat](#file-information-structure) ma per i link simbolici.

Il prototipo è il seguente:
```C
int lstat(const char *restrict pathname, struct stat *buf);
```

### fstatat
Questa funzione consente di ottenere i dettagli di tutti i files contenuti in un path.

Il prototipo è il seguente:
```C
int fstatat(int fd, const char *restrict pathname, struct stat *buf, int flag);
```

Nel caso in cui il pathname è relativo e il file descriptor è il valore speciale `AT_FDCWD` allora il pathname viene valutato a partire dalla current working directory, altrimenti viene valutato a partire dal `pathname`.

Se il pathname è assoluto, il file descriptor viene ignorato.

Tra i vari flags ve ne è uno detto `AT_SYMLINK_NOFOLLOW` che impedisce alla funzione di dereferenziare i link simbolici, permettendo così alla funzione di restituire informazioni sul link stesso. Se questo parametro non fosse specificato, allora la funzione restituirebbe anche i dettagli del file puntato dal link.

<br></br>

# File sharing
Nei prossimi paragrafi introdurremo le strutture dati che il kernel adotta per poter rappresentare i files aperti,
in quanto sono proprio le relazioni tra queste strutture che determinano gli effetti che un processo ha su un altro in termini
di condivisione di files.

## Introduzione alle strutture dati kernel per i files
Il kernel sfrutta 3 strutture dati per rappresentare un file aperto:
- process table;
- file table;
- v-node table.

## Process table
Ogni processo nel sistema ha una entry in questa tabella. Le entries della process table contengono una **open file descriptors table**, con una entry per ogni file descriptor aperto per quel processo.

A ogni file descriptor viene associata una coppia del tipo `<fd flags, file pointer>` dove al momento nel primo parametro è possibile specificare soltanto il flag `FD_CLOEXEC` (close-on-exec flag):
- se questo bit viene impostato, allora il file descriptor verrà chiuso dopo una exec andata a buon fine, altrimenti resterà aperto;
- se il bit non viene impostato, allora il file descriptor rimarrà aperto anche in caso di exec andata a buon fine.

Per impostare questo bit si sfruttano i getter e setter `F_GETFD` e `F_SETFD` con la funzione `fcntl()`.

Il secondo parametro della coppia invece è un puntatore a una **file table entry**, che vedremo nel prossimo paragrafo.

> Schema riassuntivo
>
>![](/img/process_table.png)

## File table
Ogni file aperto ha una entry in questa tabella. Ogni **file table entry** contiene:
- i **file status flags** del file: come read, write, append, sync e non-blocking;
- il **current file offset** del file;
- un puntatore alla **v-node table entry** del file.

> Schema riassuntivo
>
> ![](/img/process_table_file_table.png)

## V-node table
Ogni file o dispositivo aperto ha una entry in questa tabella, che contiene informazioni riguardanti il tipo del file e puntatori a funzioni che effettuano operazioni su di essi.

La maggior parte dei files ha all'interno della propria v-node table entry anche un **i-node** (*index-node*), che contiene la maggior parte delle informazioni contenute nella struttura stat, come ad esempio:
- proprietario del file;
- grandezza del file;
- numero di hard-links;
- puntatori alla reale posizione su disco dei blocchi di dati...

> Schema 
>
> ![](/img/v-node-i-node.png)

## Schema complessivo
>![](/img/kernel_data_structures.png)
> In questo schema potete notare come i file pointer associati ai due file descriptors, alla fine puntino a v-node structures del tutto differenti (sono files o dispositivi diversi su disco).
>
> Consideriamo un altro caso:
>![](/img/kernel_data_structures_same_file.png) 
> Qui invece due file descriptors diversi, associati a processi diversi, puntano due file table entry diverse, ma alla fine puntano la stessa v-node table entry.
>
> Ciò implica che i due processi hanno aperto un file con una *open*, che ha causato l'inserimento di due entries nella file table. Questo vuol dire che i due processi possono leggere lo stesso file da posizioni diverse,
> dato che non condividono lo stesso *current file offset*. 
>
> Si noti che eventuali scritture possono portare a race condition. Risolveremo questo problema introducendo delle opportune funzioni atomiche nei prossimi paragrafi.

## Dischi, partizioni e file system
In questo e nei prossimi paragrafi, presentiamo una vista concettuale della struttura del file system UNIX, entrando man mano nei dettagli.

> ![](/img/unix_fs_structure.png)
>
> Ogni partizione del disco contiene un `boot block` e un `super block`, insieme a un certo numero di `cylinder group`.
> 
> Il `boot block` contiene il bootstrap program, necessario per l'avvio del sistema operativo all'accensione del computer.
>
> Il `super block` descrive lo stato del file system indicando: block size, grandezza della partizione, puntatori a blocchi liberi e l'i-node number della root directory.
>
> Come potete notare, ogni cylinder group contiene inizialmente una copia del *super block*, oltre a informazioni sul cilindro stesso.
>
> La `block bitmap` e la `i-node map` tengono traccia rispettivamente dei blocchi e dei i-nodes allocati / deallocati.
>
> Gli i-nodes sono di lunghezza prefissata e come già detto contengono la maggior parte delle informazioni di un file (che sono poi quelle contenute nella struttura stat). Vi è una corrispondenza biunivoca tra index-nodes e files.

### i-nodes, data blocks e hard-links
> Descriviamo la seguente figura (in un caso specifico)
>
> ![](/img/i_nodes_data_blocks1.png)
>
> Ogni i-node ha diversi blocchi di dati su disco.
>
> I due `directory block` contengono fra le varie possibili entries dello stesso formato, una coppia `<i-node number, filename>`. Notate che entrambi gli i-node number puntano allo stesso i-node. Ciò implica che questi due files, che sono a tutti gli effetti *directory entries*, puntano allo stesso file su disco, sono ossia ***hard-links***.
>
> In generale il massimo numero di hard-links che un file può avere è specificato dalla costante POSIX: `LINK_MAX`.

### i-nodes, directory blocks e hard-links
> Descriviamo la seguente figura (in un caso specifico)
>
> ![](/img/i_nodes_directory_blocks1.png)
>
> Consideriamo prima il directory block con due entries:
> - la prima ha come i-node number 2549 e il filename è `.`, che sta ad indicare la directory corrente
> - la seconda invece ha 1267 e il filename è `..`, che indica la sua parent directory.
>
> Passiamo all'altro directory block, che tra le sue entries ne ha 3 di particolare rilevanza in questo esempio:
> - la prima ha come i-node number 1267 e viene indicata come current directory (quindi è la parent directory dell'altro directory block);
> - la seconda ha un i-node number generico e indica la sua parent directory;
> - la terza invece ha come i-node number 2549 ed indica la cartella `testdir` (confermiamo quindi la relazione di parentela tra i due blocchi).
>
> Possiamo quindi contare il numero di **hard-links** di questi i-nodes:
> - l'i-node 1267 ha 3 hard-links, uno per ogni directory block ed uno proveniente dalla parent directory contenente una directory entry con i-node number pari a 1267;
> - l'i-node 2549 invece ne ha solo 2, uno per ogni directory block.

### Hard-links
Un hard-link è un collegamento diretto all'i-node di un file.

In un sistema UNIX possono sussistere diversi hard-link verso uno stesso file, quindi se volessimo eliminare un file definitivamente, bisognerebbe eliminare i suoi hard-links fino ad azzerare il link count dell'i-node.

Ogni volta che viene creato un hard-link verso un file, il relativo link count viene aumentato.

Ricordiamo che la struttura stat dei files contiene un campo `st_nlink` che contiene proprio il numero di hard-links del file.

## Hard & Soft Links
Per ricordare:
- un hard-link è un riferimento all'i-node di un file;
- un soft-link è un riferimento al pathname di un file (è un riferimento a un hard-link).

I link simbolici vennero introdotti per poter ovviare ad alcune limitazioni degli hard-links, tra cui:
- gli hard-links a directories possono essere creati solo dal superuser, nei sistemi che danno la possibilità di farlo;
- in generale non è possibile avere hard-links che puntano files presenti in altri file systems;

Consideriamo il seguente
> Esempio
>
> Creiamo due files:
>```bash
> touch pippo; touch pluto
>```
> E inseriamo del testo al loro interno:
>```bash
> echo "This is pippo" > pippo
> echo "This is pluto" > pluto
>```
> Creiamo ora un hard-link verso `pippo` e un soft-link verso `pluto`:
>```bash
> ln pippo pippo_hard_link
> ln -s pippo pippo_soft_link
>```
> Diamo un'occhiata a ciò che ci restituisce il comando `ls`:
>
> ![](/img/hard_soft_links.png)
>
> Come potete notare, il numero di hard-links dell'i-node puntato da `pippo` e `pippo_hard_link` è aumentato.
>
> Proviamo a modificare il nome del file `pippo`:
>
> ![](/img/hard_soft_links1.png)
>
> Non è cambiato nulla, infatti le due directory entries puntano direttamente all'i-node di quel file.
>
> Il discorso cambia se si modifica il nome del file puntato dal soft-link:
>
> ![](/img/hard_soft_links2.png)
>
> Il link è diventato invalido.

## Directory hard-links
Come già detto in precedenza, gli hard-links a directories possono essere creati solo dai superuser, se il sistema che si sta usando offre questa possibilità.

Ciò è dovuto al fatto che i directory hard-links possono rompere il file system in diversi modi.

### Cicli nel file system
Un hard-link a una directory potrebbe puntare all'i-node del parent della directory corrente, creando quindi un ciclo.

> Esempio
>```bash
> mkdir -p /tmp/a/b;
> cd /tmp/a/b
> ln -d /tmp/a l
>```
> Creiamo quindi una serie di directory e ci spostiamo in quella più interna dove proviamo a creare un hard-link al parent.
> Nella maggior parte dei sistemi (come Ubuntu) l'operazione fallisce, in altri come Mac OS è possibile sfruttare l'opzione `-F`,
> anche se elimina il link precedente per poter creare quello nuovo.
>
> In sistemi in cui questa operazione è permessa si può entrare in un ciclo,
> la seguente operazione è concessa:
>```bash
> cd /tmp/a/b/l/b/l/b/l/b/l/b/l/b
>```
> Se un programma effettuasse la scansione dell'intero file system, entrerebbe tranquillamente in un ciclo da cui potrebbe non uscire mai.
>
> A questo punto il file system non ha più una struttura ad albero.

### Ambiguità nelle parent directories
Con un loop nel file system, un file può avere più directories padre.

Riprendendo in esame lo scorso esempio, la directory `b` ha due parent directories:
- `/tmp/a/`
- `/tmp/a/b/l`


### Moltiplicazione dei files
Con un loop nel file system, un file può essere identificato da path che non sono più univoci.

> Esempio
>
> Riprendiamo l'esempio precedente e supponiamo di avere un file chiamato `in_loop.txt` all'interno della directory `b`.
> 
> In questo caso il file può essere identificato da infiniti path diversi:
> - `/tmp/a/b/in_loop.txt`;
> - `/tmp/a/b/l/b/in_loop.txt` o addirittura
> - `/tmp/a/b/l/b//l/b/l/b/l/b/l/b/l/b/l/b/l/b/in_loop.txt`.
>
> e così via.
>
> Ovviamente l'i-node puntato è sempre lo stesso, quindi la moltiplicazione dei files è "fittizia".
>
> Ciò però potrebbe indurre un programma a operare su uno stesso file più volte.

## System calls per la gestione di hard-links
Presentiamo alcune system calls per la creazione e l'eliminazione di hard-links programmaticamente.

### link
Il prototipo della funzione è il seguente:

>```C
> #include <unistd.h>
>
> int link(const char *path1, const char *path2); // returns 0 if OK, 1 otherwise
>```
> Affinché le system call abbiano successo, `path1` e `path2` devono trovarsi nello stesso file system. Lo standard POSIX ha inoltre stabilito per questa funzione che `path1` <b>non</b> può essere una directory.
>
> La prima funzione crea una directory entry col valore puntato da `*path2` con gli attributi del file puntato da `*path1`.


### linkat
La funzione è quasi del tutto analoga a **link**, il prototipo è il seguente:
>```C
> #include <unistd.h>
>
> int linkat(int fd1, const char *name1, int fd2, const char *name2, int flag); // returns 0 if OK, 1 otherwise
>```
> Potendo esplicitare i file descriptors delle directory entries da linkare, con questa funzione è possibile creare hard-links a files nello stesso file system ma in path diversi.
>
> L'intepretazione dei file descriptors e dei path è analoga a quanto visto per tutte le funzioni nella loro variante `at`, come [fstatat](#fstatat).
>
> Se il file da cui creare l'hard-link è un soft-link, allora tra i flags è possibile esplicitare il flag `AT_SYMLINK_FOLLOW` per decidere se creare un hard-link verso il soft-link oppure direttamente verso l'i-node del file. Se il flag è presente allora il link viene creato verso il file puntato dal link simbolico.

### unlink
Questa funzione rimuove un hard-link dal file in input, il prototipo è il seguente:
>```C
> #include <unistd.h>
>
> int unlink(const char *pathname); // returns 0 if OK, 1 otherwise
>```
> Ricordate sempre che vi è un side-effect per ogni aggiunta / rimozione di hard-links, ed è l'aggiornamento del link counter degli i-nodes, oltre all'aggiornamento della struttura stat.

### unlinkat
La funzione è del tutto analoga a **unlink**, il prototipo è il seguente:
>```C
> #include <unistd.h>
>
> int unlinkat(int fd, const char *pathname, int flag); // returns 0 if OK, 1 otherwise
>```
> L'intepretazione dei file descriptors e dei path è analoga a quanto visto per tutte le funzioni nella loro variante `at`, come [fstatat](#fstatat).
>
> È possibile esplicitare il flag `AT_REMOVEDIR`, che se impostato permette alla funzione di comportarsi esattamente come la funzione `rmdir()`, che rimuove directories vuote. Altrimenti la funzione si comporta esattamente come `unlink`.
