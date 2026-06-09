# 02 — Infrastructure

## Vue d'ensemble

```
   Poste humain                  Serveur Git (NAS / serveur)            VM de DEV
 ┌──────────────┐    git pull   ┌─────────────────────────┐  pull/push ┌──────────────────┐
 │ Claude Cowork│ ◀───────────  │  Gitea (privé)          │ ◀────────▶ │ Claude Code      │
 │  + CLONE     │               │  - dépôt <App>          │            │  + Visual Studio │
 │  (lecture)   │               │  - dépôt <App>.Releases │            │  + SDK + base    │
 └──────────────┘               └─────────────────────────┘            └──────────────────┘
                                          ▲  pull / push (rapport)
                                          │
                                   ┌──────────────────┐
                                   │ VM de TEST        │  tâche nocturne 23h :
                                   │ env. complet      │  pull → rebuild base → tests → rapport
                                   └──────────────────┘
```

Tout transite par **Gitea**. Le cerveau (Cowork) lit un **clone**, les mains (Claude Code) travaillent sur la **VM de dev**, la **VM de test** rejoue tout chaque nuit.

## Gitea (git auto-hébergé)

- **Pourquoi** : hébergé chez toi, privé — le code ne part pas sur un forge tiers (un choix, pas une obligation). **Pour un dev WinDev, c'est le remplaçant du GDS** (Gestionnaire de Sources) — en standard et universel.
- **Installation** : paquet natif du NAS, ou conteneur (Docker/Container Manager). Léger.
- **Deux dépôts** par projet : `<App>` (sources) et `<App>.Releases` (binaires Velopack, séparés).
- **`git` classique uniquement.** Ne pas utiliser le CLI propriétaire d'un forge concurrent (incompatible). En interface ligne de commande Gitea, l'outil dédié existe mais n'est pas nécessaire.
- **Authentification** : un **token dédié** (jeton d'accès personnel, périmètre minimal) stocké dans le **gestionnaire d'identifiants Windows** (helper git `manager`), jamais en clair, jamais dans une URL.

## Velopack (installeur + mise à jour auto)

Pour les clients **desktop** et le **service** (pas les fronts web, déployés classiquement). **C'est l'équivalent de ton store privé / de l'installation depuis le réseau en WinDev** : l'appli s'installe et **se met à jour toute seule**, sans repasser sur chaque poste.

- **Feed** = le dépôt Gitea `<App>.Releases` (Velopack publie nativement sur les *releases* Gitea).
- **`VelopackApp.Build().Run()`** en toute première instruction du `Main` de chaque exécutable concerné.
- **Deux canaux** : `prod` et `recette`, via l'option `--channel`. Le client lit son canal en config.
- **Deux tokens, jamais versionnés**, en variables d'environnement sur la machine de build :
  - *publication* (write) : utilisé par l'outil de packaging, reste côté build.
  - *lecture* (read-only) : compte technique dédié, limité au dépôt Releases, **injecté au build dans un fichier généré (gitignoré) puis offusqué** dans le client. L'offuscation = dissuasion ; le périmètre read-only + le compte dédié sont la vraie barrière.

## VM de DEV

Claude Code + Visual Studio (ou éditeur léger) + SDK .NET + accès à la base de travail. C'est ici que le code s'écrit, se committe et se pousse. Cette machine n'a **pas besoin** d'héberger Gitea (c'est le serveur qui l'héberge).

## VM de TEST (le filet nocturne)

Environnement **complet et isolé**, jamais la prod :

- **Base métier dédiée** `<base-metier>_Test` (jamais la base de prod). Reconstruite chaque nuit depuis le bundle d'installation, **sous un autre nom** (substitution, sans modifier les `.sql` versionnés).
- **(si l'appli intègre un système externe)** sa base restaurée (depuis un `.bak` sur un partage — **jamais versionné**) + lien rétabli après chaque reconstruction, pour tester aussi cette intégration.
- **(si besoin)** le logiciel tiers installé, pour couvrir son éventuel SDK/export.
- **Tâche planifiée 23h** : `git pull` → reconstruire la base → lancer tous les tests → **rapport horodaté** → le pousser sur une **branche dédiée `reports`** (jamais `main`, pour ne jamais entrer en conflit).
- **Garde-fous** (ceinture + bretelles) :
  - un **marqueur machine** (variable d'env. posée par l'installeur de la VM de test) ; le script nocturne **refuse de tourner** s'il est absent → un poste de dev ne se fait jamais détruire sa base.
  - base de test **dédiée** (jamais le nom de prod).
  - **vérifier réellement** que le push du rapport a réussi (ne jamais avaler l'échec en silence) ; un test rouge = rapport rouge, mais le script ne plante pas.
- **Principe** : l'IA *conçoit et écrit* les tests ; la **machine les rejoue de façon déterministe**. On ne met jamais une IA dans le rôle du juge vert/rouge.

## Synchronisation & circulation des prompts

### Le flux des prompts (tu es le pont)
- Sur ton poste, **Cowork rédige le prompt** d'une étape — **adapté à l'état réel** (il lit d'abord le *handoff* `passation.md`).
- Tu **copies-colles ce prompt dans Claude Code, sur la VM de dev**. Claude Code exécute, committe, pousse, et écrit son **bilan** dans le handoff.
- Tu reviens vers Cowork avec le bilan pour la **relecture**. **Tu fais le pont** entre les deux IA.

### Le flux du code (git, bidirectionnel)
- **Claude Code** (VM de dev) : édite le code, committe, **pousse** sur Gitea — c'est le sens principal.
- **Toi, côté clone** : `git pull` régulièrement → Cowork relit la version à jour.
- **Tu pousses aussi depuis le clone** quand il le faut : un `CLAUDE.md`, une décision (ADR), ou les fichiers **`.claude/`** que les IA ne peuvent pas modifier elles-mêmes. À faire **correctement** :
  ```
  git pull           # se mettre à jour d'abord
  git add <fichier>
  git commit -m "..."
  git push
  ```
- **Règle anti-conflit** : on évite d'éditer **le même fichier des deux côtés en même temps**. En pratique : tu pousses tes modifs depuis le clone **avant** la prochaine session Claude Code, et Claude Code **commence toujours par `git pull`**. Ainsi les deux côtés ne s'écrasent jamais.
