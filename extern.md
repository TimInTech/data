Dein interner Speicher ist vollgelaufen. df -h zeigt zuletzt nur noch ~20 MiB frei. TestDisk meldet No space left for the file image. Das heißt: Die Wiederherstellung hat den internen Speicher gefüllt. Wenn du so weiterläufst, riskierst du Datenverlust auf dem Mac selbst.

Jetzt musst du zwei Dinge tun: 1) PhotoRec/TestDisk stoppen. 2) Alles, was bisher gerettet wurde, auf ein externes Laufwerk verschieben und Platz schaffen. Danach kannst du weitermachen, aber das Ziel muss extern liegen.

Ich gebe dir jetzt genau die Reihenfolge.

1. Laufende Rettung sauber anhalten
Im Terminal-Fenster mit PhotoRec oder TestDisk:



Wenn PhotoRec läuft: Taste Stop auswählen oder Ctrl + C.

Wenn TestDisk offen ist: q drücken bis du wieder in der Shell bist.


Ziel: Keine weiteren Dateien werden mehr auf die interne Platte geschrieben.

2. Externes Laufwerk anschließen
Nimm eine externe USB-Festplatte oder großen USB-Stick. Wichtig: Er sollte genug freien Speicher haben. Dein Quellstick hat 128 GB. Ideal mindestens 200 GB frei.



Steck ihn an das MacBook.

3. Prüfen, wie das externe Laufwerk heißt
Dieser Befehl listet alle eingehängten Volumes. Du suchst den neuen Namen, z. B. External, UNTITLED, Backup, o. ä.



ls /Volumes

4. Zielordner auf dem externen Laufwerk anlegen
Ersetze NAME durch den tatsächlichen Volume-Namen aus Schritt 3.



mkdir -p "/Volumes/NAME/recovery"

5. Finde den bisherigen Recovery-Ordner auf der internen SSD
Du hast versucht du -sh /Users/paminagrewe-remberg/recup_dir, aber das System sagte No such file or directory. Wir suchen jetzt wirklich, wo PhotoRec gespeichert hat.



find /Users/paminagrewe-remberg -maxdepth 2 -type d -iname "recup_dir*"

Ergebnis könnte z. B. sein: /Users/paminagrewe-remberg/recup_dir oder /Users/paminagrewe-remberg/recup_dir.1 oder ähnliches.

Notiere dir den exakten Pfad.

6. Prüfe Größe dieses Ordners
Ersetze den gefundenen Pfad.



du -sh "/Users/paminagrewe-remberg/recup_dir"

Das bestätigt, dass genau dieser Ordner dir den Speicher vollmacht.

7. Verschiebe die geretteten Daten auf das externe Laufwerk
Achtung: mv verschiebt, nicht kopiert. Du gewinnst damit sofort wieder freien Platz.



mv "/Users/paminagrewe-remberg/recup_dir" "/Volumes/NAME/recovery/recup_dir"

8. Erzeuge einen Symlink zurück an die alte Stelle
Grund: PhotoRec denkt später weiter, es schreibt aber real auf die externe Platte.



ln -s "/Volumes/NAME/recovery/recup_dir" "/Users/paminagrewe-remberg/recup_dir"

Ab jetzt zeigt ~/recup_dir auf die externe Platte.

9. Prüfe freien Speicher nach dem Verschieben
Wenn der Move geklappt hat, sollte wieder viel „Avail“ erscheinen.



df -h ~

Wenn da wieder zig GiB frei stehen: gut. Wenn nicht, dann gibt es noch andere große Ordner. Dann Schritt 10.

10. Größte Speicherfresser im Home-Verzeichnis finden
Das sortiert nach Größe. Gut geeignet um zu sehen, wo noch was liegt.



du -sh /Users/paminagrewe-remberg/* | sort -h

Jeder Ordner, der viele GiB hat (z. B. Downloads, Movies, Pictures, evtl. noch ein zweiter recup_dir oder recup_dir.1) kann genauso mit mv rübergeschoben werden auf /Volumes/NAME/recovery/.

Beispiel:

mv "/Users/paminagrewe-remberg/recup_dir.1" "/Volumes/NAME/recovery/recup_dir.1"

Dann wieder prüfen:

df -h ~

Ziel ist: mindestens 20 GiB wieder frei auf der internen SSD. Darunter wird macOS instabil.

11. Danach PhotoRec erneut starten und weitermachen
Wichtig: Dieses Mal gleich das externe Laufwerk als Ziel angeben, nicht wieder die interne SSD.



sudo photorec /dev/rdisk7

Konfiguration in PhotoRec:

File Opt: nur die sinnvollen Formate aktivieren (z. B. jpg, jpeg, png, heic, mov, txt, pdf). Alles andere abwählen. Weniger Typen = weniger Müll = weniger Speicherbedarf.

Options: „Keep corrupted files“ = No.

Search: „Other“.

Destination: /Volumes/NAME/recovery/recup_dir wählen.


PhotoRec legt dann Unterordner recup_dir.X an, diesmal aber direkt auf der externen Platte.

12. TestDisk-Hinweis aus deinem Screenshot
TestDisk zeigt:
Disk /dev/disk2 - 128 GB - APFS



Das sagt:

Der Rettungsstick ist APFS.

TestDisk hat versucht, ein Image zu erzeugen oder erwartet eines. Es scheitert wegen No space left for the file image. Das ist exakt dein Speicherproblem, nicht ein Bedienfehler.

APFS macht es schwer, die alte Ordnerstruktur komplett inklusive Original-Dateinamen sauber wiederherzustellen. Das bedeutet: PhotoRec ist hier dein Hauptwerkzeug. TestDisk kann manchmal trotzdem einzelne Verzeichnisse listen, aber du hast nicht genug freien Platz zum Anlegen eines kompletten Images. Lösung ist wie oben: Extern anschließen und dort arbeiten.


13. Kontrolle zwischendurch
Nach dem Neustart von PhotoRec kannst du immer wieder prüfen:



Freier Speicher interne SSD:


df -h ~

Freier Speicher auf dem externen Laufwerk:


df -h "/Volumes/NAME"

Wachstum des Zielordners:


du -sh "/Volumes/NAME/recovery/recup_dir"

Interpretation zu deinem aktuellen Zustand:

Die freie Kapazität ist in mehreren df -h Aufrufen von ~32 GiB auf ~20 MiB gefallen. Das heißt, dein Laufwerk ist praktisch voll.

Du bist an der Grenze, wo macOS keine temporären Dateien mehr sauber schreiben kann. Das kann Abstürze, Abbrüche, Datenkorruption und Startprobleme verursachen, wenn du so weiter machst.

Du musst jetzt sofort auslagern, bevor du weiter rettest.


Fazit:

1. Externes Laufwerk anschließen.


2. ls /Volumes, Ordner anlegen.


3. Bisherigen recup_dir verschieben.


4. Symlink setzen.


5. Speicher prüfen.


6. PhotoRec neu starten mit Ziel auf externem Laufwerk mit begrenzten Dateitypen.



Wenn du mir nach Schritt 3 den exakten Volume-Namen aus ls /Volumes und den Output von find /Users/paminagrewe-remberg -maxdepth 2 -type d -iname "recup_dir*" gibst, formuliere ich dir die zwei konkreten mv und ln -s Befehle exakt mit Pfad, damit du sie nur noch kopieren musst.