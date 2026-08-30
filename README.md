# FoSyMa — Exploration & Collecte Multi-Agents (Dedale)

Projet M1 ANDROIDE 2025 — UE Fondements des Systèmes Multi-Agents (FoSyMa), Sorbonne Université. Auteur : Jules MAZLUM (Groupe 19).

## Description / Sujet

Ce projet implémente un système multi-agents (SMA) coopératif basé sur la plateforme **JADE** et l'environnement **Dedale**. Il s'agit d'une variante distribuée du jeu *Hunt the Wumpus*, où une équipe d'agents doit explorer un environnement inconnu, cartographier les lieux et collecter un maximum de trésors tout en gérant les contraintes d'un environnement dynamique (Golem).

L'objectif est de déployer une équipe d'agents hétérogènes dans un environnement (graphe, grille ou labyrinthe) pour :

1. **Explorer** totalement la carte.
2. **Collecter** des ressources (or et diamants) stockées dans des coffres nécessitant des compétences spécifiques (serrurerie et force).
3. **Stocker** les ressources dans un agent Silo (Tanker).
4. **S'adapter** à un environnement dynamique où un Golem déplace les trésors et referme les coffres.

Contraintes techniques : communication limitée (rayon de portée), capacité de sac à dos limitée, coordination nécessaire pour éviter les blocages dans les couloirs étroits.

### Architecture des agents

Le système repose sur trois types d'agents spécialisés, utilisant des machines à états finis (FSM) pour leurs comportements.

**`MyExploreAgent`** — cartographie rapide de l'environnement : explore les nœuds inconnus, observe le voisinage et partage sa carte avec les alliés rencontrés.

<img width="653" height="476" alt="Capture d'écran 2025-12-06 à 15 19 05" src="https://github.com/user-attachments/assets/e1c6fe6c-ff05-412e-95d9-5680d8e57d1e" />

**`MyCollectAgent`** — récupération des trésors et transport vers le Tanker. Possède des attributs de *lockpicking* (ouverture de coffres) et de *strength* (portage) : analyse les trésors accessibles selon ses compétences, calcule le chemin optimal pour la collecte et vide son sac auprès du Tanker.

<img width="653" height="476" alt="Capture d'écran 2025-12-06 à 15 19 19" src="https://github.com/user-attachments/assets/479ee441-b891-4c31-a07c-41a5bc58b515" />

**`MyTankerAgent`** (Silo) — stockage illimité des ressources. Sert de point de déchargement mobile, et gère les situations d'impasse en signalant sa position aux autres agents pour éviter qu'ils ne se bloquent.

<img width="653" height="476" alt="Capture d'écran 2025-12-06 à 15 19 28" src="https://github.com/user-attachments/assets/1a0e5287-bc89-4d96-9c28-b177f5f4e31b" />

### Stratégies implémentées

**1. Partage incrémental de la carte** — pour optimiser la bande passante et respecter les communications limitées, les agents maintiennent une mémoire de ce qu'ils ont déjà envoyé à chaque collègue ; lors d'une rencontre, seuls les nouveaux nœuds découverts depuis la dernière interaction sont transmis (réduction drastique du volume de messages).

**2. Collecte basée sur les capacités** — les agents maintiennent une `listTreasureData` partagée et synchronisée contenant l'état des trésors (type, quantité, serrurerie requise, état ouvert/fermé). Un agent collecteur ne cible un trésor que s'il a la compétence de serrurerie suffisante (ou le coffre est déjà ouvert) et la place dans son sac, ce qui évite les déplacements inutiles.

**3. Exploration infinie** — pour contrer le Golem (qui déplace les trésors), l'exploration ne s'arrête jamais : une fois la carte complète connue, l'agent génère une « carte virtuelle » vide et ré-explore l'environnement comme s'il était inconnu, pour mettre à jour l'état des trésors et détecter les changements.

**4. Résolution de conflits et impasses** — négociation de passage (un agent bloqué par un allié envoie une requête explicite pour lui demander de se déplacer) et gestion du Tanker (s'il est coincé dans une impasse structurelle, il signale sa position aux explorateurs pour que ce nœud soit considéré comme « fermé »).

## Structure du dépôt

```text
.
├── dedale-etu/            Projet Java (Maven) implémentant les agents sur la plateforme Dedale/JADE
│   ├── src/main/          Code source des agents (MyExploreAgent, MyCollectAgent, MyTankerAgent...)
│   ├── src/test/          Tests
│   ├── resources/         Ressources du projet
│   ├── doc/               Documentation
│   └── pom.xml            Configuration Maven
├── 19-MAZLUM.pdf          Rapport du projet
└── Présentation.pdf       Support de présentation
```

## Installation / Prérequis

Projet Java géré via Maven, basé sur les plateformes JADE et Dedale (voir `dedale-etu/pom.xml`).

## Utilisation

Importer `dedale-etu/` comme projet Maven dans un IDE Java (Eclipse, IntelliJ...), puis lancer la simulation sur la plateforme Dedale pour exécuter les agents `MyExploreAgent`, `MyCollectAgent` et `MyTankerAgent`.

## Résultats principaux

- Le système remplit les objectifs d'exploration complète de la carte et de collecte coopérative des trésors décrits ci-dessus, tout en s'adaptant aux perturbations du Golem.
- Limite — **pas de collecte collaborative simultanée** : les coffres nécessitant une force supérieure à celle d'un seul agent ne sont pas ouverts (pas de coordination pour agir à deux sur un même coffre).
- Limite — **odeur du Golem non exploitée** : l'indice olfactif du Golem n'est pas utilisé pour fuir ou traquer les déplacements de trésors.
- Limite — **redondance de code** : certaines logiques (déplacement, observation) sont dupliquées entre `Explore` et `Collect` ; une refactorisation avec héritage commun serait bénéfique.

Le détail est présenté dans le rapport `19-MAZLUM.pdf`.

## Auteurs

- Jules MAZLUM (Groupe 19) — Architecture, Stratégies, Développement
