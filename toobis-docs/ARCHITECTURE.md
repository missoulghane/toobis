# OUIMOVE — Spécification produit

## 1. Description globale du produit

OUIMOVE est une application de mobilité urbaine inspirée d'applications comme Citymapper, adaptée aux réalités des villes marocaines. Elle permet à un utilisateur de trouver facilement un trajet entre deux points en combinant marche à pied, lignes de bus, et à terme grands taxis et petits taxis, avec estimation du temps de trajet, du budget, et information temps réel lorsque disponible.

**Objectifs utilisateurs** : trouver rapidement comment se déplacer en transport en commun, connaître les options disponibles, réduire l'incertitude liée aux transports, préparer ses trajets quotidiens.

**Objectifs business** : construire progressivement une plateforme de mobilité urbaine multi-ville, une base de données transport fiable, une solution proposable aux opérateurs et collectivités.

**Marché** : lancement à Kénitra, avec une extension envisagée vers d'autres villes moyennes marocaines (Oujda, Berkane...), puis potentiellement les grandes villes.

**Produit multi-composants** :
- **Application mobile** (React Native, iOS/Android) : recherche d'itinéraire, favoris, notifications, suivi temps réel, affichage cartographique.
- **Back-office web** (React) : gestion des villes, lignes, arrêts, itinéraires.
- **API Backend** (Spring Boot) : gestion utilisateurs, calcul d'itinéraires, données transport, services temps réel.

**Support multilingue** : le produit doit être compatible dès le lancement avec le français, l'arabe (avec support RTL) et l'anglais, avec une architecture i18n native et des noms de lieux multilingues.

**Architecture technique cible** : Spring Boot (backend), PostgreSQL + PostGIS (base de données spatiale), React Native (mobile), React (back-office).

**Indicateurs de succès** : nombre d'utilisateurs actifs, précision de l'estimation des trajets, nombre de lignes et arrêts référencés, taux d'adoption multi-villes.

---

## 2. MVP v1 — La première version lancée aux utilisateurs finaux

Le MVP se limite à une seule brique fonctionnelle, sur le périmètre d'une seule ville (Kénitra), pour les modes de transport marche et bus :

### Recherche et calcul d'itinéraire

L'utilisateur renseigne un point de départ et un point d'arrivée (adresse, arrêt de bus référencé, ou position GPS). L'application affiche de manière textuelle : ligne(s) de bus recommandée(s), arrêts de départ et d'arrivée, correspondances éventuelles, temps de marche estimé.

**Transverse au MVP** : Support FR/AR/EN avec RTL natif dès le lancement ; back-office permettant la création des lignes, arrêts et parcours nécessaires à l'exploitation du MVP.

**Ce qui est explicitement hors MVP** : L'estimation du temps de trajet, le compte utilisateur, la cartographie, le temps réel, l'intégration des grands taxis et petits taxis, la standardisation GTFS. Ces éléments sont traités post-lancement.

### Points de vigilance technique du MVP

Même si l'affichage reste purement textuel côté utilisateur, l'architecture sous-jacente doit déjà anticiper la suite pour éviter une réécriture ultérieure :

- **Un MVP "textuel" mais 100% spatial en base de données** : pour calculer un itinéraire textuel, le backend a déjà besoin de coordonnées géographiques pour situer les arrêts de bus et calculer les distances de marche. L'absence de carte à l'écran ne dispense donc pas de construire dès le MVP un modèle de données géospatial complet.
- **Personnalisation à moindre coût malgré l'absence de compte** : en l'absence de compte utilisateur dans le MVP, le stockage local du téléphone (AsyncStorage en React Native) peut être utilisé pour conserver les dernières recherches de l'utilisateur, donnant une forme de personnalisation sans attendre.
- **Support RTL, principal point de vigilance UI** : l'affichage textuel en arabe avec alignement de droite à gauche demande une attention particulière sur les listes d'étapes de type « Marche → Bus 12 → Marche », qui doivent rester lisibles et cohérentes dans les deux sens de lecture.

---

## 3. MVP v2 — Deuxième livraison, après le MVP v1

Une fois le MVP v1 validé auprès des premiers utilisateurs, un deuxième lot vient enrichir le produit avec les briques suivantes :

### Estimation du temps de trajet

Calcul et affichage du temps de marche jusqu'au départ, du temps d'attente estimé, de la durée du trajet en bus, du temps de correspondance et du temps de marche jusqu'à l'arrivée. Les estimations s'appuient sur la distance, la vitesse moyenne, la période de la journée et l'historique disponible.

### Compte utilisateur et personnalisation

Inscription, connexion, profil utilisateur. Gestion des lieux favoris (domicile, travail, autres), historique des recherches, favoris synchronisés, raccourcis de trajet. Ce lot permet notamment de faire basculer les recherches sauvegardées en local vers un compte synchronisé.

### Cartographie (visualisation statique)

Intégration d'un SDK de carte, affichage de la position des arrêts, du tracé des lignes, et du parcours complet de l'utilisateur (marche + bus) superposé sur la carte. Cette carte reste statique à ce stade : ni position live des bus, ni heure d'arrivée dynamique — ces éléments relèvent des orientations stratégiques à définir post MVP.

Concernant le choix du SDK, **OpenStreetMap** constitue souvent l'option la plus économique pour démarrer sur le marché marocain, en évitant les frais d'API volumineux de Google Maps ; ce choix reste néanmoins à valider selon la qualité des données OSM disponibles sur Kénitra.

---

## 4. Orientations stratégiques pour la suite

Une fois le MVP (v1 & v2) lancé, deux orientations stratégiques distinctes se dessinent pour la suite du produit. Le choix entre les deux doit être mûri à la lumière des résultats du lancement (usages observés, signaux partenaires) : elles engagent des modèles économiques, des types de partenaires et des architectures de données différents. 

### Orientation 1 — Partenariat opérateurs, logique « ERP transport »

Partenariat avec une compagnie de bus et/ou une collectivité pour obtenir l'accès aux données temps réel des bus (position GPS, heure d'arrivée estimée, alertes retard), complété par un module de gestion des chauffeurs (planning, suivi, exploitation) destiné aux opérateurs. Le produit évolue alors partiellement vers un outil B2B d'exploitation transport, avec un modèle économique de type licence/abonnement opérateur, éventuellement appuyé par un financement institutionnel.

Les données d'usage collectées grâce aux comptes utilisateurs du MVP permettront de démontrer aux compagnies de bus (à commencer par l'opérateur local à Kénitra : FOUGHAL) quelles lignes sont les plus demandées — un argument de légitimité commerciale central pour la vente du module « ERP ».

### Orientation 2 — Grand public, logique multimodale

Partenariat avec les collectivités porté sur l'attractivité du territoire, avec intégration des petits taxis et grands taxis dans le moteur de recherche (référencement des stations et lignes habituelles, itinéraires mixtes, estimation des coûts et temps d'attente). Le produit reste centré sur l'usager final et sur la couverture exhaustive des modes de déplacement disponibles en ville, avec un modèle économique orienté volume d'utilisateurs.

Dans ce cas, la cartographie statique devra évoluer pour intégrer des points d'intérêt spécifiques aux taxis (stations de grands taxis, points de passage habituels).

### Indicateurs à observer pour arbitrer entre les deux orientations

- répartition des trajets recherchés : pendulaires réguliers vs trajets ponctuels non couverts par le bus ;
- réactivité d'une compagnie de bus à un partenariat data, vs intérêt d'une collectivité pour une couverture multimodale élargie ;
- capacité interne à maintenir une base de données terrain (taxis) dans la durée ;
- disponibilité de financements fléchés « modernisation opérateur » vs « attractivité citoyenne ».

### Orientation 3 — Extension multi-ville

Décision volontairement différée et indépendante des deux orientations ci-dessus : elle sera tranchée après le lancement du MVP à Kénitra. À noter pour plus tard : la vitesse de réplication multi-ville diffère selon l'orientation retenue (un partenariat opérateur par ville pour l'Orientation 1, contre une collecte de données terrain par ville pour l'Orientation 2) — un paramètre à réintégrer dans cet arbitrage le moment venu.

---

## 5. Roadmap

**Phase MVP v1 — objectif : sortir vite.** Saisie d'adresses, moteur de calcul Spring Boot / PostGIS, intégration des données de lignes de Kénitra dans le back-office React, interface de résultats textuels en trois langues (FR/AR/EN).

**Phase MVP v2 — objectif : rendre l'application attractive.** Choix et intégration du SDK cartographique, développement du module d'estimation des temps de trajet, création du système d'authentification et de personnalisation.

**Phase Arbitrage.** Analyse des données d'usage collectées après le lancement du Lot 2 (délai indicatif : environ 3 mois) pour choisir entre l'Orientation 1 et l'Orientation 2, sur la base des indicateurs listés en section 4.
