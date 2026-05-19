# Gauge Scale - Vijeo Designer / Control Expert

Eigen kleurzones voor vacuüm en perslucht, voor achter een meter , gebouwd voor Vijeo Designer HMI in combinatie met Schneider Control Expert PLC.

---

## Achtergrond

De standaard CircleGraph in Vijeo Designer is lastig te gebruiken als je mBar-waarden rechtstreeks wilt verschalen naar hoeken op een cirkelmeter. Daarom is dit systeem opgebouwd uit twee losse delen die samenwerken:

1. **GaugeSwets.xdb** — PLC functieblok (Control Expert) dat de omrekening doet
2. **GaugeScale25.xml** — Vijeo Designer Smart Object dat de kleurzones tekent

Daarnaast is er een nieuwere versie waarbij de omrekening volledig in het HMI-object zit:

3. **GaugeScaleHMI01.xml** — Vijeo Designer Smart Object met ingebouwde omrekening, geen PLC functieblok nodig

---

## Bestandsoverzicht

| Bestand | Type | Omschrijving |
|---|---|---|
| `GaugeSwets.xdb` | Control Expert FB | Omrekening mBar naar HMI-schaalwaarden |
| `GaugeScale25.xml` | Vijeo Smart Object | Kleurzones op basis van omgerekende HMI-waarden |
| `GaugeScaleHMI01.xml` | Vijeo Smart Object | Kleurzones met ingebouwde omrekening, rechtstreeks mBar als input |

---

## GaugeSwets.xdb — PLC Functieblok

### Inputs

| Naam | Type | Omschrijving |
|---|---|---|
| `Pressure` | INT | Actuele druk in mBar (voor de wijzer) |
| `maxPress` | INT | Max druk compressor in mBar (grens oranje/rood) |
| `spHigh` | INT | Setpoint High in mBar (grens groen/oranje) |
| `spMid` | INT | Setpoint Mid in mBar (grens blauwgroen/groen) |
| `spLow` | INT | Setpoint Low in mBar (grens donkerblauw/blauwgroen) |
| `VacRelease` | INT | Vacuüm waarbij release valve opengaat in mBar (negatief) |
| `MaxVac` | INT | Max vacuüm waarbij pompen uitvallen in mBar (negatief) |
| `FullScale` | STRING | Schaalbereik: `"3000"` / `"4000"` / `"5000"` / `"6000"` / `"8000"` |

### FullScale waarden

| FullScale | Meterbereik | VacDegressScale |
|---|---|---|
| `"3000"` | -1 tot 3 bar | 2.1 |
| `"4000"` | -1 tot 4 bar | 1.55 |
| `"5000"` | -1 tot 5 bar | 1.25 |
| `"6000"` | -1 tot 6 bar | 1.05 |
| `"8000"` | -1 tot 8 bar | 0.75 |

### Output

| Naam | Type | Omschrijving |
|---|---|---|
| `HmiData` | Gauge (DDT) | Struct met alle omgerekende HMI-waarden |

### Gauge DDT structuur

| Veld | Type | Omschrijving |
|---|---|---|
| `ActualPressure` | INT | Actuele druk (doorgestuurd naar wijzer) |
| `MaxPressScale` | INT | Interne schaalmax (bijv. 5000 of 5720) |
| `MaxPress` | INT | Omgerekende max drukwaarde |
| `spHigh` | INT | Omgerekend setpoint High |
| `spMid` | INT | Omgerekend setpoint Mid |
| `spLow` | INT | Omgerekend setpoint Low |
| `MaxVacRelease` | INT | Omgerekende vacuüm release grens |
| `MaxVacPump` | INT | Omgerekende vacuüm pomp-uit grens |
| `MaxVacScale` | INT | Omgerekende max vacuüm schaalgrens |
| `TypeGauge` | INT | Gauge type (zelfde als FullScale als getal) |

---

## GaugeScale25.xml — Vijeo Smart Object (met PLC omrekening)

Dit object verwacht de **omgerekende** waarden uit `HmiData` van het PLC functieblok.

### Placeholders (inputs)

| Naam | Omschrijving |
|---|---|
| `MaxVacScale` | Koppelen aan `HmiData.MaxVacScale` |
| `MaxVacPump` | Koppelen aan `HmiData.MaxVacPump` |
| `MaxVacRelease` | Koppelen aan `HmiData.MaxVacRelease` |
| `MaxPressScale` | Koppelen aan `HmiData.MaxPressScale` |
| `MaxPress` | Koppelen aan `HmiData.MaxPress` |
| `SpHigh` | Koppelen aan `HmiData.spHigh` |
| `SpMid` | Koppelen aan `HmiData.spMid` |
| `SpLow` | Koppelen aan `HmiData.spLow` |

### Kleurzones (instelbaar via properties)

| Naam | Standaard | Zone |
|---|---|---|
| `ColorVacBlue` | Cyaan | Vacuüm normaal (van 0 tot VacRelease) |
| `ColorVacOrange` | Oranje | Vacuüm hoog (VacRelease tot MaxVacPump) |
| `ColorVacRed` | Rood | Vacuüm max (MaxVacPump tot MaxVacScale) |
| `ColorPressDarkBlue` | Donkerblauw | Pers laag (0 tot SpLow) |
| `ColorPressBlueGreen` | Blauwgroen | Pers mid-laag (SpLow tot SpMid) |
| `ColorPressGreen` | Groen | Pers normaal (SpMid tot SpHigh) |
| `ColorPressOrange` | Oranje | Pers hoog (SpHigh tot MaxPress) |
| `ColorPressRed` | Rood | Pers max (MaxPress tot MaxPressScale) |
| `BgColor` | Zwart | Achtergrond |

---

## GaugeScaleHMI01.xml — Vijeo Smart Object (omrekening in HMI)

Dit object doet de volledige omrekening zelf. Geen PLC functieblok nodig.

### Placeholders (inputs)

| Naam | Type | Omschrijving |
|---|---|---|
| `FullScale` | INT | Schaalbereik: 3000 / 4000 / 5000 / 6000 / 8000 |
| `MaxPress` | INT | Max druk compressor in mBar |
| `SpHigh` | INT | Setpoint High in mBar |
| `SpMid` | INT | Setpoint Mid in mBar |
| `SpLow` | INT | Setpoint Low in mBar |
| `VacRelease` | INT | Vacuüm release grens in mBar (negatief getal) |
| `MaxVac` | INT | Max vacuüm pomp-uit grens in mBar (negatief getal) |

Zelfde kleuropties als GaugeScale25.

---

## Gebruik

### Combinatie GaugeSwets + GaugeScale25

```
PLC (Control Expert)
  └── GaugeSwets FB
        Inputs:  Pressure, maxPress, spHigh, spMid, spLow,
                 VacRelease, MaxVac, FullScale (STRING)
        Output:  HmiData (Gauge DDT)

HMI (Vijeo Designer)
  └── Standaard Vijeo meter  ← HmiData.ActualPressure
  └── GaugeScale25           ← HmiData.MaxVacScale, MaxVacPump, etc.
```

### Standalone GaugeScaleHMI01

```
PLC (Control Expert)
  └── Rechtstreeks: maxPress, spHigh, spMid, spLow,
                    VacRelease, MaxVac, FullScale (INT)

HMI (Vijeo Designer)
  └── Standaard Vijeo meter  ← ActualPressure
  └── GaugeScaleHMI01        ← bovenstaande variabelen rechtstreeks
```

---

## Gebruikt in projecten

- 190019 Test Opstelling Bergharen
- 190117 Fujian DN455 01 & 02
- 210155 BASHUNDHARA 2115
