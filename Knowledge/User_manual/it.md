# LaserBase -- Manuale utente

Questo manuale descrive come funziona LaserBase. L'obiettivo non è
esaurire ogni dettaglio interno, ma rendere il programma utilizzabile in
modo corretto e chiarire che cosa accade tra l'immagine e il G-code.

Il testo segue il flusso di lavoro reale.

------------------------------------------------------------------------

# 1. Che Cos'è LaserBase

LaserBase è una suite software per preparare ed eseguire lavori di
incisione laser.

Le sue parti principali sono:

- **Finestra principale** -- database parametri e gestione record
- **Image Workspace** -- elaborazione immagine, preview, export G-code
- **Sender** -- invio del G-code alla macchina
- **Sketch** -- superficie semplice per disegno e schizzi

LaserBase non è un editor di immagini generico. Il suo scopo è
trasformare un'immagine in un raster allineato alla macchina e in un file
di controllo utilizzabile.

------------------------------------------------------------------------

# 2. La Base dell'Incisione

Il risultato dell'incisione è determinato insieme da tre fattori:

1.  potenza del laser
2.  velocità di avanzamento
3.  densità di punti o linee

Per questo motivo LaserBase non elabora solo un'immagine. Il programma
lega l'immagine anche a una dimensione fisica, a un DPI, alla geometria
della macchina e ai parametri G-code.

------------------------------------------------------------------------

# 3. Il Flusso di Lavoro Completo

Un lavoro tipico si svolge così:

1.  caricare l'immagine
2.  impostare dimensione, DPI e dati macchina
3.  impostare crop e geometria
4.  scegliere la modalità di elaborazione
5.  controllare la preview
6.  impostare velocità e potenza, oppure usare la raccomandazione automatica
7.  salvare il G-code o l'immagine elaborata
8.  salvare un record e ricaricarlo in seguito se necessario

------------------------------------------------------------------------

# 4. Image Workspace

L'Image Workspace è la parte centrale del programma.

L'immagine sorgente è mostrata a sinistra, la vista elaborata a destra.
Quella vista di destra non ha sempre lo stesso significato: a seconda
della modalità selezionata può essere grayscale, dither binario oppure,
in depth mode, uno dei branch elaborati.

Per questo motivo la preview va sempre interpretata insieme alla modalità
di elaborazione selezionata.

------------------------------------------------------------------------

# 5. Caricare un'Immagine

Usare **Load image** per caricare l'immagine.

Formati supportati:

- PNG
- JPG / JPEG
- BMP

Dopo il caricamento, il programma memorizza l'immagine sorgente e la usa
come punto di partenza per tutta l'elaborazione successiva.

------------------------------------------------------------------------

# 6. Immagine RAW, Immagine Elaborata e BASE

Il programma usa più livelli immagine.

**Immagine RAW**

È l'immagine sorgente caricata. Crop e rotazione reale appartengono a
questa immagine.

**Immagine elaborata**

È il raster già allineato alla dimensione fisica scelta, al DPI e alla
geometria macchina, con le trasformazioni immagine già applicate.

**BASE**

BASE non è sempre binario.

- In modalità **Grayscale**, BASE è un raster in scala di grigi
- In modalità **Hybrid**, BASE è un'immagine tonale in scala di grigi con pattern
- Nelle modalità di dither binario, BASE è un'immagine a punti bianco e nero

Il G-code viene sempre generato dal raster elaborato attivo.

------------------------------------------------------------------------

# 7. Dimensione e DPI

La dimensione dell'incisione è indicata in millimetri.

Il DPI definisce la densità di linee o punti:

    spaziatura linee (mm) = 25.4 / DPI

Il DPI richiesto non coincide sempre con il DPI fisicamente usato dalla
macchina. LaserBase costruisce un raster reale allineato al sistema di
passi della macchina, quindi la dimensione dell'immagine elaborata e il
DPI effettivo possono differire dai valori richiesti.

------------------------------------------------------------------------

# 8. Profilo Macchina

Il profilo macchina contiene i dati fisici della macchina:

- steps/mm per asse
- max rate
- acceleration
- modulo laser
- dati profilo G-code

Questi valori non servono solo all'export. Il programma li usa anche per
l'allineamento del raster e per il calcolo automatico dell'overscan.

In modalità fiber il workspace usa un profilo macchina virtuale; in
modalità diode i dati macchina fanno parte dell'elaborazione vera e
propria.

------------------------------------------------------------------------

# 9. Crop

Il crop è definito nello spazio RAW. Questo è importante.

Significa che il ritaglio non viene applicato all'immagine già elaborata,
ma all'immagine sorgente prima delle elaborazioni successive.

Forme disponibili:

- quadrato / rettangolo
- cerchio

Con crop circolare, larghezza e altezza devono coincidere. Se il crop è
attivo ma non valido, il pulsante Process non viene eseguito.

------------------------------------------------------------------------

# 10. Rotazione Reale e Preview Rotate Back

Le due rotazioni non sono la stessa cosa.

**Rotate 90**

È una trasformazione geometrica reale. Cambia l'orientamento
dell'immagine sorgente, e crop, geometria ed elaborazione seguono tale
orientamento.

**Preview rotate back**

È soltanto un livello di visualizzazione. Non modifica elaborazione o
G-code. Ruota indietro solo le preview di sinistra e di destra.

Se l'immagine deve essere ruotata per la macchina, **Rotate 90** è il
controllo corretto. Se l'obiettivo è soltanto osservare l'immagine in un
orientamento più comodo, preview rotate back è lo strumento giusto.

------------------------------------------------------------------------

# 11. Modalità di Elaborazione

La modalità di elaborazione è la decisione centrale nel workspace.

Non definisce soltanto quale algoritmo viene eseguito. Definisce anche il
tipo di BASE prodotto, il significato della preview e il modo in cui il
G-code viene generato.

LaserBase non è limitato al dither binario.

**Grayscale**

Il raster elaborato resta in scala di grigi. Il G-code assegna un valore
PWM a ogni tono di pixel.

**Hybrid**

È una modalità basata sulla scala di grigi che tratta i toni in modo più
patternizzato. Non è né dither binario classico né grayscale puro.

Hybrid è utile quando il grayscale puro appare troppo piatto e il dither
binario troppo duro. Si colloca tra campo tonale e struttura a punti.

**Modalità di dither binario**

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

In queste modalità il raster finale è binario: un punto brucia oppure no.

**Depth**

Non è un algoritmo di dither separato, ma un modello di elaborazione a
due branch.

In depth mode il programma mantiene un branch grayscale e un branch
dither, e la preview può passare da uno all'altro. In export il G-code è
costruito in due passaggi.

Depth è utile quando trasferimento tonale e struttura a punti non devono
essere affidati a un unico raster. Rispetto al semplice grayscale,
fornisce due branch elaborati separati con ruoli diversi.

------------------------------------------------------------------------

# 12. Controlli Immagine

**Contrast, Brightness, Gamma, Radius, Amount**

Questi controlli modellano l'ingresso della fase di elaborazione.

**Negative**

Inversione tonale.

**Mirror X / Mirror Y**

Specchiatura geometrica dell'immagine.

**Threshold**

Non è uno slider immagine generale. Influenza la soglia delle modalità
binarie a diffusione d'errore.

**Serpentine scan**

Nelle modalità di dither a diffusione d'errore inverte la direzione di
elaborazione di una riga sì e una no. Non è una modalità di dither
separata, ma un interruttore aggiuntivo.

**1 pixel off**

Nelle modalità binarie è una fase di pulizia che può rimuovere pixel
singoli isolati.

------------------------------------------------------------------------

# 13. Raccomandazione Auto

**Auto** non è un semplice calcolatore di rapporti.

La raccomandazione viene costruita dal database usando:

- tecnica
- materiale
- modulo laser

Il sistema cerca così di fornire valori per Speed / Max power / Min power.

Se non esiste una corrispondenza esatta, il programma usa una logica di
fallback. Non risale arbitrariamente tutti i livelli superiori; può
ricadere solo su gruppi safe-stop supportati. Quando questo accade, il
programma lo segnala e può richiedere conferma.

In modalità Auto, Speed e Max power restano collegati: se uno dei due
viene modificato manualmente, l'altro viene regolato attorno alla base
della raccomandazione corrente.

Auto è più affidabile quando si basa su una corrispondenza esatta tra
materiale e tecnica. Nei casi di fallback va trattato soprattutto come un
punto di partenza sicuro, non come valore finale.

Il sistema di raccomandazione si basa sui record memorizzati nel database.
Questi record non sono limitati a dati predefiniti: l'utente può aggiungere voci proprie.

I dati salvati possono essere esportati e ricaricati in un altro ambiente.
I file chiave importati aggiornano il set di dati preparato centralmente.

------------------------------------------------------------------------

# 14. Overscan

L'overscan è il tratto di ingresso e uscita al di fuori del bordo
dell'immagine.

In modalità automatica, LaserBase lo calcola a partire da velocità e
accelerazione dell'asse. È possibile inserire anche un override manuale.

Se l'overscan è troppo piccolo, i bordi dell'immagine possono distorcersi
perché la macchina non si muove ancora a velocità costante in quel punto.

------------------------------------------------------------------------

# 15. Elaborazione e Passo Decisionale

**Process** non esegue semplicemente un filtro.

Prima dell'elaborazione, il programma valuta:

- la dimensione richiesta
- il DPI richiesto
- la geometria di passo della macchina
- la risoluzione reale dell'immagine sorgente

Il risultato è:

- **BASE** -- l'elaborazione può essere eseguita direttamente
- **REPAIR** -- per il raster target serve condizionamento immagine / ricampionamento

Il raster elaborato effettivo viene costruito dopo questa decisione.

------------------------------------------------------------------------

# 16. Preview

La preview a destra mostra il branch elaborato selezionato.

Questa preview non è in ogni modalità un'immagine diretta 1:1
dell'incisione finale. Questo vale soprattutto in dither binario e in
depth mode, dove mostra più la logica del raster che la risposta del
materiale.

In depth mode è possibile passare tra due preview:

- Grayscale
- Dither

**Nearest preview** cambia il modo in cui l'immagine viene mostrata, non
il modo in cui viene elaborata.

In fullscreen conviene controllare:

- transizioni tonali
- striature
- eccesso di sharpening
- densità dei punti binari
- differenza tra i due branch in depth mode

------------------------------------------------------------------------

# 17. Generazione del G-code

La strategia G-code dipende dalla modalità.

**Grayscale / Hybrid**

Il programma ricava valori PWM dal tono del pixel, quindi la potenza del
laser può cambiare all'interno di una linea.

**Dither binario**

I punti neri incidono con la massima potenza configurata, i punti bianchi
non incidono.

**Depth**

Vengono prodotti due passaggi successivi: il branch dither e il branch
grayscale ricevono ciascuno il proprio blocco G-code.

------------------------------------------------------------------------

# 18. Save Image, G-code e Frame

**Save image**

Salva l'immagine elaborata attualmente attiva. Può essere il branch
grayscale, hybrid oppure dither.

**G-code**

Salva il file di controllo dell'incisione.

**Frame**

Se attivo, il programma crea anche un file frame separato. I suoi
parametri vengono inseriti in una finestra dedicata. Con crop circolare,
il frame è circolare; altrimenti è rettangolare.

------------------------------------------------------------------------

# 19. Salvataggio nel Database

La finestra principale funziona come database parametri, ma il
salvataggio del workspace contiene più informazioni.

Quando si salva, il programma memorizza:

- il riferimento all'immagine sorgente
- lo stato macchina e geometria
- lo stato del crop
- i controlli BASE e G-code
- lo stato Auto
- entrambe le impostazioni dei branch in depth mode

Per questo motivo il reload non è solo compilazione del record, ma
ripristino dello stato del workspace.

------------------------------------------------------------------------

# 20. Ricaricare

Un record di workspace salvato può essere ricaricato dalla finestra
principale.

Quando ciò avviene, il programma:

1.  ricarica l'immagine sorgente
2.  ripristina dimensione, DPI e dati macchina
3.  ripristina crop e controlli di elaborazione
4.  esegue di nuovo l'elaborazione

Lo stato ripristinato è quindi di tipo sessione, non un semplice elenco
di parametri.

------------------------------------------------------------------------

# 21. Sender

Sender viene eseguito in una finestra separata.

Funzioni di base:

1.  selezionare una porta
2.  connettere
3.  caricare il G-code
4.  avviare l'invio

Durante l'uso:

- Pause / Resume
- STOP
- E-STOP
- $X Unlock in stato di allarme
- movimento jog manuale
- marker laser
- comandi terminale

Sender supporta anche un frame separato, la memorizzazione delle posizioni
machine-start e work-start e il posizionamento basato su offset.

------------------------------------------------------------------------

# 22. Finestra Principale, New Entry, Calculator, Sketch

**Finestra principale**

Memorizza e gestisce i record.

**New Entry**

Crea un nuovo record.

**Calculator**

Aiuta a calcolare nuovi parametri proporzionali a partire da record
esistenti.

**Sketch**

Una superficie semplice e separata per disegno e schizzi.

------------------------------------------------------------------------

# 23. Problemi Comuni

- **Nessuna immagine** -- caricare prima un'immagine sorgente
- **Nessun profilo macchina in modalità diode** -- inserirne uno o caricarlo
- **Crop non valido** -- correggerlo o disattivarlo
- **Crop circolare con dimensioni diverse** -- larghezza e altezza devono coincidere
- **Nessuna raccomandazione auto** -- non esistono dati adatti per questa combinazione materiale / tecnica / modulo
- **Impossibile salvare il G-code** -- l'elaborazione deve prima completarsi correttamente

------------------------------------------------------------------------

# 24. Pratica Utile

- Su materiali nuovi, controllare prima sia la preview grayscale sia quella di dither binario.
- Se l'immagine deve essere ruotata per la macchina, usare il vero Rotate 90.
- Se l'obiettivo è solo una diversa orientazione di visualizzazione, usare preview rotate back.
- In modalità Auto, controllare sempre se la raccomandazione è esatta o basata su fallback.
- Usare depth quando i due ruoli raster devono essere gestiti separatamente.
- Usare l'export Frame prima di posizionare il lavoro reale.
- Trattare il salvataggio del workspace come salvataggio dello stato di lavoro, non solo come salvataggio record.

------------------------------------------------------------------------

# 25. Riferimento Rapido

    spaziatura linee (mm) = 25.4 / DPI
    DPI effettivo = 25.4 / actual pitch_mm

Modalità:

- Grayscale -> raster PWM basato sul tono
- Hybrid -> raster grayscale con pattern
- Dither -> raster binario a punti
- Depth -> elaborazione a due branch ed export in due passaggi

Rotazione:

- Rotate 90 -> cambiamento geometrico reale
- Preview rotate back -> solo visualizzazione

Salvataggio:

- Save image -> branch elaborato attivo
- Save db -> stato del workspace + record
- G-code -> file di controllo, opzionalmente con file frame
