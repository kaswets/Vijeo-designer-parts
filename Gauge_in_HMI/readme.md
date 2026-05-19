# GaugeScaleHMI01 — Vijeo Designer Smart Object

Eigen analoge drukmeter met kleurzones voor vacuüm en perslucht, volledig berekend in Vijeo Designer. Geen PLC functieblok nodig.

---

## Achtergrond

Dit object is de opvolger van de combinatie **GaugeSwets.xdb + GaugeScale25.xml**.

Bij die oudere variant deed een PLC functieblok (Control Expert) de omrekening van mBar-waarden naar interne HMI-schaalwaarden, en het Smart Object tekende alleen de kleurzones op basis van die al omgerekende waarden.

Bij GaugeScaleHMI01 zit die volledige omrekening nu in het Smart Object zelf. Je geeft rechtstreeks de echte mBar-waarden als input, en het object regelt de rest.

Zie ook: [README.md](README.md) voor de uitleg over de oudere variant met PLC functieblok.

---

## Hoe het werkt

De meter heeft een perslucht-kant (bovenkant cirkel, van 180° naar 0°) en een vacuüm-kant (onderkant cirkel, van 180° naar 270°).

Op basis van de `FullScale` input bepaalt `calcZones()` eerst de schaalfactor en de vacuüm-degressiefactor. Daarna rekent hij alle mBar-waarden om naar interne schaalwaarden, en vervolgens naar hoeken op de cirkel. Die hoeken worden gebruikt om 8 gekleurde taartsegmenten te tekenen.

### FullScale — schaalbereik

| FullScale waarde | Meterbereik | VacDegressScale |
|---|---|---|
| 3000 | -1 tot 3 bar | 2.1 |
| 4000 | -1 tot 4 bar | 1.55 |
| 5000 | -1 tot 5 bar | 1.25 |
| 6000 | -1 tot 6 bar | 1.05 |
| 8000 | -1 tot 8 bar | 0.75 |

De VacDegressScale corrigeert de vacuüm-kant zodat de zones bij elk schaalbereik visueel correct uitkomen op de cirkel.

---

## Placeholders (inputs)

Alle inputs zijn INT variabelen in mBar. Koppel deze in Vijeo Designer aan PLC-variabelen.

| Naam | Type | Omschrijving |
|---|---|---|
| `FullScale` | INT | Schaalbereik: 3000 / 4000 / 5000 / 6000 / 8000 |
| `MaxPress` | INT | Max druk die de compressor kan leveren (grens oranje/rood) |
| `SpHigh` | INT | Setpoint High (grens groen/oranje) |
| `SpMid` | INT | Setpoint Mid (grens blauwgroen/groen) |
| `SpLow` | INT | Setpoint Low (grens donkerblauw/blauwgroen) |
| `VacRelease` | INT | Vacuüm waarbij release valve opengaat — negatief getal, bijv. -600 |
| `MaxVac` | INT | Max vacuüm waarbij pompen uitvallen — negatief getal, bijv. -800 |

---

## Kleurzones

Het object tekent 8 zones. De kleuren zijn instelbaar via de properties in Vijeo Designer.

### Vacuüm-kant (onderkant cirkel)

| Naam | Standaard kleur | Zone |
|---|---|---|
| `ColorVacBlue` | Cyaan | Vacuüm normaal — van 0 tot VacRelease |
| `ColorVacOrange` | Oranje | Vacuüm hoog — van VacRelease tot MaxVac |
| `ColorVacRed` | Rood | Vacuüm max — van MaxVac tot schaalgrens |

### Pers-kant (bovenkant cirkel)

| Naam | Standaard kleur | Zone |
|---|---|---|
| `ColorPressDarkBlue` | Donkerblauw | Pers laag — van 0 tot SpLow |
| `ColorPressBlueGreen` | Blauwgroen | Pers mid-laag — van SpLow tot SpMid |
| `ColorPressGreen` | Groen | Pers normaal — van SpMid tot SpHigh |
| `ColorPressOrange` | Oranje | Pers hoog — van SpHigh tot MaxPress |
| `ColorPressRed` | Rood | Pers max — van MaxPress tot schaalgrens |

### Achtergrond

| Naam | Standaard kleur | Omschrijving |
|---|---|---|
| `BgColor` | Zwart | Achtergrondkleur van het object |

---

## Gebruik in Vijeo Designer

1. Importeer `GaugeScaleHMI01.xml` als Smart Object
2. Plaats het object op je scherm
3. Koppel de 7 INT-variabelen aan de placeholders
4. Stel de kleuren in via de properties (optioneel)
5. Plaats de standaard Vijeo meter bovenop voor de wijzer en schaalverdeling
   - Koppel de actuele druk (mBar) aan de wijzer van de standaard meter
   - Stel het bereik van de standaard meter in op het gewenste schaalbereik

---

## Verschil met de oudere variant

| | GaugeSwets + GaugeScale25 | GaugeScaleHMI01 |
|---|---|---|
| Omrekening | In PLC (Control Expert) | In HMI (Vijeo Designer) |
| PLC functieblok nodig | Ja | Nee |
| Input type FullScale | STRING (`"6000"`) | INT (`6000`) |
| Inputs aan HMI | Omgerekende HMI-waarden | Rechtstreeks mBar |
| Geschikt voor | Bestaande projecten met GaugeSwets | Nieuwe projecten zonder PLC omrekening |

Beide varianten geven hetzelfde visuele resultaat.

---

## Gebruikt in projecten

- Nieuw object, nog niet in projecten uitgerold
