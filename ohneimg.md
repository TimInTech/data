Kurz: Du kannst direkt vom Stick retten. Risiko ist höher als mit Image, aber ok. So gehst du auf dem **Mac** weiter. Am Ende steht eine Kurzvariante für Linux.

## macOS: direkte Rettung ohne Image

**0) Grundregeln**

* Nichts auf den Stick schreiben.
* Zielordner auf **interner SSD** oder besser **externer Platte**.
* Mac wach halten.

**1) Mac wach halten**

```
caffeinate -dimsu
```

**2) Quellgerät prüfen**

```
diskutil list
```

**3) Alle Partitionen des Sticks aushängen**
(Beispiel `disk7`; verwende deine Nummer)

```
diskutil unmountDisk /dev/disk7
```

**4) Zielordner anlegen (intern)**

```
mkdir -p ~/recup_dir
```

**Alternative Zielordner (extern, empfohlen)**
(erst Laufwerksnamen prüfen, dann Ordner erstellen)

```
ls /Volumes
```

```
mkdir -p "/Volumes/NAME/recup_dir"
```

**5) PhotoRec direkt auf dem Rohgerät starten**
(du nutzt bereits `rdisk7`, das ist gut und schnell)

```
sudo photorec /dev/rdisk7
```

**6) PhotoRec-Einstellungen**

* **File Opt**: nur benötigte Typen aktivieren, z. B. `jpg`, `jpeg`, `png`, `heic`, `mov`, `txt`, `pdf`.
* **Options**: „Keep corrupted files“ = **No**.
* **Search**: **Other** wählen.
* **Destination**: `~/recup_dir` oder `"/Volumes/NAME/recup_dir"`.

**7) Speicher überwachen**
(Freiheit intern)

```
df -h ~
```

(Größe des Recovery-Ordners)

```
du -sh ~/recup_dir
```

(Extern prüfen)

```
df -h "/Volumes/NAME"
```

**8) Falls Platz knapp wird**
(PhotoRec stoppen, dann Ziel umziehen)

```
mv ~/recup_dir "/Volumes/NAME/recup_dir"
```

```
ln -s "/Volumes/NAME/recup_dir" ~/recup_dir
```

(Danach PhotoRec wieder starten und denselben Zielpfad wählen – es schreibt nun auf die externe Platte.)

**9) Nach Abschluss prüfen**

```
open ~/recup_dir
```

```
find ~/recup_dir -type f | wc -l
```

**10) Optional: strukturierte Kopien mit TestDisk direkt vom Stick**
(nur Lesen, keine Writes)

```
sudo testdisk /dev/rdisk7
```

Analyse → Quick Search → Partition wählen → **p** Dateien ansehen → **C** kopieren → Zielordner wählen.

**11) Typische Probleme**

* „No such file or directory“ bei `du`: Pfad exakt und in Anführungszeichen schreiben.
* Fortschritt langsam: Dateitypen enger wählen.
* Abbruch: PhotoRec erneut starten, denselben Zielordner wählen, es setzt fort.

---

## Linux (Kubuntu/Deepin): Kurzvariante ohne Image

**Wach halten**

```
systemd-inhibit --what=handle-lid-switch:sleep --why="recovery" bash
```

**Gerät finden**

```
lsblk -o NAME,MODEL,SIZE,FSTYPE,MOUNTPOINT
```

**Partitionen aushängen**
(Beispiel `sdb1`, `sdb2`)

```
sudo umount -f /dev/sdb1
```

```
sudo umount -f /dev/sdb2
```

**Zielordner**

```
mkdir -p ~/recup_dir
```

**PhotoRec**

```
sudo photorec /dev/sdb
```

**Speicher prüfen**

```
df -h ~
```

```
du -sh ~/recup_dir
```

**Optional TestDisk**

```
sudo testdisk /dev/sdb
```

Wenn du mir die exakte Gerätebezeichnung und den gewünschten Zielpfad nennst, gebe ich dir die minimalen PhotoRec-Dateitypen zum Aktivieren.
