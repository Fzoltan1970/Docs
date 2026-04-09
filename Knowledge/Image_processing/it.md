# LaserBase -- Elaborazione immagini a livello officina

Questo documento descrive il modello di elaborazione immagini di
LaserBase. L'obiettivo è comprenderne il funzionamento: quali
rappresentazioni esistono, in quale ordine viene eseguita la pipeline e
come il programma passa dall'immagine RAW al G-code.

Non è una specifica per sviluppatori. L'attenzione è posta su un modello
mentale utilizzabile e tecnicamente accurato.

------------------------------------------------------------------------

## 1. Quadro Centrale del Sistema

LaserBase non è costruito soltanto attorno al dither binario.

Il sistema supporta diverse strategie di elaborazione:

- raster in scala di grigi
- raster di dither binario
- modello tonale ibrido
- elaborazione depth a due branch

Per questo motivo, la domanda centrale non è soltanto quale algoritmo di
dither scegliere, ma quale rappresentazione immagine produce la modalità
scelta e quale tipo di G-code ne deriva.

------------------------------------------------------------------------

## 2. Rappresentazioni

### 2.1 RAW

RAW è l'immagine sorgente caricata.

È collegata a:

- rotazione reale di 90°
- crop
- orientamento sorgente

Il crop non è definito sull'immagine già elaborata, ma nello spazio RAW.

### 2.2 Raster Elaborato

È l'immagine già:

- legata alla dimensione fisica target
- allineata ai passi macchina
- comprensiva delle trasformazioni immagine
- pronta per l'export

### 2.3 BASE

BASE dipende dalla modalità.

Non è corretto insegnarlo come sempre binario.

- In **Grayscale**, BASE è in scala di grigi
- In **Hybrid**, BASE è in scala di grigi con pattern
- Nelle modalità di dither binario, BASE è binario

### 2.4 Stato Depth

In depth mode il sistema mantiene due raster elaborati:

- un branch grayscale
- un branch dither

La preview può passare da uno all'altro, e l'export è costruito in due
passaggi.

------------------------------------------------------------------------

## 3. Geometria, DPI e Allineamento Macchina

### 3.1 Raster Richiesto e Raster Reale

L'utente fornisce:

- dimensione fisica in millimetri
- DPI richiesto
- asse di scansione
- dati macchina

Il programma non disegna direttamente un raster a partire da questi dati.
Calcola invece un raster reale allineato a ciò che la macchina può
riprodurre passo per passo.

Se le linee sull'asse di passo possono essere collocate solo a intervalli
specifici, il pitch reale e il DPI effettivo vengono adattati a tale
geometria consentita.

### 3.2 DPI Effettivo

Il DPI effettivo è:

    DPI effettivo = 25.4 / real_pitch_mm

Può differire dal DPI richiesto.

### 3.3 Passo Decisionale: BASE o REPAIR

Prima dell'elaborazione il sistema decide:

- se l'immagine sorgente è sufficiente per il raster reale richiesto
- oppure se è prima necessario un condizionamento immagine / ricampionamento

Se il numero richiesto di linee o colonne è maggiore di quanto
l'immagine sorgente possa fornire direttamente, la decisione diventa
**REPAIR**. Altrimenti è **BASE**.

------------------------------------------------------------------------

## 4. Spazio RAW e Orientamento

### 4.1 Rotate 90

`rotate_90` è una trasformazione geometrica reale.

Non è un trucco di preview. Cambia l'orientamento dell'immagine sorgente.
Per questo motivo, i passaggi seguenti lavorano già sull'immagine RAW
ruotata:

- crop
- valutazione geometrica
- ricampionamento se necessario
- costruzione del raster finale

### 4.2 Preview Rotate Back

Preview rotate back è un livello di visualizzazione separato.

Trasforma solo la vista. Non modifica:

- lo stato del kernel
- la definizione del crop
- l'immagine BASE
- il G-code

Le due rotazioni restano quindi separate:

- `rotate_90` = geometria di elaborazione
- preview rotate back = comodità visiva

------------------------------------------------------------------------

## 5. Modello di Crop

### 5.1 Perché Crop in Spazio RAW

Il crop è definito sull'immagine sorgente affinché il ritaglio avvenga il
prima possibile.

Questo significa:

- l'elaborazione successiva parte già dall'area ritagliata
- le decisioni geometriche possono usare la dimensione sorgente effettiva ritagliata
- il crop può essere salvato e ripristinato con precisione

### 5.2 Forme di Crop

- rettangolo / quadrato
- cerchio

Con crop circolare, la geometria deve restare quadrata. Questo vincolo è
imposto anche a livello UI.

------------------------------------------------------------------------

## 6. Modalità di Elaborazione

La modalità di elaborazione è la scelta centrale del sistema.

Definisce quale rappresentazione assume BASE, che cosa significa la
preview e come il G-code traduce i dati raster in potenza e movimento.

### 6.1 Grayscale

In questa modalità il raster resta in scala di grigi. Il generatore
G-code mappa il tono del pixel su valori PWM.

Non è un'incisione on/off, ma un profilo di potenza più continuo.

### 6.2 Dither Binario

Le modalità a diffusione d'errore e quelle ordinate appartengono a questo
gruppo:

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Qui il risultato finale è binario. A livello G-code, un pixel riceve
bruciatura piena oppure nulla.

### 6.3 Hybrid

Hybrid non è una modalità di dither classica.

Parte da un'immagine in scala di grigi e aggiunge una correzione di
pattern dipendente dal tono. È più forte nei mezzitoni e più debole agli
estremi.

Il suo ruolo è:

- non grayscale puro
- non dither binario puro
- trasferimento più strutturato dei campi tonali

In termini pratici, Hybrid colma il divario tra un grayscale troppo
uniforme e un dither binario troppo duro. È utile quando è importante
mantenere il tono, ma un mapping grayscale completamente liscio non
fornisce abbastanza struttura.

### 6.4 Depth

Depth mantiene due branch separati:

- branch grayscale
- branch dither

Ogni branch può memorizzare il proprio stato dei controlli. La preview
passa da uno all'altro e l'export genera due passaggi.

Depth è utile quando tono e struttura a punti non devono essere affidati
allo stesso raster. Rispetto al semplice grayscale, offre spazio di
lavoro separato per questi due ruoli.

------------------------------------------------------------------------

## 7. Il Ruolo Reale dei Controlli Immagine

### 7.1 Negative

Nella pipeline, `negative` viene eseguito presto.

Questo significa che i passaggi successivi di brightness, contrast,
gamma e sharpening lavorano già nello spazio tonale invertito.

### 7.2 Brightness e Contrast

Entrambi operano sull'immagine di lavoro in scala di grigi.

- brightness: spostamento lineare
- contrast: scalatura attorno al punto medio

### 7.3 Gamma

Gamma è una correzione non lineare nello spazio tonale. Il sistema limita
il valore inserito a un intervallo consentito e lo applica tramite LUT.

### 7.4 Radius e Amount

Lo sharpening è basato su unsharp mask.

### 7.5 Mirror X / Mirror Y

Le operazioni di specchio fanno parte delle trasformazioni immagine e
vengono eseguite dopo le modifiche tonali.

### 7.6 Threshold

Threshold non è uno slider universale.

Influenza solo la soglia usata dalle modalità binarie a diffusione
d'errore. In Bayer, halftone clustered, grayscale e hybrid, il suo
significato è diverso o non materialmente rilevante.

### 7.7 Serpentine Scan

Non è un algoritmo di dither separato, ma un'alternanza della direzione
di propagazione dell'errore riga per riga.

Il suo ruolo:

- riguarda solo le modalità a diffusione d'errore
- non ha lo stesso effetto in Bayer o nelle modalità non binarie

### 7.8 One Pixel Off

È una fase di pulizia applicata all'output raster binario.

Non è un filtro immagine generale. Il suo scopo è rimuovere pixel singoli
isolati che sporgono rispetto all'intorno.

------------------------------------------------------------------------

## 8. L'Ordine Reale della Pipeline

L'ordine logico è:

1.  caricare l'immagine RAW
2.  `rotate_90` opzionale
3.  crop nello spazio RAW
4.  condizionamento geometrico / ricampionamento se necessario
5.  `negative`
6.  `contrast` + `brightness`
7.  `gamma`
8.  sharpening (`radius` + `amount`)
9.  `mirror_x` / `mirror_y`
10. resize finale alla dimensione raster
11. operazione dither / hybrid dipendente dalla modalità
12. `one_pixel_off` opzionale

Le conseguenze principali sono:

- il crop è un passaggio iniziale lato RAW
- `rotate_90` viene applicato prima del crop
- `negative` non è dopo il dithering
- mirror viene eseguito dopo i controlli tonali

Questo ordine è importante perché il comportamento dei controlli dipende
da esso. La stessa impostazione produce risultati diversi a seconda del
punto in cui entra nella pipeline.

------------------------------------------------------------------------

## 9. Modello di Preview

### 9.1 La Preview Non Ha un Significato Unico

La preview di destra è costruita a partire da `_right_view_mode` e dal
branch elaborato attivo.

Questa preview non è sempre un'immagine diretta 1:1 dell'incisione
finale. In dither binario e in depth mode mostra soprattutto la struttura
di elaborazione e il carattere del raster, più che la risposta finale del
materiale.

In pratica questo significa:

- può non esserci preview
- può esserci una vista BASE elaborata
- in depth la preview attiva può essere grayscale o dither

### 9.2 Nearest Preview

Questo cambia solo l'interpolazione di visualizzazione. L'immagine
elaborata non viene modificata.

### 9.3 Fullscreen

Il fullscreen usa lo stesso branch preview attivo della normale vista a
destra, ma in scala più grande e più facile da ispezionare.

------------------------------------------------------------------------

## 10. Raccomandazione Auto

### 10.1 Che Cosa Usa

Gli input sono:

- chiave materiale
- chiave tecnica
- wattaggio del modulo utente

Il sistema di raccomandazione aggrega record dal database.

### 10.2 Non una Singola Formula Lineare

Auto non è soltanto una formula "potenza proporzionale alla velocità".

La logica combina due tipi di raccomandazione:

1.  raccomandazione modulo basata sulle mediane dei record
2.  raccomandazione basata su un proxy energetico

L'output finale può fondere entrambe.

### 10.3 Fallback

Il fallback non è una risalita gerarchica arbitraria.

Il sistema:

- cerca prima una corrispondenza esatta di materiale
- in alternativa può ricadere su specifici prefissi safe-stop
- non usa un fallback generale di famiglia di livello superiore

Questo definisce i limiti del comportamento di fallback.

### 10.4 Comportamento UI

In modalità Auto, Speed e Max power raccomandati sono legati a una base.
Se l'utente modifica uno dei due, l'altro cambia attorno a quella base.

La raccomandazione è più forte quando proviene da un match esatto. Con
fallback safe-stop resta un punto di partenza più protetto, ma meno
diretto per il materiale specifico.

Il sistema di raccomandazione si basa sui record memorizzati nel database.
Questi record non sono limitati a dati predefiniti: l'utente può aggiungere voci proprie.

I dati salvati possono essere esportati e ricaricati in un altro ambiente.
I file chiave importati aggiornano il set di dati preparato centralmente.

------------------------------------------------------------------------

## 11. Overscan

La formula dell'overscan è:

    overscan ≈ 1.15 × v² / (2a)

dove:

- `v` è la velocità di scansione in mm/s
- `a` è l'accelerazione dell'asse attivo in mm/s²

Il fattore 1.15 agisce come margine di sicurezza.

Dal punto di vista dell'export esistono tre stati:

- off
- auto
- manual

In modalità manual, il valore inserito sovrascrive il calcolo automatico.

------------------------------------------------------------------------

## 12. Strategia G-code

### 12.1 Grayscale e Hybrid

Qui il generatore G-code mappa i valori dei pixel su una potenza
continua:

    power = s_min + (s_max - s_min) × tone

Per questo motivo, `min_power` agisce come limite inferiore del mapping
tonale.

### 12.2 Dither Binario

Nelle modalità binarie, i pixel neri ricevono il livello di potenza più
alto e i pixel bianchi ricevono zero.

In questo caso, `min_power` non è l'elemento principale di controllo
della valutazione binaria.

### 12.3 Export Depth

In depth mode il programma:

1.  costruisce il primo passaggio dal branch dither
2.  costruisce il secondo passaggio dal branch grayscale
3.  concatena entrambi i blocchi G-code

Il secondo passaggio eredita anche alcuni valori di stato export dal
primo, quindi non si tratta di due export completamente indipendenti.

------------------------------------------------------------------------

## 13. Frame

L'export frame è più di una semplice casella di controllo.

Quando frame è attivo:

- il programma apre una finestra parametri separata
- si possono impostare speed, power e pass count dedicati
- viene generato un file frame separato accanto al G-code salvato

Con crop circolare, il percorso frame è circolare; altrimenti è
rettangolare.

------------------------------------------------------------------------

## 14. Salvataggio e Ricarica

### 14.1 Save Image

Salva l'immagine elaborata attualmente attiva.

In depth mode questo significa che l'output salvato non è un "BASE"
astratto, ma il branch attivo in quel momento.

### 14.2 Save DB

L'azione di salvataggio memorizza anche lo stato del workspace basato su
sidecar.

Contenuto tipico:

- riferimento immagine
- modalità macchina e profilo macchina
- campi geometrici
- selezione materiale / tecnica
- base control
- gcode control
- auto control
- stato del crop
- stato depth e controlli dei branch

### 14.3 Reload

Reload ricostruisce il workspace e poi esegue di nuovo l'elaborazione.

Per questo motivo, save/reload ha natura di sessione, non solo di record.

------------------------------------------------------------------------

## 15. Sender in Breve, Vista Officina

Sender non è soltanto streaming grezzo.

Funzioni rilevanti:

- gestione porta e connessione
- line / byte / auto stream modes
- pause / resume / stop / e-stop
- alarm unlock
- jog
- marker laser
- esecuzione frame separata
- gestione posizioni machine-start e work-start
- gestione start point consapevole del homing

Dal punto di vista officina, questo conta perché anche frame e
posizionamento sono processi a più stati in Sender.

------------------------------------------------------------------------

## 16. Modello Mentale Corretto in Sintesi

In breve:

- RAW e preview non sono la stessa cosa
- BASE non è sempre binario
- la preview non è sempre un'unica immagine diretta dell'incisione finale
- `rotate_90` è una vera operazione geometrica
- preview rotate back riguarda solo la visualizzazione
- il crop vive nello spazio RAW
- la raccomandazione Auto è basata su database e limitata da safe-stop
- la strategia G-code dipende dalla modalità
- depth ha due branch e due passaggi
- il salvataggio porta con sé lo stato del workspace

Questo fornisce un quadro corretto e utilizzabile di come funziona
LaserBase.
