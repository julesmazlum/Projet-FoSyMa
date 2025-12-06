# Projet FoSyMa : Exploration & Collecte Multi-Agents (Dedale)

**Master 1 ANDROIDE - Sorbonne Université (2025)** *UE Fondements des Systèmes Multi-Agents*

Ce projet implémente un système multi-agents (SMA) coopératif basé sur la plateforme **JADE** et l'environnement **Dedale**. Il s'agit d'une variante distribuée du jeu *Hunt the Wumpus*, où une équipe d'agents doit explorer un environnement inconnu, cartographier les lieux et collecter un maximum de trésors tout en gérant les contraintes d'un environnement dynamique (Golem).

---

## Table des Matières
- [Description du Projet](#-description-du-projet)
- [Architecture des Agents](#-architecture-des-agents)
- [Stratégies Implémentées](#-stratégies-implémentées)
  - [Exploration & Partage de Carte](#1-partage-incrémental-de-la-carte)
  - [Collecte Coopérative](#2-collecte-basée-sur-les-capacités)
  - [Gestion Dynamique](#3-exploration-infinie)
  - [Gestion des Blocages](#4-résolution-de-conflits-et-impasses)
- [Limitations & Perspectives](#-limitations--perspectives)
- [Auteurs](#-auteurs)

---

## Description du Projet

L'objectif est de déployer une équipe d'agents hétérogènes dans un environnement (graphe, grille ou labyrinthe) pour :
1. **Explorer** totalement la carte.
2. **Collecter** des ressources (Or et Diamants) stockées dans des coffres nécessitant des compétences spécifiques (Serrurerie et Force).
3. **Stocker** les ressources dans un agent Silo (Tanker).
4. **S'adapter** à un environnement dynamique où un Golem déplace les trésors et referme les coffres.

**Contraintes techniques :**
* Communication limitée (rayon de portée).
* Capacité de sac à dos limitée.
* Coordination nécessaire pour éviter les blocages dans les couloirs étroits.

---

## Architecture des Agents

Le système repose sur trois types d'agents spécialisés, utilisant des machines à états finis (FSM) pour leurs comportements :

### 1. `MyExploreAgent`
* **Rôle :** Cartographie rapide de l'environnement.
* **Comportement :** Explore les nœuds inconnus, observe le voisinage et partage sa carte avec les alliés rencontrés.
<img width="653" height="476" alt="Capture d’écran 2025-12-06 à 15 19 05" src="https://github.com/user-attachments/assets/e1c6fe6c-ff05-412e-95d9-5680d8e57d1e" />

### 2. `MyCollectAgent`
* **Rôle :** Récupération des trésors et transport vers le Tanker.
* **Compétences :** Possède des attributs de *Lockpicking* (ouverture de coffres) et de *Strength* (portage).
* **Comportement :** Analyse les trésors accessibles selon ses compétences, calcule le chemin optimal pour la collecte et vide son sac auprès du Tanker.
<img width="653" height="476" alt="Capture d’écran 2025-12-06 à 15 19 19" src="https://github.com/user-attachments/assets/479ee441-b891-4c31-a07c-41a5bc58b515" />

### 3. `MyTankerAgent` (Silo)
* **Rôle :** Stockage illimité des ressources.
* **Comportement :** Sert de point de déchargement mobile. Il gère également les situations d'impasse en signalant sa position aux autres agents pour éviter qu'ils ne se bloquent.
<img width="653" height="476" alt="Capture d’écran 2025-12-06 à 15 19 28" src="https://github.com/user-attachments/assets/1a0e5287-bc89-4d96-9c28-b177f5f4e31b" />

---

## Stratégies Implémentées

### 1. Partage Incrémental de la Carte
Pour optimiser la bande passante et respecter les communications limitées :
* Les agents maintiennent une mémoire de ce qu'ils ont déjà envoyé à chaque collègue.
* Lors d'une rencontre, seuls les **nouveaux nœuds découverts** depuis la dernière interaction sont transmis.
* *Avantage :* Réduction drastique du volume de messages.

### 2. Collecte Basée sur les Capacités
Les agents maintiennent une `listTreasureData` partagée et synchronisée contenant l'état des trésors (type, quantité, serrurerie requise, état ouvert/fermé).
* Un agent collecteur ne cible un trésor que si :
    * Il a la compétence de serrurerie suffisante OU le coffre est déjà ouvert.
    * Il a la place dans son sac.
* Cela évite les déplacements inutiles vers des coffres impossibles à ouvrir pour l'agent.

### 3. Exploration Infinie
Pour contrer le **Golem** (qui déplace les trésors), l'exploration ne s'arrête jamais :
* Une fois la carte complète connue, l'agent génère une "carte virtuelle" vide.
* Il ré-explore l'environnement comme s'il était inconnu pour mettre à jour l'état des trésors et détecter les changements.

### 4. Résolution de Conflits et Impasses
* **Négociation de passage :** Si un agent est bloqué par un allié, il envoie une requête explicite pour demander à l'autre de se déplacer sur un nœud adjacent libre.
* **Gestion Tanker :** Si le Tanker est coincé dans une impasse structurelle (cul-de-sac), il envoie sa position aux explorateurs pour qu'ils considèrent ce nœud comme "fermé" et ne tentent pas d'y accéder inutilement.

---

## Limitations & Perspectives

Actuellement, le projet présente certaines limites documentées dans le rapport :
* **Pas de collecte collaborative simultanée :** Les coffres nécessitant une force supérieure à celle d'un seul agent ne sont pas ouverts (pas de coordination pour agir à deux sur un même coffre).
* **Odeur du Golem :** L'indice olfactif du Golem n'est pas exploité pour fuir ou traquer les déplacements de trésors.
* **Redondance de code :** Certaines logiques (déplacement, observation) sont dupliquées entre `Explore` et `Collect`. Une refactorisation avec héritage commun serait bénéfique.

---

## Auteurs

**Groupe 19**
* **Jules MAZLUM** - *Architecture, Stratégies, Développement*

*Projet réalisé dans le cadre du Master 1 ANDROIDE, Sorbonne Université.*
