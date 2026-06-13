# Rancilio Silvia – PID-Umbau (Individueller Plan für ESP8266 / Rev 1.2)

Diese Anleitung ist speziell auf dein Setup mit der **Rancilio Silvia**, dem **ESP8266 (NodeMCU Amica V2)** und der **CleverCoffee Platine Rev 1.2** zugeschnitten.

**⚠️ WICHTIGER HINWEIS ZUR SOFTWARE-VERSION:**
Laut dem aktuellen CleverCoffee Projekt wird ab **Version 4.0.0** die Entwicklung *ausschließlich für den ESP32* fortgesetzt! Da du einen **ESP8266** verwendest, bist du für die Firmware auf eine ältere Version (z. B. v3.x.x) angewiesen. Die neuen Features wie Web-Flasher oder BLE-Waagen aus v4.0.0 sind mit deinem Microcontroller leider nicht kompatibel. Du musst die Firmware klassisch über **VSCode + PlatformIO** flashen.

---

## 🛠 Deine Hardware-Materialliste

| Komponente | Spezifikation / Modell | Funktion im Umbau |
|---|---|---|
| **Platine** | CleverCoffee PCB **Rev 1.2** | Trägerplatine (Shield), verteilt Spannung & Signale. |
| **Mikrocontroller** | AZ-Delivery NodeMCU Amica V2 (ESP8266MOD 12-F) | Das „Gehirn“: Liest den Sensor, regelt die Heizung per PID, stellt WLAN bereit. |
| **Display** | AZ-Delivery 0,96″ I²C OLED | Anzeige von Ist-/Soll-Temperatur und Status an der Maschine. |
| **Relais (SSR)** | SSR E32-0U3 RA 2425-D06 | Schaltet die 230V-Heizung sicher per 5V-Steuersignal vom ESP. |
| **Netzteil** | Mean-Well IA-433 SNT RS-15-5 (5 V) | Versorgt ESP, Platine und das OLED-Display mit 5V Gleichstrom. |
| **Sensor** | TSIC 306 (TO-92) | Misst hochpräzise die aktuelle Boilertemperatur. |

**Referenzen:**
- [CleverCoffee Handbuch (Deutsch)](https://manual.rancilio-pid.de/de/)
- [Pinbelegung Platine Rev 1.2 für ESP8266](https://manual.rancilio-pid.de/de/platinen/ESP8266.html)

---

## ⚠️ Schritt 1 – Sicherheit zuerst (Lebenswichtig!)

Du arbeitest mit **230 V Netzspannung** und einem **~1100 W Heizelement**. Fehler können lebensgefährlich sein!

- **Netzstecker ziehen** und Maschine vollständig abkühlen lassen, bevor du sie öffnest.
- Arbeite **niemals** an der Maschine, solange sie am Netz hängt.
- Der **Sicherheits-/Übertemperaturthermostat (Klixon)** der Silvia **muss in der Heizung in Reihe bleiben**. Er ist die Notabschaltung. Niemals überbrücken!
- Der **Brüh-Thermostat (Bimetall, i.d.R. 100°C)** wird durch das SSR ersetzt – die Temperaturregelung übernimmt ab jetzt der ESP.
- Alle Adern mit 230V müssen korrekt gecrimpt (Aderendhülsen oder Flachstecker) und isoliert werden.
- Wenn du dir bei der 230-V-Verdrahtung unsicher bist: **Elektrofachkraft hinzuziehen.**

---

## 💻 Schritt 2 – Firmware vorbereiten und ESP8266 flashen

Da Version 4.0 den ESP8266 nicht mehr unterstützt, musst du eine Version aus der **3.x.x Serie** (in deinem Fall **v3.3.0**) installieren.

1. **Software installieren**: Lade dir Visual Studio Code (VSCode) und die Erweiterung **PlatformIO** herunter.
2. **Quellcode laden**: Lade dir den Quellcode einer v3-Version als ZIP von der GitHub-Seite herunter und entpacke ihn.
3. Öffne den entpackten Ordner in VSCode (PlatformIO erkennt das Projekt automatisch).
4. **Konfigurationsdatei anpassen**: Kopiere die `userConfig_sample.h` zu `userConfig.h`. Setze dort die für dich passenden Werte:
   - **WLAN-Zugangsdaten (Wichtig für v3.3.0!):** Trage bei `WIFI_SSID` und `WIFI_PASS` die Daten für dein Heimnetzwerk ein. Der WiFi-Manager wurde in dieser Version entfernt, eine Konfiguration per Weboberfläche ist beim ersten Start nicht mehr möglich.
   - **Temperatursensor**: Setze `TEMPSENSOR` auf `2` für den TSIC.
   - **Display**: Setze `OLED_DISPLAY` auf `2` für das SSD1306 0,96" Display.
   - Prüfe auch die weiteren Einstellungen wie `MACHINEID` und passe sie bei Bedarf an.
5. **ESP flashen**: Verbinde den AZ-Delivery NodeMCU per Micro-USB mit dem PC und klicke in PlatformIO auf "Upload".

> *Tipp: Teste den NodeMCU + Platine + OLED kurz am USB-Kabel (vor dem Einbau in die Maschine). Wenn das OLED etwas anzeigt, war das Flashen erfolgreich. Der ESP wird versuchen, sich direkt mit dem von dir konfigurierten WLAN zu verbinden.*

**Referenzen:**
- CleverCoffee GitHub Releases (für v3.x.x)
- Anleitung: Software installieren (CleverCoffee Handbuch) [11]
- Download Visual Studio Code
- PlatformIO IDE für VSCode

---

## 🧩 Schritt 3 – Platine Rev 1.2 bestücken

1. **NodeMCU aufstecken**: Achte darauf, dass der Micro-USB Anschluss in die Richtung zeigt, die auf der Rev 1.2 Platine markiert ist (Pin-1 Aufdruck beachten).
2. **OLED anschließen**: Verbinde das Display an den I²C-Anschluss der Platine. *Achtung: Bei AZ-Delivery sind oft VCC und GND sowie SDA und SCL vertauscht! Überprüfe die Beschriftung des Displays exakt mit der Beschriftung auf der Platine.*
3. **Klemmen identifizieren**: Mache dich mit den Anschlüssen für das Relais (SSR-Steuerung), den Sensor (TSIC) und den 5V Eingang vertraut. Siehe dazu den Platinen-Guide.

**Referenzen:**
- Pinbelegung Platine Rev 1.2 für ESP8266
- Allgemeiner Aufbau der Hardware (CleverCoffee Handbuch) [17]

---

## 🔧 Schritt 4 – Maschine öffnen & Dokumentieren

1. Stecker gezogen, Maschine kalt.
2. Wassertank entnehmen, obere Abdeckung abschrauben, Gehäuse öffnen.
3. **Fotos machen!** Fotografiere die Werksverdrahtung am Boiler und den Thermostaten, bevor du Kabel abziehst.
4. Finde den **Brüh-Thermostat** (wird stillgelegt/überbrückt) und den **Sicherheitsthermostat** (bleibt erhalten!).

---

## 🌡️ Schritt 5 – Temperatursensor (TSIC 306) anbringen

Der Sensor benötigt drei Verbindungen: GND, VCC (3,3V) und Signal.

1. Prüfe das Datenblatt des TSIC 306 (TO-92), um die 3 Beinchen korrekt zuzuordnen.
2. Befestige den Sensor mit etwas **Wärmeleitpaste** am Kessel (z. B. unter der Klammer des alten Brüh-Thermostats oder mit einer passenden Schraube).
3. Verlege ein 3-adriges, hitzebeständiges Kabel (z. B. Silikonlitze) von der Platine zum Sensor.
4. Schließe die Adern an die "TSIC" Pins der Platine (GND / 3.3V / SIG) an.

**Referenzen:**
- Datenblatt TSIC 306 (Beispiel) [14]
- Anleitung: Temperatursensor anschließen (CleverCoffee Handbuch)
---

## ⚡ Schritt 6 – Heizung (SSR E32-0U3) & Netzteil (Mean-Well RS-15-5)

**Das SSR (E32-0U3 RA 2425-D06):**
1. **Lastseite (230 V):** Trenne die beiden Kabel, die an den Brüh-Thermostat gehen. Diese werden an die Last-Klemmen des SSR angeschlossen. *Der Stromkreislauf verläuft nun über das SSR zur Heizung und den Sicherheitsthermostat.*
2. **Steuerseite (DC 5V):** Verbinde den Schaltausgang der Platine Rev 1.2 mit dem Steuereingang (+ und -) des SSR. Polung beachten!
3. Platziere das SSR so, dass es Wärme an ein Metallteil des Gehäuses abgeben kann.

**Das Netzteil (Mean-Well IA-433 SNT RS-15-5):**
1. **230 V Eingang:** Klemme Phase (L) und Nullleiter (N) parallel an den Netzanschluss der Maschine, **nach** dem Hauptschalter, damit der PID nur an ist, wenn die Maschine läuft. Schließe auch die Erdung (PE) an das Netzteil an.
2. **5 V Ausgang:** Verbinde V+ und V- mit dem **5V In** Terminal der Rev 1.2 Platine. Prüfe die Spannung idealerweise vorher kurz mit einem Multimeter (sollte bei ca. 5.0V liegen).

**Referenzen:**
- Schaltpläne (CleverCoffee Handbuch) [24]
- Anleitung: SSR anschließen (CleverCoffee Handbuch)
- Datenblatt Mean-Well RS-15 Serie [13]

---

## 📦 Schritt 7 – Endmontage & Erster Start

1. Fixiere Platine, SSR und das Mean-Well Netzteil sicher im Gehäuse. Achte auf genügend Abstand zum heißen Boiler (Hitze kann das Netzteil oder den ESP beschädigen).
2. Platziere das OLED-Display so, dass es von außen gut ablesbar ist.
3. **Erster Start (mit Wasser, Boiler voll!):** Stecker rein, Hauptschalter umlegen.
   - Das OLED sollte aufleuchten und die aktuelle Kesseltemperatur anzeigen.
   - Der ESP verbindet sich direkt mit dem in `userConfig.h` eingetragenen WLAN. Du musst die IP-Adresse des Geräts (z.B. über die Weboberfläche deines Routers) herausfinden, um auf das Web-Interface zuzugreifen.
4. **Heiz-Test & PID Tuning:** Öffne das Web-Interface über die IP-Adresse der Maschine und stelle einen Soll-Wert (z. B. 95°C) ein. Beobachte, ob das SSR heizt und die Temperatur wie erwartet ansteigt und sich stabilisiert. Falls die Temperatur über das Ziel hinausschießt, musst du die PID-Werte im Web-Interface anpassen (siehe Community-Empfehlungen für die Silvia).

**Referenzen:**
- Anleitung: PID-Werte einstellen (CleverCoffee Handbuch) [19]
- CleverCoffee Discord Community (für Hilfe & Austausch)

---

## ✅ Checkliste vor dem ersten Espresso

- [ ] Sicherheitsthermostat ist in Reihe verblieben und nicht überbrückt?
- [ ] ESP8266 wurde mit einer v3.x.x Firmware (nicht v4.0.0) erfolgreich geflasht?
- [ ] SSR leuchtet beim Heizen auf und schaltet am Soll-Wert zuverlässig ab?
- [ ] Temperaturanzeige ist plausibel (startet bei ~20-25°C, wenn Maschine kalt ist)?
- [ ] Alle 230V Kontakte sind fest, fachgerecht gecrimpt und vollständig isoliert?

*Viel Erfolg und guten Espresso! ☕*
