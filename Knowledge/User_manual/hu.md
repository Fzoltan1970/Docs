# LaserBase -- Felhasználói kézikönyv

Ez a kézikönyv a LaserBase működését írja le. A cél nem az,
hogy minden belső részletet kimerítsen, hanem az, hogy a programot
helyesen lehessen használni, és közben világos legyen, mi történik a kép
és a G-code között.

A szöveg a valós munkafolyamatot követi.

------------------------------------------------------------------------

# 1. Mi a LaserBase

A LaserBase lézergravírozó munkák előkészítésére és futtatására készült
programcsomag.

Fő részei:

- **Főablak** -- paraméteradatbázis és rekordkezelés
- **Image Workspace** -- képfeldolgozás, preview, G-code export
- **Sender** -- a G-code elküldése a gépre
- **Sketch** -- egyszerű rajzoló és vázlatkészítő felület

A LaserBase nem általános képszerkesztő. A célja az, hogy egy képből
gravírozható, géphez illesztett raszter és vezérlőfájl készüljön.

------------------------------------------------------------------------

# 2. A gravírozás alapja

A lézergravírozás eredményét három tényező együtt határozza meg:

1.  a lézer teljesítménye
2.  a haladási sebesség
3.  a pont- vagy sorsűrűség

A LaserBase ezért nem csak képet dolgoz fel, hanem a képhez fizikai
méretet, DPI-t, gépgeometriát és G-code paramétereket is rendel.

------------------------------------------------------------------------

# 3. A teljes munkafolyamat

Egy tipikus munka:

1.  kép betöltése
2.  méret, DPI és gépadatok megadása
3.  crop és geometriai beállítás
4.  képfeldolgozási mód kiválasztása
5.  preview ellenőrzése
6.  sebesség és teljesítmény beállítása vagy automatikus ajánlása
7.  G-code vagy feldolgozott kép mentése
8.  szükség esetén rekord mentése és későbbi visszatöltése

------------------------------------------------------------------------

# 4. Image Workspace

Az Image Workspace a program központi része.

Bal oldalon a forráskép látszik, jobb oldalon a feldolgozott nézet.
Ez a jobb oldali nézet nem mindig ugyanazt jelenti: a kiválasztott
módtól függően lehet szürkeárnyalatos, bináris ditheres vagy depth
módnál többféle feldolgozott ág egyike.

Ezért a preview-t mindig a választott feldolgozási móddal együtt kell
értelmezni.

------------------------------------------------------------------------

# 5. Kép betöltése

A **Load image** gombbal tölthető be a kép.

Támogatott formátumok:

- PNG
- JPG / JPEG
- BMP

A kép betöltése után a program eltárolja a forrásképet, és ebből indul
minden további feldolgozás.

------------------------------------------------------------------------

# 6. RAW kép, feldolgozott kép és BASE

A program több képszintet használ.

**RAW kép**

Ez a betöltött forráskép. A crop és a valódi forgatás ehhez a képhez
kapcsolódik.

**Feldolgozott kép**

Ez az a raszter, amely már a megadott fizikai mérethez, DPI-hez és
gépgeometriához igazodik, és tartalmazza a képi transzformációkat is.

**BASE**

A BASE nem mindig bináris.

- **Grayscale** módban a BASE szürkeárnyalatos raszter
- **Hybrid** módban a BASE szürkeárnyalatos, mintázott tónuskép
- bináris dither módokban a BASE fekete-fehér pontkép

A G-code mindig az aktuális feldolgozott raszterből készül.

------------------------------------------------------------------------

# 7. Méret és DPI

A gravírozás méretét milliméterben adjuk meg.

A DPI meghatározza a sor- vagy pontsűrűséget:

    vonalköz (mm) = 25,4 / DPI

A megadott DPI nem mindig azonos a gépen ténylegesen használható DPI-vel.
A LaserBase a gép léptetéséhez igazított valódi rasztert készít, ezért a
feldolgozott kép mérete és effektív DPI-je eltérhet a pusztán kért
értékektől.

------------------------------------------------------------------------

# 8. Machine profile

A Machine profile a gép fizikai adatait tartalmazza:

- tengelyenkénti steps/mm
- max rate
- acceleration
- lézermodul
- G-code profiladatok

Ezek nem csak az exporthoz kellenek. A program ezekből számolja a
raszter illesztését és az automatikus overscan értéket is.

Fiber módban a workspace virtuális gépprofilt használ; diode módban a
gépadatok ténylegesen részei a feldolgozásnak.

------------------------------------------------------------------------

# 9. Crop

A crop a RAW kép terében van definiálva. Ez fontos.

Ez azt jelenti, hogy a kivágás nem a már feldolgozott képen történik,
hanem a forrásképen, még a további feldolgozás előtt.

Elérhető alakok:

- négyzet / téglalap
- kör

Kör cropnál a szélesség és magasság azonos kell legyen. Ha a crop aktív,
de érvénytelen, a Process gomb nem indul.

------------------------------------------------------------------------

# 10. Valódi forgatás és preview-visszaforgatás

A két forgatás nem ugyanaz.

**Rotate 90**

Ez valódi geometriai transzformáció. A forráskép orientációját
megváltoztatja, és a crop, a geometria, valamint a feldolgozás is ehhez
igazodik.

**Preview visszaforgatás**

Ez csak megjelenítési réteg. A feldolgozást és a G-code-ot nem módosítja,
csak a bal és jobb oldali preview nézetet forgatja vissza.

Ha a képet a gép miatt el kell forgatni, a **Rotate 90** a fontos.
Ha csak kényelmesebben szeretnéd nézni, a preview-visszaforgatás való.

------------------------------------------------------------------------

# 11. Feldolgozási módok

A feldolgozási mód a workspace központi döntése.

Nem csak azt határozza meg, hogy milyen algoritmus fusson, hanem azt is,
hogy milyen típusú lesz a BASE, mit mutat a preview, és hogyan készül a
G-code.

A LaserBase nem csak bináris ditheringgel dolgozik.

**Grayscale**

A feldolgozott raszter szürkeárnyalatos marad. A G-code pixelről pixelre
PWM értéket rendel a tónushoz.

**Hybrid**

Szürkeárnyalatos alapú mód, amely a tónusokat mintázottabban kezeli.
Nem klasszikus bináris dither, és nem tiszta grayscale.

A Hybrid ott hasznos, ahol a tiszta grayscale túl laposnak, a bináris
dither pedig túl keménynek bizonyul. Köztes megoldást ad a tónusmező és a
pontszerkezet között.

**Bináris dither módok**

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Ezekben a végső raszter bináris: a pont vagy ég, vagy nem ég.

**Depth**

Ez nem külön dither algoritmus, hanem kétágú feldolgozás.

Depth módban a program külön kezel egy grayscale és egy dither ágat, és a
preview-ben ezek között lehet váltani. Exportnál a G-code két passzból
épül fel.

A Depth akkor hasznos, ha a tónusátvitelt és a pontstruktúrát nem
ugyanarra az egyetlen raszterre akarod bízni. A sima grayscale-hez képest
két külön funkciójú feldolgozott ágat ad.

------------------------------------------------------------------------

# 12. A képi kontrollok

**Contrast, Brightness, Gamma, Radius, Amount**

Ezek a feldolgozás bemenetét alakítják.

**Negative**

A tónusok inverziója.

**Mirror X / Mirror Y**

Geometriai tükrözés a képen.

**Threshold**

Nem általános képi csúszka. A bináris error-diffusion módok küszöbére hat.

**Serpentine scan**

Hibaterjesztéses dither módoknál minden második sor feldolgozási irányát
megfordítja. Nem külön dither, hanem kiegészítő kapcsoló.

**1 pixel off**

Bináris módoknál egy utólagos tisztító lépés, amely izolált egyedi
pontokat tud eltüntetni.

------------------------------------------------------------------------

# 13. Auto ajánló

Az **Auto** nem egyszerű arányszámoló.

Az ajánlás az adatbázisból dolgozik:

- technika
- anyag
- lézermodul

alapján próbál Speed / Max power / Min power értéket adni.

Ha nincs pontos találat, a program fallbackot használ. Nem
lép fel minden felső szintre; csak a támogatott safe-stop csoportokra
eshet vissza. Ha ilyen fallback történik, a program ezt jelzi, és
megerősítést kérhet.

Auto módban a Speed és a Max power összekapcsolva mozog: ha az egyiket
kézzel módosítod, a másikat a rendszer az aktuális ajánlás bázisához
igazítja.

Az Auto akkor a legerősebb, ha pontos anyag- és technikaalapú találatra
támaszkodik. Fallback esetén inkább biztonságos kiindulásnak tekinthető,
mint közvetlen végértéknek.

Az ajánló az adatbázisban szereplő rekordokra támaszkodik.
Ezek a rekordok nem csak előre definiáltak: a felhasználó saját adatokat is hozzáadhat.

A rögzített adatok exportálhatók, és más környezetben visszatölthetők.
Az importált kulcsfájlok a központilag előkészített adatbázist frissítik.

------------------------------------------------------------------------

# 14. Overscan

Az overscan az a ráfutási kifutási távolság, amelyen a fej a kép
szélén kívül mozog.

Automatikus módban a LaserBase ezt a sebességből és a tengely
gyorsulásából számolja. Kézi override is megadható.

Ha az overscan túl kicsi, a kép széle torzulhat, mert a gép még nem
állandó sebességen mozog.

------------------------------------------------------------------------

# 15. Processing és döntési lépés

A **Process** nem egyszerűen csak lefuttat egy szűrőt.

Előtte a program megvizsgálja:

- a kívánt méretet
- a kért DPI-t
- a gép léptetési geometriáját
- a forráskép tényleges felbontását

Ennek eredménye:

- **BASE** -- a feldolgozás közvetlenül végrehajtható
- **REPAIR** -- a kívánt raszterhez képkondicionálás / újramintázás kell

Ezután készül el a tényleges feldolgozott raszter.

------------------------------------------------------------------------

# 16. Preview

A jobb oldali preview a kiválasztott feldolgozott ágat mutatja.

Ez a preview nem minden módban a végső gravírozás közvetlen 1:1 képe.
Különösen bináris dither és depth mód esetén inkább a raszterlogikát
mutatja meg.

Depth módban két preview között lehet váltani:

- Grayscale
- Dither

A **Nearest preview** a megjelenítés módját változtatja, nem magát a
feldolgozást.

Fullscreen módban érdemes ellenőrizni:

- tónusátmenetek
- csíkosodás
- túl erős élesítés
- a bináris pontkép sűrűségét
- depth módban a két ág közti különbséget

------------------------------------------------------------------------

# 17. G-code generálás

A G-code stratégia módfüggő.

**Grayscale / Hybrid**

A program a pixel tónusából számít PWM értéket, ezért a lézer teljesítménye
soron belül is változhat.

**Bináris dither**

A fekete pontok a beállított maximális teljesítménnyel égnek, a fehér
pontok nem égnek.

**Depth**

Két egymásra épített passz készül: a dither és a grayscale ág külön
G-code blokkot kap.

------------------------------------------------------------------------

# 18. Save image, G-code és Keret

**Save image**

Az aktuálisan aktív feldolgozott képet menti. Ez lehet grayscale, hybrid
vagy dither ág is.

**G-code**

A gravírozási vezérlőfájlt menti.

**Keret**

Ha be van kapcsolva, a program külön keretfájlt is készít. Ennek
paraméterei külön ablakban adhatók meg. Kör cropnál kör, egyébként
téglalap keret készül.

------------------------------------------------------------------------

# 19. Mentés az adatbázisba

A főablak paraméteradatbázisként működik, de a workspace
mentése ennél több.

Mentéskor a program eltárolja:

- a forráskép hivatkozását
- a gép- és geometriai állapotot
- a crop állapotát
- a base és G-code kontrollokat
- az auto állapotot
- depth módban a két ág beállításait

Ezért a visszatöltés nem csak rekordkitöltés, hanem workspace-állapot
helyreállítása.

------------------------------------------------------------------------

# 20. Visszatöltés

A mentett workspace rekord visszatölthető a főablakból.

Ilyenkor a program:

1.  visszatölti a forrásképet
2.  visszaállítja a méretet, DPI-t és gépadatokat
3.  visszaállítja a cropot és a feldolgozási kontrollokat
4.  újrafuttatja a feldolgozást

Ezért a visszatöltött állapot munkamenet-jellegű, nem egyszerű
paraméterlista.

------------------------------------------------------------------------

# 21. Sender

A Sender külön ablakban fut.

Alapfunkciók:

1.  port kiválasztása
2.  kapcsolódás
3.  G-code betöltése
4.  küldés indítása

Működés közben:

- Pause / Resume
- STOP
- E-STOP
- $X Unlock alarm esetén
- kézi jog mozgatás
- marker lézer
- terminálparancsok

Emellett a Sender tud külön frame futtatást, gépi kezdőpont és
work-start pozíció mentést, valamint offsetes pozicionálást is.

------------------------------------------------------------------------

# 22. Főablak, New Entry, Calculator, Sketch

**Főablak**

Tárolja és kezeli a rekordokat.

**New Entry**

Új rekord létrehozása.

**Calculator**

Meglévő rekordokból segít arányos új paramétert számolni.

**Sketch**

Egyszerű külön ablakos rajzoló és vázlatkészítő felület.

------------------------------------------------------------------------

# 23. Gyakori hibák

- **Nincs kép** -- előbb tölts be forrásképet
- **Nincs gépprofil diode módban** -- add meg vagy töltsd be
- **Érvénytelen crop** -- javítsd vagy kapcsold ki
- **Kör crop és eltérő méret** -- a szélesség és magasság legyen azonos
- **Nincs auto ajánlás** -- nincs megfelelő adat ehhez az anyag / technika / modul kombinációhoz
- **Nem menthető G-code** -- a feldolgozásnak előbb sikeresen le kell futnia

------------------------------------------------------------------------

# 24. Hasznos gyakorlat

- Új anyagnál először grayscale és bináris dither módban is nézd meg a preview-t.
- Ha a gép miatt kell elforgatni a képet, a valódi Rotate 90-et használd.
- Ha csak nézni szeretnéd más orientációban, a preview-visszaforgatást használd.
- Auto ajánlónál mindig figyeld, hogy pontos vagy fallback ajánlást kaptál-e.
- Depth módot akkor érdemes használni, ha a kétféle raszter szerepét külön akarod kezelni.
- A Keret exportot használd pozicionálás előtt.
- A workspace mentést kezeld munkamentésként, ne csak rekordként.

------------------------------------------------------------------------

# 25. Gyors referencia

    vonalköz (mm) = 25,4 / DPI
    effektív DPI = 25,4 / valódi pitch_mm

Módok:

- Grayscale → tónusalapú PWM raszter
- Hybrid → szürkeárnyalatos, mintázott raszter
- Dither → bináris pontkép
- Depth → kétágú feldolgozás és kétpasszos export

Forgatás:

- Rotate 90 → valódi geometriai változás
- Preview visszaforgatás → csak megjelenítés

Mentés:

- Save image → aktuális feldolgozott ág
- Save db → workspace-állapot + rekord
- G-code → vezérlőfájl, opcionális keretfájllal
