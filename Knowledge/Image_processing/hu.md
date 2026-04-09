# LaserBase -- Képfeldolgozás műhely szinten

Ez a dokumentum a LaserBase képfeldolgozásának modelljét írja
le. A cél a működés megértése: milyen reprezentációk vannak, milyen
sorrendben fut a pipeline, és hogyan jutunk el a RAW képtől a G-code-ig.

Nem fejlesztői specifikáció. A hangsúly a használható, technikailag
pontos mentális modellen van.

------------------------------------------------------------------------

## 1. A rendszer alapképe

A LaserBase nem kizárólag bináris ditheringre épül.

A rendszer többféle feldolgozási stratégiát támogat:

- szürkeárnyalatos raszter
- bináris dither raszter
- hibrid tónusrendszer
- kétágú depth feldolgozás

Ezért a központi kérdés nem az, hogy „melyik dither algoritmust
használjuk”, hanem az, hogy a választott mód milyen képreprezentációt ad,
és abból milyen G-code készül.

------------------------------------------------------------------------

## 2. Reprezentációk

### 2.1 RAW

A RAW a betöltött forráskép.

Ehhez kapcsolódik:

- a valódi 90°-os forgatás
- a crop
- a forráskép orientációja

A crop nem a már feldolgozott képre vonatkozik, hanem a RAW térre.

### 2.2 Feldolgozott raszter

Ez az a kép, amely már:

- a kívánt fizikai mérethez van rendelve
- a gép léptetéséhez van igazítva
- tartalmazza a képi transzformációkat
- közvetlenül exportálható

### 2.3 BASE

A BASE fogalma módfüggő.

Nem helyes úgy tanítani, hogy a BASE mindig bináris.

- **Grayscale** esetén a BASE szürkeárnyalatos
- **Hybrid** esetén a BASE szürkeárnyalatos, mintázott
- bináris dither módoknál a BASE bináris

### 2.4 Depth állapot

Depth módban a rendszer két feldolgozott rasztert tart:

- egy grayscale ágat
- egy dither ágat

A preview ezek között válthat, az export pedig két passzból épül fel.

------------------------------------------------------------------------

## 3. Geometria, DPI és gépillesztés

### 3.1 Kért és valós raszter

A felhasználó megad:

- fizikai méretet milliméterben
- kért DPI-t
- scan tengelyt
- gépadatokat

A program ebből nem közvetlenül rajzol rasztert, hanem a gép
léptethetőségéhez illesztett valós rasztert számol.

Ha a step-tengelyen a sorok csak meghatározott lépésközökkel vihetők fel,
akkor a tényleges pitch és az effektív DPI a gép által megengedett
értékhez igazodik.

### 3.2 Effektív DPI

Az effektív DPI:

    effektív DPI = 25,4 / real_pitch_mm

Ez lehet eltérő a kért DPI-től.

### 3.3 Döntési lépés: BASE vagy REPAIR

A feldolgozás előtt a rendszer eldönti:

- a forráskép elegendő-e a szükséges valós raszterhez
- vagy előbb képkondicionálás / újramintázás kell

Ha a szükséges sor- vagy oszlopszám nagyobb, mint ami a forrásképből
közvetlenül kijön, a döntés **REPAIR** lesz. Ellenkező esetben **BASE**.

------------------------------------------------------------------------

## 4. RAW tér és orientáció

### 4.1 Rotate 90

A `rotate_90` valódi geometriai transzformáció.

Nem preview trükk, hanem a forráskép orientációját módosítja. Emiatt a
következő lépések már az elforgatott RAW képpel dolgoznak:

- crop
- geometriai kiértékelés
- szükség szerinti resample
- végső raszterépítés

### 4.2 Preview visszaforgatás

A preview-visszaforgatás külön megjelenítési réteg.

Ez csak a nézetet transzformálja. Nem módosítja:

- a kernel állapotát
- a crop definícióját
- a BASE képet
- a G-code-ot

Ezért a két forgatást szét kell választani:

- `rotate_90` = feldolgozási geometria
- preview rotate back = vizuális kényelem

------------------------------------------------------------------------

## 5. Crop modell

### 5.1 Miért RAW-space crop?

A crop a forrásképen értelmezett, mert így a kivágás a lehető
legkorábban történik.

Ennek következménye:

- a további feldolgozás már a kivágott területből indul
- a geometriai döntés is a cropolt effektív forrásmérettel számolhat
- a crop menthető és később pontosan visszatölthető

### 5.2 Crop alakok

- téglalap / négyzet
- kör

Kör cropnál a geometriának négyzetesnek kell lennie. Ez UI oldalon is
ellenőrzött feltétel.

------------------------------------------------------------------------

## 6. Feldolgozási módok

A feldolgozási mód a rendszer központi választása.

Ez határozza meg, hogy a BASE milyen reprezentáció lesz, a preview mit
jelent, és a G-code hogyan képezi át a rasztert teljesítménnyé és
mozgássá.

### 6.1 Grayscale

Ebben a módban a raszter szürkeárnyalatos marad. A G-code generátor a
pixel tónusát PWM értékre képezi le.

Ez nem be/ki alapú gravírozás, hanem folytonosabb teljesítményprofil.

### 6.2 Bináris dither

Az error-diffusion és ordered módok ebbe a csoportba tartoznak:

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Itt a végeredmény bináris. A G-code szinten a pixel vagy kap teljes
égetést, vagy nem kap.

### 6.3 Hybrid

A Hybrid nem klasszikus dither.

Szürkeárnyalatos képből indul, és egy tónusfüggő mintázati korrekciót ad
hozzá. A középtónusokban erősebb, a szélső tartományokban gyengébb.

Ennek szerepe:

- nem tiszta grayscale
- nem tiszta bináris dither
- a tónusmezőket texturáltabban viszi át

Gyakorlati értelemben a Hybrid a túl sima grayscale és a túl durva
bináris dither közti rést hidalja át. Akkor hasznos, amikor a tónusmező
megtartása fontos, de a teljesen sima szürkeleképezés nem ad elég
struktúrát.

### 6.4 Depth

A Depth két külön branch-et tart fenn:

- grayscale branch
- dither branch

Mindkettő saját kontrollállapotot tárolhat. A preview a két ág között
vált, az export pedig két passzt generál.

A Depth akkor indokolt, amikor a tónus és a pontszerkezet nem ugyanazzal
az egyetlen raszterrel kezelhető jól. A sima grayscale-hez képest külön
munkateret ad a két szerepnek.

------------------------------------------------------------------------

## 7. A képi kontrollok valódi szerepe

### 7.1 Negative

A pipeline-ban a `negative` korán fut.

Ez azt jelenti, hogy a későbbi brightness, contrast, gamma és sharpening
már az invertált tónustérre dolgozik.

Ez azt jelenti, hogy a későbbi műveletek már az invertált tónustéren
dolgoznak.

### 7.2 Brightness és Contrast

Mindkettő a szürkeárnyalatos munkaképen fut.

- brightness: lineáris eltolás
- contrast: középpont körüli skálázás

### 7.3 Gamma

Nemlineáris korrekció a tónustéren. A rendszer a megadott gamma értéket
korlátok közé szorítja, és LUT alapon alkalmazza.

### 7.4 Radius és Amount

Az élesítés unsharp mask jellegű.

### 7.5 Mirror X / Mirror Y

A tükörműveletek a képi transzformáció részei, és a
implementációban a tónusmódosítások után futnak.

### 7.6 Threshold

A threshold nem univerzális csúszka.

Csak az error-diffusion jellegű bináris dither módok küszöbére hat.
Bayer, halftone clustered, grayscale és hybrid esetén más az értelme vagy
nincs érdemi szerepe.

### 7.7 Serpentine scan

Ez nem külön dither algoritmus, hanem a hibaterjesztés irányának
váltogatása soronként.

Jelenlegi szerepe:

- csak az error-diffusion módokra hat
- Bayerre és nem bináris módokra nem ad ugyanilyen hatást

### 7.8 One pixel off

Ez utólagos, bináris raszteren futó tisztítás.

Nem általános képszűrő. A cél az izolált, környezetéből kilógó egyedi
pixelek eltüntetése.

------------------------------------------------------------------------

## 8. A pipeline tényleges sorrendje

A logikai sorrend:

1.  RAW kép betöltése
2.  opcionális `rotate_90`
3.  RAW-space crop
4.  szükség esetén geometriai kondicionálás / resample
5.  `negative`
6.  `contrast` + `brightness`
7.  `gamma`
8.  élesítés (`radius` + `amount`)
9.  `mirror_x` / `mirror_y`
10. végső raszterméretre átméretezés
11. módfüggő dithering / hybrid művelet
12. opcionális `one_pixel_off`

A sorrend fő pontjai:

- a crop korai, RAW oldali lépés
- a `rotate_90` még a crop előtt érvényes
- a `negative` nem a dithering után van
- a mirror a tónusmódosítások után fut

Ez a sorrend azért fontos, mert a kontrollok hatása ebből következik.
Ugyanaz a beállítás más eredményt ad attól függően, hogy a pipeline melyik
pontján lép be.

------------------------------------------------------------------------

## 9. Preview modell

### 9.1 A preview nem egyetlen jelentésű

A jobb oldali preview a `_right_view_mode` és az aktív feldolgozott ág
szerint épül fel.

Ez a preview nem minden esetben a végső gravírozás közvetlen 1:1 képe.
Bináris dither és depth mód esetén inkább a feldolgozási szerkezetet és a
raszter jellegét mutatja meg.

Gyakorlati jelentése:

- lehet nincs preview
- lehet feldolgozott BASE nézet
- depth módban lehet grayscale vagy dither branch

### 9.2 Nearest preview

Ez csak a megjelenítés interpolációját változtatja. A feldolgozott képet
nem módosítja.

### 9.3 Fullscreen

A fullscreen ugyanabból az aktív preview-ágából dolgozik, mint a normál
jobb oldali nézet, csak nagyobb és részletesebb ellenőrzésre alkalmas.

------------------------------------------------------------------------

## 10. Auto ajánló

### 10.1 Mi alapján dolgozik

Az ajánlás bemenete:

- material key
- technique key
- user module watt

Az ajánló az adatbázisból származó rekordokat aggregálja.

### 10.2 Nem lineáris egyképletű rendszer

Az Auto nem csak egy „power arányos speed” képlet.

A logika két jellegű ajánlást kombinál:

1.  rekord-medián alapú modul-ajánlás
2.  energia-proxy alapú ajánlás

A kettőből végső kevert ajánlás készülhet.

### 10.3 Fallback

A fallback nem tetszőleges hierarchikus felmászás.

A rendszer:

- pontos anyagtalálatot keres
- ha nincs, meghatározott safe-stop prefixekre eshet vissza
- nincs általános top-level family fallback

Ez meghatározza a fallback határait.

### 10.4 UI viselkedés

Auto módban az ajánlott Speed és Max power egy bázisértékhez kötött.
Ha a felhasználó az egyiket átírja, a másik ehhez igazodva módosul.

Az ajánlás akkor tekinthető a legerősebbnek, ha exact találatból származik.
Safe-stop fallback esetén védettebb kiindulást ad, de kevésbé
közvetlenül az adott anyagra vonatkozik.

Az ajánló az adatbázisban szereplő rekordokra támaszkodik.
Ezek a rekordok nem csak előre definiáltak: a felhasználó saját adatokat is hozzáadhat.

A rögzített adatok exportálhatók, és más környezetben visszatölthetők.
Az importált kulcsfájlok a központilag előkészített adatbázist frissítik.

------------------------------------------------------------------------

## 11. Overscan

Az overscan képlete:

    overscan ≈ 1,15 × v² / (2a)

ahol:

- `v` a scan sebesség mm/s-ban
- `a` az adott tengely gyorsulása mm/s²-ban

Az 1,15 szorzó egy biztonsági tartalék.

Az export szempontjából három állapot van:

- off
- auto
- manual

Manual esetén a felhasználó által megadott érték írja felül az automatikus
számítást.

------------------------------------------------------------------------

## 12. G-code stratégia

### 12.1 Grayscale és Hybrid

Itt a G-code generátor a pixelértéket folytonos teljesítményre képezi:

    power = s_min + (s_max - s_min) × tone

Ezért a `min_power` ténylegesen a tónusleképezés alsó határa.

### 12.2 Bináris dither

Bináris módoknál a fekete pixel fixen a felső teljesítményszintet kapja,
a fehér pixel nullát.

Ilyenkor a `min_power` nem a bináris kiértékelés fő vezérlőeleme.

### 12.3 Depth export

Depth módban a program:

1.  elkészíti az első passzt a dither branch-ből
2.  elkészíti a második passzt a grayscale branch-ből
3.  a két G-code blokkot egymás után fűzi

A második passz átveszi az első passz végállapotának bizonyos export
paramétereit is, ezért ez nem két teljesen független külön export.

------------------------------------------------------------------------

## 13. Frame / Keret

A frame export nem csak egy kapcsoló.

Ha a keret aktív:

- a program külön paraméterablakot nyit
- külön sebesség, teljesítmény és pass count adható meg
- a mentett G-code mellett külön frame fájl is készül

Kör cropnál a frame körív alapú körpálya, egyébként téglalap.

------------------------------------------------------------------------

## 14. Mentés és visszatöltés

### 14.1 Save image

Az aktuálisan aktív feldolgozott képet menti.

Depth módban ez azt jelenti, hogy nem elvont „BASE” kerül mentésre, hanem
az a branch, amely éppen aktív.

### 14.2 Save db

A mentés sidecar alapú workspace-állapotot is eltárol.

Tipikusan bekerül:

- image referencia
- machine mode és machine profile
- geometriai mezők
- material / technique kiválasztás
- base control
- gcode control
- auto control
- crop állapot
- depth állapot és branch kontrollok

### 14.3 Reload

A visszatöltés ténylegesen rekonstruálja a workspace-t, majd újrafuttatja
a feldolgozást.

Ezért a mentés-visszatöltés modellje munkamenet-jellegű, nem csak
adatbázisrekord-jellegű.

------------------------------------------------------------------------

## 15. Sender röviden, workshop szemmel

A Sender szerepe nem csak a nyers streamelés.

Releváns mai funkciók:

- port és kapcsolatkezelés
- line / byte / auto stream mód
- pause / resume / stop / e-stop
- alarm unlock
- jog
- marker lézer
- külön frame futtatás
- machine start és work start pozíciókezelés
- homing-aware indulópont kezelés

Ez a user szinthez képest annyiban fontos, hogy a frame és a pozicionálás
a Senderben is többállapotú folyamat.

------------------------------------------------------------------------

## 16. Helyes mentális modell összefoglalása

Röviden:

- a RAW és a preview nem ugyanaz
- a BASE nem mindig bináris
- a preview nem mindig a végső gravírozás egyetlen alakja
- a `rotate_90` valódi geometriai művelet
- a preview-visszaforgatás csak nézet
- a crop RAW térben él
- az Auto ajánló adatbázis-alapú és safe-stop fallbackos
- a G-code stratégia a módtól függ
- depth módban két branch és két passz van
- a mentés workspace-állapotot is hordoz

Ez a LaserBase működésének használható, helyes képe.
