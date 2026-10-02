# Paper Ball VR

> Un jeu de lancer en réalité virtuelle : vise la poubelle, une canette à la fois, pendant que le temps file.

![Unreal Engine](https://img.shields.io/badge/Unreal%20Engine-5.x-313131?logo=unrealengine&logoColor=white)
![OpenXR](https://img.shields.io/badge/OpenXR-SteamVR-1f6feb)
![Blueprint](https://img.shields.io/badge/Blueprint-visual%20scripting-0e7fc0)
![Statut](https://img.shields.io/badge/statut-en%20d%C3%A9veloppement-orange)

<!-- Remplacer par un GIF ou une capture du gameplay -->
<p align="center">
  <img src="Docs/gameplay.gif" alt="Aperçu du gameplay" width="720">
</p>

---

## Le projet

**Paper Ball VR** est une expérience VR courte (environ 3 minutes) conçue pour l'atelier technologies immersives des portes ouvertes du Studio, le **8 octobre 2026**, à destination d'étudiants de l'enseignement supérieur.

L'objectif : une démo jouable **sans aucune explication**, qui reste amusante à regarder pour le public qui attend son tour.

Le joueur est assis à un bureau. Il ferme la main, une canette apparaît. Il ouvre la main, la canette part. À lui de la mettre dans la poubelle de tri, alors que chaque niveau ajoute une nouvelle contrainte.

## Gameplay

| Niveau | Règle | Ce que le joueur apprend |
| --- | --- | --- |
| **0 – Tuto** | La poubelle se place toute seule sous la canette | Le geste de lancer, réussite garantie |
| **1 – Fixe** | La poubelle ne bouge plus | Viser et doser sa force |
| **2 – Mobile** | La poubelle se déplace, puis se fige au lancer | Choisir le bon moment |
| **3 – Ventilos** | Des ventilateurs dévient la canette | Compenser le vent |

**La tension monte avec le chrono** : une horloge murale silencieuse au début, puis un tic-tac qui accélère et s'amplifie dans les dernières secondes.

## Points techniques

- **Interactions VR avec OpenXR** : un même build pour les casques compatibles SteamVR
- **Lancer à vélocité moyennée** : la vitesse de la manette est moyennée sur les dernières frames pour un lancer naturel et prévisible
- **Apparition en main** : main fermée, une canette apparaît attachée à la manette ; main ouverte, elle est lancée
- **Tuto aimanté** : le point d'impact est prédit au lâcher (`Predict Projectile Path`) et la poubelle s'y déplace
- **Niveaux pilotés par les données** : chaque niveau est décrit dans une DataTable (mode de la poubelle, ventilateurs, paniers requis, ambiance sonore), ce qui permet de rééquilibrer le jeu sans toucher à la logique
- **Physique** : canette en métal à faible restitution, détection de collision continue (CCD), vent appliqué par forces continues
- **Communication par événements** : la canette, la poubelle et le gestionnaire de niveaux échangent via des Event Dispatchers (`OnThrown`, `OnScored`)
- **Vue spectateur** : un affichage dédié sur l'écran du PC avec score et classement
- **Pawn de debug clavier-souris** : toute la logique de jeu se teste dans l'éditeur, sans casque

## Architecture

```
DT_Levels ──lit──▶ BP_LevelManager ──affiche──▶ Vue spectateur
                        │   ▲
              configure │   │ OnScored
                        ▼   │
      BP_Bin   BP_Fan   BP_WallClock
        ▲
        │ OnThrown
     BP_Can ◀──fait apparaître── BP_VRPawn
```

| Blueprint | Rôle |
| --- | --- |
| `BP_VRPawn` | Fait apparaître la canette en main et gère le lancer |
| `BP_Can` | La canette : physique, état « déjà comptée », durée de vie |
| `BP_Bin` | La poubelle : modes aimanté, fixe et mobile, détection du panier |
| `BP_Fan` | Zone de vent qui dévie la canette |
| `BP_WallClock` | Affichage du temps restant et montée du tic-tac |
| `BP_LevelManager` | Niveau courant, score, chrono, enchaînement des niveaux |
| `BP_DebugPawn` | Lancer au clic pour tester sans casque |

## Lancer le projet

### Prérequis

- Unreal Engine **5.x**
- Un casque compatible SteamVR (Valve Index, HTC Vive…) et SteamVR installé
- Un PC compatible VR

### Installation

```bash
git clone https://github.com/Nath9666/<nom-du-depot>.git
```

1. Ouvrir le fichier `.uproject` avec Unreal Engine 5
2. Vérifier que le plugin **OpenXR** est activé (*Edit → Plugins*)
3. Lancer SteamVR, puis **Play → VR Preview**

### Commandes

| Action | VR | Debug (clavier-souris) |
| --- | --- | --- |
| Faire apparaître une canette | Fermer la main (gâchette de préhension) | — |
| Lancer | Ouvrir la main | Clic gauche |
| Se déplacer / regarder | Mouvements de la tête | ZQSD + souris |

## Feuille de route

- [x] Projet UE5 sur le VR Template, OpenXR
- [x] Décor de bureau
- [ ] Pawn de debug et lancer de canette
- [ ] Poubelle et détection du panier
- [ ] Niveaux 0 à 3 et gestionnaire de niveaux
- [ ] Horloge et montée de la tension sonore
- [ ] Vue spectateur et classement
- [ ] Sons et finitions
- [ ] Test grandeur nature le 8 octobre 2026

## Auteur

**Nathan Morel** — Ingénieur numérique (Efrei Paris, promotion 2025), spécialisé en XR et temps réel

[LinkedIn](https://www.linkedin.com/in/nathan-morel-4b993b1b7) · [GitHub](https://github.com/Nath9666)
