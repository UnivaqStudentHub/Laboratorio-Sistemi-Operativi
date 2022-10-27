# Wildcards

* `Cosa sono?` Sono dei simboli che nascondono dei pattern! Definiscono di fatti dei metodi per filtrare dei file per estensione / nome ed eseguire, nel caso, delle operazioni su di loro come una copia o uno spostamento da una cartella ad un'altra!

*IMPORTANTE!*
*Le wildcards sono tradotte dalla shell prima di essere passate al comando, e quindi non sono parte del comando stesso* 

Di seguito elenchiamo le wildcards introdotte a lezione:

* `*` : restituisce in output tutti i path 
    * `<parola/lettera/wildcard>*`  : restituisce tutte i files che **iniziano** con quella `<parola>` o `<lettera>` o  `<wildcard>`  
    * `*<parola/lettera/wildcard>` : restituisce tutte i files che **terminano** con quella `<parola>` o  `<lettera>` o `<wildcard>`


* `?` : restituisce tutti i files che hanno **un solo carattere** nel nome
    * `<parola/lettera/wildcard?>` : restituisce tutti i files che dopo la `<parola>` o  `<lettera>` o `<wildcard>` terminano con **un solo carattere**
    * `?<parola/lettera/wildcard>` : restituisce tutti i files che prima della `<parola>` o  `<lettera>` o `<wildcard>`  iniziano con **un solo** carattere 


* `[]` : restituisce tutti i files formati da **un solo carattere** o da un certo **range di caratteri** ( di solito si utilizza insieme a `*`) 
    * `[agd]` : restituisce tutte le directory formate da **una** ed una sola delle lettere specificate (esempio a.txt, g.txt, d.txt ma non ag.txt, etc...)
    * `[a-d]` : restituisce tutte le directory formate da **una** delle lettere all'interno del range (esempio a.txt, b.txt, c.txt, d.txt)
    * `[^a-d]` oppure `[!a-d]`: restituisce tutte le directory formate da **una** delle lettere che **non sono specificate** (esempio k.txt, z.txt, e.txt)


* `{}` : restituisce tutti i files formati da delle combinazioni di wildcard e di parole scritte all'interno delle parentesi (si possono usare più combinazioni basta che vengano divise da una `,`)


* `\` : si usa prima di una wildcard per farla leggere come stringa (anche lo stesso \ (backslash))
    * per scrivere il carattere `\` si usa `\\`