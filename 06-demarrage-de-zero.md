# 06 — Démarrer un projet (l'universel)

La partie **commune à tout projet**. Au-delà, **quoi construire et dans quel ordre, c'est ton cadrage qui le décide** (cf. [03 — Périmètre](03-perimetre.md)) — pas ce guide.

## 1. Monter l'infra (une fois)
Comptes, logiciels, serveur Git, VM Claude Code (+ VM de test conseillée). → [00 — Prérequis & installation](00-prerequis-et-installation.md).

## 2. Préparer les sources
Range l'existant dans **`origine/`** (`code`, `data`, `screenshoot`, `autres`). **Plus il y a de matière, mieux Claude analyse.**

## 3. Cartographier l'existant (avant de coder)
Premier prompt à Claude Code = la **cartographie** : des **sous-agents en parallèle** analysent `origine/` et produisent des **fiches `.md` de synthèse + de règles** (entités, écrans, règles métier, conventions de nommage, pièges, dépendances externes, **secrets en dur à révoquer**). Fais-les **relire** par d'autres agents. C'est le *brief* d'analyse.

## 4. Cadrer avec Cowork
Colle **`PromptInit.md`** dans Cowork. À partir des fiches, il te pose **les questions qui changent le plan** et **vous décidez ensemble** : la cible, le périmètre, **l'architecture**, **la séquence de construction**, les **choix techniques**. Cowork pose alors le `CLAUDE.md`, les premiers **ADR**, et la **roadmap** — l'ordre des étapes **que vous avez choisi**. Parmi ces choix, le **style d'IHM** (designer-first façon WinDev, ou code-first / MVVM) : si tu vises le rendu WinDev-like, ajoute la consigne dédiée à tes prompts d'étape (exemple dans `templates/prompt-etape.md`).

## 5. Dérouler la boucle
À chaque étape (cf. [05 — Boucle de travail](05-boucle-de-travail.md)) : Cowork rédige le prompt → tu le colles dans Claude Code → il exécute / teste / committe / pousse + bilan → tu `git pull` → Cowork relit (3 catégories) → décision = ADR → statut à jour.

---

> **La méthode du guide s'arrête là.** L'ordre des modules, l'architecture, le paradigme : ton **cadrage** les a fixés selon **ton** projet. Le guide t'a donné les **outils** et la **méthode** pour les tenir — pas la recette de ton appli.
>
> **Pour aller plus loin** *(optionnel, hors périmètre garanti)* : remplacer les dernières briques PcSoft — télémétrie, identité, vitrine de distribution — par des équivalents .NET → **[07 — Perspectives](07-perspectives.md)**.
