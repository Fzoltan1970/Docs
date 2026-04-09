# Első lépések

A tényleges munka az **Image Workspace** ablakban indul.
A feldolgozás általában meglévő kép betöltésével indul.
Lehetőség van arra is, hogy a felhasználó a **Sketch** ablakban készítsen rajzot, majd azt átküldje a képfeldolgozóba.
A Sketch az indító EXE-ből érhető el, rajz készíthető benne, a kész rajz pedig átküldhető a képfeldolgozóba.
A főablak hasznos, de nem ez a kötelező kezdőlépés.

## 1. Kép betöltése

Nyisd meg az Image Workspace-t, és töltsd be a képet.
Innen indul minden további lépés.
Kép nélkül nincs feldolgozás.

Ezután állítsd be a géphez tartozó alap környezetet.

## 2. Machine mode és profile

Először válaszd ki a **machine mode**-ot.
**Diode** módban saját gépprofilt használsz.
**Fiber** módban ez nem ugyanilyen blokkoló lépés.

Diode módban adj meg vagy tölts be **machine profile**-t.
A profile a gép viselkedését írja le.
Erre épül a helyes feldolgozás és a helyes G-code.
Ha ezt kihagyod, diode módban nem lesz érvényes feldolgozás.

Nem kell a firmware értékeit fejből tudnod.
A program USB kapcsolaton keresztül tud kommunikálni a géppel, és ki tudja olvasni a kiinduló gépadatokat.
Ha ez sikeres, a profile alapjai automatikusan kitöltődnek.

A profile része a **modul érték** is.
Ha több különböző modulod van, érdemes külön profile-t menteni hozzájuk.
A modul nem csak adat: a rendszer később ezt is figyelembe veszi.

A profile-t mentsd el, amint használható.
Ezzel a későbbi munkáknál nem kell mindent újra beállítanod.
A mentett profile később visszatölthető.

Ha megvan a machine mode és a profile, lépj tovább a geometriára.

## 3. Geometria

Add meg együtt a következőket:
- **width**
- **height**
- **DPI**
- **scan axis**

Ezek együtt határozzák meg a rasztert és a tényleges gravírozási méretet.
Ha ezek közül valamelyik hiányzik vagy rossz, az eredmény sem lesz helyes.

Ezután készítsd elő magát a képet.

## 4. Kép előkészítése

Ha nem a teljes képet használod, állíts be **crop**-ot.
Ez opcionális.

Ha a képet valóban el kell forgatni, használd a **Rotate 90** lépést.
Ez a feldolgozásra is hat.

Ha csak más nézetben akarod látni a képet, használd a preview forgatását.
Ez csak megjelenítési réteg.

Most jön a legfontosabb döntés.

## 5. Feldolgozási mód

Válassz **feldolgozási módot**.
Ez határozza meg a végeredményt és a G-code viselkedését is.

**Grayscale** akkor jó, ha folytonosabb tónust akarsz.
**Dither** akkor jó, ha bináris pontképpel akarsz dolgozni.
**Hybrid** akkor hasznos, ha a grayscale túl sima, a dither túl kemény.
**Depth** akkor való, ha két külön feldolgozott ággal akarsz dolgozni.

Ezután ellenőrizd a preview-t.

## 6. Preview és Process

A preview ellenőrzésre való.
Nem minden módban a végső gravírozás közvetlen 1:1 képe.

Ha a beállítások rendben vannak, indítsd el a **Process** lépést.
Itt készül el a feldolgozott kép.
Ez lesz az export alapja.

Ezután mentsd el a vezérlőfájlt.

## 7. Export

Mentsd el a **G-code**-ot.
Ez még nem futtatás.

Az **overscan** export oldali beállítás.
Ezzel azt szabályozod, hogyan fusson ki a gép a kép szélein.

A soreltolás / line shift a profilhoz kötött korrekció.
Ez nem kezdő lépés, és nem külön képfeldolgozási döntés.

Ha elkészült a G-code, lépj át a futtatásra.

## 8. Futtatás

A mentett G-code a **Sender** ablakban fut.
Itt történik a tényleges futtatás.

Nyisd meg a Sender-t.
Válaszd ki a portot, majd csatlakozz a géphez.
Ha a port nem jelenik meg, ellenőrizd a kábelezést és hogy a gép csatlakoztatva van.

Töltsd be az előbb elmentett G-code-ot.
Állítsd be a pozíciót.

Ha kell, futtass külön **frame**-et.
Ez opcionális, és pozicionálás előtt hasznos.

Ezután indítsd el a munkát.
Szükség esetén használhatod a Pause, Resume és Stop vezérlést.

## Megjegyzés

Háromféle mentést érdemes megkülönböztetni.

A **profile mentés** a gép újrahasználatához kell.
Ezzel ugyanazt a gépet vagy modult később gyorsan vissza tudod tölteni.

A **workspace mentés** a teljes aktuális állapotot menti.
Ebben benne van a konkrét kép és a feldolgozási beállítások is.
Ezt akkor használd, ha ugyanarra a munkára akarsz később visszatérni.

A **DB mentés** ennél szűkebb.
Ez inkább rekord- és paraméterszintű mentés.
Nem ugyanaz, mint a teljes workspace állapot megőrzése.

Korábbi munkát a főablakból tudsz visszatölteni.
A részletes működést a Tudástár tartalmazza.
