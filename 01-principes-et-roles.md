# 01 — Principes & rôles

## Les deux IA

| | **Claude Cowork** (le cerveau) | **Claude Code** (les mains) |
|---|---|---|
| Où | Sur ton poste, avec un **clone** du dépôt (lecture) | Sur la **VM de dev**, dans le dépôt de travail |
| Fait quoi | Architecture, découpage en étapes, **rédige les prompts**, **relit** le travail, traduit, tient la roadmap et la mémoire | **Exécute** : code, SQL, déploie, teste, committe, pousse, **écrit un bilan** |
| Ne fait pas | N'écrit pas le code de prod | Ne décide pas seul de l'architecture |

**L'humain est toujours aux commandes** : il valide chaque étape, tranche les décisions, lance les prompts. L'IA assiste avec puissance, l'humain garde le volant.

## La règle d'or

> **Le prompt est temporaire, la structure est permanente.**

Un bon prompt améliore une tâche ; une bonne **architecture d'information** (CLAUDE.md, docs/, ADR, runbooks, skills) améliore **toutes** les sessions. On investit dans la structure, pas dans des prompts géants.

## Ce qui est non négociable

1. **Une décision = un ADR.** Toute décision d'architecture ou métier est tracée dans `docs/decisions/` (contexte → décision → conséquences). Sinon elle se perd et se re-débat.
2. **Un bilan à chaque étape (handoff).** Claude Code termine toujours par un bilan écrit dans `docs/runbooks/passation.md`. C'est le point d'entrée du cerveau à la session suivante.
3. **Le dépôt et la base font foi.** On **vérifie**, on ne suppose pas. Avant de produire un prompt d'étape : lire le handoff + l'état réel du dépôt. Avant d'écrire du SQL : **introspecter la base live** (ne jamais inventer une colonne).
4. **Aucun secret versionné.** Licences, chaînes de connexion, tokens → config locale gitignorée ou variables d'environnement.
5. **Vérité unique.** Un schéma de base = **un** script source de vérité. Une info détaillée vit dans `docs/`, pas dans un CLAUDE.md surchargé.
6. **Séparer dev et test.** La machine de test ne touche jamais une base de prod ; garde-fous explicites.

## L'état d'esprit qui fait la différence

- On **cadre avant de coder** (cartographie de l'existant, schéma cible validé, architecture validée) — comme on le ferait sans IA, mais dix fois plus vite.
- On avance **par étapes courtes, ordonnées, testées**, chacune close par un bilan.
- On **relit systématiquement** en trois catégories : (1) erreur bloquante, (2) incohérence de nommage/convention, (3) remarque métier/cosmétique. On ne re-discute pas les choix déjà validés.
- **Pense à la vraie data tôt** : sans données réelles, difficile de valider quoi que ce soit. *Comment* tu obtiendras de la vraie data fait partie des questions de cadrage (l'ordre, lui, c'est ton choix — pas une règle du guide).
