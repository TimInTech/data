Ziel: Die externe Festplatte soll auf dem Mac normal im Finder erscheinen, ohne Linux. Du bekommst eine klare Schritt-für-Schritt-Anleitung, die eine nicht-technische Person am Mac alleine ausführen kann. Sie deckt beide Fälle ab: Platte ist leer oder Platte enthält schon Daten.

Ich nenne die Festplatte hier „RECOVERY500“. Wenn bei dir ein anderer Name angezeigt wird, ist das egal.

---

1. Festplatte anschließen
   Schließe die externe Festplatte per USB am Mac an.
   Wenn ein Fenster kommt mit
   „Das angeschlossene Medium konnte von diesem Computer nicht gelesen werden“
   dann klicke auf „Ignorieren“. Nicht „Auswerfen“.

---

2. Terminal öffnen
   Öffne das Terminal auf dem Mac:
   Klicke unten im Dock auf das Launchpad (graues Rakettsymbol)
   Suche nach „Terminal“
   Starte Terminal.

Wir arbeiten jetzt nur noch in diesem Terminal.

---

3. Prüfen, wie die Festplatte heißt
   Gib im Terminal ein:

```bash
diskutil list
```

Drücke Enter.

Der Mac zeigt jetzt alle Speichergeräte. Du siehst mehrere Blöcke:

* `disk0` ist fast immer die eingebaute Mac-SSD. Finger weg.
* Weiter unten findest du etwas wie `disk2` oder `disk3` oder `disk4` mit Größe ungefähr 500 GB. Das ist deine externe Festplatte.
  In diesem Block steht oben z. B. `GUID_partition_scheme`.
  Darunter steht eine Zeile mit etwas wie `disk3s1` und Typ `Microsoft Basic Data` oder `Windows_NTFS` oder `Linux Filesystem` oder eventuell `Untitled`.

Wichtig ist:

* der Gerätename der ganzen Platte, z. B. `/dev/disk3`
* der Gerätename der Partition, z. B. `/dev/disk3s1`

Schreibe dir diese zwei Bezeichnungen genau auf (disk3 / disk3s1). Wir brauchen die gleich.

Wenn du unsicher bist, nimm die Zeile mit der größten Größe (z. B. 500.1 GB) die NICHT disk0 ist.

---

4. Versuch, die Platte manuell einzuhängen (mounten)
   Jetzt versuchen wir, ob macOS das Volume einfach nicht automatisch eingebunden hat.

Im Terminal (pass den Namen an, falls deine Platte anders heißt als `disk3s1`):

```bash
diskutil mount /dev/disk3s1
```

Drücke Enter.

Erwartetes Ergebnis:

* Wenn keine Fehlermeldung kommt:
  Finder öffnen. Links in der Seitenleiste unter „Orte“ sollte jetzt ein Eintrag erscheinen, z. B. „RECOVERY500“.
  Dann bist du fertig. Die Platte ist gemountet und du kannst Dateien darauf kopieren.

* Wenn eine Fehlermeldung kommt wie „Volume on disk3s1 could not be mounted“ oder „Filesystem not recognized“ gehe zu Schritt 5.

---

5. Prüfen, was macOS von der Partition hält
   Wir fragen jetzt den Mac ab, ob er das Dateisystem erkennt. Das sagt uns, ob die Platte neu formatiert werden muss.

Im Terminal:

```bash
diskutil info /dev/disk3s1
```

Drücke Enter.

Du bekommst viele Zeilen. Die wichtigen Felder:

* `File System Personality:`

  * Wenn hier `exfat` steht, dann versteht macOS das Dateisystem grundsätzlich.
  * Wenn hier sowas steht wie `Unknown` oder es steht gar nichts sinnvolles da, dann wurde das Volume von Linux so angelegt, dass macOS damit nichts anfangen kann.

* `Mounted:`

  * Wenn hier `Yes`, bist du fertig.
  * Wenn hier `No`, hat macOS es erkannt, aber nicht eingehängt.

Jetzt zwei Fälle:

Fall A: File System Personality ist `exfat`, aber `Mounted: No`
Das bedeutet: Das Volume ist eigentlich ok, aber wurde nicht sauber ausgeworfen oder hat ein „schmutziges Bit“. Wir reparieren es kurz und mounten es dann.

Gib ein:

```bash
sudo fsck_exfat -y /dev/disk3s1
diskutil mount /dev/disk3s1
```

Hinweis: Nach `sudo fsck_exfat` fragt macOS eventuell nach deinem Mac-Login-Passwort. Das ist normal.

Wenn dieser Mount klappt: Finder öffnen. Die Festplatte sollte jetzt sichtbar sein. Fertig.

Fall B: File System Personality ist leer oder Unknown
Das heißt: macOS erkennt das Dateisystem nicht als gültiges exFAT. Dann muss die Platte am Mac neu formatiert werden, damit sie in Zukunft zuverlässig erkannt wird. Danach funktioniert sie dauerhaft mit macOS und iPadOS. Das ist der sichere Weg. Achtung: dabei gehen alle Daten weg, die auf der Platte drauf sind.

Wenn die Platte im Moment leer ist bzw. du einverstanden bist, dass sie neu beschrieben wird, mach Schritt 6.

Wenn auf der Platte schon wichtige wiederhergestellte Daten liegen, halte sofort an und sag einem technisch erfahrenen Menschen, dass vorher kopiert werden muss. Nicht weitermachen.

---

6. Platte am Mac neu formatieren (ExFAT mit GUID)
   Dieser Schritt macht die Festplatte 100 % Mac- und iPad-kompatibel. Danach erscheint sie sofort im Finder.

Wichtig: Nimm den Gerätenamen der gesamten Platte, also ohne „s1“. Beispiel: `/dev/disk3`, nicht `/dev/disk3s1`.

Im Terminal:

```bash
diskutil eraseDisk ExFAT RECOVERY500 GPT /dev/disk3
```

Erklärung für den Menschen am Mac:

* `eraseDisk` = löscht alles auf dieser externen Platte
* `ExFAT` = plattformunabhängiges Dateisystem, von macOS, iPadOS und Windows les- und schreibbar
* `RECOVERY500` = der Name, wie die Platte dann im Finder heißt
* `GPT` = moderne Partitionsstruktur (GUID Partition Table)
* `/dev/disk3` = deine Platte. Nicht vertippen. Nicht `disk0` nehmen, das wäre die interne Mac-Platte.

Nach diesem Befehl:

* Der Finder sollte sofort automatisch ein neues Laufwerk `RECOVERY500` anzeigen.
* Das Laufwerk ist leer und einsatzbereit.
* Ab jetzt kann man ganz normal Dateien rüberziehen wie bei einem USB-Stick.

---

7. Sichtbarkeit in Finder aktivieren (Kontrollpunkt für absolute Anfänger)
   Falls die Platte da ist, aber man sie nicht sieht:

8. Öffne Finder.

9. In der Menüleiste oben: „Finder“ anklicken → „Einstellungen“.

10. Tab „Allgemein“.

11. Dort anhaken: „Externe Festplatten“.

12. Dann Tab „Seitenleiste“.

13. Dort auch „Externe Festplatten“ anhaken.

Danach taucht sie links in der Seitenleiste unter „Orte“ auf.

---

8. Was du am Ende erwarten sollst
   Nach Schritt 4 oder Schritt 6 soll im Finder links ein Eintrag sein wie „RECOVERY500“.
   Wenn du den anklickst, öffnet sich ein komplett leeres Fenster. Ab jetzt kannst du Ordner wie `recup_dir.1`, `recup_dir.2`, `recup_dir.3` einfach per Drag & Drop vom anderen Rechner (oder über Netzwerkfreigabe) rüberkopieren.

---

Kurzfassung für die Person am Mac, ohne Erklärtext:

1. Terminal öffnen.
2. Prüfen welche Disk die externe ist:

```bash
diskutil list
```

Merke dir `/dev/diskX` und `/dev/diskXs1`.

3. Mount versuchen:

```bash
diskutil mount /dev/diskXs1
```

4. Wenn Fehler:

```bash
diskutil info /dev/diskXs1
```

* Wenn `File System Personality: exfat`:

```bash
sudo fsck_exfat -y /dev/diskXs1
diskutil mount /dev/diskXs1
```

* Wenn nicht exFAT erkannt ODER Platte darf gelöscht werden:

```bash
diskutil eraseDisk ExFAT RECOVERY500 GPT /dev/diskX
```

5. Finder öffnen. Externe Festplatten in den Finder-Einstellungen sichtbar schalten.

---

Das ist der komplette Ablauf. Danach ist die Platte les- und schreibbar auf macOS und iPadOS und kann normal für die geretteten Daten genutzt werden.
