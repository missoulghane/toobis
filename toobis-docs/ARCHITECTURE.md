# Spécification Globale Produit
Code Produit : "OUIMOVE"

## Application de mobilité urbaine - Maroc
**Version du document : 1.1** **Produit : Application mobile de calcul d'itinéraires transport** **Marché initial : Kénitra, Maroc** **Marché cible : Villes moyennes : Oujda, Berkane ...**

---

# 1. Vision du produit

Le produit a pour objectif de fournir une solution de mobilité urbaine inspirée des applications comme Citymapper, adaptée aux réalités des villes marocaines.
L'application permet aux utilisateurs de trouver facilement un trajet en transport entre deux points en combinant :
- marche à pied ;
- lignes de bus disponibles ;
- correspondances ;
- estimation du temps de trajet ;
- information temps réel lorsque disponible.

---

# 2. Objectifs produit

## Objectifs utilisateurs
Permettre à un utilisateur de :
- trouver rapidement comment se déplacer en bus et grands taxis ;
- connaître les lignes disponibles ;
- réduire l'incertitude liée aux transports publics ;
- préparer ses trajets quotidiens.

## Objectifs business
Construire progressivement :
- une plateforme de mobilité urbaine multi-ville ;
- une base de données transport fiable ;
- une solution pouvant être proposée aux opérateurs et collectivités.

---

# 3. Périmètre fonctionnel global

## Applications
Le produit comprend :

### Application mobile
Technologie : React Native (iOS / Android)  
Fonctions principales : 
- recherche d'itinéraire (textuel en V1) ; 
- favoris (Domicile, Travail, Autres ...) via stockage local puis compte utilisateur ; 
- Notifications (Problèmes sur les lignes, accidents ...) ;
- suivi temps réel ;
- affichage cartographique (introduit en Version 5).

### Back-office Web
Technologie : React Web  
Fonctions : 
- Gestion des villes ; 
- Gestion des lignes (Bus & Grands Taxis) ; 
- gestion des arrêts ; 
- gestion des itinéraires ; 
- import/export des données transport.

### API Backend
Technologie : Spring Boot  
Responsabilités : 
- gestion utilisateurs ; 
- calcul itinéraires ; 
- gestion données transport ; 
- services temps réel.

---

# 4. Support multilingue

Le produit doit être compatible dès la première version avec :
- Français (FR)
- Arabe (AR)
- Anglais (EN)

Contraintes :
- support RTL (Right-to-Left) pour l'arabe ;
- noms des lieux multilingues ;
- architecture i18n native.

---

# 5. Roadmap produit

## Version 1 - Recherche et calcul d'itinéraire (MVP textuel)
### Objectif
Permettre à un utilisateur de trouver un trajet en transport entre deux points de manière simple et textuelle, **sans interface cartographique (Pas d'affichage carte)**.

### Fonctionnalités
#### Recherche trajet
L'utilisateur peut renseigner :
- point de départ
- point d'arrivée

Les points peuvent être une adresse ou un arrêt de bus référencé.

#### Calcul itinéraire
L'application affiche de manière textuelle / liste :
- ligne(s) de bus recommandée(s) ;
- arrêts de départ et d'arrivée ;
- correspondances éventuelles ;
- temps de marche estimé.

### Livrables Version 1
#### Application mobile
- écran recherche trajet ;
- écran résultats (mode liste / textuel, sans carte) ;
- gestion FR/AR/EN.
#### Backend
- API recherche trajet ;
- moteur de calcul itinéraire initial ;
- gestion réseau transport.
#### Back-office
- création ville ;
- création lignes ;
- création arrêts ;
- création parcours.
#### Données
- intégration réseau bus Kénitra ;
- base initiale lignes et arrêts.

---

## Version 2 - Estimation du temps de trajet
### Objectif
Fournir une estimation réaliste du temps nécessaire pour effectuer un trajet.

### Fonctionnalités
Ajout du calcul :
- temps de marche départ ;
- temps d'attente estimé ;
- durée trajet bus ;
- temps correspondance ;
- temps marche arrivée.

### Modèle d'estimation
Les estimations prennent en compte :
- distance ;
- vitesse moyenne ;
- période de la journée ;
- historique disponible.

### Livrables Version 2
#### Application mobile
- affichage durée totale ;
- détail par étape ;
- indication attente estimée.
#### Backend
- service calcul temps ;
- gestion paramètres vitesse ;
- statistiques trajet.
#### Back-office
- configuration temps moyens ;
- analyse performances lignes.

---

## Version 3 - Compte utilisateur et personnalisation
### Objectif
Améliorer l'expérience quotidienne et la fidélisation.

### Fonctionnalités
#### Création compte utilisateur
- inscription ;
- connexion ;
- profil utilisateur.
#### Personnalisation
- domicile ;
- travail ;
- lieux favoris ;
- historique recherches.

### Livrables Version 3
#### Application mobile
- authentification ;
- espace utilisateur ;
- favoris synchronisés ;
- raccourcis trajet.
#### Backend
- gestion comptes ;
- stockage préférences ;
- sécurité utilisateur.
#### Base de données
Ajout des tables : utilisateurs, préférences, historique.

---

## Version 4 - Information temps réel GPS
### Objectif
Afficher la position réelle des bus et améliorer fortement la précision des horaires.

### Fonctionnalités
- position GPS des bus ;
- estimation d'arrivée prochaine (ETA) ;
- retard éventuel ;
- information trafic.

### Architecture
Flux : Bus GPS → Service temps réel → API → Application mobile

### Livrables Version 4
#### Backend
- service GPS ;
- gestion positions temps réel ;
- calcul ETA.
#### Application mobile
- arrivée estimée temps réel ;
- notifications de retard.
#### Intégration opérateur
- connexion système GPS existant ou installation d'équipements.

---

## Version 5 - Cartographie et Visualisation (MAP)
### Objectif
*Introduit après le temps réel.* Fournir un affichage cartographique visuel complet pour faciliter l'orientation spatiale de l'utilisateur.

### Fonctionnalités
- Intégration d'un SDK de carte (**Décision technique à valider** : Google Maps VS Mapbox VS OpenStreetMap).
- Position visuelle des arrêts sur la carte.
- Parcours graphique des lignes.
- Tracé complet du trajet de l'utilisateur (marche + bus) superposé sur la carte.
- Affichage des bus en mouvement (exploitant les données temps réel de la V4).

### Livrables Version 5
#### Application mobile
- Composant carte intégré ;
- Écran de guidage visuel dynamique.

---

## Version 6 - Lot "Grands Taxis"
### Objectif
Enrichir le moteur de recherche en intégrant le réseau des Grands Taxis (taxis collectifs), composante indispensable et structurante de la mobilité urbaine et périurbaine au Maroc.

### Fonctionnalités
- Référencement des stations de grands taxis, des points de passage et des lignes/trajets habituels.
- Calcul d'itinéraires mixtes ou alternatifs (Bus + Grand Taxi ou Marche + Grand Taxi).
- Modèle d'estimation des coûts (tarifs fixes par place) et temps d'attente estimé selon le remplissage théorique.

---

## Version 7 - Standardisation GTFS & Industrialisation
### Objectif
Passer à l'échelle sur le plan technique en adoptant les standards du marché mondial pour simplifier l'extension multi-villes et les partenariats institutionnels.

### Fonctionnalités
- Refonte de la base de données transport pour s'aligner nativement sur le format standard mondial **GTFS** (General Transit Feed Specification).
- Back-office : Intégration de modules d'import et d'export de fichiers GTFS pour automatiser les mises à jour de données auprès des collectivités ou des opérateurs de transport.

---

# 6. Architecture technique cible

## Backend
Spring Boot (Modules : authentification, utilisateurs, transport, itinéraires, temps réel, module d'import/export GTFS).

## Base de données
PostgreSQL + PostGIS (Gestion spatiale des villes, lignes, arrêts, coordonnées GPS et parcours).

## Mobile
React Native (Android / iOS) avec support RTL natif.

## Web Admin
React (Données transport, administration, supervision, gestion des fichiers GTFS).

---

# 7. Évolution géographique

## Phase pilote
Ville : Kénitra  
Objectif : Valider l'usage utilisateur, la qualité des données initiales et le modèle opérationnel en mode textuel/liste avant de déployer l'infrastructure lourde de la carte.

## Extension
Déploiement progressif : Autres villes moyennes marocaines (Oujda, Berkane...) puis grandes villes et réseau national.

---

# 8. Indicateurs de succès
- Nombre d'utilisateurs actifs ;
- Précision de l'estimation des trajets ;
- Nombre de lignes et arrêts référencés (Bus + Taxis) ;
- Taux d'adoption multi-villes.

---

# 9. Principes directeurs et Décisions à valider

1. La qualité des données transport est prioritaire.
2. Le produit est multi-ville dès sa conception.
3. Le support FR/AR/EN (avec RTL) est obligatoire dès la V1.
4. **L'affichage de la carte (MAP) est repoussé en Version 5**, s'adossant à l'infrastructure temps réel (V4), permettant un lancement initial plus rapide, focalisé sur la fiabilité de l'itinéraire textuel.
5. **[DÉCISION À VALIDER] Choix du SDK Cartographique (V5) :** Arbitrage financier et technique entre Google Maps (coûteux à l'échelle), Mapbox, ou OpenStreetMap.
6. **[DÉCISION À VALIDER] Standardisation GTFS (V7) :** Choix d'implémenter le format standard mondial en fin de roadmap pour accélérer le développement initial, ou évaluation d'un alignement dès le départ.
