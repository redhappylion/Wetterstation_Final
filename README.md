# Status-LED Erweiterung – Wetterstation 2025  
Marc & Filip  
Datum: 04.06.2025

## Projektübersicht  
Dieses Teilprojekt erweitert die bestehende ESP32-C3 Wetterstation um eine intelligente Statusanzeige mit der eingebauten RGB-LED. Ziel war es, den Verbindungs- und Sensorstatus durch klare Farbgebung direkt am Gerät sichtbar zu machen – insbesondere bei fehlender WLAN-Verbindung.

## Verwendete Komponenten

- **ESP32 C3**  
  Mikrocontroller mit integrierter WS2812 RGB-LED (GPIO 8)

- **DHT11 Sensor**  
  Misst Temperatur und Luftfeuchtigkeit

- **Ultraschallsensor**  
  Misst die Distanz zu einem Objekt

- **Webserver**  
  Zeigt aktuelle Messwerte und erlaubt manuelles Umschalten der LED-Farbe

- **EEPROM Speicher**  
  Merkt sich, ob die LED automatisch (Status) oder manuell gesteuert wird

## Status-LED Logik

Die interne LED zeigt den aktuellen Systemzustand durch definierte Farben an:

| Farbe         | Bedeutung                              |
|---------------|-----------------------------------------|
| 🔴 Rot        | Keine WLAN-Verbindung                   |
| 🟠 Orange     | Temperatur über 30 °C                   |
| 🟡 Gelb       | Ultraschallsensor meldet Fehler         |
| 🟢 Grün       | Alles in Ordnung                        |
| 🔵 Blau       | Initialer Zustand bei WLAN-Verbindung   |

> Hinweis: Wenn die LED-Steuerung im Webinterface deaktiviert wird, kann eine beliebige Farbe manuell gesetzt werden.

## Webinterface  

Im Webinterface ist eine Checkbox integriert, um die automatische LED-Anzeige zu aktivieren oder zu deaktivieren. Die Einstellung wird dauerhaft im EEPROM gespeichert.

## Fehleranalyse und Lösung  

### Problem  
Die rote LED leuchtete nicht, wenn keine WLAN-Verbindung bestand. Grund war eine blockierende Zeitsynchronisation (`setupTime()`), die auf NTP-Zeit wartete, obwohl kein WLAN verfügbar war.

### Korrekturen  

- Reihenfolge im `setup()` geändert:  
  - EEPROM-Einstellungen zuerst lesen  
  - Danach WLAN verbinden  
  - Status direkt über LED anzeigen  
  - Erst danach Zeit synchronisieren

- `setupTime()` mit Timeout ergänzt:  
  - Wenn kein WLAN: Synchronisation überspringen  
  - Wenn WLAN vorhanden: Max. 10 s warten

## Testprotokoll  

1. WLAN-Daten absichtlich falsch konfiguriert  
2. Code hochgeladen  
3. Seriellen Monitor beobachtet

### Ergebnis  
- Fehlgeschlagene WLAN-Verbindung erkannt  
- LED sofort auf **rot** geschaltet  
- Programm lief stabil weiter (nicht blockiert)  
- Meldung im Serial Monitor: *"Kein WLAN, Zeit Synchronisierung übersprungen."*

**Test bestanden**

## Fazit  

Die LED reagiert nun sofort sichtbar auf Systemfehler. Durch das Entschärfen der Blockaden im Setup ist das System robuster, besser testbar und benutzerfreundlicher geworden.

