# Erste Schritte

Die eigentliche Arbeit beginnt im Fenster **Image Workspace**.
Die Verarbeitung beginnt in der Regel mit dem Laden eines vorhandenen Bildes.
Es ist auch möglich, im Fenster **Sketch** eine Zeichnung zu erstellen und sie an die Bildverarbeitung zu senden.
Sketch ist über die Starter-EXE erreichbar, dort kann gezeichnet werden, und die fertige Zeichnung kann an die Bildverarbeitung übergeben werden.
Das Hauptfenster ist nützlich, aber es ist nicht der erforderliche erste Schritt.

## 1. Bild Laden

Öffne den Image Workspace und lade das Bild.
Von hier aus beginnen alle weiteren Schritte.
Ohne Bild gibt es keine Verarbeitung.

Danach richtest du die grundlegende Maschinenumgebung ein.

## 2. Machine Mode und Profil

Wähle zuerst den **machine mode**.
Im Modus **Diode** arbeitest du mit einem eigenen Maschinenprofil.
Im Modus **Fiber** ist das kein gleichartiger blockierender Schritt.

Im Diode-Modus gib ein **machine profile** an oder lade eines.
Das Profil beschreibt das Verhalten der Maschine.
Darauf bauen die korrekte Verarbeitung und der korrekte G-Code auf.
Wenn du es auslässt, ist die Verarbeitung im Diode-Modus nicht gültig.

Du musst die Firmware-Werte nicht auswendig kennen.
Das Programm kann über USB mit der Maschine kommunizieren und die grundlegenden Maschinendaten auslesen.
Wenn das gelingt, werden die Grundlagen des Profils automatisch ausgefüllt.

Zum Profil gehört auch der **Modulwert**.
Wenn du mehrere Lasermodule hast, ist es sinnvoll, für jedes ein eigenes Profil zu speichern.
Das Modul ist nicht nur ein gespeicherter Wert: das System berücksichtigt es später ebenfalls.

Speichere das Profil, sobald es verwendbar ist.
So musst du bei späteren Arbeiten nicht alles erneut eingeben.
Das gespeicherte Profil kann später wieder geladen werden.

Wenn machine mode und Profil feststehen, gehe zur Geometrie weiter.

## 3. Geometrie

Gib diese Werte zusammen an:
- **width**
- **height**
- **DPI**
- **scan axis**

Zusammen bestimmen sie das Raster und die tatsächliche Gravurgröße.
Wenn einer dieser Werte fehlt oder falsch ist, wird auch das Ergebnis nicht korrekt sein.

Danach bereitest du das Bild selbst vor.

## 4. Bild Vorbereiten

Wenn du nicht das ganze Bild verwendest, setze einen **Crop**.
Das ist optional.

Wenn das Bild wirklich gedreht werden muss, benutze **Rotate 90**.
Das wirkt sich auch auf die Verarbeitung aus.

Wenn du das Bild nur in einer anderen Ansicht sehen willst, benutze die Preview-Drehung.
Das ist nur eine Darstellungsebene.

Jetzt kommt die wichtigste Entscheidung.

## 5. Verarbeitungsmodus

Wähle einen **Verarbeitungsmodus**.
Er bestimmt das Endergebnis und auch das Verhalten des G-Codes.

**Grayscale** ist gut, wenn du weichere Tonübergänge willst.
**Dither** ist gut, wenn du mit einem binären Punktbild arbeiten willst.
**Hybrid** ist nützlich, wenn Grayscale zu glatt und Dither zu hart ist.
**Depth** ist für Fälle gedacht, in denen du mit zwei getrennten verarbeiteten Branches arbeiten willst.

Danach prüfst du die Preview.

## 6. Preview und Process

Die Preview dient zur Kontrolle.
In manchen Modi ist sie kein direktes 1:1-Bild der endgültigen Gravur.

Wenn die Einstellungen passen, starte **Process**.
Dabei wird das verarbeitete Bild erzeugt.
Dieses Bild bildet die Grundlage für den Export.

Danach speicherst du die Steuerdatei.

## 7. Export

Speichere den **G-Code**.
Damit wird noch nicht ausgeführt.

**Overscan** ist eine exportseitige Einstellung.
Damit steuerst du, wie die Maschine über die Bildränder hinaus fährt.

Zeilenversatz / line shift / bidirectional compensation ist eine profilgebundene Korrektur.
Das ist kein Schritt für den Einstieg und keine eigene Bildverarbeitungsentscheidung.

Wenn der G-Code fertig ist, gehst du zum Ausführen weiter.

## 8. Ausführen

Der gespeicherte G-Code läuft im Fenster **Sender**.
Dort findet die eigentliche Ausführung statt.

Öffne den Sender.
Wähle den Port und verbinde dich dann mit der Maschine.
Wenn der Port nicht erscheint, prüfe die Verkabelung und ob die Maschine verbunden ist.

Lade den zuvor gespeicherten G-Code.
Stelle die Position ein.

Falls nötig, führe einen separaten **Frame** aus.
Das ist optional und vor dem Positionieren nützlich.

Danach startest du die Arbeit.
Bei Bedarf kannst du Pause, Resume und Stop verwenden.

## Hinweis

Es ist sinnvoll, drei Arten des Speicherns zu unterscheiden.

Das **Speichern des Profils** dient der Wiederverwendung der Maschine.
Damit kannst du dieselbe Maschine oder dasselbe Modul später schnell erneut laden.

Das **Speichern des Workspace** sichert den vollständigen aktuellen Zustand.
Dazu gehören das konkrete Bild und die Verarbeitungseinstellungen.
Verwende das, wenn du später zu derselben Arbeit zurückkehren willst.

Das **DB-Speichern** ist enger gefasst.
Es ist eher eine Speicherung auf Datensatz- und Parameterebene.
Es ist nicht dasselbe wie das Bewahren des vollständigen Workspace-Zustands.

Eine frühere Arbeit kannst du aus dem Hauptfenster wieder laden.
Die detaillierte Funktionsweise ist im Wissensbereich beschrieben.
