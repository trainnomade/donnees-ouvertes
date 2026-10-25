# Train Nomade : données ouvertes ferroviaires

Jeux de données ouverts sur les trains en France, publiés par [Train Nomade](https://trainnomade.fr) : desserte de
chaque gare, tracés des voies entre deux arrêts, trains de nuit, correspondances entre la gare et les bus, trams et cars.
Ils sont calculés à partir des données ouvertes officielles (horaires SNCF Voyageurs, réseaux de transport locaux,
OpenStreetMap) et publiés sous licence [ODbL 1.0](LICENSE).

*English below.*

## Les jeux

| Jeu | Contenu | Mise à jour | Fichiers en ligne | data.gouv.fr |
| --- | --- | --- | --- | --- |
| [Correspondances gare-bus, tram et cars](correspondances-gares/) | arrêts à moins de 350 m des gares, lignes et horaires types | chaque semaine | [`correspondances-gares.csv`](https://api.trainnomade.fr/donnees/v1/correspondances-gares.csv) | [fiche](https://www.data.gouv.fr/datasets/correspondances-gare-bus-tram-et-cars-arrets-a-moins-de-350-m-des-gares-lignes-et-horaires-types) |
| [Desserte des gares de France](desserte-gares/) | trains par jour, premier et dernier départ, destinations directes | chaque nuit | [`desserte-gares.csv`](https://api.trainnomade.fr/donnees/v1/desserte-gares.csv), [`desserte-gares-destinations.csv`](https://api.trainnomade.fr/donnees/v1/desserte-gares-destinations.csv) | [fiche](https://www.data.gouv.fr/datasets/desserte-des-gares-de-france-trains-par-jour-premier-et-dernier-depart-destinations-directes) |
| [Trains de nuit au départ de la France](horaires-trains-de-nuit/) | circulations prévues, arrêts et horaires | chaque nuit | [`horaires-trains-de-nuit.csv`](https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit.csv), [`horaires-trains-de-nuit-arrets.csv`](https://api.trainnomade.fr/donnees/v1/horaires-trains-de-nuit-arrets.csv) | [fiche](https://www.data.gouv.fr/datasets/trains-de-nuit-au-depart-de-la-france-circulations-prevues-arrets-et-horaires) |
| [Tracés ferroviaires entre deux arrêts en France](traces-ferroviaires/) | géométrie, longueur, vitesse, ponts, tunnels | chaque mois | [`traces-ferroviaires.csv`](https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.csv), [`traces-ferroviaires.geojson`](https://api.trainnomade.fr/donnees/v1/traces-ferroviaires.geojson), [`traces-ferroviaires-shapes.txt`](https://api.trainnomade.fr/donnees/v1/traces-ferroviaires-shapes.txt) | [fiche](https://www.data.gouv.fr/datasets/traces-ferroviaires-entre-deux-arrets-en-france-geometrie-longueur-vitesse-ponts-tunnels) |

Chaque dossier contient la description complète du jeu (colonnes, méthode, sources, limites), sa licence, un fichier
`CITATION.cff` et un **instantané daté** des données dans `data/` (sa date et ses chiffres sont dans le README du jeu). Pour
la version la plus récente, lisez les fichiers en ligne : ils sont recalculés au rythme indiqué dans le tableau.

Format : CSV en UTF-8, séparateur virgule, en-tête sur la première ligne, heures de Paris ; GeoJSON en WGS84. Une cellule
vide veut dire « inconnu » ou « sans objet », jamais zéro.

## Lire les données

```sh
curl -sO https://api.trainnomade.fr/donnees/v1/desserte-gares.csv
```

```python
import pandas as pd

gares = pd.read_csv("https://api.trainnomade.fr/donnees/v1/desserte-gares.csv", dtype={"code_uic": str, "code_commune": str})
print(gares.nlargest(10, "trains_jour_ouvre")[["nom", "trains_jour_ouvre"]])
```

```javascript
const texte = await (await fetch("https://api.trainnomade.fr/donnees/v1/desserte-gares.csv")).text();
const [entete, ...lignes] = texte.trim().split("\n");
console.log(entete.split(","), lignes.length, "gares");
```

Les fichiers se lisent depuis un serveur comme depuis une page web (CORS ouvert), sans clé ni compte. Limite : 30 requêtes
au plus par période de 10 secondes et par adresse IP pour l'ensemble de `api.trainnomade.fr` (au-delà : réponse 429).

## API en temps réel

Train Nomade publie aussi une API ouverte : trains SNCF du jour avec retard et voie, départs et arrivées d'une gare,
disponibilités TGV MAX. Documentation, exemples et conditions : https://trainnomade.fr/api/

## Licence et crédit

Les données sont sous **Open Database License (ODbL) 1.0** : vous pouvez les copier, les modifier et les réutiliser, y
compris dans un service commercial, à deux conditions : **citer la source** (le crédit à afficher est donné dans chaque
dossier, par exemple « Desserte des gares : Train Nomade (https://trainnomade.fr), d'après les horaires SNCF Voyageurs
(ODbL) ») et **partager à l'identique** toute base de données qui en dérive. Les exemples de code de ce fichier sont
libres de droits.

Train Nomade est un projet indépendant, sans lien avec la SNCF ni avec les opérateurs cités. Ne pas confondre avec
d'autres services au nom proche : le nôtre est **Train Nomade**, à l'adresse trainnomade.fr.

## Signaler une erreur

Une valeur fausse, une gare manquante, un besoin d'un autre format : ouvrez un
[ticket](https://github.com/trainnomade/donnees-ouvertes/issues/new/choose) ou écrivez à contact@trainnomade.fr. Pour
une faille de sécurité, voir [SECURITY.md](SECURITY.md).

---

## English

Open datasets about trains in France, published by [Train Nomade](https://trainnomade.fr): service at every railway
station, railway geometry between two stops, night trains, connections between stations and local buses, trams and
coaches. They are computed from official open data (SNCF Voyageurs timetables, local transport networks, OpenStreetMap)
and released under the [Open Database License 1.0](LICENSE). Each folder holds the full description (columns, method,
sources, limits), the licence, a `CITATION.cff` file and a dated snapshot in `data/`; the live files listed above are
recomputed every night, week or month. CSV files are UTF-8, comma separated, Paris local time. Free access, no key, CORS
enabled, 30 requests per 10 seconds per IP address. Credit the source as written in each folder and share derived
databases under the same licence. Train Nomade is independent and not affiliated with SNCF. Contact:
contact@trainnomade.fr
