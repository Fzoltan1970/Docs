# LaserBase -- Bildverarbeitung auf Werkstattniveau

Dieses Dokument beschreibt das Bildverarbeitungsmodell von LaserBase. Das
Ziel ist, die Funktionsweise zu verstehen: welche Repräsentationen
existieren, in welcher Reihenfolge die Pipeline läuft und wie das
Programm vom RAW-Bild zum G-Code gelangt.

Dies ist keine Entwickler-Spezifikation. Der Fokus liegt auf einem
nutzbaren, technisch präzisen mentalen Modell.

------------------------------------------------------------------------

## 1. Das Kernbild des Systems

LaserBase ist nicht nur um binäres Dithering aufgebaut.

Das System unterstützt mehrere Verarbeitungsstrategien:

- Graustufenraster
- binäres Dither-Raster
- hybrides Tonmodell
- zweigeteilte Depth-Verarbeitung

Darum lautet die zentrale Frage nicht nur, welchen Dither-Algorithmus man
wählt, sondern welche Bildrepräsentation der gewählte Modus erzeugt und
welche Art von G-Code daraus folgt.

------------------------------------------------------------------------

## 2. Repräsentationen

### 2.1 RAW

RAW ist das geladene Quellbild.

Es ist verknüpft mit:

- echter 90°-Rotation
- Crop
- Quellorientierung

Crop ist nicht auf dem bereits verarbeiteten Bild definiert, sondern im
RAW-Raum.

### 2.2 Verarbeitetes Raster

Das ist das Bild, das bereits:

- an die Zielgröße in der Realität gebunden ist
- an das Maschinenschrittsystem angepasst ist
- die Bildtransformationen trägt
- exportbereit ist

### 2.3 BASE

BASE ist modusabhängig.

Es ist nicht korrekt, BASE immer als binär zu lehren.

- In **Grayscale** ist BASE graustufig
- In **Hybrid** ist BASE graustufig mit Musterung
- In binären Dither-Modi ist BASE binär

### 2.4 Depth-Zustand

Im Depth-Modus hält das System zwei verarbeitete Raster:

- einen Grayscale-Branch
- einen Dither-Branch

Die Preview kann zwischen ihnen wechseln, und der Export besteht aus zwei
Durchläufen.

------------------------------------------------------------------------

## 3. Geometrie, DPI und Maschinenausrichtung

### 3.1 Angefordertes und Reales Raster

Der Benutzer gibt an:

- physische Größe in Millimetern
- angeforderte DPI
- Scanachse
- Maschinendaten

Das Programm zeichnet daraus nicht direkt ein Raster. Es berechnet ein
reales Raster, das an das angepasst ist, was die Maschine schrittweise
abbilden kann.

Wenn Zeilen auf der Schrittachse nur in bestimmten Schrittintervallen
platziert werden können, werden tatsächlicher Pitch und effektive DPI an
diese erlaubte Geometrie angepasst.

### 3.2 Effektive DPI

Effektive DPI ist:

    effektive DPI = 25.4 / real_pitch_mm

Sie kann von der angeforderten DPI abweichen.

### 3.3 Entscheidungsschritt: BASE oder REPAIR

Vor der Verarbeitung entscheidet das System:

- ob das Quellbild für das benötigte reale Raster ausreicht
- oder ob zuerst Bildkonditionierung / Resampling nötig ist

Wenn die benötigte Zahl von Zeilen oder Spalten größer ist als das, was
das Quellbild direkt liefern kann, wird die Entscheidung **REPAIR**.
Andernfalls ist sie **BASE**.

------------------------------------------------------------------------

## 4. RAW-Raum und Orientierung

### 4.1 Rotate 90

`rotate_90` ist eine echte geometrische Transformation.

Es ist kein Preview-Trick. Es ändert die Orientierung des Quellbildes.
Darum arbeiten die folgenden Schritte bereits auf dem rotierten
RAW-Bild:

- Crop
- Geometrieauswertung
- Resample, wenn nötig
- Aufbau des Endrasters

### 4.2 Preview Rotate Back

Preview rotate back ist eine separate Darstellungsebene.

Es transformiert nur die Ansicht. Es verändert nicht:

- den Kernel-Zustand
- die Crop-Definition
- das BASE-Bild
- den G-Code

Die beiden Rotationen bleiben deshalb getrennt:

- `rotate_90` = Verarbeitungsgeometrie
- preview rotate back = visuelle Bequemlichkeit

------------------------------------------------------------------------

## 5. Crop-Modell

### 5.1 Warum Crop im RAW-Raum?

Crop ist auf dem Quellbild definiert, damit der Ausschnitt so früh wie
möglich erfolgt.

Das bedeutet:

- die weitere Verarbeitung startet bereits aus dem beschnittenen Bereich
- Geometrieentscheidungen können die beschnittene effektive Quellgröße verwenden
- Crop kann präzise gespeichert und wiederhergestellt werden

### 5.2 Crop-Formen

- Rechteck / Quadrat
- Kreis

Bei Kreis-Crop muss die Geometrie quadratisch bleiben. Das wird auch auf
UI-Ebene erzwungen.

------------------------------------------------------------------------

## 6. Verarbeitungsmodi

Der Verarbeitungsmodus ist die zentrale Wahl im System.

Er bestimmt, welche Repräsentation BASE annimmt, was die Preview
bedeutet und wie G-Code Rasterdaten in Leistung und Bewegung umsetzt.

### 6.1 Grayscale

In diesem Modus bleibt das Raster graustufig. Der G-Code-Generator bildet
den Pixelton auf PWM-Werte ab.

Dies ist keine Ein/Aus-Gravur, sondern ein kontinuierlicheres
Leistungsprofil.

### 6.2 Binärer Dither

Die Error-Diffusion- und Ordered-Modi gehören hierher:

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Hier ist das Endergebnis binär. Auf G-Code-Ebene erhält ein Pixel
entweder volle Energie oder keine.

### 6.3 Hybrid

Hybrid ist kein klassischer Dither-Modus.

Es startet mit einem Graustufenbild und fügt eine tonabhängige
Musterkorrektur hinzu. Im Mitteltonbereich ist sie stärker, an den
Extremen schwächer.

Seine Rolle ist:

- nicht reines Grayscale
- nicht reiner binärer Dither
- stärker strukturierte Übertragung von Tonflächen

Praktisch überbrückt Hybrid den Abstand zwischen zu glattem Grayscale und
zu hartem binärem Dither. Es ist nützlich, wenn Tonerhalt wichtig ist,
vollständig glattes Grayscale aber nicht genug Struktur liefert.

### 6.4 Depth

Depth hält zwei getrennte Branches:

- Grayscale-Branch
- Dither-Branch

Jeder Branch kann seinen eigenen Kontrollzustand speichern. Die Preview
wechselt zwischen ihnen, und der Export erzeugt zwei Durchläufe.

Depth ist nützlich, wenn Ton und Punktstruktur nicht von demselben Raster
getragen werden sollen. Gegenüber einfachem Grayscale gibt es getrennten
Arbeitsraum für diese zwei Rollen.

------------------------------------------------------------------------

## 7. Die Tatsächliche Rolle der Bildkontrollen

### 7.1 Negative

In der Pipeline läuft `negative` früh.

Das bedeutet, dass die späteren Schritte für Brightness, Contrast, Gamma
und Schärfung bereits im invertierten Tonraum arbeiten.

### 7.2 Brightness und Contrast

Beide arbeiten auf dem graustufigen Arbeitsbild.

- brightness: lineare Verschiebung
- contrast: Skalierung um den Mittelpunkt

### 7.3 Gamma

Gamma ist eine nichtlineare Korrektur im Tonraum. Das System begrenzt den
eingegebenen Wert auf einen zulässigen Bereich und wendet ihn per LUT an.

### 7.4 Radius und Amount

Schärfung basiert auf Unsharp Mask.

### 7.5 Mirror X / Mirror Y

Spiegeloperationen gehören zu den Bildtransformationen und laufen nach
den Tonmodifikationen.

### 7.6 Threshold

Threshold ist kein universeller Regler.

Er beeinflusst nur den Schwellenwert der binären
Error-Diffusion-Modi. In Bayer, halftone clustered, grayscale und hybrid
ist seine Bedeutung anders oder materiell nicht relevant.

### 7.7 Serpentine Scan

Dies ist kein eigener Dither-Algorithmus, sondern ein Wechsel der
Fehlerfortpflanzungsrichtung von Zeile zu Zeile.

Seine Rolle:

- es betrifft nur Error-Diffusion-Modi
- in Bayer oder nichtbinären Modi hat es nicht dieselbe Wirkung

### 7.8 One Pixel Off

Dies ist ein Reinigungsschritt, der auf binärer Rasterausgabe läuft.

Es ist kein allgemeiner Bildfilter. Sein Zweck ist es, isolierte einzelne
Pixel zu entfernen, die aus ihrer Umgebung herausstechen.

------------------------------------------------------------------------

## 8. Die Tatsächliche Reihenfolge der Pipeline

Die logische Reihenfolge ist:

1.  RAW-Bild laden
2.  optional `rotate_90`
3.  Crop im RAW-Raum
4.  Geometriekonditionierung / Resample, falls nötig
5.  `negative`
6.  `contrast` + `brightness`
7.  `gamma`
8.  Schärfung (`radius` + `amount`)
9.  `mirror_x` / `mirror_y`
10. endgültige Größenanpassung auf Rastergröße
11. modusabhängige Dither- / Hybrid-Operation
12. optional `one_pixel_off`

Die Hauptfolgen sind:

- Crop ist ein früher RAW-seitiger Schritt
- `rotate_90` wird vor Crop angewendet
- `negative` liegt nicht nach dem Dithering
- Mirror läuft nach den Tonkontrollen

Diese Reihenfolge ist wichtig, weil sich das Verhalten der Kontrollen
daraus ergibt. Dieselbe Einstellung liefert unterschiedliche Ergebnisse,
je nachdem, wo sie in die Pipeline eintritt.

------------------------------------------------------------------------

## 9. Preview-Modell

### 9.1 Preview Hat Keine Einzige Bedeutung

Die rechte Preview wird aus `_right_view_mode` und dem aktiven
verarbeiteten Branch aufgebaut.

Diese Preview ist nicht immer ein direktes 1:1-Bild der endgültigen
Gravur. In binärem Dither und Depth zeigt sie Verarbeitungsstruktur und
Rastercharakter stärker als die spätere Materialreaktion.

Praktisch bedeutet das:

- es kann keine Preview geben
- es kann eine verarbeitete BASE-Ansicht geben
- in Depth kann die aktive Preview Grayscale oder Dither sein

### 9.2 Nearest Preview

Dies ändert nur die Anzeigeinterpolation. Das verarbeitete Bild wird
nicht verändert.

### 9.3 Fullscreen

Fullscreen verwendet denselben aktiven Preview-Branch wie die normale
rechte Ansicht, nur größer und besser prüfbar.

------------------------------------------------------------------------

## 10. Auto-Empfehlung

### 10.1 Was Sie Verwendet

Die Eingaben sind:

- Materialschlüssel
- Technikschlüssel
- Modulwattzahl des Benutzers

Der Empfehlungsmechanismus aggregiert Datensätze aus der Datenbank.

### 10.2 Keine Einzige Lineare Formel

Auto ist nicht nur eine Formel "Leistung proportional zur Geschwindigkeit".

Die Logik kombiniert zwei Arten von Empfehlungen:

1.  Modulempfehlung auf Basis von Datensatz-Medianen
2.  Empfehlung auf Basis eines Energie-Proxys

Die Endausgabe kann beide mischen.

### 10.3 Fallback

Fallback ist kein beliebiges hierarchisches Aufsteigen.

Das System:

- sucht zuerst einen exakten Materialtreffer
- kann sonst auf bestimmte Safe-Stop-Präfixe zurückfallen
- verwendet keinen allgemeinen Top-Level-Familien-Fallback

Das definiert die Grenzen des Fallback-Verhaltens.

### 10.4 UI-Verhalten

Im Auto-Modus sind empfohlene Speed und Max power an eine Basis gebunden.
Wenn der Benutzer einen Wert verändert, bewegt sich der andere um diese
Basis mit.

Die Empfehlung ist am stärksten, wenn sie aus einem exakten Treffer
kommt. Bei Safe-Stop-Fallback bleibt sie ein geschützter Startpunkt, ist
aber weniger direkt für das gegebene Material.

Die Empfehlung stützt sich auf die in der Datenbank gespeicherten Datensätze.
Diese Datensätze sind nicht auf vordefinierte Inhalte beschränkt: der Benutzer kann eigene Einträge ergänzen.

Gespeicherte Daten können exportiert und in einer anderen Umgebung wieder geladen werden.
Importierte Schlüsseldateien aktualisieren den zentral vorbereiteten Datenbestand.

------------------------------------------------------------------------

## 11. Overscan

Die Overscan-Formel lautet:

    overscan ≈ 1.15 × v² / (2a)

wobei:

- `v` die Scan-Geschwindigkeit in mm/s ist
- `a` die Beschleunigung der aktiven Achse in mm/s² ist

Der Faktor 1.15 wirkt als Sicherheitsreserve.

Aus Sicht des Exports gibt es drei Zustände:

- off
- auto
- manual

Im manuellen Modus überschreibt der eingegebene Wert die automatische
Berechnung.

------------------------------------------------------------------------

## 12. G-Code-Strategie

### 12.1 Grayscale und Hybrid

Hier bildet der G-Code-Generator Pixelwerte auf kontinuierliche Leistung
ab:

    power = s_min + (s_max - s_min) × tone

Darum wirkt `min_power` als untere Grenze der Tonabbildung.

### 12.2 Binärer Dither

In binären Modi erhalten schwarze Pixel die obere Leistungsstufe und
weiße Pixel Null.

In diesem Fall ist `min_power` nicht das zentrale Steuerelement der
binären Auswertung.

### 12.3 Depth-Export

Im Depth-Modus führt das Programm aus:

1.  Aufbau des ersten Passes aus dem Dither-Branch
2.  Aufbau des zweiten Passes aus dem Grayscale-Branch
3.  Verkettung beider G-Code-Blöcke

Der zweite Pass übernimmt auch bestimmte Export-Zustandswerte aus dem
ersten, daher sind dies keine zwei vollständig unabhängigen Exporte.

------------------------------------------------------------------------

## 13. Frame

Frame-Export ist mehr als ein einfaches Kontrollkästchen.

Wenn Frame aktiv ist:

- öffnet das Programm einen separaten Parameterdialog
- können eigene Werte für Speed, Power und Pass count gesetzt werden
- wird neben dem gespeicherten G-Code eine eigene Frame-Datei erzeugt

Bei Kreis-Crop ist der Frame-Pfad kreisförmig, sonst rechteckig.

------------------------------------------------------------------------

## 14. Speichern und Wieder Laden

### 14.1 Save Image

Speichert das aktuell aktive verarbeitete Bild.

Im Depth-Modus bedeutet das, dass nicht ein abstraktes "BASE" gespeichert
wird, sondern der Branch, der in diesem Moment aktiv ist.

### 14.2 Save DB

Die Speicheraktion legt zusätzlich Sidecar-basierten Workspace-Zustand ab.

Typischer Inhalt:

- Bildreferenz
- Maschinenmodus und Maschinenprofil
- Geometriefelder
- Material- / Technik-Auswahl
- Base control
- G-code control
- auto control
- Crop-Zustand
- Depth-Zustand und Branch-Kontrollen

### 14.3 Reload

Reload rekonstruiert den Workspace und führt danach die Verarbeitung
erneut aus.

Darum ist Save/Reload sitzungsartig und nicht nur datensatzartig.

------------------------------------------------------------------------

## 15. Sender Kurz, aus Werkstattsicht

Sender ist nicht nur rohes Streaming.

Relevante Funktionen sind:

- Port- und Verbindungsverwaltung
- line / byte / auto stream modes
- pause / resume / stop / e-stop
- alarm unlock
- jog
- Marker-Laser
- separater Frame-Lauf
- Verwaltung von machine-start und work-start Positionen
- homing-bewusste Startpunktbehandlung

Auf Werkstattniveau ist das wichtig, weil Frame-Behandlung und
Positionierung auch im Sender mehrstufige Abläufe sind.

------------------------------------------------------------------------

## 16. Korrektes Mentales Modell in Kürze

Kurz gesagt:

- RAW und Preview sind nicht dasselbe
- BASE ist nicht immer binär
- Preview ist nicht immer ein einzelnes direktes Bild der Endgravur
- `rotate_90` ist eine echte geometrische Operation
- preview rotate back betrifft nur die Darstellung
- Crop lebt im RAW-Raum
- Auto-Empfehlung ist datenbankbasiert und safe-stop-begrenzt
- die G-Code-Strategie hängt vom Modus ab
- Depth hat zwei Branches und zwei Durchläufe
- Speichern trägt Workspace-Zustand mit

Dies ist ein nutzbares und korrektes Bild davon, wie LaserBase arbeitet.
