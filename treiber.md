Hier ist die ergänzte Markdown-Datei. Ich habe die spezifischen Prüfungen für gängige Chipsets (Arduino/USB-Serial) sowie den kompletten Windows-Update-Prozess integriert.

-----

# Hardware-Initialisierung & System-Update Guide

Diese Anleitung dient der vollständigen Vorbereitung eines Windows-Systems sowie der gezielten Treiber-Installation für Hardware-Kombinationen (z.B. Arduino, ESP32, Sensoren).

## 1\. System-Berechtigungen setzen

Führe die PowerShell als **Administrator** aus und aktiviere die Skript-Ausführung, um administrative Befehle nutzen zu können.

```powershell
# Erlaubt das Ausführen lokaler Skripte
Set-ExecutionPolicy RemoteSigned -Force
```

-----

## 2\. Vollständiges System-Update

Ein aktuelles System minimiert Inkompatibilitäten zwischen dem USB-Stack und den Gerätetreibern.

### A. Windows Kern-Updates

```powershell
# Installation des Update-Moduls, falls nicht vorhanden
Install-Module PSWindowsUpdate -Force

# Suche, Download und Installation aller Windows-Updates inkl. automatischem Neustart
Get-WindowsUpdate -AcceptAll -Install -AutoReboot
```

### B. Applikations- & Runtime-Updates

```powershell
# Aktualisiert alle via Winget installierten Apps und Bibliotheken
winget upgrade --all
```

-----

## 3\. Ziel-Hardware identifizieren (Vendor-IDs)

Häufig genutzte Controller verwenden spezifische Vendor-IDs (VID). Nutze diese Befehle, um zu prüfen, ob deine Geräte erkannt werden.

| Hardware-Typ | Vendor-ID (VID) |
| :--- | :--- |
| **Arduino Uno / Mega** | `2341` |
| **CH340 (Günstige Klone)** | `1A86` |
| **CP210x (ESP32/Sensoren)** | `10C4` |
| **FTDI** | `0403` |

### Automatisierte Prüfung auf angeschlossene Geräte:

```powershell
# Prüft, ob Geräte der o.g. Hersteller aktuell angeschlossen sind
$vids = @("2341", "1A86", "10C4", "0403")
Get-PnpDevice -PresentOnly | Where-Object { $id = $_.InstanceId; $vids | Where-Object { $id -like "*VID_$(_)*" } } | Select-Object FriendlyName, Status, InstanceId
```

-----

## 4\. Treiber-Installation (Deployment)

Sobald die Geräte erkannt wurden (ggf. mit Status "Error" aufgrund fehlender Treiber), wird das Treiberpaket injiziert.

```powershell
# 1. Navigiere in dein lokales Treiber-Verzeichnis (Beispiel)
cd "C:\Pfad\zu\deinen\Treibern"

# 2. Treiber dem Windows-Store hinzufügen und installieren
pnputil /add-driver "*.inf" /install
```

-----

## 5\. Verifikation & Fehlerbehebung

Nach der Installation sollte der Status auf `OK` stehen.

```powershell
# Detaillierte Prüfung der neuen Hardware
Get-PnpDevice -PresentOnly | Where-Object {$_.Status -ne 'OK'} # Zeigt nur Geräte mit Problemen
```

**Bei Konflikten (Treiber-Reset):**
Falls ein falscher Treiber geladen wurde, entferne ihn über die OEM-Nummer:

```powershell
# OEM-Nummer herausfinden
pnputil /enum-drivers

# Treiber erzwingt löschen (Beispiel oem10.inf)
pnputil /delete-driver oem10.inf /uninstall /force
```

-----

## Zusammenfassung des Workflows

1.  **Rechte:** `Set-ExecutionPolicy RemoteSigned`
2.  **Update:** `Get-WindowsUpdate` -\> `winget upgrade --all`
3.  **Check:** VID-Abfrage durchführen.
4.  **Install:** `pnputil /add-driver`
5.  **Final:** Funktionsprüfung via `Get-PnpDevice`.

-----

Möchtest du, dass ich für die VID-Abfrage ein dediziertes PowerShell-Skript (.ps1) erstelle, das du direkt in dein Repository einbinden kannst?
