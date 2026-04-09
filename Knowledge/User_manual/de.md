# LaserBase -- Benutzerhandbuch

Dieses Handbuch beschreibt die Funktionsweise von LaserBase. Das Ziel ist
nicht, jedes interne Detail auszuschöpfen, sondern das Programm korrekt
nutzbar zu machen und klar zu zeigen, was zwischen Bild und G-Code
geschieht.

Der Text folgt dem realen Arbeitsablauf.

------------------------------------------------------------------------

# 1. Was LaserBase Ist

LaserBase ist eine Software-Suite zum Vorbereiten und Ausführen von
Lasergravur-Aufträgen.

Ihre Hauptteile sind:

- **Hauptfenster** -- Parameterdatenbank und Datensatzverwaltung
- **Image Workspace** -- Bildverarbeitung, Preview, G-Code-Export
- **Sender** -- Senden des G-Codes an die Maschine
- **Sketch** -- einfache Zeichen- und Skizzenfläche

LaserBase ist kein allgemeiner Bildeditor. Der Zweck ist, aus einem Bild
ein maschinenangepasstes Raster und eine verwendbare Steuerdatei zu
erzeugen.

------------------------------------------------------------------------

# 2. Grundlage der Gravur

Das Gravurergebnis wird gemeinsam durch drei Faktoren bestimmt:

1.  Laserleistung
2.  Fahrgeschwindigkeit
3.  Punkt- oder Liniendichte

Darum verarbeitet LaserBase nicht nur ein Bild. Das Programm verknüpft
das Bild auch mit physischer Größe, DPI, Maschinengeometrie und
G-Code-Parametern.

------------------------------------------------------------------------

# 3. Der Gesamte Arbeitsablauf

Ein typischer Auftrag sieht so aus:

1.  Bild laden
2.  Größe, DPI und Maschinendaten festlegen
3.  Crop und Geometrie einstellen
4.  Verarbeitungsmodus wählen
5.  Preview prüfen
6.  Geschwindigkeit und Leistung setzen oder die Auto-Empfehlung nutzen
7.  G-Code oder das verarbeitete Bild speichern
8.  bei Bedarf einen Datensatz speichern und später wieder laden

------------------------------------------------------------------------

# 4. Image Workspace

Der Image Workspace ist der zentrale Teil des Programms.

Links wird das Quellbild gezeigt, rechts die verarbeitete Ansicht. Diese
rechte Ansicht bedeutet nicht immer dasselbe: je nach gewähltem Modus
kann sie Graustufen, binären Dither oder in Depth eine der verarbeiteten
Branches zeigen.

Darum muss die Preview immer zusammen mit dem gewählten
Verarbeitungsmodus gelesen werden.

------------------------------------------------------------------------

# 5. Bild Laden

Mit **Load image** wird das Bild geladen.

Unterstützte Formate:

- PNG
- JPG / JPEG
- BMP

Nach dem Laden speichert das Programm das Quellbild und verwendet es als
Ausgangspunkt für die weitere Verarbeitung.

------------------------------------------------------------------------

# 6. RAW-Bild, Verarbeitetes Bild und BASE

Das Programm verwendet mehrere Bildebenen.

**RAW-Bild**

Das ist das geladene Quellbild. Crop und echte Rotation beziehen sich
auf dieses Bild.

**Verarbeitetes Bild**

Das ist das Raster, das bereits auf die gewählte physische Größe, DPI und
Maschinengeometrie ausgerichtet ist und die Bildtransformationen bereits
enthält.

**BASE**

BASE ist nicht immer binär.

- Im Modus **Grayscale** ist BASE ein Graustufenraster
- Im Modus **Hybrid** ist BASE ein gemustertes Graustufen-Tonbild
- In binären Dither-Modi ist BASE ein Schwarz-Weiß-Punktbild

G-Code wird immer aus dem aktiven verarbeiteten Raster erzeugt.

------------------------------------------------------------------------

# 7. Größe und DPI

Die Gravurgröße wird in Millimetern angegeben.

DPI bestimmt die Linien- oder Punktdichte:

    Linienabstand (mm) = 25.4 / DPI

Die angeforderte DPI ist nicht immer identisch mit der physisch von der
Maschine verwendeten DPI. LaserBase erzeugt ein reales Raster, das auf
das Schrittsystem der Maschine ausgerichtet ist, deshalb können sich
verarbeitete Bildgröße und effektive DPI von den angeforderten Werten
unterscheiden.

------------------------------------------------------------------------

# 8. Maschinenprofil

Das Maschinenprofil enthält die physischen Daten der Maschine:

- steps/mm je Achse
- max rate
- acceleration
- Lasermodul
- G-Code-Profildaten

Diese Werte werden nicht nur für den Export benötigt. Das Programm
verwendet sie auch für Rasterausrichtung und automatische
Overscan-Berechnung.

Im Fiber-Modus verwendet der Workspace ein virtuelles Maschinenprofil; im
Diode-Modus sind die Maschinendaten Teil der eigentlichen Verarbeitung.

------------------------------------------------------------------------

# 9. Crop

Crop ist im RAW-Bildraum definiert. Das ist wichtig.

Das bedeutet, dass der Ausschnitt nicht auf das bereits verarbeitete
Bild, sondern vor der weiteren Verarbeitung auf das Quellbild angewendet
wird.

Verfügbare Formen:

- Quadrat / Rechteck
- Kreis

Für Kreis-Crop müssen Breite und Höhe übereinstimmen. Wenn Crop aktiv,
aber ungültig ist, läuft der Process-Button nicht.

------------------------------------------------------------------------

# 10. Echte Rotation und Preview-Zurückdrehen

Die beiden Rotationen sind nicht dasselbe.

**Rotate 90**

Das ist eine echte geometrische Transformation. Sie ändert die
Ausrichtung des Quellbildes, und Crop, Geometrie und Verarbeitung folgen
dieser Ausrichtung.

**Preview rotate back**

Das ist nur eine Darstellungsebene. Verarbeitung und G-Code werden nicht
verändert. Nur die linke und rechte Preview-Ansicht werden zurückgedreht.

Wenn das Bild wegen der Maschine gedreht werden muss, ist **Rotate 90**
das relevante Werkzeug. Wenn es nur um eine bequemere Ansicht geht, ist
preview rotate back richtig.

------------------------------------------------------------------------

# 11. Verarbeitungsmodi

Der Verarbeitungsmodus ist die zentrale Entscheidung im Workspace.

Er legt nicht nur fest, welcher Algorithmus läuft. Er bestimmt auch,
welcher BASE-Typ entsteht, was die Preview bedeutet und wie der G-Code
erzeugt wird.

LaserBase ist nicht auf binäres Dithering beschränkt.

**Grayscale**

Das verarbeitete Raster bleibt graustufig. Der G-Code weist jedem
Pixelton einen PWM-Wert zu.

**Hybrid**

Dies ist ein Graustufen-basierter Modus, der Töne stärker gemustert
behandelt. Er ist weder klassischer binärer Dither noch reines
Graustufenbild.

Hybrid ist nützlich, wenn reines Grayscale zu flach und binärer Dither zu
hart wirkt. Es liegt zwischen Tonfläche und Punktstruktur.

**Binäre Dither-Modi**

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

In diesen Modi ist das Endraster binär: ein Punkt brennt oder er brennt
nicht.

**Depth**

Dies ist kein eigener Dither-Algorithmus, sondern ein
Zwei-Branch-Verarbeitungsmodell.

Im Depth-Modus hält das Programm einen Grayscale-Branch und einen
Dither-Branch, und in der Preview kann zwischen beiden umgeschaltet
werden. Beim Export entsteht der G-Code aus zwei Durchläufen.

Depth ist nützlich, wenn Tonübertragung und Punktstruktur nicht von einem
einzigen Raster getragen werden sollen. Im Vergleich zu einfachem
Grayscale gibt es zwei getrennte verarbeitete Branches mit verschiedenen
Rollen.

------------------------------------------------------------------------

# 12. Bildkontrollen

**Contrast, Brightness, Gamma, Radius, Amount**

Diese Kontrollen formen den Eingang der Verarbeitungsstufe.

**Negative**

Toninvertierung.

**Mirror X / Mirror Y**

Geometrische Spiegelung des Bildes.

**Threshold**

Dies ist kein allgemeiner Bildregler. Er beeinflusst den Schwellenwert
binärer Error-Diffusion-Modi.

**Serpentine scan**

In Error-Diffusion-Dither-Modi kehrt es die Verarbeitungsrichtung jeder
zweiten Zeile um. Es ist kein eigener Dither-Modus, sondern ein
zusätzlicher Schalter.

**1 pixel off**

In binären Modi ist dies ein Reinigungsschritt, der isolierte einzelne
Pixel entfernen kann.

------------------------------------------------------------------------

# 13. Auto-Empfehlung

**Auto** ist kein einfacher Verhältnisrechner.

Die Empfehlung wird aus der Datenbank aufgebaut anhand von:

- Technik
- Material
- Lasermodul

Daraus versucht das System Werte für Speed / Max power / Min power zu
liefern.

Wenn es keine exakte Übereinstimmung gibt, verwendet das Programm eine
Fallback-Logik. Es steigt nicht beliebig jede höhere Ebene hinauf, sondern
kann nur auf unterstützte Safe-Stop-Gruppen zurückfallen. Wenn das
geschieht, zeigt das Programm es an und kann eine Bestätigung verlangen.

Im Auto-Modus bleiben Speed und Max power gekoppelt: Wenn einer der Werte
von Hand geändert wird, wird der andere um die aktuelle
Empfehlungsbasis herum angepasst.

Auto ist am stärksten, wenn es auf einer exakten Material- und
Technik-Übereinstimmung basiert. In Fallback-Fällen sollte es eher als
sicherer Startpunkt denn als Endwert behandelt werden.

Die Empfehlung stützt sich auf die in der Datenbank gespeicherten Datensätze.
Diese Datensätze sind nicht auf vordefinierte Inhalte beschränkt: der Benutzer kann eigene Einträge ergänzen.

Gespeicherte Daten können exportiert und in einer anderen Umgebung wieder geladen werden.
Importierte Schlüsseldateien aktualisieren den zentral vorbereiteten Datenbestand.

------------------------------------------------------------------------

# 14. Overscan

Overscan ist der Einlauf- und Auslaufweg außerhalb der Bildgrenze.

Im Automatikmodus berechnet LaserBase ihn aus Geschwindigkeit und
Achsbeschleunigung. Eine manuelle Überschreibung kann ebenfalls gesetzt
werden.

Ist Overscan zu klein, können sich die Bildränder verziehen, weil die
Maschine dort noch nicht mit konstanter Geschwindigkeit fährt.

------------------------------------------------------------------------

# 15. Verarbeitung und Entscheidungsschritt

**Process** führt nicht einfach nur einen Filter aus.

Vor der Verarbeitung bewertet das Programm:

- die angeforderte Größe
- die angeforderte DPI
- die Schrittgeometrie der Maschine
- die tatsächliche Auflösung des Quellbildes

Das Ergebnis ist:

- **BASE** -- die Verarbeitung kann direkt ausgeführt werden
- **REPAIR** -- Bildkonditionierung / Resampling ist für das Zielraster erforderlich

Danach wird das eigentliche verarbeitete Raster aufgebaut.

------------------------------------------------------------------------

# 16. Preview

Die rechte Preview zeigt den ausgewählten verarbeiteten Branch.

Diese Preview ist nicht in jedem Modus ein direktes 1:1-Bild der
endgültigen Gravur. Das gilt besonders für binären Dither und Depth, wo
sie mehr Rasterlogik als Materialreaktion zeigt.

Im Depth-Modus kann zwischen zwei Previews umgeschaltet werden:

- Grayscale
- Dither

**Nearest preview** ändert die Art der Darstellung, nicht die
Verarbeitung.

Im Fullscreen-Modus lohnt es sich zu prüfen:

- Tonübergänge
- Streifenbildung
- Überschärfung
- binäre Punktdichte
- den Unterschied zwischen beiden Branches im Depth-Modus

------------------------------------------------------------------------

# 17. G-Code-Erzeugung

Die G-Code-Strategie hängt vom Modus ab.

**Grayscale / Hybrid**

Das Programm leitet PWM-Werte aus dem Pixelton ab, daher kann sich die
Laserleistung innerhalb einer Zeile ändern.

**Binärer Dither**

Schwarze Punkte brennen mit der eingestellten Maximalleistung, weiße
Punkte brennen nicht.

**Depth**

Es werden zwei aufeinanderfolgende Durchläufe erzeugt: Dither-Branch und
Grayscale-Branch erhalten jeweils ihren eigenen G-Code-Block.

------------------------------------------------------------------------

# 18. Save Image, G-Code und Frame

**Save image**

Speichert das aktuell aktive verarbeitete Bild. Das kann der Grayscale-,
Hybrid- oder Dither-Branch sein.

**G-code**

Speichert die Steuerdatei für die Gravur.

**Frame**

Wenn aktiviert, erzeugt das Programm zusätzlich eine separate
Frame-Datei. Die Parameter werden in einem eigenen Dialog eingegeben. Bei
Kreis-Crop ist der Frame kreisförmig, sonst rechteckig.

------------------------------------------------------------------------

# 19. In die Datenbank Speichern

Das Hauptfenster arbeitet als Parameterdatenbank, aber das Speichern des
Workspace enthält mehr als das.

Beim Speichern legt das Programm ab:

- die Referenz auf das Quellbild
- Maschinen- und Geometriezustand
- Crop-Zustand
- BASE- und G-Code-Kontrollen
- Auto-Zustand
- beide Branch-Einstellungen im Depth-Modus

Darum ist Reload nicht nur Datensatzbefüllung, sondern Wiederherstellung
des Workspace-Zustands.

------------------------------------------------------------------------

# 20. Wieder Laden

Ein gespeicherter Workspace-Datensatz kann aus dem Hauptfenster erneut
geladen werden.

Dann führt das Programm aus:

1.  Quellbild erneut laden
2.  Größe, DPI und Maschinendaten wiederherstellen
3.  Crop und Verarbeitungskontrollen wiederherstellen
4.  die Verarbeitung erneut ausführen

Der wiederhergestellte Zustand ist damit sitzungsartig und keine bloße
Parameterliste.

------------------------------------------------------------------------

# 21. Sender

Sender läuft in einem separaten Fenster.

Grundfunktionen:

1.  Port auswählen
2.  verbinden
3.  G-Code laden
4.  Senden starten

Während des Betriebs:

- Pause / Resume
- STOP
- E-STOP
- $X Unlock im Alarmzustand
- manuelle Jog-Bewegung
- Marker-Laser
- Terminalbefehle

Sender unterstützt außerdem einen separaten Frame-Lauf, das Speichern von
Machine-Start- und Work-Start-Positionen sowie offsetbasierte
Positionierung.

------------------------------------------------------------------------

# 22. Hauptfenster, New Entry, Calculator, Sketch

**Hauptfenster**

Speichert und verwaltet Datensätze.

**New Entry**

Erstellt einen neuen Datensatz.

**Calculator**

Hilft, aus vorhandenen Datensätzen proportionale neue Parameter zu
berechnen.

**Sketch**

Eine einfache separate Zeichen- und Skizzenfläche.

------------------------------------------------------------------------

# 23. Häufige Probleme

- **Kein Bild** -- zuerst ein Quellbild laden
- **Kein Maschinenprofil im Diode-Modus** -- eines eingeben oder laden
- **Ungültiger Crop** -- korrigieren oder deaktivieren
- **Kreis-Crop mit ungleichen Maßen** -- Breite und Höhe müssen übereinstimmen
- **Keine Auto-Empfehlung** -- für diese Material- / Technik- / Modulkombination gibt es keine passenden Daten
- **G-Code kann nicht gespeichert werden** -- die Verarbeitung muss zuerst erfolgreich abgeschlossen sein

------------------------------------------------------------------------

# 24. Nützliche Praxis

- Bei neuen Materialien zuerst sowohl Grayscale- als auch binäre Dither-Preview prüfen.
- Wenn das Bild wegen der Maschine gedreht werden muss, echtes Rotate 90 verwenden.
- Wenn nur eine andere Ansichtsorientierung gewünscht ist, preview rotate back verwenden.
- Im Auto-Modus immer prüfen, ob die Empfehlung exakt oder fallback-basiert ist.
- Depth verwenden, wenn die zwei Rasterrollen getrennt behandelt werden sollen.
- Frame-Export vor der Positionierung des echten Auftrags verwenden.
- Workspace-Speichern als Speichern des Arbeitszustands behandeln, nicht nur als Datensatzspeichern.

------------------------------------------------------------------------

# 25. Kurzübersicht

    Linienabstand (mm) = 25.4 / DPI
    effektive DPI = 25.4 / actual pitch_mm

Modi:

- Grayscale -> tonbasiertes PWM-Raster
- Hybrid -> gemustertes Graustufenraster
- Dither -> binäres Punktraster
- Depth -> Zwei-Branch-Verarbeitung und Zwei-Pass-Export

Rotation:

- Rotate 90 -> echte geometrische Änderung
- Preview rotate back -> nur Darstellung

Speichern:

- Save image -> aktiver verarbeiteter Branch
- Save db -> Workspace-Zustand + Datensatz
- G-code -> Steuerdatei, optional mit Frame-Datei
