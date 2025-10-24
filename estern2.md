Ziel: Dein Mac ist voll. Er droht abzustürzen. Wir müssen jetzt Platz freimachen, die bereits geretteten Dateien sichern und die Wiederherstellung korrekt neu starten. Du bekommst eine klare Schritt-für-Schritt-Anleitung mit den tatsächlichen Pfaden aus deinen Screenshots.

Wichtige Fakten aus deinen Bildern:

* Benutzername: `paminagrewe-remberg`
* Bisheriges Wiederherstellungsziel: `/Users/paminagrewe-remberg/recup_dir`
* Quellgerät: `/dev/rdisk7` (USB-Stick, 128 GB)
* Der Mac hat fast 0 Speicher frei. Fehlermeldung „Dein System hat keinen Programmspeicher mehr.“
* TestDisk wollte ein Image `image.dd` erzeugen und hat die Platte gefüllt.
* Ergebnis: System ist kurz vorm Absturz.

Wir gehen jetzt so vor:

1. Rettung anhalten.
2. Speicher freiräumen.
3. Externe Festplatte einbinden.
4. Gerettete Daten von der internen SSD auf die externe Festplatte verschieben.
5. Symlink setzen, damit PhotoRec weiterarbeiten kann, aber auf extern schreibt.
6. Speicher prüfen.
7. Recovery korrekt neu starten.

Ich schreibe dir jeden Schritt. Nach jedem Schritt führst du den angegebenen Befehl in der Konsole aus. Immer Enter drücken nach dem Befehl.

---

1. Schritt: Offene Recovery stoppen

---

Falls PhotoRec oder TestDisk noch laufen und Text im Terminal hochscrollt oder ein Menü offen ist:
Drücke gleichzeitig die Tasten `ctrl` und `c`.

Das stoppt den Prozess.
Wenn PhotoRec ein Menü zeigt und unten eine Auswahl wie `[Stop]`, kannst du auch mit den Pfeiltasten auf „Stop“ gehen und Enter drücken.

Kein weiterer Befehl hier.

---

2. Schritt: Die große angefangene Image-Datei löschen

---

In deinen Bildern lag eine Datei `image.dd` auf dem Schreibtisch (Desktop).
Diese Datei ist unvollständig und belegt viel Speicher. Die brauchen wir nicht.

Gib ein:

```bash
rm -i ~/Desktop/image.dd
```

Der Mac fragt dich dann `remove ...?`.
Gib `y` ein.
Drück Enter.

Damit löschst du das halbfertige Abbild, das TestDisk angelegt hat.

---

3. Schritt: Externe Festplatte anschließen

---

Stecke jetzt ein externes Laufwerk an den Mac (USB-Festplatte oder großer USB-Stick).
Wichtig: Das Laufwerk muss mehr freien Speicher haben als die internen 53 GiB, besser deutlich mehr (128 GiB oder mehr ist sinnvoll).

Wenn das Laufwerk eingesteckt ist, gib ein:

```bash
ls /Volumes
```

Du bekommst eine Liste.
Da steht immer `Macintosh HD` (das ist intern) und zusätzlich der Name deiner externen Festplatte. Beispiel: `Backup` oder `UNTITLED`.
Du brauchst diesen Namen gleich. Ich nenne ihn im Rest der Anleitung `NAME`. Du ersetzt `NAME` durch den echten Namen aus der Liste.

Kein weiterer Befehl hier. Du merkst dir nur den Namen.

---

4. Schritt: Zielordner auf externer Festplatte anlegen

---

Wir erstellen dort einen Ordner für die geretteten Dateien.

```bash
mkdir -p "/Volumes/NAME/recovery"
```

Wichtig: `"NAME"` wirklich ersetzen durch den angezeigten Namen des externen Laufwerks aus Schritt 3.
Die Anführungszeichen unbedingt mit eingeben. Die verhindern Probleme bei Leerzeichen.

Beispiel falls das Laufwerk „Backup Pamina“ heißt:

```bash
mkdir -p "/Volumes/Backup Pamina/recovery"
```

---

5. Schritt: Finde deine bisherigen Wiederherstellungsdaten

---

PhotoRec hat laut Screenshot nach `/Users/paminagrewe-remberg/recup_dir` geschrieben.
Wir prüfen jetzt, wo diese Ordner genau liegen und wie sie heißen.

Gib ein:

```bash
find /Users/paminagrewe-remberg -maxdepth 2 -type d -iname "recup_dir*"
```

Erwartung: Du siehst einen oder mehrere Pfade wie
`/Users/paminagrewe-remberg/recup_dir`
oder
`/Users/paminagrewe-remberg/recup_dir.1`

Diese Ordner enthalten deine bereits geretteten Fotos, Videos, Texte.

Kein weiterer Befehl hier. Du merkst dir die Pfade.

---

6. Schritt: Diese geretteten Daten auf die externe Festplatte verschieben

---

Beispiel: Angenommen `find` hat dir genau diesen Ordner gezeigt:

`/Users/paminagrewe-remberg/recup_dir`

Dann verschiebst du ihn jetzt rüber auf die externe Platte in den `recovery` Ordner:

```bash
mv "/Users/paminagrewe-remberg/recup_dir" "/Volumes/NAME/recovery/recup_dir"
```

Wenn `find` zusätzlich noch so etwas gefunden hat wie
`/Users/paminagrewe-remberg/recup_dir.1`
dann verschiebst du den auch:

```bash
mv "/Users/paminagrewe-remberg/recup_dir.1" "/Volumes/NAME/recovery/recup_dir.1"
```

Diese `mv` Befehle geben dir sofort wieder viel freien Speicher auf der internen SSD, weil du nichts kopierst, sondern verschiebst.

---

7. Schritt: Verknüpfung zurück anlegen (Symlink)

---

Warum das wichtig ist:
PhotoRec versucht beim nächsten Lauf wieder nach `/Users/paminagrewe-remberg/recup_dir` zu schreiben.
Wir legen jetzt eine Art „Alias“ an. Dieser Alias zeigt intern auf den Ordner, der in Wahrheit auf der externen Festplatte liegt.
Das heißt: Ab jetzt landet jede neue gerettete Datei direkt auf der externen Festplatte.

Gib ein:

```bash
ln -s "/Volumes/NAME/recovery/recup_dir" "/Users/paminagrewe-remberg/recup_dir"
```

Hinweis: Wenn der Befehl meckert, dass `/Users/paminagrewe-remberg/recup_dir` schon existiert, dann wurde der Ordner vorher nicht verschoben. In dem Fall musst du Schritt 6 zuerst richtig ausführen, so dass `recup_dir` weg ist, bevor du den Link setzt.

---

8. Schritt: Prüfen ob jetzt wieder Speicher frei ist

---

Gib ein:

```bash
df -h ~
```

Wichtig: In der Spalte `Avail` solltest du wieder mehrere Gigabyte frei sehen.
Wenn da wieder nur Megabyte frei stehen (z. B. „20Mi“ oder „300Mi“), dann ist dein interner Speicher immer noch zu voll.
Dann musst du weitere große Ordner von deinem Benutzer-Ordner auf die externe Platte verschieben.

Liste die größten Ordner in deinem Benutzerverzeichnis sortiert nach Größe:

```bash
du -sh /Users/paminagrewe-remberg/* | sort -h
```

Wenn du hier z. B. einen Ordner siehst wie `Downloads` mit vielen GB, kannst du den ebenfalls verschieben:

Beispiel:

```bash
mv "/Users/paminagrewe-remberg/Downloads" "/Volumes/NAME/recovery/Downloads"
```

Danach wieder testen:

```bash
df -h ~
```

Zielwert: mindestens 10 GiB frei, besser 20 GiB frei, bevor du weitermachst.

---

9. Schritt: PhotoRec korrekt neu starten

---

Ab jetzt wird direkt auf die externe Festplatte gespeichert, aber PhotoRec denkt, er schreibt nach `recup_dir` im Benutzerordner.
Das ist korrekt so. Genau das wollten wir.

Starte PhotoRec wieder:

```bash
sudo photorec /dev/rdisk7
```

Du bist wieder im PhotoRec-Menü.

Jetzt die Einstellungen machen, damit nicht unnötig Müll gezogen wird und damit der Speicher nicht sofort wieder vollläuft:

1. Wähle das Laufwerk `/dev/rdisk7`.
2. Wähle „Whole disk“ oder die APFS-Partition.
3. Gehe in `File Opt`.
4. Schalte nur diese Dateitypen ein:

   * `jpg`, `jpeg`, `png`, `heic` (Fotos)
   * `mov` (Videos vom iPhone oder Kamera)
   * `txt`, `pdf` (Dokumente/Notizen)
     Alle anderen Typen abwählen. Du brauchst z. B. keine `mp3`, keine `DS_Store`, keinen Apple-Metadaten-Schrott.
5. Gehe auf `Options`.
6. Stelle `Keep corrupted files` auf `No`.
7. Wähle `Search`.
8. Wähle `Other`.
9. Wenn PhotoRec dich nach dem Ziel fragt, zeige ihm den Pfad:
   `/Users/paminagrewe-remberg/recup_dir`
   Dieser Pfad zeigt jetzt in Wirklichkeit auf `/Volumes/NAME/recovery/recup_dir`, also direkt externe Festplatte.

Dann starten lassen.

---

10. Schritt: Während der Recovery wach bleiben

---

Das hast du richtig gemacht. Weiter so.

Offenes Terminal-Fenster auflassen mit:

```bash
caffeinate -dimsu
```

Solange dieses Fenster offen bleibt und der Befehl läuft, geht der Mac nicht schlafen.

---

11. Was du nach der Recovery machst

---

Wenn PhotoRec fertig ist, kannst du die gefundenen Dateien im Finder prüfen:

* Öffne den Finder.
* Wechsle zu deiner externen Festplatte (`NAME`).
* Öffne den Ordner `recovery`.
* Darin liegen Ordner wie `recup_dir`, `recup_dir.1`, `recup_dir.2`, usw.
* Diese enthalten die geretteten Fotos, Videos, Texte.

---

## Wichtige Dinge, die du NICHT tun darfst

Du darfst den USB-Stick `/dev/rdisk7` nicht formatieren.
Du darfst deinen Mac jetzt nicht gewaltsam ausschalten solange die Platte 0 Byte frei hat (Crash-Risiko, Datenverlust). Erst Speicher freimachen, dann kannst du normal neu starten.
Du darfst TestDisk nicht mehr „Image Creation“ auf interne SSD machen, weil die SSD nicht groß genug ist. (Das hat dir `image.dd` erzeugt und das System fast gekillt.)

---

## Zusammenfassung in einem Satz

Du verschiebst jetzt den bisherigen Ordner `/Users/paminagrewe-remberg/recup_dir` auf ein externes Laufwerk unter `/Volumes/NAME/recovery/`, setzt dann mit `ln -s` einen Link zurück, prüfst mit `df -h ~`, und startest `sudo photorec /dev/rdisk7` neu mit reduziertem Dateitypenset, damit nur wirklich wichtige Dateien weitergerettet werden.
