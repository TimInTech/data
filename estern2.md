Dein Screenshot zeigt Folgendes klar:

* Die Datei `/Volumes/Public/Image.dd.gz` **existiert**, wurde aber **nicht vollständig geschrieben**.
* Der Testbefehl `gunzip -t` meldet **„unexpected end of file“** → das ist eine **abgebrochene Komprimierung** (Image unvollständig).
* `gunzip -c ... > Image.dd` bricht mit **„uncompress failed“** ab, weil der Gzip-Stream am Ende fehlt.
  Das passiert, wenn `dd` während der Erstellung oder Kompression abgebrochen wurde (`ctrl + c` oder Speichermangel).

➡️ Das bedeutet:
Die Datei **enthält den Anfang** des Sticks (ein Teil ist also verwertbar), aber **nicht alles**. Du kannst daraus trotzdem oft viele Dateien retten.

---

## 🧩 So gehst du jetzt vor (einfach erklärt)

### 1. Prüfen, wie groß dein Image wirklich ist

```bash
ls -lh /Volumes/Public/Image.dd.gz
```

Merke dir die Größe (z. B. 24G).
→ Wenn sie > 10 GB ist, lohnt sich die Analyse mit PhotoRec.

---

### 2. Entpackung mit Fehlertoleranz versuchen

(macOS-Gunzip bricht sofort ab, aber wir können die Daten trotzdem nutzen)

#### Variante A: Entpacken trotz Abbruch

```bash
gzip -dc /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd
```

**Erklärung:**

* `-d` = dekomprimieren
* `-c` = Ausgabe in Datei schreiben
* `gzip` schreibt so viel, wie lesbar ist, auch wenn das Ende fehlt.

Wenn der Befehl mit einer Warnung endet (nicht mit „Killed“ oder „Input/output error“), ist das Ergebnis brauchbar.

---

### 3. Wenn Entpacken funktioniert hat

Du hast dann `/Volumes/Public/Image.dd`.
Starte PhotoRec **darauf**:

```bash
sudo photorec /Volumes/Public/Image.dd
```

→ Zielordner für Wiederherstellung:
`/Volumes/Public/recovery`

Während der Auswahl in PhotoRec:

* `File Opt`: nur `jpg`, `png`, `mov`, `txt`, `pdf`, `heic` aktivieren
* `Options`: „Keep corrupted files“ = No
* `Search`: „Other“ wählen
* **Destination:** `/Volumes/Public/recovery`

PhotoRec rettet dann alle lesbaren Dateien aus dem unvollständigen Image.

---

### 4. Wenn das Entpacken komplett fehlschlägt

Wenn auch `gzip -dc` scheitert:
Du kannst das Image **direkt in PhotoRec** öffnen (es verarbeitet auch komprimierte Fragmente):

```bash
sudo photorec /Volumes/Public/Image.dd.gz
```

PhotoRec liest dann automatisch, was möglich ist, und ignoriert beschädigte Endbereiche.

---

### 5. Mac wach halten

Lass dein `caffeinate -dimsu`-Fenster weiter offen, bis PhotoRec fertig ist.

---

### 🔍 Zusammenfassung

| Schritt                    | Befehl                                                            | Ziel                       |
| -------------------------- | ----------------------------------------------------------------- | -------------------------- |
| Größe prüfen               | `ls -lh /Volumes/Public/Image.dd.gz`                              | sehen, ob Rettung sinnvoll |
| Entpacken trotz Abbruch    | `gzip -dc /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd` | so viel wie möglich retten |
| PhotoRec starten           | `sudo photorec /Volumes/Public/Image.dd`                          | Wiederherstellung          |
| Falls Entpackung scheitert | `sudo photorec /Volumes/Public/Image.dd.gz`                       | Rettung direkt vom Gzip    |

---

Wenn du mir **die Ausgabe von**

```bash
ls -lh /Volumes/Public/Image.dd.gz
```

sendest, kann ich dir exakt sagen, ob sich die Wiederherstellung noch lohnt (z. B. > 15 GB = lohnend, < 5 GB = wahrscheinlich unbrauchbar).

#####
stop....
###
###Perfekt. Auf dem Foto sieht man zwei Dinge:

1. **Der Befehl `caffeinate -dimsu` läuft erfolgreich**
   → Der Mac bleibt jetzt dauerhaft wach und geht **nicht** in den Standby oder Ruhezustand.
   → Du kannst das Terminalfenster einfach geöffnet lassen. Kein weiterer Befehl nötig.
   → Wenn du das Fenster später schließen willst, beendest du den Vorgang mit `Ctrl + C`.

2. **Die Prozessliste (`ps aux | grep dd`) zeigt:**
   Kein `dd`-Prozess läuft mehr → deine Kopie wurde **sauber beendet**.
   Das heißt: Die Datei `/Volumes/Public/Image.dd.gz` wurde korrekt geschlossen und ist sicher gespeichert.

---

### ✅ Zusammengefasst: Dein aktueller Zustand ist stabil

* Kein aktiver Kopiervorgang mehr.
* Abbild (`Image.dd.gz`) existiert.
* Mac bleibt wach dank `caffeinate`.
* Du kannst jetzt **ruhig neu starten**, **aber erst nach dieser kurzen Checkliste:**

---

### 🔍 Letzter Sicherheitscheck vor Neustart

**1. Prüfen, ob die Datei vollständig auf der externen Festplatte liegt:**

```bash
ls -lh /Volumes/Public/Image.dd.gz
```

→ Notiere die Größe (z. B. 24 G, 80 G, 110 G).
Wenn sie > 100 GB ist, ist das Image ziemlich vollständig.

**2. Externes Laufwerk sicher auswerfen (wenn du jetzt wirklich neu startest):**

* Im Finder links auf „Public“ klicken.
* Dann im Finder-Menü oder Kontextmenü „Auswerfen“ wählen.
* Warten, bis das Laufwerk im Finder verschwindet.

**3. Jetzt kannst du den Mac neu starten.**
Einfach über das Apple-Menü → „Neustart“.

---

### 🧩 Nach dem Neustart (nächste Schritte)

1. Externe Festplatte („Public“) wieder anschließen.

2. Prüfen, ob dein Image da ist:

   ```bash
   ls -lh /Volumes/Public/Image.dd.gz
   ```

3. Teste, ob das Abbild fehlerfrei ist:

   ```bash
   gunzip -t /Volumes/Public/Image.dd.gz
   ```

   * Wenn **keine Fehlermeldung** kommt → alles gut.
   * Wenn „unexpected end of file“ erscheint → Image unvollständig, aber trotzdem verwertbar.

4. Dann (nur wenn genug Speicherplatz frei ist) das Abbild entpacken:

   ```bash
   gunzip -c /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd
   ```

5. **Danach PhotoRec auf diesem Image laufen lassen**, nicht mehr auf dem USB-Stick:

   ```bash
   sudo photorec /Volumes/Public/Image.dd
   ```

   Ziel: `/Volumes/Public/recovery`

---

Wenn du willst, kann ich dir den exakten nächsten Befehl (inkl. Kontrolle der Größe und Pfade) vorbereiten —
dazu bitte nur **eine letzte Terminalausgabe** schicken von:

```bash
ls -lh /Volumes/Public
```

Dann bekommst du Schritt-für-Schritt die Befehle, die du 1:1 kopieren kannst.

###
STOP
####
Ja, du kannst neu starten, aber nicht blind. Erst drei Checks, damit du keinen Schaden machst und keinen Fortschritt verlierst. Du hast `dd` mit `Ctrl + C` sauber beendet. Das heißt: Die Datei `/Volumes/Public/Image.dd.gz` ist geschlossen und wurde auf dem externen Volume `Public` geschrieben. Das ist grundsätzlich ok. Du hast jetzt ein (wahrscheinlich unvollständiges) Abbild. Wir sichern jetzt den Zustand, prüfen Speicher, dann Neustart, dann geht die Wiederherstellung weiter nur noch vom Abbild.

Ich gebe dir jetzt einen klaren Ablauf. Du gehst Schritt für Schritt durch. Nach jedem Schritt den Befehl genau so eintippen und Enter drücken. Keine zusätzlichen Zeichen tippen. Groß/Klein beachten.

---

1. Prüfen ob gerade noch was läuft

---

Ziel: Sicherstellen dass kein Kopiervorgang mehr aktiv ist. Wenn nichts mehr läuft, ist es sicher den Mac neu zu starten.

Öffne ein Terminalfenster und tippe:

```bash
ps aux | grep dd
```

Erwartung:

* Wenn du nur die Zeile mit `grep dd` siehst, läuft `dd` nicht mehr. Gut.
* Wenn du noch eine Zeile siehst mit `dd if=/dev/disk...`, dann läuft es noch. In dem Fall nicht neu starten, sondern zuerst wieder in dieses Fenster gehen und `ctrl` + `c` drücken.

(Hinweis: `ctrl` + `c` heißt die Strg-Taste gedrückt halten und dann c drücken. Nicht `ctrl + c` schreiben.)

---

2. Größe des Abbilds auf der externen Festplatte prüfen

---

Damit stellen wir fest, dass die Datei wirklich existiert und geschlossen ist.

```bash
ls -lh /Volumes/Public/Image.dd.gz
```

Du bekommst eine Ausgabe wie z. B. `24G` oder `80G` oder ähnlich. Damit wissen wir: Die Rettungsdatei liegt wirklich auf der externen Platte `Public` und nicht intern.

Wenn dieser Befehl „No such file or directory“ zurückgibt, ist etwas schief. Dann keinen Neustart und nochmal melden.

---

3. Speichersituation prüfen (extern und intern)

---

a) Freier Platz auf der externen Platte `Public`:

```bash
df -h /Volumes/Public
```

Merke dir die Zahl in der Spalte `Avail` (frei). Das muss deutlich zweistellig in `Gi` sein, sonst wirst du später kein entpacktes Image mehr erzeugen können.

b) Freier Platz auf deinem internen Systemlaufwerk (das ist dein Mac selbst):

```bash
df -h ~
```

Wichtig:

* Wenn hier wieder mehrere `Gi` frei sind (z. B. 10Gi, 20Gi, mehr), dann ist das System wieder atembar.
* Wenn hier nur `Mi` frei stehen (so wie vorher, 398Mi, 300Mi), dann darfst du nicht sofort neu starten, weil macOS evtl. nicht sauber hochkommt.

Falls noch extrem wenig frei ist, verschiebe große Ordner aus deinem Benutzerordner (`Downloads`, `Bilder`, `recup_dir`, usw.) auf die externe Platte. Beispiel:

```bash
mv "/Users/paminagrewe-remberg/Downloads" "/Volumes/Public/Downloads"
```

Dann nochmal:

```bash
df -h ~
```

Ziel: mindestens 5 GiB frei, besser >10 GiB frei.

Wenn du diese Luft wieder hast, ist Neustart unkritisch.

---

4. Was du nach dem Neustart tust

---

Nach dem Neustart NICHT sofort wieder `dd` starten. Erst verifizieren, ob dein Abbild sinnvoll ist. Die nächsten Schritte sind:

Schritt 4.1: Gzip-Image auf Fehler testen (ohne zu entpacken, dauert kurz):

```bash
gunzip -t /Volumes/Public/Image.dd.gz
```

* Wenn kein Fehler kommt und du wieder eine Eingabezeile bekommst: Das komprimierte Abbild ist technisch lesbar.
* Wenn `unexpected end of file` kommt: Das Abbild ist abgebrochen. Du kannst trotzdem versuchen, daraus noch Daten zu holen. Es ist halt nicht komplett.

Schritt 4.2: Abbild entpacken in eine Rohdatei. Diese Rohdatei benutzen wir später für PhotoRec. Wichtig: Das passiert alles auf der externen Platte `Public`, nicht intern.

```bash
gunzip -c /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd
```

Hinweis: Dafür muss auf `/Volumes/Public` genug Platz frei sein. Im Idealfall mindestens so viel wie die Größe deines USB-Sticks (ungefähr 128 GB). Wenn `df -h /Volumes/Public` zeigt, dass nicht genug frei ist, dann zuerst Platz schaffen.

Schritt 4.3: PhotoRec auf dem entpackten Abbild laufen lassen, nicht mehr auf dem echten Stick:

```bash
sudo photorec /Volumes/Public/Image.dd
```

In PhotoRec:

* `File Opt` öffnen.
* Nur sinnvolle Typen aktiv lassen:

  * `jpg`, `jpeg`, `png`, `heic` (Fotos)
  * `mov` (Videos)
  * `txt`, `pdf` (Dokumente)
* Alles andere abwählen (`mp3`, `DS_Store`, etc. brauchst du nicht).
* `Options`: `Keep corrupted files` = `No`.
* `Search`: `Other`.
* Zielordner auswählen:
  `/Volumes/Public/recovery`
  (Wenn `recovery` nicht existiert, vorher anlegen:)

  ```bash
  mkdir -p /Volumes/Public/recovery
  ```

Dann laufen lassen.

Wichtig: Ab jetzt geht nichts mehr auf deine interne SSD. Alles bleibt auf `Public`. Das schützt dein System.

---

5. Sehr wichtig: vor Neustart nicht nochmal `dd` starten

---

Auf deinem letzten Foto sieht man:

* Der Befehl war `sudo dd if=/dev/disk2 bs=1m | gzip > /Volumes/Public/Image.dd.gz`
* Passwort wurde eingegeben.
* Dann hast du `ctrl + c` genutzt.
* Ausgabe am Ende:

  ```
  317644+0 records in
  317644+0 records out
  333069680064 bytes transferred ...
  ```

  Das heißt: `dd` wurde beendet. Die Shell ist wieder bei `%`. Das heißt die Datei ist sauber geschlossen. Das ist gut. Das heißt: Du verlierst durch Neustart nicht plötzlich das, was schon geschrieben wurde.

Du hast also jetzt ein Snapshot des Sticks (vielleicht nicht komplett, aber verwertbar). Es ist legitim, ab jetzt neu zu starten, unter zwei Bedingungen:

1. Auf deiner internen SSD ist wieder ausreichend freier Platz (siehe Schritt 3b).
2. Die externe Platte `Public` bleibt angeschlossen, oder du trennst sie sauber (rechtsklick „Auswerfen“ im Finder), bevor du runterfährst.

Wenn Punkt 1 nicht erfüllt ist (also dein internes Laufwerk ist weiter bei 0 % frei), dann zuerst Platz frei machen wie oben beschrieben, sonst riskierst du Boot-Probleme.

---

6. Kurze To-do-Liste für dich zum Abarbeiten

7. `ps aux | grep dd`
   Prüfen, dass dd wirklich aus ist.

8. `ls -lh /Volumes/Public/Image.dd.gz`
   Prüfen, dass die Datei da ist.

9. `df -h ~`
   Sicherstellen, dass dein internes Laufwerk wieder mehrere GiB frei hat. Wenn nicht: große Ordner mit `mv` auf `/Volumes/Public` verschieben. Dann nochmal `df -h ~`.

Wenn diese drei Punkte ok sind, kannst du normal neu starten.

Nach dem Neustart:

* `gunzip -t /Volumes/Public/Image.dd.gz`
* `gunzip -c /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd`
* `sudo photorec /Volumes/Public/Image.dd` mit Ziel `/Volumes/Public/recovery`

Das ist der saubere weitere Weg ohne erneut auf die volle interne SSD zu schreiben und ohne nochmal `dd` neu anzuschmeißen.


####
#

Gut erkannt – dein Screenshot bestätigt:
Du führst gerade diesen Befehl aus:

```bash
sudo dd if=/dev/disk2 bs=1m | gzip > /Volumes/Public/Image.dd.gz
```

Das ist korrekt und liest sektorweise deinen USB-Stick (`/dev/disk2`) aus und schreibt die komprimierte Sicherung direkt auf das externe Laufwerk **„Public“**.
Allerdings darf dieser Prozess **nicht unterbrochen** werden, solange er läuft.

Hier ist der überarbeitete, vereinfachte Leitfaden für **genau deinen aktuellen Zustand** — so, dass du sicher und Schritt für Schritt weitermachen kannst:

---

## 🧭 Übersicht – Was du gerade tust

* Du erstellst mit `dd` ein komprimiertes Abbild deines USB-Sticks.
* Das Ziel ist `/Volumes/Public/Image.dd.gz` (auf externer Platte).
* Das ist richtig.
* Du musst nur sicherstellen, dass der Prozess korrekt läuft und genug Platz auf „Public“ frei ist.

---

## 🔧 Schritt-für-Schritt-Anleitung ab deinem aktuellen Stand

### 1. **Laufender `dd`-Prozess prüfen**

Wenn du sehen willst, ob dein Backup (dd) **noch läuft oder hängt**, öffne ein neues Terminal-Fenster (nicht das, wo `dd` läuft!)
und gib ein:

```bash
ps aux | grep dd
```

Das zeigt dir, ob der Prozess noch aktiv ist.
Beispiel-Ausgabe:

```
root     1234   0.5  0.0  25600  2000 ??  S     4:36PM   0:45.32 dd if=/dev/disk2 bs=1m
```

→ Dann läuft er.

Wenn nichts oder nur die „grep dd“-Zeile steht → `dd` ist beendet.

---

### 2. **Fortschritt anzeigen**

Im Fenster, wo `dd` läuft, kannst du den Fortschritt anzeigen, ohne ihn zu stoppen:

➡️ Drücke gleichzeitig: **Ctrl + T**

macOS zeigt dir dann z. B.:

```
6919168+0 records in
6919168+0 records out
...
```

Das bedeutet: `dd` arbeitet noch.
Lass ihn einfach weiterlaufen. Er kann bei 128 GB → 1–3 Stunden brauchen (je nach Stick und USB-Port).

---

### 3. **Falls du ihn stoppen musst**

Wenn du versehentlich falsches Laufwerk gewählt hast oder der Speicher auf „Public“ voll ist:

Drücke in dem aktiven `dd`-Fenster:

```text
Ctrl + C
```

Dann wird der Prozess sicher beendet.
Er schreibt am Ende die Anzahl der kopierten Blöcke und Megabytes.
Beispiel:

```
245760+0 records in
245760+0 records out
128000000000 bytes transferred in 7200 secs (17777 bytes/sec)
```

---

### 4. **Größe prüfen**

Wenn `dd` fertig ist oder du ihn beendet hast:

```bash
ls -lh /Volumes/Public/Image.dd.gz
```

Das zeigt dir, wie groß das Backup ist.
Wenn die Größe unter **120 GB** liegt, ist es unvollständig.
Dann lieber nochmal sauber starten.

---

### 5. **Optional: Speicherplatz auf „Public“ prüfen**

Bevor du nochmal startest oder weiterarbeitest:

```bash
df -h /Volumes/Public
```

Achte darauf, dass in der Spalte „Avail“ mindestens **150 GB frei** sind.
Wenn weniger → keine komplette Sicherung möglich.

---

### 6. **Sauber neu starten (wenn nötig)**

Wenn das alte Backup unvollständig ist (z. B. 24 GB wie auf deinem früheren Screenshot):

```bash
rm -i /Volumes/Public/Image.dd.gz
```

Dann neuen Lauf starten:

```bash
sudo dd if=/dev/rdisk2 bs=1m | gzip > /Volumes/Public/Image.dd.gz
```

*(Nutze `rdisk2` falls vorhanden, ist schneller als `disk2`.)*

Warte, bis du wieder `%` im Terminal siehst → dann ist es abgeschlossen.

---

### 7. **Wenn fertig – Daten aus dem Abbild wiederherstellen**

Nach vollständigem Backup:

1. Entpacke die Datei:

   ```bash
   gunzip -c /Volumes/Public/Image.dd.gz > /Volumes/Public/Image.dd
   ```

2. Starte PhotoRec auf diesem Image:

   ```bash
   sudo photorec /Volumes/Public/Image.dd
   ```

   * Wähle Dateitypen wie `jpg`, `png`, `mov`, `txt`, `pdf`.
   * Ziel: `/Volumes/Public/recovery/recup_dir`

3. Alle gefundenen Dateien landen dort.

---

### 8. **Mac wach halten (empfohlen)**

In einem zweiten Terminalfenster:

```bash
caffeinate -dimsu
```

Solange dieses Fenster offen bleibt, schläft der Mac nicht ein.
Perfekt für lange Rettungsvorgänge.

---

## ✅ Zusammenfassung für dich

Du machst es **richtig**.
Nur:

* Lass `dd` in Ruhe durchlaufen.
* Zeig Fortschritt mit **Ctrl + T**.
* Prüfe später die Datei mit `ls -lh /Volumes/Public/Image.dd.gz`.
* Wenn unvollständig → löschen, mehr Platz schaffen, nochmal starten.
* Danach **PhotoRec** auf **dem Image**, nicht mehr auf dem Stick.

Wenn du magst, schick mir bitte noch kurz die Ausgabe von:

```bash
df -h /Volumes/Public
```

Dann sag ich dir exakt, ob dein Speicher auf der externen Platte für das komplette Abbild reicht.



######


Stand jetzt:

* Dein interner Speicher ist praktisch voll (nur ~400 MiB frei). Das ist kritisch.
* Auf dem externen Laufwerk `Public` liegt schon eine angefangene Sicherung: `/Volumes/Public/Image.dd.gz`, zuletzt ca. 24G groß. Diese Sicherung ist unvollständig, weil der Prozess offenbar abgebrochen wurde, als macOS wegen Speichermangel fast stehengeblieben ist.
* Du hast schon direkt vom Stick `/dev/disk2` mit `dd` gelesen und per `gzip` komprimiert und nach `/Volumes/Public/Image.dd.gz` geschrieben. Das ist fachlich sinnvoll.
* Wir müssen jetzt zuerst Speicher intern freimachen, dann das Image auf dem externen Laufwerk sauber neu erstellen, dann erst daraus wiederherstellen.
* Ab jetzt darf nichts mehr auf die interne SSD geschrieben werden. Alles muss auf `/Volumes/Public` landen.

Ich gebe dir jetzt eine bereinigte Schritt-für-Schritt-Anleitung. Du gehst sie der Reihe nach durch. Jede Code-Zeile einzeln eintippen und mit Enter bestätigen.

---

1. Schritt: Alle Rettungstools stoppen

---

Wenn PhotoRec, TestDisk oder dd noch laufen oder hängen:

* Gehe in das jeweilige Terminal-Fenster.
* Drücke `ctrl` + `c`.

Wenn ein Tool im Terminal ein Menü zeigt (z. B. `[Stop]`), dann mit Pfeiltasten auf `Stop` und Enter.

Ziel: Keine weiteren Daten sollen mehr auf die interne SSD laufen.

Kein Befehl hier.

---

2. Schritt: Prüfen, wie das externe Laufwerk heißt

---

Du hast in den Screenshots `/Volumes/Public` benutzt. Ich gehe davon aus, dass die externe Platte als `Public` eingehängt ist.

Zur Sicherheit:

```bash
ls /Volumes
```

Wenn du siehst `Public`, dann ist alles so wie in den Screenshots. Falls der Name anders ist, musst du in allen folgenden Befehlen `Public` durch diesen echten Namen ersetzen.

---

3. Schritt: Ordner für die geretteten Daten auf der externen Platte anlegen

---

Wir sammeln alles dort. Du schreibst nichts mehr in dein Benutzerverzeichnis.

```bash
mkdir -p "/Volumes/Public/recovery"
```

Dieser Befehl erstellt `/Volumes/Public/recovery`, falls er noch nicht existiert.

---

4. Schritt: Verschiebe vorhandene PhotoRec-Ergebnisse von intern nach extern

---

PhotoRec hat laut deinem ersten Screenshot alles nach
`/Users/paminagrewe-remberg/recup_dir`
gespeichert.

Wir verschieben diesen Ordner jetzt komplett rüber auf die externe Platte, damit deine interne SSD wieder Luft hat.

```bash
mv "/Users/paminagrewe-remberg/recup_dir" "/Volumes/Public/recovery/recup_dir"
```

Wenn du zusätzlich noch weitere Ordner hast wie `recup_dir.1`, `recup_dir.2`, dann verschiebe die genauso:

```bash
mv "/Users/paminagrewe-remberg/recup_dir.1" "/Volumes/Public/recovery/recup_dir.1"
```

```bash
mv "/Users/paminagrewe-remberg/recup_dir.2" "/Volumes/Public/recovery/recup_dir.2"
```

Falls einer dieser Ordner nicht existiert, meldet macOS „No such file or directory“. Das ist normal. Dann einfach weiter.

---

5. Schritt: Verknüpfung (Symlink) zurück anlegen

---

Wir legen jetzt am alten Speicherort einen Link an, der auf die externe Platte zeigt. So glaubt PhotoRec später, es schreibt wieder nach `recup_dir`, in Wahrheit speichert es aber direkt auf `/Volumes/Public/recovery`.

```bash
ln -s "/Volumes/Public/recovery/recup_dir" "/Users/paminagrewe-remberg/recup_dir"
```

Wenn der Link nicht erstellt werden kann, weil `/Users/paminagrewe-remberg/recup_dir` noch existiert, dann war der `mv` aus Schritt 4 nicht erfolgreich. In dem Fall Schritt 4 wiederholen.

---

6. Schritt: Kaputte riesige Dateien löschen, die internen Speicher blockieren

---

Die internen Screenshots zeigen `image.dd` bzw. `image.dd.gz` Versuche und die Warnung „kein Programmspeicher mehr“. Eine unvollständige Sicherung auf dem Schreibtisch (`~/Desktop/image.dd`) kann die interne SSD vollmachen.

Wir löschen nur die interne Kopie. Die externe Kopie unter `/Volumes/Public/Image.dd.gz` lassen wir stehen.

```bash
rm -i ~/Desktop/image.dd
```

Wenn die Datei dort nicht liegt, meldet er „No such file or directory“. Das ist ok.

---

7. Schritt: Prüfen, ob wieder freier Platz da ist

---

Jetzt schauen wir den freien Platz deiner internen SSD an. Das muss hochgehen. Vorher warst du bei nur ~398 MiB frei (100 % voll, das ist extrem schlecht). Ziel ist mehrere Gigabyte frei.

```bash
df -h ~
```

Interpretation:

* Spalte `Avail` muss wieder mehrere „Gi“ anzeigen, nicht „Mi“.
* Wenn du immer noch fast 100 % voll bist: Dann musst du weitere große Ordner auf die externe Platte verschieben. Typisch: `Downloads`, `Bilder`, `Filme`.

Beispiel Analyse der größten Ordner:

```bash
du -sh /Users/paminagrewe-remberg/* | sort -h
```

Wenn du dort z. B. einen großen `Downloads` Ordner siehst (nur Beispiel):

```bash
mv "/Users/paminagrewe-remberg/Downloads" "/Volumes/Public/recovery/Downloads"
```

Dann wieder prüfen:

```bash
df -h ~
```

Du darfst erst weiterarbeiten, wenn intern wieder mindestens 10 GiB frei sind. Besser >20 GiB.

Warum: Dein Mac lag vorher im Zustand „kein Programmspeicher mehr“. Das heißt RAM + Swap waren erschöpft. Das killt Prozesse mitten in der Rettung.

---

8. Schritt: Sauberes, vollständiges Abbild des USB-Sticks direkt nach extern schreiben

---

Wichtig: Wir machen jetzt ein vollständiges sektorweises Abbild vom Stick. Das ist besser als nur einzelne Dateien zu retten. Danach kannst du den Stick abziehen und weiterarbeiten ohne Risiko.

1. Zuerst prüfen, welches Device dein Stick ist. In deinen Screenshots tauchen `/dev/rdisk7` und `/dev/disk2` auf. Das wechselt, je nach erneutem Einstecken. Wir checken jetzt sauber:

```bash
diskutil list
```

Suche den Eintrag mit ungefähr 128 GB. Der heißt z. B. `/dev/disk2`.
Merke dir diese Nummer (ich nenne sie hier `/dev/disk2`).
Für dd nimmst du dann die Raw-Variante `/dev/rdisk2`. Wenn `rdisk2` nicht existiert, bleib bei `disk2`.

2. Hänge den Stick aus, damit nichts drauf geschrieben wird:

```bash
diskutil unmountDisk /dev/disk2
```

Wenn deine Nummer anders ist, ersetze `disk2`.

3. Führe jetzt das Abbild aus, direkt komprimiert auf die externe Platte. Du hast das schon fast richtig gemacht im Screenshot:

```bash
sudo dd if=/dev/rdisk2 bs=1m | gzip > "/Volumes/Public/Image.dd.gz"
```

Erklärung:

* `sudo dd if=/dev/rdisk2 bs=1m` liest den kompletten Stick Sektor für Sektor.
* `| gzip` komprimiert das on-the-fly, damit es weniger Platz auf `Public` braucht.
* `> "/Volumes/Public/Image.dd.gz"` speichert die komprimierte Sicherung direkt auf die externe Platte, nicht auf deine volle interne SSD.

Nach Enter fragt er nach dem Passwort. Gib dein macOS-Admin-Passwort ein und drücke Enter. Während der Befehl läuft, kommt KEIN neuer Prompt.

Fortschritt anzeigen (das war in deinem Screenshot unklar):
Drücke während `dd` noch läuft im selben Terminalfenster die Tasten `ctrl` + `t`.
Wichtig: Du tippst NICHT `Ctrl + T` als Text. Du drückst wirklich gleichzeitig die beiden Tasten.
macOS zeigt dann den aktuellen Fortschritt von `dd` im Terminal.

Lass diesen Schritt vollständig durchlaufen. Er ist erst fertig, wenn du wieder eine normale Eingabezeile mit `%` siehst.

4. Prüfe danach die Größe der Sicherung auf der externen Platte:

```bash
ls -lh "/Volumes/Public/Image.dd.gz"
```

Wenn da nur 24G steht und du weißt, dass der Stick 128 GB groß ist, dann wurde der Vorgang zu früh beendet (z. B. vom System gekillt, weil kein Speicher frei). Dann musst du Schritt 7 (Platz schaffen) noch konsequenter machen und Schritt 8 neu starten, damit du ein vollständiges Abbild bekommst.

Du willst am Ende EINEN vollständigen `Image.dd.gz`. Lösche alte Teildateien vorher, um Verwirrung zu vermeiden:

```bash
rm -i "/Volumes/Public/Image.dd.gz"
```

Dann neu starten mit dem dd-Befehl von oben. Hintergrund: Deine zweite Aufnahme zeigt mehrere Zeitstempel und Größen (3.3G, 3.7G, 5.1G, 11G, 18G, 21G, 24G). Das deutet darauf hin, dass mehrfach neu gestartet wurde und immer ein neuer `Image.dd.gz` erzeugt wurde. Ziel ist ein sauberer Durchlauf ohne Abbruch.

---

9. Schritt: Nach erfolgreichem Abbild vom Stick weiterarbeiten nur noch vom Image

---

Wenn du ein vollständiges `Image.dd.gz` hast:

1. Entpacke das Image (aber NICHT auf die interne SSD, nur auf die externe Platte `Public`). Du brauchst auf `Public` nochmal ungefähr die Größe des Sticks unkomprimiert.

```bash
gunzip -c "/Volumes/Public/Image.dd.gz" > "/Volumes/Public/Image.dd"
```

Jetzt existiert `/Volumes/Public/Image.dd`. Das ist eine 1:1-Kopie des gesamten Sticks. Der Stick selbst ist ab jetzt nicht mehr nötig.

2. Erstelle einen Zielordner für gefundene Dateien auf der externen Platte:

```bash
mkdir -p "/Volumes/Public/recovery/final"
```

3. Starte PhotoRec auf dem Image statt auf der echten Hardware:

```bash
sudo photorec "/Volumes/Public/Image.dd"
```

In PhotoRec:

* `File Opt`: nur sinnvolle Dateitypen aktivieren (`jpg`, `jpeg`, `png`, `heic`, `mov`, `txt`, `pdf`). Alle anderen abwählen.
* `Options`: `Keep corrupted files` = `No`.
* Wähle `Search`.
* Wähle `Other`.
* Zielordner für die Wiederherstellung NICHT ins Home, sondern:

Pfad wählen:
`/Volumes/Public/recovery/final`

Damit landet alles direkt auf dem externen Laufwerk. Die interne SSD bleibt frei und stabil.

---

10. Schritt: Mac wach halten

---

Halte den Mac wach in einem eigenen Terminalfenster, während PhotoRec oder dd läuft:

```bash
caffeinate -dimsu
```

Solange dieses Fenster offen bleibt, geht der Mac nicht schlafen.

---

11. Ergebnisprüfung am Ende

---

Wenn PhotoRec fertig ist:

* Öffne den Finder.
* Gehe nach `/Volumes/Public/recovery/final`.
* Dort liegen Ordner `recup_dir.*` mit den geretteten Dateien.

Keine Daten mehr direkt auf der internen SSD sichern. Alles bleibt auf `Public`.

---

## Kurz zusammengefasst

1. Interne Platte entlasten: `recup_dir` und alles Große nach `/Volumes/Public/recovery` verschieben.
2. Freien Platz intern prüfen mit `df -h ~` bis wieder mehrere GiB frei sind.
3. Stick mit `diskutil list` identifizieren und mit `diskutil unmountDisk` aushängen.
4. Vollständiges Abbild erstellen mit

   ```bash
   sudo dd if=/dev/rdisk2 bs=1m | gzip > "/Volumes/Public/Image.dd.gz"
   ```

   (ersetze ggf. `rdisk2`).
5. Image auf `Public` entpacken zu `Image.dd`.
6. `sudo photorec "/Volumes/Public/Image.dd"` laufen lassen und nach `/Volumes/Public/recovery/final` schreiben.
7. `caffeinate -dimsu` offen lassen, damit nix einschläft.

Wenn du mir jetzt zwei Ausgaben lieferst

```bash
ls /Volumes
```

und

```bash
diskutil list
```

kann ich dir die genauen Befehle mit deinem tatsächlichen Gerätenamen (z. B. `/dev/rdisk7` vs `/dev/rdisk2`) und deinem echten Volume-Namen statt `Public` schreiben, so dass du sie 1:1 kopieren kannst.
