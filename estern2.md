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
