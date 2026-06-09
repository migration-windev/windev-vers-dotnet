---
name: revue
description: Relire le code/SQL d'une étape et ne signaler que les vrais points, en 3 catégories. À utiliser après chaque livraison de Claude Code.
---

# Skill — Revue d'étape

Relis le travail produit (code, SQL, scripts, doc) et **ne rapporte QUE** :

1. **Erreur bloquante** — ça ne compile pas / ça plante / une règle dure est violée.
2. **Incohérence de nommage / convention** — par rapport à `docs/conventions-sql.md` et au `CLAUDE.md`.
3. **Remarque métier / cosmétique** — hypothèse à valider, simplification, lisibilité.

**Pas de réécriture non demandée.** On **propose**, l'humain arbitre. On ne remet pas en cause les choix déjà validés (voir `docs/decisions/`).

## Contrôles systématiques

**Universels** (quelle que soit l'archi que tu as choisie) :
- **Introspection** : le code/SQL produit s'appuie sur des champs/tables/membres **réels** (vérifiés en base ou dans l'API réelle), jamais supposés.
- **Structure de données** : aucune suppression (`DROP`…) ni modif de structure non proposée ; scripts de schéma **idempotents**.
- **Secrets** : aucun en dur ni versionné.
- **Tests** : la validation a-t-elle été **réellement exécutée** (build + tests + déploiement), pas seulement « ça compile » ?

**Selon les choix actés de TON projet** (`CLAUDE.md`, `docs/architecture.md`, ADR — ne garde que ce qui te concerne) :
- **(si archi en couches)** le **sens des dépendances acté** est respecté (ex. présentation → métier → données, jamais l'inverse) ; aucune couche n'en court-circuite une autre.
- **(si accès à un système externe)** accès **encapsulé dans un repository dédié**, conforme au **mode d'accès défini** (lecture seule, ou écritures limitées à une whitelist) — jamais ailleurs.
- **(selon tes conventions)** patterns imposés respectés (ex. async/annulation, contexte de session, nommage) tels que fixés dans `docs/conventions-*.md`.

## Sortie attendue

Une liste courte, classée par les 3 catégories. Pour chaque point : **où** (fichier/ligne) et **quoi corriger**, pas tout le film.
