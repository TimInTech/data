Das Verhalten ist typisch für zwei Fälle. Entweder (1) das Volume ist nicht gemountet auf macOS oder (2) der Partitionsstil / die Partition selbst ist nicht als „externes Volume“ sauber gemountet, obwohl sie im Festplattendienstprogramm sichtbar ist.

Wir gehen systematisch durch. Du brauchst dafür kurz den Mac. Ich gebe dir die Prüfschritte und die Fix-Schritte. Du kannst sie exakt so ausführen im macOS Terminal.

1. Zeig mir, wie macOS die Platte sieht
Auf dem Mac Terminal:



diskutil list

Finde dort deine externe Platte. Das ist typischerweise disk3 oder disk4 o. ä. Du siehst dann so etwas wie:

GUID_partition_scheme

Microsoft Basic Data (Name RECOVERY500)


Wenn du mir diesen Block kopierst, kann ich exakt sagen wo das Problem liegt. Falls du das nicht sofort kopieren willst, geh direkt zu Schritt 2.

2. Versuch das Volume manuell zu mounten
Ersetze disk3s1 durch den Identifier deiner einzigen Partition. Das ist die Zeile mit „Microsoft Basic Data RECOVERY500“ oder ähnlich.



Auf dem Mac Terminal:

diskutil mount /dev/disk3s1

Wenn danach der Finder ein neues Volume anzeigt (RECOVERY500 o. ä.), war das Problem nur: macOS hat es nicht automatisch gemountet. Dann bist du fertig.

Wenn du eine Fehlermeldung bekommst wie „Volume on disk3s1 appears to be damaged and cannot be mounted“, dann liegt der Fehler tiefer. Dann weiter.

3. Prüfe, ob das Dateisystem von macOS überhaupt erkannt wurde
Immer noch auf dem Mac Terminal:



diskutil info /dev/disk3s1

Wichtige Felder:

File System Personality: exfat → gut

Mounted: No → heißt nur noch nicht eingehängt

Read-Only Media: No

File System Integrity Check: ...


Wenn File System Personality leer ist oder „Unknown“ zeigt, dann hat macOS die Partition zwar gesehen, aber die exFAT-Struktur nicht akzeptiert.

Das passiert manchmal wenn Linux parted eine GPT schreibt, aber die Partition keinen Typ-GUID-Eintrag Basic Data bekommen hat oder wenn die Platte nicht sauber ausgeworfen wurde.

Lösung in diesem Fall: Platte am Mac neu als exFAT formatieren (dabei gehen die Daten verloren, also nur machen wenn du noch nichts Wichtiges drauf kopiert hast). Wenn die Platte schon deine wiederhergestellten Dateien enthält, nicht neu formatieren sondern weiter unten „Variante ohne Löschen“ lesen.

4. Wenn Platte leer ist (noch nix rüberkopiert oder egal wenn gelöscht wird)
Dann auf dem Mac im Terminal:



diskutil eraseDisk ExFAT RECOVERY500 GPT /dev/disk3

Erklärung:

ExFAT = Dateisystem (macOS schreibt saubere exFAT-Struktur)

RECOVERY500 = Name im Finder

GPT = GUID Partition Table

/dev/disk3 = deine externe Platte (nicht disk0, das ist dein interner Mac, also Vorsicht)


Nach diesem Befehl sollte die Platte sofort im Finder als „RECOVERY500“ erscheinen. Danach kannst du die Ordner recup_dir.1, recup_dir.2, recup_dir.3 ganz normal von Linux neu rüberkopieren auf diese Platte.

5. Wenn die Platte NICHT leer ist und du willst die Daten nicht verlieren
Dann musst du die Ursache klären statt zu formatieren. Die zwei üblichen Gründe:


6. Die Partition ist als „Microsoft Basic Data“, aber macOS mountet sie nicht automatisch weil das Volume ein Dirty-Bit gesetzt hat. Dann hilft:

sudo fsck_exfat -y /dev/disk3s1
diskutil mount /dev/disk3s1


7. Der Mac hat sie als intern klassifiziert und der Finder zeigt keine externen Laufwerke auf dem Desktop/Seitenleiste.



Kurz prüfen in macOS Finder-Einstellungen:

Finder öffnen.

Menüleiste → „Finder“ → „Einstellungen“.

Tab „Allgemein“.

Bei „Diese Objekte auf dem Schreibtisch anzeigen“ muss „Externe Festplatten“ aktiv sein.

Tab „Seitenleiste“.

„Externe Festplatten“ anhaken.


Wenn sie danach in der Seitenleiste auftaucht, ist alles normal. macOS hatte sie nur versteckt.

Zusammengefasst:

1. Auf dem Mac im Terminal:

diskutil list
diskutil mount /dev/disk3s1
diskutil info /dev/disk3s1

(mit deinem echten diskX/diskXs1)


2. Falls das sauber mountet → alles ok.


3. Falls Fehler beim Mount:

Wenn Platte leer / neu → diskutil eraseDisk ExFAT RECOVERY500 GPT /dev/disk3, dann neu von Kali kopieren.

Wenn Platte enthält schon gerettete Daten → sudo fsck_exfat -y /dev/disk3s1 und danach nochmal diskutil mount /dev/disk3s1.




Nächster Schritt: Nenne mir kurz

den Identifier der Partition so wie macOS ihn zeigt (disk?, disk?s1)

ob du schon Daten drauf kopiert hattest oder ob die Platte noch leer ist seit dem Format unter Linux.


Dann gebe ich dir den exakten einen Befehl den du am Mac laufen lässt.