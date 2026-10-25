# Train service at every French railway station: trains per day, first and last departure, direct destinations

For each of the 2,792 passenger stations listed by SNCF: how many trains call there on an average working day, on
Saturday and on Sunday, how many regional coaches, the first and last departure, how many stations can be reached
without changing trains, and which kinds of trains stop there (TGV INOUI, OUIGO, Intercités, TER…). A second file lists,
station by station, every destination reached without a change. Computed every night from the official SNCF timetables
by [Train Nomade](https://trainnomade.fr/donnees/desserte-gares/).

*Version française plus bas.*

- Files: [`data/desserte-gares.csv`](data/desserte-gares.csv) and
  [`data/desserte-gares-destinations.csv`](data/desserte-gares-destinations.csv)
- Live versions (recomputed every night): https://api.trainnomade.fr/donnees/v1/desserte-gares.csv and
  https://api.trainnomade.fr/donnees/v1/desserte-gares-destinations.csv
- Description page: https://trainnomade.fr/donnees/desserte-gares/

Both files are comma separated, UTF-8, with the header on the first line. Times are Paris local time (`HH:MM`). An empty
cell means "unknown" or "not applicable", never zero.

## `desserte-gares.csv`: one line per station

One line per passenger station of the SNCF list, sorted by `code_uic`. The period is the 14 days starting on the day of
computation (fewer if the published timetable ends earlier).

| Column | Format | Content |
| --- | --- | --- |
| `code_uic` | 8 digits | UIC code of the station, as in the SNCF timetables (`StopArea:OCE` + code) |
| `autres_codes_uic` | codes separated by `;` | other UIC codes of the same station (for example the suburban part of a Paris terminus); usually empty |
| `nom` | text | official station name (SNCF list) |
| `code_commune` | 5 characters | INSEE code of the municipality |
| `latitude` | decimal degrees, WGS84 | position of the station |
| `longitude` | decimal degrees, WGS84 | position of the station |
| `segment_drg` | `A`, `B` or `C` | SNCF station category: `A` national, `B` regional, `C` local |
| `desserte_connue` | `oui` or `non` | `oui` if at least one train or coach of the source calls at the station during the period; `non`: the counts are unknown (empty) |
| `trains_jour_ouvre` | decimal, trains | average number of trains calling at the station on a working day (Monday to Friday, public holidays excluded) |
| `trains_samedi` | decimal, trains | average number of trains on a Saturday |
| `trains_dimanche` | decimal, trains | average number of trains on a Sunday |
| `cars_jour_ouvre` | decimal, coaches | average number of regional coaches (Car TER, on-demand coaches, shuttles) on a working day |
| `premier_depart` | `HH:MM` | earliest departure of a train on a working day of the period |
| `dernier_depart` | `HH:MM` | latest departure of a train on a working day of the period (may be after midnight) |
| `destinations_directes` | integer, stations | stations reached by at least one train without a change during the period |
| `types_trains` | labels separated by `;` | kinds of trains calling at the station: `TGV INOUI`, `OUIGO`, `TGV Lyria`, `ICE`, `Intercités`, `Intercités de nuit`, `OUIGO Train Classique`, `TER`, `Tram-train`, `Train` |
| `periode_debut` | `YYYY-MM-DD` | first day of the period counted |
| `periode_fin` | `YYYY-MM-DD` | last day of the period counted |

```
code_uic,autres_codes_uic,nom,code_commune,latitude,longitude,segment_drg,desserte_connue,trains_jour_ouvre,trains_samedi,trains_dimanche,cars_jour_ouvre,premier_depart,dernier_depart,destinations_directes,types_trains,periode_debut,periode_fin
87713040,,Dijon,21231,47.323370,5.027208,A,oui,243.8,145.5,159,12,05:09,22:12,135,Intercités;TER;TGV INOUI;TGV Lyria,2026-10-25,2026-11-07
```

## `desserte-gares-destinations.csv`: one line per direct link

One line per pair (departure station, station reached without a change), sorted by `code_uic_depart` then
`code_uic_arrivee`. Departure stations are those of `desserte-gares.csv`; destinations may be abroad (Brussels, Geneva,
Barcelona…). Same period as `desserte-gares.csv`.

| Column | Format | Content |
| --- | --- | --- |
| `code_uic_depart` | 8 digits | UIC code of the departure station (`code_uic` of `desserte-gares.csv`) |
| `gare_depart` | text | name of the departure station |
| `code_uic_arrivee` | 8 digits | UIC code of the station reached |
| `gare_arrivee` | text | name of the station reached |
| `trains_par_semaine` | decimal, trains | direct trains per week (trains of the period × 7 / number of days) |
| `duree_min_minutes` | integer, minutes | shortest scheduled travel time |
| `premier_depart` | `HH:MM` | earliest departure on a working day; empty if the link only runs at weekends |
| `dernier_depart` | `HH:MM` | latest departure on a working day; empty if the link only runs at weekends |
| `types_trains` | labels separated by `;` | kinds of trains running the link |

```
code_uic_depart,gare_depart,code_uic_arrivee,gare_arrivee,trains_par_semaine,duree_min_minutes,premier_depart,dernier_depart,types_trains
87713040,Dijon,87686006,Paris Gare de Lyon,92.5,95,06:27,20:27,TER;TGV INOUI;TGV Lyria
```

## Method

- Source timetables: every trip of the SNCF GTFS feed running on each day of the period (`calendar_dates.txt`).
- A train "calls" at a station when boarding or alighting is allowed there; a train calling twice is counted once.
  Coaches (`route_type` 3 or a coach product) are counted apart in `cars_jour_ouvre` and are not used for destinations.
- Averages: working days are Monday to Friday except the 11 national public holidays; Saturday and Sunday averages use
  the Saturdays and Sundays of the period; a public holiday on a weekday or a Saturday is left out of the averages (on a
  Sunday it counts as a Sunday).
- Times are those of the timetable, in Paris time. A departure after midnight belongs to the service day that started
  the evening before (for example `00:35`).
- A station with several UIC codes (Paris Gare de Lyon: `87686006` main lines, `87686030` suburban) is one line; its
  `code_uic` is the code most used in the timetables.
- Checks run before each update (the previous version stays online if one fails): at least 2,700 stations, at least
  2,000 stations with known service and 90 % of the previous snapshot, unique 8-digit codes, a 7 to 14-day period,
  plausible values (at most 1,500 trains a day), large stations (Lyon Part-Dieu, Marseille Saint-Charles, Bordeaux
  Saint-Jean, Lille Flandres) above a minimum, at least 20,000 direct links.

## Content of this snapshot (25 October 2026)

Period 25 October to 7 November 2026: 2,792 stations, 2,383 with known service, 49,056 direct links. Busiest stations on
a working day in this source: Strasbourg (585 trains), Lille Flandres (547), Nantes (484), Lyon Part-Dieu (473).

## Sources

- **Timetables**: SNCF Voyageurs, GTFS dataset "Réseau SNCF TGV, Intercités et TER"
  (https://transport.data.gouv.fr/datasets/horaires-sncf; file
  https://eu.ftp.opendatasoft.com/sncf/plandata/Export_OpenData_SNCF_GTFS_NewTripId.zip), licence ODbL.
- **Stations**: SNCF, dataset "Gares de voyageurs"
  (https://ressources.data.sncf.com/explore/dataset/gares-de-voyageurs/), licence ODbL: names, UIC codes, INSEE codes,
  positions and categories.

## Limits

- Scheduled timetables only: no delays, cancellations or last-minute changes. Engineering works and holidays of the
  period weigh on the averages; compare files of different dates with care.
- The source does not include Transilien and RER trains (Île-de-France), Eurostar, tourist railways or other operators:
  Paris termini show only main-line and TER trains, and a station served only by those trains has `desserte_connue` =
  `non` and empty counts.
- `Train` is the label used by the source for some regional trains that are not branded TER.
- Direct destinations count stations reached by train without a change; coaches are not included.

## How to cite

> Train Nomade, *Train service at every French railway station*, from SNCF Voyageurs timetables (GTFS, ODbL).
> Version of [date]. https://api.trainnomade.fr/donnees/v1/desserte-gares.csv

## Licence

This database is made available under the **Open Database License (ODbL) 1.0**, see [LICENSE](LICENSE). Credit to
display: **"Desserte des gares : Train Nomade (https://trainnomade.fr), d'après les horaires SNCF Voyageurs (ODbL)"**.
If you publish a database derived from it, share it under the same licence.

Contact: contact@trainnomade.fr

---

## En français

**Desserte des gares de France** : pour chacune des 2 792 gares de voyageurs SNCF, le nombre moyen de trains par jour
ouvré, le samedi et le dimanche, les cars TER, le premier et le dernier départ, le nombre de destinations directes et les
types de trains. Un second fichier donne chaque destination atteinte sans changement. Recalculé chaque nuit sur les
14 jours qui suivent : https://api.trainnomade.fr/donnees/v1/desserte-gares.csv et
https://api.trainnomade.fr/donnees/v1/desserte-gares-destinations.csv (page : https://trainnomade.fr/donnees/desserte-gares/).

| Colonne | Contenu |
| --- | --- |
| `code_uic` | code UIC de la gare (8 chiffres), celui des horaires SNCF |
| `autres_codes_uic` | autres codes UIC de la même gare (partie banlieue d'une gare parisienne), séparés par `;` |
| `nom` | nom officiel de la gare |
| `code_commune` | code INSEE de la commune |
| `latitude` | latitude (WGS84) |
| `longitude` | longitude (WGS84) |
| `segment_drg` | catégorie SNCF : `A` nationale, `B` régionale, `C` locale |
| `desserte_connue` | `oui` si au moins un train ou car de la source s'y arrête sur la période ; sinon cellules vides |
| `trains_jour_ouvre` | trains par jour ouvré (lundi-vendredi hors fériés), moyenne |
| `trains_samedi` | trains par samedi, moyenne |
| `trains_dimanche` | trains par dimanche, moyenne |
| `cars_jour_ouvre` | cars TER, cars à réservation et navettes par jour ouvré, moyenne |
| `premier_depart` | premier départ d'un train un jour ouvré (HH:MM, heure de Paris) |
| `dernier_depart` | dernier départ d'un train un jour ouvré (peut être après minuit) |
| `destinations_directes` | gares atteintes en train sans changement |
| `types_trains` | types de trains qui s'arrêtent, séparés par `;` |
| `periode_debut` | premier jour compté |
| `periode_fin` | dernier jour compté |
| `code_uic_depart` | code UIC de la gare de départ (fichier des destinations) |
| `gare_depart` | gare de départ |
| `code_uic_arrivee` | code UIC de la gare atteinte |
| `gare_arrivee` | gare atteinte (parfois à l'étranger) |
| `trains_par_semaine` | trains directs par semaine |
| `duree_min_minutes` | durée la plus courte prévue, en minutes |

Les colonnes `premier_depart`, `dernier_depart` et `types_trains` du fichier des destinations ont le même sens, pour la
liaison.

- Méthode : comptage des courses des horaires théoriques SNCF qui s'arrêtent à la gare (montée ou descente permise),
  jour par jour ; moyenne des jours ouvrés hors fériés, des samedis et des dimanches ; un départ après minuit appartient
  au jour de service commencé la veille ; une gare à plusieurs codes UIC tient sur une ligne.
- Contrôles avant chaque mise à jour (sinon l'ancienne version reste en ligne) : au moins 2 700 gares, au moins
  2 000 gares desservies et 90 % de l'instantané précédent, codes uniques, période de 7 à 14 jours, valeurs plausibles,
  minimum pour quatre grandes gares, au moins 20 000 liaisons directes.
- Sources : horaires GTFS « Réseau SNCF TGV, Intercités et TER » de SNCF Voyageurs
  (https://transport.data.gouv.fr/datasets/horaires-sncf, ODbL) ; « Gares de voyageurs » de SNCF
  (https://ressources.data.sncf.com/explore/dataset/gares-de-voyageurs/, ODbL).
- Limites : horaires prévus (ni retards ni suppressions) ; Transilien, RER, Eurostar, trains touristiques et autres
  opérateurs absents de la source (gares parisiennes : grandes lignes et TER seulement) ; travaux et fériés de la période
  pèsent sur les moyennes ; « Train » est le libellé de la source pour des trains régionaux sans marque TER.
- Citation : **« Desserte des gares : Train Nomade (https://trainnomade.fr), d'après les horaires SNCF Voyageurs
  (ODbL) »**. Licence **ODbL 1.0** : partage à l'identique de toute base dérivée. Train Nomade est un site indépendant,
  sans lien avec la SNCF.
