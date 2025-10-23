
1. Terminal öffnen
   Spotlight öffnen (⌘+Leertaste) → „Terminal“ → Enter.

2. Homebrew prüfen oder installieren

```bash
brew --version || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Danach Brew ins PATH übernehmen, wenn am Ende eine Anweisung erscheint, diese exakt ausführen.
```

3. Tools installieren

```bash
brew install ddrescue testdisk
```

4. USB-Stick finden und aushängen

```bash
diskutil list
# Den richtigen Eintrag mit Größe/Name erkennen, z. B. /dev/disk2
diskutil unmountDisk /dev/disk2
```

5. 1:1-Image des Sticks erstellen (immer zuerst klonen)

```bash
sudo ddrescue -n /dev/disk2 ~/Desktop/usb.img ~/Desktop/usb.log
sudo ddrescue -r3 /dev/disk2 ~/Desktop/usb.img ~/Desktop/usb.log
```

6. Ordner/Dateinamen retten versuchen (TestDisk auf dem Image)

```bash
testdisk ~/Desktop/usb.img
```

Navigation:

* „Analyse“ → „Quick Search“.
* Gefundene Partition(en) mit Pfeiltasten wählen → **p** drückt Inhalte anzeigen.
* Mit **C** markierte Dateien/Ordner in einen Zielordner kopieren.
* Nichts auf den Original-Stick schreiben. „Write“ nur auf dem Image verwenden, und nur wenn sicher.

7. Inhalte roh wiederherstellen (PhotoRec auf dem Image)

```bash
photorec ~/Desktop/usb.img
```

Empfohlene Einstellungen:

* **File Opt**: nur benötigte Typen aktivieren, z. B. `jpg`, `jpeg`, `png`, `heic`, `pdf`, `docx`, `txt`.
* **Options**: „Keep corrupted files“ = No.
* **Search**: Dateisystemtyp meist **Other** (für FAT/exFAT).
* Zielordner: interne SSD (nicht der Stick).

8. Optional: Dateisystemtyp des Images prüfen

```bash
hdiutil attach -nomount ~/Desktop/usb.img
# Ausgabe zeigt z. B. /dev/disk3
diskutil info /dev/disk3
```

Nur lesen, nichts reparieren auf dem Original.

Wichtige Regeln:

* Keine Schreibzugriffe auf den Stick. Immer auf dem **Image** arbeiten.
* Erst TestDisk für Struktur/Dateinamen, dann PhotoRec für alles, was fehlt.
* Ziel der Wiederherstellung nie der Stick selbst.

