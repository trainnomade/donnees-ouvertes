# Bus, tram and coach connections at French railway stations: stops within 350 m, lines and typical times

For the large railway stations covered by Train Nomade: every bus, tram, metro or coach stop within 350 m of the
station, its lines and their directions, the first and last departure on a working day, the daytime frequency and the
number of departures on a working day, a Saturday and a Sunday, from the GTFS timetables that the local networks publish
on transport.data.gouv.fr, the French national access point to mobility data. Updated every week by
[Train Nomade](https://trainnomade.fr/donnees/correspondances-gares/).

*Version française plus bas.*

- File: [`data/correspondances-gares.csv`](data/correspondances-gares.csv)
- Live version (updated every week): https://api.trainnomade.fr/donnees/v1/correspondances-gares.csv
- Description page: https://trainnomade.fr/donnees/correspondances-gares/

The file is comma separated, UTF-8, with the header on the first line. Times are local time (`HH:MM`). An empty cell
means "unknown" or "not applicable", never zero.

## `correspondances-gares.csv`: one line per station, stop, line and direction

Sorted by `code_uic_gare`, then by distance to the station, stop, network, line and direction.

| Column | Format | Content |
| --- | --- | --- |
| `code_uic_gare` | 8 digits | UIC code of the railway station (SNCF timetables) |
| `gare` | text | railway station name |
| `page_gare` | URL | page of the station on trainnomade.fr (departures, access map) |
| `arret` | text | name of the bus, tram or coach stop, as published by the network |
| `ids_arrets_gtfs` | identifiers separated by `;` | `stop_id` of the network GTFS grouped under this name (both sides of the street, platforms) |
| `latitude` | decimal degrees, WGS84 | average position of these GTFS stops |
| `longitude` | decimal degrees, WGS84 | average position of these GTFS stops |
| `distance_m` | integer, metres (rounded to 10) | straight-line distance from the station |
| `reseau` | text | transport network |
| `mode` | `bus`, `tram`, `metro` | mode as declared in the network GTFS (`route_type`) |
| `ligne` | text | line number or name |
| `couleur_ligne` | `#RRGGBB` | line colour (adjusted for readability when the network gives none or an unreadable one) |
| `direction` | text | direction (terminus or headsign) |
| `premier_passage` | `HH:MM` | first departure from the stop on the reference working day |
| `dernier_passage` | `HH:MM` | last departure on the reference working day (a time after midnight ends the service day) |
| `frequence_min` | integer, minutes | median interval between departures from 07:00 to 19:00 on the working day; empty when fewer than 5 departures |
| `passages_jour_ouvre` | integer | departures on the reference working day |
| `passages_samedi` | integer | departures on the reference Saturday |
| `passages_dimanche` | integer | departures on the reference Sunday |
| `jour_ouvre_reference` | `YYYY-MM-DD` | working day used (next Tuesday after the reading) |
| `samedi_reference` | `YYYY-MM-DD` | Saturday used |
| `dimanche_reference` | `YYYY-MM-DD` | Sunday used |
| `licence_source` | `Licence Ouverte 2.0` or `ODbL` | licence of the network data |
| `jeu_source` | URL | dataset of the network on transport.data.gouv.fr |
| `releve_le` | `YYYY-MM-DD` | date on which the network timetables were read |

```
code_uic_gare,gare,page_gare,arret,ids_arrets_gtfs,latitude,longitude,distance_m,reseau,mode,ligne,couleur_ligne,direction,premier_passage,dernier_passage,frequence_min,passages_jour_ouvre,passages_samedi,passages_dimanche,jour_ouvre_reference,samedi_reference,dimanche_reference,licence_source,jeu_source,releve_le
87141002,Nancy,https://trainnomade.fr/gares/gare-de-nancy/,Gare - Poirel,NYGAR1,48.690437,6.175657,100,Stan,bus,11,#FF7900,Jardins Fleuris,07:00,20:33,30,28,22,21,2026-10-27,2026-10-31,2026-10-25,Licence Ouverte 2.0,https://transport.data.gouv.fr/datasets/fr-200052264-t0034-0000-1,2026-10-24
```

## Method

1. Networks: every GTFS of the national access point with stops within 350 m of the station. Regional aggregates that
   duplicate local networks, railway datasets, long-distance coaches, school-only services and zonal on-demand
   transport are left out; a network published twice is read once (the version with real-time data, then the most
   recent one).
2. Stops: GTFS stops of the same name are grouped; their position is the average of the grouped stops.
3. Times: departures of a coming Tuesday, Saturday and Sunday (the "reference" days) from `calendar.txt` and
   `calendar_dates.txt`; for each line, the two main directions are kept (partial services of a few trips are left out);
   a line duplicated by two networks with the same times is kept once.
4. Frequency: median interval between two departures between 07:00 and 19:00, when there are at least 5 departures.
5. Licences: only networks published under Licence Ouverte 2.0 (Etalab) or ODbL are included; each line names its
   network, its licence and its dataset.
6. Checks before each update (the previous version stays online if one fails): at least 45 stations and 90 % of the
   previous snapshot, at least 2,000 lines and 85 % of the previous snapshot, all distances within 400 m, valid times,
   timetables read less than 21 days ago, licences limited to the two above, coordinates in mainland France.

## Content of this snapshot (timetables read on 24 October 2026)

50 railway stations, 349 stops, 75 networks, 2,599 line-directions (2,494 bus, 87 tram, 18 metro); reference days
27, 31 and 25 October 2026.

## Sources

- Timetables (GTFS) published by each network on **transport.data.gouv.fr** (https://transport.data.gouv.fr/), cited
  line by line in `reseau`, `jeu_source` and `licence_source`: Licence Ouverte 2.0 (Etalab) or ODbL depending on the
  network.
- Railway stations: UIC codes and positions from the SNCF dataset "Gares de voyageurs"
  (https://ressources.data.sncf.com/explore/dataset/gares-de-voyageurs/, ODbL).

## Limits

- Scheduled timetables of one typical day: no engineering works, strikes or real-time information.
- Networks under the Licence Mobilités (Île-de-France Mobilités) or with no declared licence are left out: Paris
  stations, Amiens regional coaches and the Caen urban network are not included yet.
- Only the stations covered by Train Nomade (50 today); the list grows with the site.
- "Stop within 350 m" is measured as the crow flies, not as a walking route.

## How to cite

> Train Nomade, *Bus, tram and coach connections at French railway stations*, from the timetables of the networks
> published on transport.data.gouv.fr. Version of [date]. https://api.trainnomade.fr/donnees/v1/correspondances-gares.csv

## Licence

This database is made available under the **Open Database License (ODbL) 1.0**, see [LICENSE](LICENSE), because part of
its sources are under ODbL. Credit to display: **"Correspondances gare-bus : Train Nomade (https://trainnomade.fr),
d'après les horaires des réseaux publiés sur transport.data.gouv.fr"**, and the networks used. If you publish a database
derived from it, share it under the same licence.

Contact: contact@trainnomade.fr

---

## En français

**Correspondances gare ↔ bus, tram et cars** : pour les grandes gares suivies par Train Nomade, chaque arrêt de bus,
tram, métro ou car à moins de 350 m de la gare, ses lignes et directions, le premier et le dernier passage d'un jour
ouvré, la fréquence en journée et le nombre de passages un jour ouvré, un samedi et un dimanche, d'après les horaires
GTFS publiés par les réseaux sur transport.data.gouv.fr. Mis à jour chaque semaine :
https://api.trainnomade.fr/donnees/v1/correspondances-gares.csv (page : https://trainnomade.fr/donnees/correspondances-gares/).

| Colonne | Contenu |
| --- | --- |
| `code_uic_gare` | code UIC de la gare (horaires SNCF) |
| `gare` | nom de la gare |
| `page_gare` | page de la gare sur trainnomade.fr |
| `arret` | nom de l'arrêt publié par le réseau |
| `ids_arrets_gtfs` | identifiants `stop_id` du GTFS du réseau regroupés sous ce nom |
| `latitude` | latitude moyenne de ces arrêts (WGS84) |
| `longitude` | longitude moyenne de ces arrêts (WGS84) |
| `distance_m` | distance à vol d'oiseau depuis la gare, en mètres (arrondie à 10) |
| `reseau` | réseau de transport |
| `mode` | mode déclaré dans le GTFS du réseau : `bus`, `tram`, `metro` |
| `ligne` | numéro ou nom de la ligne |
| `couleur_ligne` | couleur de la ligne (`#RRGGBB`) |
| `direction` | direction (terminus ou girouette) |
| `premier_passage` | premier passage le jour ouvré de référence |
| `dernier_passage` | dernier passage le jour ouvré de référence |
| `frequence_min` | intervalle médian entre deux passages de 7 h à 19 h, en minutes (vide sous 5 passages) |
| `passages_jour_ouvre` | passages le jour ouvré de référence |
| `passages_samedi` | passages le samedi de référence |
| `passages_dimanche` | passages le dimanche de référence |
| `jour_ouvre_reference` | jour ouvré utilisé (le mardi suivant le relevé) |
| `samedi_reference` | samedi utilisé |
| `dimanche_reference` | dimanche utilisé |
| `licence_source` | licence des données du réseau : `Licence Ouverte 2.0` ou `ODbL` |
| `jeu_source` | jeu du réseau sur transport.data.gouv.fr |
| `releve_le` | date de lecture des horaires des réseaux |

- Méthode : arrêts de tous les GTFS du point d'accès national à moins de 350 m de la gare, regroupés par nom ; agrégats
  régionaux en doublon, jeux ferroviaires, cars longue distance, scolaire seul et transport à la demande zonal écartés ;
  horaires d'un mardi, d'un samedi et d'un dimanche à venir ; deux directions principales par ligne ; fréquence =
  écart médian de 7 h à 19 h.
- Contrôles avant chaque mise à jour (sinon l'ancienne version reste en ligne) : au moins 45 gares et 90 % de
  l'instantané précédent, au moins 2 000 lignes et 85 % de l'instantané précédent, distances de 400 m au plus, heures
  valides, horaires lus il y a moins de 21 jours, deux licences seulement, coordonnées en France métropolitaine.
- Cet instantané (horaires lus le 24 octobre 2026) : 50 gares, 349 arrêts, 75 réseaux, 2 599 lignes-directions.
- Sources : horaires GTFS de chaque réseau sur https://transport.data.gouv.fr/ (Licence Ouverte 2.0 ou ODbL selon le
  réseau, cités ligne par ligne) ; codes et positions des gares : « Gares de voyageurs » de SNCF (ODbL).
- Limites : horaires prévus d'un jour type, ni travaux ni temps réel ; réseaux sous Licence Mobilités
  (Île-de-France Mobilités) ou sans licence déclarée écartés (gares parisiennes, cars de la Somme et de l'Oise à Amiens,
  réseau de Caen) ; seulement les gares suivies par Train Nomade ; distance à vol d'oiseau, pas à pied.
- Citation : **« Correspondances gare-bus : Train Nomade (https://trainnomade.fr), d'après les horaires des réseaux
  publiés sur transport.data.gouv.fr »**, avec les réseaux utilisés. Licence **ODbL 1.0** : partage à l'identique de
  toute base dérivée. Train Nomade est un site indépendant, sans lien avec les réseaux cités.
