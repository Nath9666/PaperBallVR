# Paper Ball VR – Game Design Doc

Oct 1, 2026 · @nathan

## Contexte

Paper Ball VR est la démo Unreal de l'atelier technologies immersives des portes ouvertes du Studio, le 8 octobre 2026 après-midi. Elle doit être jouable sans briefing par des étudiants en informatique, et servir ensuite de pièce de portfolio en entretien.

| Poste | Casque | Activité |
| --- | --- | --- |
| Ma démo | Casque PC filaire Steam avec balises | Paper Ball VR (Unreal Engine 5) |
| Créer | Meta Quest 3 | Dessin 3D en réalité mixte (Open Brush) |
| Découvrir | Meta Quest 2 | Appli de prise en main, ou second poste de dessin |

Contraintes : sessions de 3 minutes environ, public qui défile, spectateurs qui regardent sur un écran.

## Concept

Le joueur est assis à un bureau et lance des canettes vides dans une poubelle de tri. Chaque niveau ajoute une seule nouvelle règle, donc on apprend en jouant, sans explication.

La seule consigne donnée au joueur : **« Mets la canette dans la poubelle. »**

Intentions :

- **Immédiat** : un geste que tout le monde connaît, réussi dès la première seconde grâce au tuto aimanté
- **Progressif** : viser, puis choisir le bon moment, puis compenser le vent
- **Tension douce** : le chrono passe d'une horloge silencieuse à un tic-tac qui s'emballe
- **Regardable** : une vue spectateur avec score et classement pour ceux qui attendent
- **Maîtrisé** : un décor, quelques objets, une démo finie et soignée plutôt qu'ambitieuse et fragile

**Règle de prise en main** : fermer la main fait apparaître une canette, ouvrir la main la lance. Pas de réserve à aller chercher, les deux mains fonctionnent indépendamment.

**Pourquoi une canette** : plus lisible qu'une boulette, et le bruit métallique rend chaque panier ou raté satisfaisant. Une poubelle de tri jaune donne un clin d'œil recyclage gratuit.

## Les niveaux

Quatre niveaux, une partie d'environ 3 minutes. On passe au niveau suivant dès que le nombre de paniers requis est atteint.

| Niveau | Règle | Ce que le joueur apprend | Pour passer |
| --- | --- | --- | --- |
| 0 – Tuto | La corbeille se place toute seule sous la canette | Le geste de lancer, réussite garantie | 2 paniers |
| 1 – Fixe | La corbeille ne bouge plus | Viser et doser sa force | 3 paniers |
| 2 – Mobile | La corbeille se déplace, et se fige dès que la canette est lâchée | Choisir le bon moment | 3 paniers |
| 3 – Ventilos | Des ventilateurs apparaissent et dévient la canette | Compenser le vent | Score max avant la fin du chrono |

Pistes de bonus si le temps le permet : une corbeille plus petite au niveau 3, un panier « rebond sur le mur » qui rapporte double.

## Le chrono et la tension

Le temps reste visible tout au long de la partie, mais il ne se fait entendre que progressivement.

1. **Début** : une horloge murale silencieuse affiche le temps restant
2. **Mi-parcours** : un tic-tac léger démarre
3. **15 dernières secondes** : le tic-tac accélère et devient plus fort, l'horloge passe au rouge
4. **Fin** : une sonnerie, puis l'écran de score

Côté son, un seul son de tic-tac suffit : on joue sur son volume et sa vitesse de répétition (ou son pitch) selon le temps restant.

## Architecture Unreal

Un manager central lit la description des niveaux et configure les objets ; la boulette et la corbeille communiquent par événements.

&#91;embedded content: architecture Blueprint · 8 éléments\]

Quand la canette est lâchée, `OnThrown` dit à la corbeille de s'aimanter (niveau 0) ou de se figer (niveau 2). Un panier remonte au manager par `OnScored`, qui met à jour le score et passe au niveau suivant.

| Champ de DT\_Levels | Exemple |
| --- | --- |
| Mode de la corbeille | Aimantée, Fixe, Mobile |
| Nombre de ventilos | 0 à 2 |
| Paniers requis | 2, 3, ou 0 pour « jusqu'à la fin du chrono » |
| Son du chrono | Silencieux, tic-tac, tic-tac rapide |

## Points techniques sensibles

**Le lancer en VR.** La vitesse de la main au moment exact du lâcher donne un lancer mou ou imprévisible. Garder la vélocité de la manette sur les 3 à 5 dernières frames, en faire la moyenne, et appliquer un léger multiplicateur réglable.

**Le tuto aimanté.** Au lâcher, calculer le point d'atterrissage avec `Predict Projectile Path By TraceChannel`, puis faire glisser la corbeille vers ce point. Ajouter un léger guidage de la canette en fin de trajectoire pour que la réussite soit vraiment garantie.

**La physique de la canette.** Le métal rebondit et roule : sans réglage, les canettes ressortent de la poubelle et le jeu devient frustrant. Lui donner un Physical Material avec peu de restitution et un peu de friction, et activer la détection de collision continue (CCD) pour qu'elle ne traverse pas la poubelle.

**La détection du panier.** Ne compter le panier que si la canette entre dans la zone par le haut (vitesse verticale négative), et une seule fois par canette.

**Le vent.** Appliquer une force continue (`AddForce`) et pas une vitesse fixe, pour que la déviation reste naturelle. Rendre le vent visible : pales qui tournent, rubans ou particules.

**La vue spectateur.** Utiliser le Spectator Screen d'Unreal pour afficher sur l'écran du PC une vue dédiée avec le score et le classement, indépendante de ce que voit le joueur.

**L'apparition en main.** Appui sur la gâchette de préhension avec la main vide : la canette apparaît attachée à la manette, avec un petit son et une vibration. Au relâchement, on la détache et on lui applique la vélocité moyennée. Une simple fonction dans le pawn VR remplace la réserve de canettes.

**Le pawn de debug.** Un `BP_DebugPawn` clavier-souris lance une canette depuis la caméra au clic, avec une force qui dépend de la durée d'appui. Il permet de tester toute la logique de jeu dans l'éditeur, sans casque.

## Planning

Règle d'or : une version jouable de bout en bout dimanche 4 au soir, même moche. Tout ce qui suit est du bonus.

**Vendredi 2 octobre – Les fondations**

- [x] Projet UE5 sur le VR Template, OpenXR activé
- [x] Décor de bureau simple (assets gratuits, licences vérifiées)
- [x] BP\_DebugPawn clavier-souris : lancer au clic, force selon la durée d'appui
- [x] BP\_Can : mesh, collision, Physical Material peu rebondissant
- [x] BP\_Bin : détection du panier, testée avec le pawn de debug
- [x] Score affiché en debug à l'écran

**Samedi 3 octobre – Le panier**

- [ ] Test sur le PC et le casque filaire du jour J
- [ ] Canette qui apparaît en main, lancer moyenné qui se sent bien
- [ ] Niveau 1 (fixe), puis niveau 0 (tuto aimanté)

**Dimanche 4 octobre – La boucle complète**

- [ ] Niveau 2 (corbeille mobile, figée au lancer)
- [ ] LevelManager et DataTable des niveaux
- [ ] Enchaînement des niveaux, début et fin de partie
- [ ] **Partie jouable de bout en bout**

**Lundi 5 octobre – Tension**

- [ ] Niveau 3 (ventilos)
- [ ] Horloge murale et montée du tic-tac

**Mardi 6 octobre – Finition**

- [ ] Sons : froissement, panier, raté, sonnerie
- [ ] Écran de fin et classement sur la vue spectateur
- [ ] Bouton reset pour l'opérateur

**Mercredi 7 octobre – Répétition**

- [ ] Installation sur site, balises calibrées, zone de jeu définie
- [ ] Tests avec 2 ou 3 vrais joueurs, réglage de la difficulté
- [ ] Build de secours sur clé USB
- [ ] Capture vidéo de la démo pour le portfolio et LinkedIn

## Jour J : checklist

- [ ] Lingettes et charlottes ou masques jetables pour l'hygiène des casques
- [ ] Chargeurs et câbles pour les Quest (autonomie d'environ 2 h, insuffisante pour l'après-midi)
- [ ] Zone de jeu dégagée et balisée au sol, balises calibrées
- [ ] Open Brush installé et testé sur le Quest 3
- [ ] Casting des Quest vers un écran pour les spectateurs
- [ ] Vue spectateur de Paper Ball VR sur la télé ou le moniteur
- [ ] Une personne par poste pour guider le public
- [ ] Build de secours et câble de rechange à portée de main

## En entretien

Une démo simple en apparence, mais qui montre une vraie démarche de conception d'expérience VR, livrée en une semaine et testée par un vrai public.

- **Game design pédagogique** : une règle nouvelle par niveau, un tuto qui garantit la réussite. C'est la logique d'une simulation de formation : on apprend le geste avant de le complexifier.
- **Interactions VR** : OpenXR (compatible Index, Vive et Quest via Link), lancer basé sur la vélocité moyennée, retour haptique.
- **Architecture pilotée par les données** : niveaux décrits dans une DataTable, équilibrage sans toucher au code.
- **Physique** : trajectoire prédite, forces de vent, détection fiable.
- **Expérience spectateur** : vue dédiée et classement, pensés pour un événement public.
- **Retour terrain** : nombre de joueurs du jour J, réglages faits après observation, vidéo à l'appui.
