# 03 — Périmètre : ce que ce guide décide (et ce qu'il te laisse)

Ce playbook t'apprend à **travailler avec les outils** et te donne des **règles de travail**. Il **ne décide pas** de l'architecture ni du style de ton application — **ça, c'est à toi.**

## Ce que ce guide couvre

- **Les outils** : Claude Cowork (le cerveau) et Claude Code (les mains), le serveur Git, la mise à jour auto, les VM (dev, test), la synchronisation. → chapitres **00**, **02**.
- **Les règles de travail** : la boucle (prompt → exécution → bilan → revue), le handoff, les ADR, la roadmap, la mémoire projet (`CLAUDE.md` / `docs/` / skills), la discipline (vérifier avant de supposer, aucun secret versionné…). → chapitres **01**, **04**, **05**.

## Ce que ce guide NE décide PAS pour toi

Ces choix sont **les tiens**, pris **au cadrage avec Cowork** (il te propose une reco argumentée, tu tranches), puis tracés en **ADR** :

- **L'architecture** : couches, API-first, MVP, microservices, DDD, monolithe modulaire… comme tu veux.
- **Le paradigme / style** de programmation et le découpage en projets.
- **La séquence de construction** : quoi avant quoi (la data d'abord ? les écrans d'abord ? un module vertical en premier ?).
- **Les choix techniques** : gestion du schéma (figé + bundle, migrations, code-first…), intégrations externes et leur mode d'accès, stratégie de déploiement.

> Le guide te donne **la méthode pour prendre et tenir ces décisions** — les poser comme règles dans `CLAUDE.md`, les tracer en ADR, les faire respecter par Claude Code — **pas les réponses**. Deux projets différents auront des architectures différentes, et c'est très bien.

## La seule exigence côté code (discipline, pas architecture)

- **Vérifier la réalité avant de coder** : introspecter la base, lire l'API réelle des bibliothèques — **ne jamais inventer** une colonne, une table, un membre. Ça vaut quelle que soit l'architecture que tu choisis.
