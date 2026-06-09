# 04 — Mémoire projet (l'anatomie Claude Code)

Le principe : **séparer** la mémoire centrale, la documentation détaillée, les décisions, les procédures, les workflows réutilisables et les garde-fous. Chacun à sa place → l'IA retrouve le bon contexte au bon moment sans être noyée.

```
<App>/
├── CLAUDE.md                 ← mémoire centrale COURTE (lue à chaque session)
├── docs/
│   ├── architecture.md       ← l'archi en détail
│   ├── conventions-sql.md    ← nommage, typage, patterns triggers
│   ├── deploiement.md        ← Velopack, canaux, tokens, bundle
│   ├── observabilite.md
│   ├── decisions/            ← ADR : une décision = un fichier
│   └── runbooks/
│       ├── passation.md      ← HANDOFF roulant (toujours le dernier bilan d'étape)
│       └── *.md              ← procédures (rebuild base, publier release, monter VM test…)
├── database/
│   └── CLAUDE.md             ← consignes locales : structure figée, no DROP, whitelist ERP
├── <App>.Data/
│   └── CLAUDE.md             ← consignes locales : entités générées (ne pas éditer à la main)
├── build/
│   └── CLAUDE.md             ← consignes locales : tokens en env, jamais committés
└── .claude/
    ├── skills/               ← workflows réutilisables (revue, format de bilan)
    └── hooks/                ← garde-fous déterministes
```

## CLAUDE.md racine — COURT

Pas un pavé. C'est la **boussole**, pas l'encyclopédie. ~50-60 lignes :
- la **mission** (3 lignes) ;
- la **carte du dépôt** (projets, dossiers) ;
- les **règles non négociables** (8-10 règles dures) ;
- les **commandes utiles** (build, test, install, publish) ;
- des **pointeurs** vers `docs/*` et les CLAUDE.md locaux.

Tout le détail part dans `docs/`. Le piège classique : vouloir tout mettre dans CLAUDE.md → il devient un placard de cuisine.

## CLAUDE.md LOCAUX

Près des zones sensibles, pour que l'IA voie les pièges **sur place** :
- `database/` : structure figée, aucun DROP, whitelist ERP, régénérer le bundle après toute modif.
- dossier des entités générées : ne jamais éditer les fichiers générés à la main (régénérer).
- `build/` : secrets uniquement en variables d'environnement, jamais en dur ni committés.

## docs/ — la vérité détaillée

Ce qui doit **exister** dans le projet sans devoir être dans la tête de l'IA en permanence : architecture, conventions, déploiement, observabilité. On structure, on n'entasse pas.

## docs/decisions/ — les ADR (la traçabilité)

**Un fichier par décision**, court : `00X-titre.md` avec **Contexte → Décision → Conséquences**. C'est ce qui évite de re-débattre dans 3 mois et explique *pourquoi* tel choix. Numérotés, jamais réécrits (on en ajoute).

## docs/runbooks/ — procédures + handoff

- **`passation.md` = le handoff roulant** : il contient **toujours** le bilan de la **dernière** étape close (écrasé à chaque passation). C'est le **point d'entrée** du cerveau : chaque prompt commence par « lis `passation.md` ». Pour l'historique complet : `git log`.
- Les autres runbooks = procédures opérationnelles (reconstruire la base, publier une release, monter une machine de test, faire tourner une rotation de token…).

## .claude/skills/ — workflows réutilisables

Pour ne pas répéter les mêmes consignes : on encapsule une méthode stable. Les plus utiles :
- **revue** : la checklist de relecture (3 catégories, contrôles systématiques).
- **format de bilan** : la structure imposée du handoff (Livré / Hypothèses & écarts / Validations / Reste à faire / Décisions).

## .claude/hooks/ — garde-fous déterministes

Une IA peut oublier une consigne ; un hook, non. Déterministes, à minima (1-2 suffisent) :
- bloquer un `DROP TABLE` dans un script de schéma ;
- bloquer une commande interdite.

> ⚠️ Les fichiers sous `.claude/` sont souvent **protégés** en édition (policy de self-modification de l'agent) : c'est **l'humain** qui les modifie.
