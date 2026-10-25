# Railway track geometry between consecutive stops in France: length, speed, bridges, tunnels, electrification

The track followed by passenger trains between every pair of consecutive stops of the SNCF timetables (about 3,200
sections in France and towards neighbouring countries), routed on the railway tracks of OpenStreetMap, with its length,
maximum speed, bridges, tunnels and electrification. Three formats: a CSV of attributes, a simplified GeoJSON and a
GTFS `shapes.txt` whose identifiers are the UIC codes of the SNCF GTFS. Recomputed every month by
[Train Nomade](https://trainnomade.fr/donnees/traces-ferroviaires/).

*Version française plus bas.*

- Files: [`data/traces-ferroviaires.csv`](data/traces-ferroviaires.csv),
  [`data/traces-ferroviaires.geojson`](data/traces-ferroviaires.geojson) and
  [`data/traces-ferroviaires-shapes.txt`](data/traces-ferroviaires-shapes.txt)
- Live versions (recomputed every month): https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.csv,
  https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.geojson and
  https://api.trainnomade.fr/donnees/v1/traces-ferroviaires-shapes.txt
- Description page: https://trainnomade.fr/donnees/traces-ferroviaires/

CSV files are comma separated, UTF-8, with the header on the first line. An empty cell means "unknown", never zero. The
GeoJSON follows RFC 7946 (WGS84 longitude, latitude; 5 decimals, about 1 m).

## `traces-ferroviaires.csv`: one line per section

One line per section (pair of stations served one after the other by at least one train), sorted by `troncon_id`. Both
directions are grouped: the published direction goes from the smaller UIC code (`code_uic_a`) to the larger one
(`code_uic_b`).

| Column | Format | Content |
| --- | --- | --- |
| `troncon_id` | `UIC1-UIC2` | section identifier: UIC codes of the two stations, smaller first |
| `code_uic_a` | 8 digits | UIC code of the first station (as in the SNCF GTFS, `StopArea:OCE` + code) |
| `gare_a` | text | name of the first station |
| `code_uic_b` | 8 digits | UIC code of the second station |
| `gare_b` | text | name of the second station |
| `longueur_m` | integer, metres | length along the track |
| `vol_oiseau_m` | integer, metres | straight-line distance between the two stations |
| `vitesse_max_kmh` | integer, km/h | highest speed limit along the section (OpenStreetMap `maxspeed`, see limits) |
| `nb_ponts` | integer | bridges and viaducts of 30 m or more |
| `longueur_ponts_m` | integer, metres | total length of these bridges |
| `nb_tunnels` | integer | tunnels of 30 m or more |
| `longueur_tunnels_m` | integer, metres | total length of these tunnels |
| `longueur_electrifiee_m` | integer, metres | length of electrified track |
| `longueur_electrification_inconnue_m` | integer, metres | length whose electrification is not recorded in OpenStreetMap |
| `part_electrifiee` | decimal 0 to 1, 3 decimals | `longueur_electrifiee_m` divided by `longueur_m` |
| `deux_sens` | `oui` or `non` | `oui` if trains run the section in both directions in the timetables |
| `nb_points` | integer | points of the simplified geometry (GeoJSON and shapes) |
| `date_calcul` | `YYYY-MM-DD` | date of the computation (OpenStreetMap extract and timetable of that day) |

```
troncon_id,code_uic_a,gare_a,code_uic_b,gare_b,longueur_m,vol_oiseau_m,vitesse_max_kmh,nb_ponts,longueur_ponts_m,nb_tunnels,longueur_tunnels_m,longueur_electrifiee_m,longueur_electrification_inconnue_m,part_electrifiee,deux_sens,nb_points,date_calcul
71043075-87784009,71043075,Figueres-Vilafant,87784009,Perpignan,53972,48263,300,16,3490,5,10460,53972,0,1,oui,89,2026-10-17
```

## `traces-ferroviaires.geojson`

A `FeatureCollection` of `LineString` features, one per section, in the order of the CSV. Each feature has `id` =
`troncon_id` and the same properties as the CSV columns above. The geometry goes from station A to station B and is
simplified with the Douglas-Peucker method: no point of the computed track is more than 5 m away from it.

## `traces-ferroviaires-shapes.txt`: GTFS shapes

The same geometries in the GTFS `shapes.txt` format, ready to be added to the SNCF GTFS feed: `shape_id` is the
`troncon_id`, so the shape of a trip between two consecutive stops `StopArea:OCE87713040` and `StopArea:OCE87713545` is
`87713040-87713545` (or the reverse of it when the trip goes the other way).

| Column | Format | Content |
| --- | --- | --- |
| `shape_id` | `UIC1-UIC2` | section identifier (`troncon_id`) |
| `shape_pt_lat` | decimal degrees, WGS84 | latitude of the point |
| `shape_pt_lon` | decimal degrees, WGS84 | longitude of the point |
| `shape_pt_sequence` | integer from 1 | order of the point along the section, from station A to station B |
| `shape_dist_traveled` | decimal, metres | distance along the track from station A |

```
shape_id,shape_pt_lat,shape_pt_lon,shape_pt_sequence,shape_dist_traveled
71043075-71793000,42.264638,2.942736,1,0
```

## Method

1. Sections: pairs of stations served one after the other by at least one train of the SNCF GTFS feed of the day of
   computation (coaches excluded).
2. Tracks: OpenStreetMap railway lines in service (`railway=rail`, main, branch and high-speed lines, station tracks)
   from the Geofabrik extracts of the French regions and of the neighbouring regions reached by these trains.
3. Routing: fastest path on the track graph between the two stations (travel time from `maxspeed`, preferred direction
   of each track respected, no reversal at a switch), resampled every 10 m to compute lengths and attributes.
4. Quality filter: a track is left out when it is much longer than the timetable allows, when it makes a large detour
   (more than 2.5 times the straight line) or when it reverses; about 0.3 % of the sections are left out this way.
5. Bridges and tunnels: OpenStreetMap `bridge` and `tunnel` tags on runs of 30 m or more (`building_passage` under a
   station roof is not a tunnel).
6. Checks before each update (the previous version stays online if one fails): at least 3,000 sections and 95 % of the
   previous snapshot, total length between 30,000 and 120,000 km, speeds between 10 and 350 km/h, bridge and tunnel
   lengths not above the section length, the three files describing the same sections, coordinates in Western Europe,
   each file under 25 MB.

## Content of this snapshot (computation of 17 October 2026)

3,218 sections (3,035 run in both directions, 115 with a station abroad), 77,176 km of track summed over the sections
(overlaps included), 195,680 points. 2,171 sections are fully electrified, 864 not at all; 126 reach 300 km/h or more.

## Sources

- **Tracks**: © OpenStreetMap contributors, data under the Open Database License
  (https://www.openstreetmap.org/copyright), regional extracts by Geofabrik (https://download.geofabrik.de/).
- **Sections and station codes**: SNCF Voyageurs, GTFS dataset "Réseau SNCF TGV, Intercités et TER"
  (https://transport.data.gouv.fr/datasets/horaires-sncf), licence ODbL.

## Limits

- Computed geometry, not an official survey of the infrastructure manager (SNCF Réseau): the path is the one the
  OpenStreetMap tracks allow; at large junctions the exact track used by a given train may differ.
- `vitesse_max_kmh` is the highest `maxspeed` tag found along the section; where a track has no `maxspeed`, the routing
  uses a default by type of line (300 km/h high-speed, 120 main line, 80 branch line) and that default can appear here.
  It is a speed limit of the track, not the speed of the trains.
- Sections overlap (a non-stop TGV and a stopping TER use the same tracks): do not add up lengths to measure the
  network. One direction per pair is published; for the other direction, read the geometry backwards.
- Sections whose station is outside the covered extracts (a few foreign terminuses) are missing.

## How to cite

> Train Nomade, *Railway track geometry between consecutive stops in France*, from © OpenStreetMap contributors and
> SNCF Voyageurs timetables (ODbL). Version of [date]. https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.geojson

## Licence

This database is made available under the **Open Database License (ODbL) 1.0**, see [LICENSE](LICENSE). Credit to
display: **"Tracés ferroviaires : Train Nomade (https://trainnomade.fr), d'après © les contributeurs OpenStreetMap et les
horaires SNCF Voyageurs (ODbL)"**. If you publish a database derived from it, share it under the same licence.

Contact: contact@trainnomade.fr

---

## En français

**Tracés ferroviaires entre deux arrêts** : le tracé sur les voies de chaque tronçon voyageurs entre deux arrêts
consécutifs des trains SNCF (≈ 3 200 tronçons, France et liaisons vers les pays voisins), calculé sur les voies
d'OpenStreetMap, avec longueur, vitesse maximale, ponts, tunnels et électrification. Trois formats : CSV des
attributs, GeoJSON simplifié (WGS84) et `shapes.txt` au format GTFS, identifiants = codes UIC du GTFS SNCF. Recalculé
chaque mois : https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.csv,
https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.geojson et
https://api.trainnomade.fr/donnees/v1/traces-ferroviaires-shapes.txt (page : https://trainnomade.fr/donnees/traces-ferroviaires/).

| Colonne | Contenu |
| --- | --- |
| `troncon_id` | identifiant du tronçon : codes UIC des deux gares, le plus petit d'abord |
| `code_uic_a` | code UIC de la première gare |
| `gare_a` | nom de la première gare |
| `code_uic_b` | code UIC de la seconde gare |
| `gare_b` | nom de la seconde gare |
| `longueur_m` | longueur sur la voie, en mètres |
| `vol_oiseau_m` | distance à vol d'oiseau, en mètres |
| `vitesse_max_kmh` | vitesse maximale autorisée la plus haute du tronçon (OpenStreetMap), km/h |
| `nb_ponts` | ponts et viaducs de 30 m ou plus |
| `longueur_ponts_m` | longueur totale de ces ponts, en mètres |
| `nb_tunnels` | tunnels de 30 m ou plus |
| `longueur_tunnels_m` | longueur totale de ces tunnels, en mètres |
| `longueur_electrifiee_m` | longueur de voie électrifiée, en mètres |
| `longueur_electrification_inconnue_m` | longueur dont l'électrification n'est pas renseignée dans OpenStreetMap |
| `part_electrifiee` | part électrifiée, de 0 à 1 |
| `deux_sens` | `oui` si des trains parcourent le tronçon dans les deux sens |
| `nb_points` | points de la géométrie simplifiée |
| `date_calcul` | date du calcul (extrait OpenStreetMap et horaires de ce jour) |
| `shape_id` | identifiant de la forme GTFS (= `troncon_id`) |
| `shape_pt_lat` | latitude du point |
| `shape_pt_lon` | longitude du point |
| `shape_pt_sequence` | rang du point, de la gare A à la gare B |
| `shape_dist_traveled` | distance depuis la gare A le long de la voie, en mètres |

- Méthode : paires de gares desservies l'une après l'autre par au moins un train du GTFS SNCF ; chemin le plus rapide
  sur les voies OpenStreetMap en service (vitesses `maxspeed`, sens préférentiel respecté, pas de rebroussement) ;
  tracés douteux écartés (≈ 0,3 %) ; ponts et tunnels de 30 m et plus ; géométrie simplifiée à 5 m près
  (Douglas-Peucker).
- Contrôles avant chaque mise à jour (sinon l'ancienne version reste en ligne) : au moins 3 000 tronçons et 95 % de
  l'instantané précédent, longueur totale de 30 000 à 120 000 km, vitesses de 10 à 350 km/h, longueurs cohérentes,
  mêmes tronçons dans les trois fichiers, coordonnées en Europe de l'Ouest, fichiers de moins de 25 Mo.
- Sources : © contributeurs OpenStreetMap (https://www.openstreetmap.org/copyright, ODbL, extraits Geofabrik) ; horaires
  GTFS « Réseau SNCF TGV, Intercités et TER » de SNCF Voyageurs (https://transport.data.gouv.fr/datasets/horaires-sncf,
  ODbL).
- Limites : tracé calculé, pas un relevé officiel de SNCF Réseau ; vitesse maximale lue dans OpenStreetMap (valeur par
  défaut selon le type de ligne là où elle manque : 300, 120 ou 80 km/h), vitesse de la voie et non des trains ; les
  tronçons se superposent, ne pas additionner les longueurs ; un seul sens publié par paire (sens inverse : géométrie à
  l'envers).
- Citation : **« Tracés ferroviaires : Train Nomade (https://trainnomade.fr), d'après © les contributeurs OpenStreetMap
  et les horaires SNCF Voyageurs (ODbL) »**. Licence **ODbL 1.0** : partage à l'identique de toute base dérivée.
  Train Nomade est un site indépendant, sans lien avec la SNCF ni avec SNCF Réseau.
