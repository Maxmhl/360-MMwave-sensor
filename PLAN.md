# 360° motorisierter mmWave-Sensor – Grobe Pläne

## Ziel
Ein mmWave-Radarsensor (60 GHz oder 24 GHz), der per Motor 360° rotiert und so einen Rundum-Scan
(Präsenz, Distanz, Bewegung, evtl. Punktwolke / Karte) liefert. Typische Anwendung: Raumüberwachung,
Präsenzerkennung in Smart Home, einfache Radar-Kartierung.

## Grundproblem
Günstige mmWave-Module haben ein begrenztes Sichtfeld (FoV), z. B. ca. 60–120° horizontal.
Ein einzelnes Modul sieht also nie 360°. Zwei Lösungswege: **drehen** (dieser Plan) oder **mehrere Module im Ring**.

---

## Plan A – Günstig & schnell: LD2410 / LD2450 + Schrittmotor (Smart Home)

| Teil | Vorschlag |
|---|---|
| Sensor | HLK-LD2450 (24 GHz, Multi-Target-Tracking X/Y) oder LD2410C (Präsenz/Distanz) |
| MCU | ESP32 (WiFi, ESPHome / MQTT / Home Assistant) |
| Motor | 28BYJ-48 + ULN2003 (billig) oder NEMA17 + A4988/TMC2209 (präziser, leiser) |
| Winkelreferenz | Hall-Sensor / Lichtschranke als Home-Position |
| Stromversorgung | USB-C 5 V / 12 V für NEMA17 |
| Signalübertragung | Wenn Sensor mitdreht: **Schleifring** (4–6 Leitungen) oder Sensor-Platine + ESP32 + Akku komplett mitdrehen und per WiFi senden |

Ablauf: Motor dreht in Schritten (z. B. 10°–30°), pro Winkel Sensordaten lesen, Daten
(Winkel, Distanz, Energie) zu einem Polarplot / Raumkarte zusammenführen.

Vorteile: billig (~30–60 €), schnell aufgebaut, Home-Assistant-tauglich.
Nachteile: langsame Rotation nötig, geringe Auflösung, kein echtes Radarbild.

## Plan B – Besser: 60-GHz-FMCW (Infineon BGT60 / Acconeer A121)

| Teil | Vorschlag |
|---|---|
| Sensor | Infineon BGT60TR13C (Radar Baseboard / Sense2GoL) oder Acconeer XM125/A121 |
| MCU/SBC | Raspberry Pi / RP2040 / ESP32-S3 |
| Motor | NEMA17 + TMC2209 oder Schrittmotor mit Encoder (AS5600) |
| Verbindung | Schleifring mit SPI/USB oder Onboard-Verarbeitung auf der rotierenden Plattform + WiFi/ETH |

Ablauf: Range-FFT pro Winkel, Winkel × Distanz → Polar-Heatmap („Radar-Display“),
optional Doppler für Bewegung/Vitalzeichen.

## Plan C – Ambitioniert: TI IWR6843 / IWR1443 + Rotation

- TI IWR6843ISK / AWR1843 liefert bereits Punktwolke (Range, Azimut, Elevation, Doppler).
- FoV ca. ±60° → mit Rotation um 180° / 360° zusammenfügen (Scan-Stitching / SLAM).
- Rechenleistung: Raspberry Pi 4/5 oder Jetson; ROS 2 für Punktwolken.
- Drehung: Schrittmotor/BLDC + Encoder, Schleifring mit USB/UART/Power.
- Vorteil: echte 3D-Punktwolke, Mapping; Nachteil: teurer (~250–500 €), komplexere Software.

## Plan D – Ohne bewegte Teile (Alternative)
Ring aus 3–6 LD2450-/LD2410-Modulen (je 60–120° FoV) um einen ESP32; keine Mechanik,
dafür Interferenzen zwischen Modulen beachten (zeitliches Multiplexing / Kanalwahl).

---

## Mechanik
- Rotierende Plattform auf Kugel-/Axiallager (z. B. 608ZZ / Drehkranz), Antrieb per Zahnriemen (GT2)
  oder direkt.
- Sensor **frei** vom Gehäuse – Radom aus Kunststoff (PLA/PETG/ABS, dünn, kein Metall/Carbon im Strahlengang).
- 3D-Druck: Basis, Drehteller, Sensorhalterung, Deckel.
- Schwerpunkt auf der Drehachse, um Vibrationen (Radar sieht Mikrobewegungen!) zu minimieren.
- Sensor möglichst weit weg von Motor (Motorvibration/Elektronik-Störungen).

## Elektronik / Daten
- **Schleifring** (Slip Ring 6–12 Leitungen) *oder* mitdrehende Elektronik + Akku / kabellose Leistungsübertragung.
- Winkel-Feedback: Magnetencoder (AS5600/AS5048) – wichtig für Synchronisation Winkel ↔ Radardaten.
- Zeitstempel: Radar-Frames mit Winkel koppeln (Interpolation bei kontinuierlicher Drehung).
- Schnittstellen: MQTT / Home Assistant / REST / ROS 2.

## Software
1. Treiber Sensor (UART/SPI) + Motorsteuerung (Stepper, Homing).
2. Scan-Modus: **Step-and-measure** (einfach) vs. **kontinuierliche Rotation** (schneller, Winkel-Interpolation).
3. Signalverarbeitung: Range-FFT, ggf. Doppler-FFT, Clutter-Filter (statischen Hintergrund abziehen), CFAR-Detektion.
4. Visualisierung: Polarplot (matplotlib / Web-Canvas), Heatmap, Home-Assistant-Dashboard.
5. Optional: Kartierung/SLAM, Personentracking, Zonen.

## Herausforderungen / Risiken
- Begrenztes FoV → Rotationsgeschwindigkeit vs. Aktualisierungsrate (Person kann sich während Rotation bewegen).
- Frame-Rate des Sensors begrenzt Winkelauflösung bei kontinuierlicher Drehung.
- Schleifring-Rauschen / Datenübertragung hochratiger Radar-Daten.
- Vibrationen und Motor-EMI stören das Radar.
- Funkzulassung/Regulierung (60-GHz-Module nur fertige, zugelassene Module nutzen).
- Statische Objekte vs. bewegte Ziele: LD24xx filtern oft nur bewegte/vitale Ziele.

## Empfohlener Fahrplan
1. **Prototyp (Wochenende):** LD2450 + ESP32 + 28BYJ-48 auf Drehteller, Step-and-measure, Polarplot.
2. **Verbesserung:** NEMA17 + TMC2209 + AS5600 + Homing, Schleifring, 3D-gedrucktes Gehäuse.
3. **Upgrade (optional):** BGT60 oder IWR6843 für höhere Auflösung/Punktwolke, ROS 2.
4. **Integration:** MQTT / Home Assistant, Dashboard, Kalibrierung.

## Offene Fragen
- Zweck: Präsenz im Smart Home, Kartierung/Robotik oder Forschung/Hobby?
- Budget und gewünschte Auflösung/Reichweite?
- Innen oder Außen? Netzbetrieb oder Akku?
- Bevorzugte Plattform (ESP32/ESPHome, Raspberry Pi, ROS)?
