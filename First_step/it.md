# Primi passi

Il lavoro reale inizia nella finestra **Image Workspace**.
L'elaborazione inizia di solito caricando un'immagine esistente.
È anche possibile creare un disegno nella finestra **Sketch** e inviarlo all'elaborazione immagini.
Sketch è disponibile dall'EXE di avvio, lì si può disegnare e il disegno finito può essere inviato all'elaborazione immagini.
La finestra principale è utile, ma non è il primo passaggio obbligatorio.

## 1. Caricamento dell'Immagine

Apri l'Image Workspace e carica l'immagine.
Tutti i passaggi successivi partono da qui.
Senza immagine non c'è elaborazione.

Dopo questo, imposta l'ambiente macchina di base.

## 2. Machine Mode e Profilo

Per prima cosa scegli il **machine mode**.
In modalità **Diode** lavori con un tuo profilo macchina.
In modalità **Fiber** non è lo stesso tipo di passaggio bloccante.

In modalità Diode, inserisci o carica un **machine profile**.
Il profilo descrive il comportamento della macchina.
Su questo si basano l'elaborazione corretta e il G-code corretto.
Se lo salti, l'elaborazione in modalità Diode non sarà valida.

Non devi conoscere a memoria i valori del firmware.
Il programma può comunicare con la macchina tramite USB e leggere i dati macchina iniziali.
Se questo riesce, la base del profilo viene compilata automaticamente.

Anche il **valore del modulo** fa parte del profilo.
Se hai più moduli laser, conviene salvare un profilo separato per ciascuno.
Il modulo non è soltanto un valore memorizzato: il sistema lo considera anche in seguito.

Salva il profilo appena è utilizzabile.
In questo modo nei lavori successivi non dovrai reinserire tutto.
Il profilo salvato può essere ricaricato in seguito.

Quando machine mode e profilo sono pronti, passa alla geometria.

## 3. Geometria

Inserisci insieme questi elementi:
- **width**
- **height**
- **DPI**
- **scan axis**

Insieme definiscono il raster e la dimensione reale dell'incisione.
Se uno di questi elementi manca o è sbagliato, anche il risultato non sarà corretto.

Poi prepara l'immagine vera e propria.

## 4. Preparazione dell'Immagine

Se non usi l'immagine intera, imposta un **crop**.
È opzionale.

Se l'immagine deve davvero essere ruotata, usa **Rotate 90**.
Questo influisce anche sull'elaborazione.

Se vuoi solo vedere l'immagine con un orientamento diverso, usa la rotazione della preview.
Questo è solo un livello di visualizzazione.

Ora arriva la decisione più importante.

## 5. Modalità di Elaborazione

Scegli una **modalità di elaborazione**.
Questa determina il risultato finale e anche il comportamento del G-code.

**Grayscale** va bene se vuoi un tono più continuo.
**Dither** va bene se vuoi lavorare con un'immagine binaria a punti.
**Hybrid** è utile se il grayscale è troppo uniforme e il dither troppo duro.
**Depth** serve quando vuoi lavorare con due branch elaborati separati.

Dopo questo, controlla la preview.

## 6. Preview e Process

La preview serve per il controllo.
In alcune modalità non è un'immagine diretta 1:1 dell'incisione finale.

Se le impostazioni sono corrette, avvia **Process**.
Qui viene creata l'immagine elaborata.
Questa immagine diventa la base dell'export.

Dopo questo, salva il file di controllo.

## 7. Export

Salva il **G-code**.
Questo non è ancora l'avvio del lavoro.

L'**overscan** è un'impostazione lato export.
Controlla il modo in cui la macchina supera i bordi dell'immagine.

Lo spostamento di linea / line shift / bidirectional compensation è una correzione legata al profilo.
Non è un passaggio da principiante, e non è una decisione separata di elaborazione immagini.

Quando il G-code è pronto, passa all'esecuzione.

## 8. Esecuzione

Il G-code salvato viene eseguito nella finestra **Sender**.
Qui avviene l'esecuzione reale.

Apri Sender.
Seleziona la porta, poi connettiti alla macchina.
Se la porta non compare, controlla il cablaggio e che la macchina sia collegata.

Carica il G-code salvato in precedenza.
Imposta la posizione.

Se serve, esegui un **frame** separato.
È opzionale, ed è utile prima del posizionamento.

Poi avvia il lavoro.
Se serve, puoi usare Pause, Resume e Stop.

## Nota

È utile distinguere tre tipi di salvataggio.

Il **salvataggio del profilo** serve per riutilizzare la macchina.
Ti permette di ricaricare rapidamente in seguito la stessa macchina o lo stesso modulo.

Il **salvataggio del workspace** memorizza lo stato completo corrente.
Questo include l'immagine concreta e le impostazioni di elaborazione.
Usalo se vuoi tornare più tardi sullo stesso lavoro.

Il **salvataggio DB** è più limitato.
È più un salvataggio a livello di record e parametri.
Non è la stessa cosa che conservare lo stato completo del workspace.

Puoi ricaricare un lavoro precedente dalla finestra principale.
Il funzionamento dettagliato è descritto nella sezione Knowledge.
