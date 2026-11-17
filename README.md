# carto-structures-mda

Cartographie interactive des structures d'accompagnement (établissements et services médico-sociaux, guichets publics, associations) par département.

L'outil tient dans **un seul fichier HTML** (HTML, CSS et JavaScript, sans dépendance externe à installer). Il s'ouvre par double-clic dans un navigateur, **sans serveur**, et interroge en direct des API publiques de l'État.

Conçu à la MDA de l'Orne pour repérer, présenter et partager l'offre d'accompagnement d'un territoire.

---

## Fonctionnalités

- **Carte de France des départements** : cliquez un département pour le sélectionner, Ctrl + clic pour en ajouter d'autres. Tout ce qui est hors de la sélection est voilé.
- **Données chargées automatiquement** pour chaque département sélectionné :
  - guichets publics de l'Annuaire de l'administration (CAF, France Travail, Mission locale, CLIC, PMI, tribunaux…) ;
  - établissements ayant un n° FINESS, via l'API Recherche d'entreprises.
- **Dix thématiques** filtrables (Accueil et orientation, Emploi et formation, Logement, Parcours de soins, Prestation à domicile, Scolarité, Démarches numériques, Protection judiciaire, ESMS, Association de soutien), plus « Autres ».
- **Recherche** par nom, commune, gestionnaire ou n° FINESS.
- **Fiche détaillée au clic** : type, gestionnaire, adresse, téléphone, courriel, site, horaires d'ouverture, n° FINESS, SIRET. Liens vers :
  - la fiche **ViaTrajectoire** (calculée à partir du n° FINESS) ;
  - la fiche Service-public ou Annuaire des entreprises ;
  - un itinéraire Google Maps et OpenStreetMap.
- **Pointeur de sélection** sur l'établissement ouvert.
- **Fonds de carte au choix** : Plan IGN, fond épuré sans routes, photographies aériennes, OpenStreetMap. Les axes routiers peuvent être superposés à volonté.
- **Import CSV** (fenêtre « Données… ») pour les structures absentes des API : saisies manuelles, extraction FINESS, permanences, associations locales. Les colonnes sont reconnues automatiquement. Les adresses sans coordonnées sont géocodées.
- **Export GeoJSON** de tout ce qui est affiché.
- **Mémorisation locale** (dans le navigateur) des imports, du fond de carte et de l'affichage des routes.

## Utilisation

1. Téléchargez `cartographie-mda.html`.
2. Ouvrez-le dans Edge, Chrome ou Firefox (double-clic).
3. Cliquez un département sur la carte, ou choisissez-le dans la liste puis « Charger ».
4. Cliquez un marqueur ou un nom dans la liste de gauche pour ouvrir sa fiche.

Une connexion Internet est nécessaire pour les fonds de carte IGN et OpenStreetMap, les API et le géocodage. Sans connexion restent disponibles : le fond épuré, les contours des départements, les données importées, la recherche et l'export.

## Sources de données

| Source | Contenu | Mode d'accès |
| --- | --- | --- |
| [API Annuaire de l'administration](https://www.data.gouv.fr/dataservices/api-annuaire-de-ladministration-competence-geographique-des-services-locaux) (DILA, Service-public.gouv.fr) | Guichets publics locaux : adresse, téléphone, horaires, géolocalisation | En direct, par département |
| [API Recherche d'entreprises](https://www.data.gouv.fr/dataservices/api-recherche-dentreprises/) (DINUM) | Établissements ayant un n° FINESS, avec coordonnées et activité (NAF) | En direct, par département (7 requêtes/s maximum) |
| Base Adresse Nationale | Géocodage des adresses importées sans coordonnées | En direct, à l'import |
| [france-geojson](https://github.com/gregoiredavid/france-geojson) (d'après IGN ADMIN EXPRESS) | Contours simplifiés des 96 départements de métropole | Intégré au fichier |
| Géoplateforme IGN, OpenStreetMap | Fonds de carte | Tuiles en direct |
| ViaTrajectoire | Fiche publique d'un établissement | Lien construit à partir du n° FINESS |

Les données restent la propriété de leurs producteurs. Merci de respecter leurs conditions de réutilisation, notamment la mention de la source, par exemple « Service-Public.gouv.fr / DILA » pour l'Annuaire de l'administration.

## Import CSV

Séparateur `;` ou `,`, encodage UTF-8. Les en-têtes reconnus automatiquement sont les suivants ; la correspondance reste modifiable avant l'import.

```
finess;nom;categorie;thematique;type;adresse;code_postal;commune;telephone;courriel;site;horaires;latitude;longitude
```

- `categorie` : code de catégorie FINESS (ex. `246` pour un ESAT). Il sert à déterminer la thématique et le lien ViaTrajectoire.
- `thematique` : libellé exact d'une des thématiques. S'il est présent, il est prioritaire.
- `latitude` / `longitude` : facultatives. Sans elles, l'adresse est géocodée.

Un modèle est fourni : `modele-import.csv`.

## Configuration

Les réglages se trouvent en tête du script, dans le fichier HTML :

- `CFG` : adresses des API, des géocodeurs et schéma des liens ViaTrajectoire.
  - Handicap : `https://usager.viatrajectoire.fr/fiche-etablissement/personnes-situation-handicap/{finess}`.
  - Personnes âgées : schéma non renseigné (`null`).
- `THEMES` : thématiques, libellés et couleurs.
- `PIVOT` : rattachement des types de guichets de l'Annuaire (codes pivot DILA) aux thématiques.
- `FINESS_CAT` : rattachement des catégories FINESS (pour les CSV importés) aux thématiques.
- `NAF` : rattachement des codes d'activité NAF (pour l'API Recherche d'entreprises) aux thématiques.

## Limites connues

- **Classement des établissements de l'API.** Les établissements issus de l'API Recherche d'entreprises sont classés par **code NAF**, plus grossier que la catégorie FINESS. Un même code peut ainsi regrouper IME, ITEP et SESSAD. Pour une typologie fine, importez un CSV avec la colonne `categorie`.
- **Outre-mer.** Les départements d'outre-mer sont sélectionnables dans la liste, mais leurs contours ne sont pas dessinés.
- **Rattachement géographique.** Les contours sont simplifiés : une structure située tout près d'une limite peut être rattachée au département voisin.
- **Liens ViaTrajectoire.** Ils ne sont générés que pour les établissements du handicap.
- **Disponibilité des sources.** Le fonctionnement dépend de la disponibilité des API et de leur acceptation des requêtes émises depuis un fichier local. Un proxy d'entreprise peut aussi les bloquer.
- **Persistance des imports.** Les imports sont conservés dans le navigateur utilisé uniquement : ils ne sont ni partagés ni sauvegardés ailleurs. Pensez à l'export GeoJSON.

## Technique

- JavaScript sans framework ni étape de compilation.
- [Leaflet](https://leafletjs.com/) 1.9.4 et [Leaflet.markercluster](https://github.com/Leaflet/Leaflet.markercluster) 1.5.3, intégrés au fichier.

## Licences des composants intégrés

- Leaflet : BSD-2-Clause.
- Leaflet.markercluster : MIT.
- Contours des départements : voir le dépôt [france-geojson](https://github.com/gregoiredavid/france-geojson).

## Auteur

Frédérick Murat — MDA de l'Orne.
