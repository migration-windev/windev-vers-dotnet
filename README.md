# Playbook — Migration WinDev/HFSQL → .NET, pilotée par IA

Méthode complète pour **migrer un logiciel métier WinDev/HFSQL (PC SOFT) vers une solution .NET moderne**, pilotée par deux assistants IA Claude (Cowork = le cerveau, Claude Code = les mains), l'humain validant à chaque étape.

Ce dépôt est un **modèle réutilisable** : tu le clones au démarrage de chaque projet et il sert de cadre permanent.

## L'idée en une phrase

> Deux IA, deux rôles : **Claude Cowork = le cerveau** (architecte / chef de projet / relecteur qui travaille avec toi), **Claude Code = les mains** (exécute dans le dépôt, code, teste, committe). **Tu valides à chaque étape.** Ce n'est pas magique : c'est une chaîne d'ingénierie où la **structure prime sur le prompt**.

## Les rôles : deux IA + toi

| **Cowork** — le cerveau | **Claude Code** — les mains | **Toi** — le pilote |
|---|---|---|
| cadre, architecture, **rédige les prompts**, **relit**, tient la roadmap et la mémoire | code/SQL dans le dépôt (sur sa VM), **teste**, committe, pousse, écrit le **bilan** | **valides** chaque étape, **tranches** les décisions, fais le **pont** (copier-coller des prompts, `git pull`) |

Cowork tourne dans l'app Claude **sur ton poste** ; Claude Code **sur sa VM dédiée** ; **toi**, tu fais le lien entre les deux.

## Le parcours : de ton appli WinDev à l'appli .NET

Tu as une appli **WinDev / WebDev / WinDev Mobile** et tu veux la **migrer vers .NET avec Claude Code**. Voici le chemin **étape par étape** — chacune renvoie au chapitre qui la détaille.

### Étape 0 — L'infrastructure *(une fois)*
Comptes, **serveur Git** (il remplace ton GDS WinDev), **VM Claude Code**, et une **VM de test** si possible. Détaillé de zéro dans **[00 — Prérequis & installation](00-prerequis-et-installation.md)** et **[02 — Infrastructure](02-infrastructure.md)**. À faire une seule fois ; chaque projet réutilise cette base.

### Étape 1 — Préparer les sources *(le dossier `origine/`)*
Crée le dépôt du projet, déposes-y **`templates/`** (copié tel quel depuis ce playbook) et un dossier **`origine/`** avec ton appli à migrer. **Avant**, prépare la matière côté WinDev — c'est elle qui conditionne la qualité de l'analyse :

| sous-dossier | contenu | comment l'obtenir |
|---|---|---|
| `origine/code` | code source **en texte** | passe le projet en **format de sauvegarde texte** (requis pour Git), puis copie les sources |
| `origine/autres` | **structure SQL** de la base, specs | **génère le script SQL depuis ton analyse** |
| `origine/data` | **données réelles** | exporte des tables en **CSV** |
| `origine/screenshoot` | captures des **écrans** | copies d'écran des principales fenêtres / pages |

> *Côté WinDev — comment faire* : [passer le projet en **format texte** (requis pour Git)](https://doc.pcsoft.fr/fr-FR/?9500230&name=partagez_vos_projets_via_git) · [générer le **script SQL** depuis l'analyse](https://doc.pcsoft.fr/fr-FR/?2011027).

> *Astuce WinDev 2026* : le **Companion IA** intégré propose la commande **`/init_projet`**, qui génère un **fichier décrivant le contexte du projet** (logique métier, éléments). Dépose-le dans `origine/` : Claude Code comprendra ton appli plus vite. *(Sinon, la cartographie de l'étape 2 reconstitue ce contexte.)*

*(`origine/` reste local / gitignoré si volumineux ou sensible — plus il y a de matière, mieux Claude analyse.)*

### Étape 2 — Le prompt d'initialisation *(cadrage)*
Ouvre **Claude Cowork**, remplis le bloc « Infos projet » de **`templates/PromptInit.md`** et colle-le. Cowork commande alors une **cartographie** à Claude Code (sous-agents en parallèle → fiches de synthèse : entités, écrans, règles métier, pièges, secrets à révoquer), te pose **les questions qui changent le plan** (cible, périmètre, architecture, séquence, techno — **tes** choix), puis met en place le `CLAUDE.md`, l'arbo `docs/` et la **roadmap**.

### Étape 3 — La boucle de prompts *(aller-retour, le cœur du travail)*
Répétée à chaque étape de la roadmap :
1. **Cowork rédige un prompt** (préambule « lis le handoff + `CLAUDE.md` » → clôture « commit + push + mets à jour le handoff »).
2. **Tu le colles dans Claude Code** (VM de dev) : il exécute, **teste pour de vrai**, committe, pousse, écrit son **bilan** dans `docs/runbooks/passation.md`.
3. **Tu fais `git pull`** → Cowork relit la version à jour.
4. **Cowork relit** en 3 catégories (bloquant / nommage / métier) et ne signale que les vrais points.
5. **Chaque décision → un ADR** ; statut de l'étape à jour. On enchaîne.

→ détail dans **[05 — Boucle de travail](05-boucle-de-travail.md)** ; le pourquoi dans **[01 — Principes & rôles](01-principes-et-roles.md)**.

### Étape 4 — La VM de test *(si possible)*
Une machine qui **rejoue tout chaque nuit** (reconstruit la base, lance les tests, pousse un rapport horodaté) — le filet de sécurité. → **[02 — Infrastructure](02-infrastructure.md)**.

### Étape X — L'application .NET produite
Au bout de la roadmap : l'appli migrée, testée, déployable. La séquence complète est récapitulée dans **[06 — Démarrer un projet](06-demarrage-de-zero.md)**.

> Pour aller plus loin *(optionnel)* : remplacer les dernières briques PcSoft → **[07 — Perspectives](07-perspectives.md)** ; choisir la base cible → **[08 — Données](08-donnees.md)**.

## Sommaire

| Chapitre | Contenu |
|---|---|
| [00 — Prérequis & installation](00-prerequis-et-installation.md) | **De zéro** : comptes (Claude, Microsoft), logiciels, serveur Git, VM Claude Code |
| [01 — Principes & rôles](01-principes-et-roles.md) | Philosophie, les deux IA, ce qui est non négociable |
| [02 — Infrastructure](02-infrastructure.md) | Serveur Git, mise à jour auto, VM de dev, VM de test, synchronisation |
| [03 — Périmètre](03-perimetre.md) | Ce que le guide décide (outils + règles) et ce qu'il te laisse (archi, paradigme, séquence) |
| [04 — Mémoire projet (anatomie Claude)](04-memoire-projet-claude.md) | `CLAUDE.md`, `docs/`, ADR, runbooks, handoff, skills, hooks |
| [05 — Boucle de travail](05-boucle-de-travail.md) | Roadmap, cycle d'une étape, bilans, revue, décisions |
| [06 — Démarrer un projet](06-demarrage-de-zero.md) | Le parcours détaillé : infra → cartographie → cadrage → boucle |
| [07 — Perspectives (annexe)](07-perspectives.md) | *Optionnel, hors périmètre* : remplacer les dernières briques PcSoft (télémétrie, identité, distribution) par des équivalents .NET |
| [08 — Données : alternatives à HFSQL](08-donnees.md) | Choisir une base cible : Classic → SQLite, Client/Serveur → PostgreSQL / SQL Server ; ce qu'on retrouve nativement et ce qui est à recâbler |
| [templates/](templates/) | `CLAUDE.template.md`, `PromptInit.md`, ADR, passation, prompt d'étape, skill de revue |

## Public

Tout développeur **WinDev / WebDev / WinDev Mobile** qui veut **migrer un projet vers .NET avec Claude Code**.

## Notations

Les valeurs propres à un projet sont notées entre chevrons : `<App>` (nom de la solution), `<url-gitea>`, `<base>`, etc. — remplace-les à l'usage. *Si* ta solution **complète un logiciel externe** (ERP, compta, gestion commerciale…), on parle de « système tiers » ; **sinon, ignore ces mentions** (elles ne concernent pas ton projet).
