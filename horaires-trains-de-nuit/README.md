# Night trains from France: every scheduled run, its stops and times

Every scheduled run of the night trains departing from France that are published as open timetables (Intercités de nuit
of SNCF Voyageurs), from today to the end of the published timetable: date, train number, origin, terminus, departure
and arrival times, duration, and every stop with its times and whether passengers may board or alight. Updated every
night by [Train Nomade](https://trainnomade.fr/donnees/horaires-trains-de-nuit/).

*Version française plus bas.*

- Files: [`data/horaires-trains-de-nuit.csv`](data/horaires-trains-de-nuit.csv) and
  [`data/horaires-trains-de-nuit-arrets.csv`](data/horaires-trains-de-nuit-arrets.csv)
- Live versions (updated every night): https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit.csv and
  https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit-arrets.csv
- Description page: https://trainnomade.fr/donnees/horaires-trains-de-nuit/

Both files are comma separated, UTF-8, with the header on the first line. Times are Paris local time (`HH:MM`); the
"day" columns count days from the departure date of the train (`0` = same day, `1` = next morning). An empty cell
means "not applicable" (no arrival time at the origin, no departure time at the terminus).

## `horaires-trains-de-nuit.csv`: one line per run

One line per scheduled run, sorted by departure date, departure time and train number.

| Column | Format | Content |
| --- | --- | --- |
| `id_circulation` | `YYYY-MM-DD-number-UIC` | identifier of the run: departure date, train number, UIC code of the terminus |
| `date_depart` | `YYYY-MM-DD` | date on which the train leaves its origin |
| `numero_train` | text | commercial train number |
| `operateur` | text | railway undertaking (`SNCF Voyageurs`) |
| `marque` | text | commercial brand (`Intercités de nuit`) |
| `code_uic_origine` | 8 digits | UIC code of the origin station |
| `origine` | text | origin station |
| `code_uic_destination` | 8 digits | UIC code of the terminus |
| `destination` | text | terminus |
| `heure_depart` | `HH:MM` | departure from the origin |
| `heure_arrivee` | `HH:MM` | arrival at the terminus |
| `jour_arrivee` | integer, days | day of arrival at the terminus, counted from `date_depart` (`1` = next day) |
| `duree_minutes` | integer, minutes | scheduled duration from origin to terminus |
| `nb_arrets` | integer | number of stops, origin and terminus included |

```
id_circulation,date_depart,numero_train,operateur,marque,code_uic_origine,origine,code_uic_destination,destination,heure_depart,heure_arrivee,jour_arrivee,duree_minutes,nb_arrets
2026-10-25-5772-87547000,2026-10-25,5772,SNCF Voyageurs,Intercités de nuit,87756056,Nice,87547000,Paris Austerlitz,18:57,07:53,1,776,8
```

## `horaires-trains-de-nuit-arrets.csv`: one line per stop

One line per stop of each run, in the order of the runs file, then by `ordre`.

| Column | Format | Content |
| --- | --- | --- |
| `id_circulation` | text | identifier of the run (`id_circulation` of the runs file) |
| `date_depart` | `YYYY-MM-DD` | departure date of the train from its origin |
| `numero_train` | text | commercial train number |
| `ordre` | integer from 1 | order of the stop along the run |
| `code_uic` | 8 digits | UIC code of the station |
| `gare` | text | station name |
| `heure_arrivee` | `HH:MM` | arrival time; empty at the origin |
| `jour_arrivee` | integer, days | day of the arrival, counted from `date_depart`; empty at the origin |
| `heure_depart` | `HH:MM` | departure time; empty at the terminus |
| `jour_depart` | integer, days | day of the departure, counted from `date_depart`; empty at the terminus |
| `montee` | `oui` or `non` | boarding allowed at this stop |
| `descente` | `oui` or `non` | alighting allowed at this stop |

```
id_circulation,date_depart,numero_train,ordre,code_uic,gare,heure_arrivee,jour_arrivee,heure_depart,jour_depart,montee,descente
2026-10-25-3970-87547000,2026-10-25,3970,2,87611509,Porté-Puymorens,19:03,0,19:05,0,oui,non
```

## Method

- Trips of the SNCF GTFS feed whose stops carry the product "INTERCITES de nuit"; one line per day on which the trip
  runs (`calendar_dates.txt`), from the day of computation to the last date of the feed.
- Times of the feed are counted from midnight of the departure day and may exceed 24:00; they are written as Paris
  clock times with the number of days in `jour_arrivee` and `jour_depart`.
- Boarding and alighting come from `pickup_type` and `drop_off_type`: on the way out, night trains often only let
  passengers board at the first stops and only alight at the last ones.
- Checks before each update (the previous version stays online if one fails): at least 100 runs and 4 train numbers,
  first date within 7 days, durations between 3 and 18 hours, unique identifiers, same runs and number of stops in both
  files.

## Content of this snapshot (25 October 2026)

805 runs from 25 October 2026 to 28 March 2027, 29 train numbers, 87 stations, all from or to Paris Austerlitz (two runs
use Paris Bercy): Tarbes (Lourdes), Toulouse, Latour-de-Carol, Cerbère, Rodez, Albi, Aurillac, Figeac, Nice and Briançon.

## Sources

- **Timetables**: SNCF Voyageurs, GTFS dataset "Réseau SNCF TGV, Intercités et TER"
  (https://transport.data.gouv.fr/datasets/horaires-sncf; file
  https://eu.ftp.opendatasoft.com/sncf/plandata/Export_OpenData_SNCF_GTFS_NewTripId.zip), licence ODbL.

## Limits

- Scheduled timetables only: no delays, last-minute cancellations or replacement coaches.
- SNCF publishes its timetables in steps: the last weeks of the period are often incomplete and fill in as new versions
  are published (the file is recomputed every night).
- Other night trains from France (European Sleeper between Paris and Berlin, for instance) do not publish reusable open
  timetables; they will be added when they do. Trains that only cross France are not included.
- No seat availability and no fares.

## How to cite

> Train Nomade, *Night trains from France: every scheduled run, its stops and times*, from SNCF Voyageurs timetables
> (GTFS, ODbL). Version of [date]. https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit.csv

## Licence

This database is made available under the **Open Database License (ODbL) 1.0**, see [LICENSE](LICENSE). Credit to
display: **"Trains de nuit : Train Nomade (https://trainnomade.fr), d'après les horaires SNCF Voyageurs (ODbL)"**. If you
publish a database derived from it, share it under the same licence.

Contact: contact@trainnomade.fr

---

## En français

**Trains de nuit au départ de la France** : chaque circulation prévue des Intercités de nuit de SNCF Voyageurs,
d'aujourd'hui à la fin des horaires publiés, avec ses arrêts et ses horaires. Mis à jour chaque nuit :
https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit.csv et
https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit-arrets.csv (page :
https://trainnomade.fr/donnees/horaires-trains-de-nuit/).

| Colonne | Contenu |
| --- | --- |
| `id_circulation` | identifiant de la circulation : date de départ, numéro de train, code UIC du terminus |
| `date_depart` | date à laquelle le train quitte sa gare d'origine |
| `numero_train` | numéro commercial du train |
| `operateur` | entreprise ferroviaire (`SNCF Voyageurs`) |
| `marque` | marque commerciale (`Intercités de nuit`) |
| `code_uic_origine` | code UIC de la gare d'origine |
| `origine` | gare d'origine |
| `code_uic_destination` | code UIC du terminus |
| `destination` | terminus |
| `heure_depart` | heure de départ de l'origine (heure de Paris) |
| `heure_arrivee` | heure d'arrivée au terminus (dans le fichier des circulations) ou à l'arrêt (dans le fichier des arrêts) |
| `jour_arrivee` | jour de l'arrivée compté depuis `date_depart` (1 = le lendemain) |
| `duree_minutes` | durée prévue de l'origine au terminus, en minutes |
| `nb_arrets` | nombre d'arrêts, origine et terminus compris |
| `ordre` | rang de l'arrêt dans la circulation |
| `code_uic` | code UIC de la gare de l'arrêt |
| `gare` | nom de la gare de l'arrêt |
| `jour_depart` | jour du départ de l'arrêt compté depuis `date_depart` |
| `montee` | montée permise (`oui`/`non`) |
| `descente` | descente permise (`oui`/`non`) |

- Méthode : courses des horaires théoriques SNCF portant le produit « INTERCITES de nuit », une ligne par jour de
  circulation ; les heures du GTFS (qui dépassent 24:00 après minuit) sont écrites en heure de Paris avec le nombre de
  jours dans `jour_arrivee` et `jour_depart` ; montée et descente d'après `pickup_type` et `drop_off_type`.
- Contrôles avant chaque mise à jour (sinon l'ancienne version reste en ligne) : au moins 100 circulations et
  4 numéros de train, première date sous 7 jours, durées de 3 à 18 heures, identifiants uniques, mêmes circulations et
  même nombre d'arrêts dans les deux fichiers.
- Cet instantané (25 octobre 2026) : 805 circulations du 25 octobre 2026 au 28 mars 2027, 29 numéros de train,
  87 gares, toutes au départ ou à destination de Paris-Austerlitz (deux par Paris-Bercy).
- Source : horaires GTFS « Réseau SNCF TGV, Intercités et TER » de SNCF Voyageurs
  (https://transport.data.gouv.fr/datasets/horaires-sncf, ODbL).
- Limites : horaires prévus, sans retards ni suppressions ; horaires publiés par étapes (dernières semaines souvent
  incomplètes) ; European Sleeper et les autres opérateurs ne publient pas d'horaires ouverts réutilisables ; ni places
  ni prix.
- Citation : **« Trains de nuit : Train Nomade (https://trainnomade.fr), d'après les horaires SNCF Voyageurs (ODbL) »**.
  Licence **ODbL 1.0** : partage à l'identique de toute base dérivée. Train Nomade est un site indépendant, sans lien
  avec la SNCF.
