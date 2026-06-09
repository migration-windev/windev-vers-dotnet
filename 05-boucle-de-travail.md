# 05 — Boucle de travail

## La roadmap

Le cerveau (Cowork) maintient une **roadmap d'étapes ordonnées** : numéro, phase, projet concerné, tâche, qui fait (Claude Code / Cowork / sous-agent), prompt, livrable attendu, **dépendances**, **statut** (À faire / En cours / Terminé).

- Les **dépendances** dictent l'ordre ; on ne saute pas une fondation.
- Le **statut** est tenu à jour à chaque étape close.
- Format pratique : un tableur, ou un fichier versionné. Peu importe, du moment qu'il y a **un ordre et des statuts**.

## Le cycle d'une étape

```
  (1) Cowork rédige le PROMPT  ─▶  (2) Claude Code EXÉCUTE  ─▶  (3) humain: git pull
        │  préambule + clôture          code/SQL, déploie, teste,        (clone à jour)
        │                               committe, pousse, écrit le BILAN        │
        ▼                                                                       ▼
  (5) statut + ADR  ◀───────────────  (4) Cowork RELIT (3 catégories) ◀─────────┘
```

**1. Le prompt** (rédigé par le cerveau) commence **toujours** par :
> « Avant toute chose, lis `docs/runbooks/passation.md` puis le `CLAUDE.md` racine. Respecte les règles non négociables. »

…et finit **toujours** par :
> « …commit + push, et **mets à jour `docs/runbooks/passation.md`** (bilan d'étape). »

**2. Claude Code exécute** dans le dépôt, **déploie/teste pour de vrai** (pas juste « ça compile »), committe, pousse, et écrit son **bilan**.

**3. L'humain** fait `git pull` côté clone.

**4. Cowork relit** en **trois catégories**, et ne signale **que** les vrais points :
- (1) **erreur bloquante** ;
- (2) **incohérence de nommage / convention** ;
- (3) **remarque métier / cosmétique**.
Pas de réécriture non demandée. On **propose**, l'humain arbitre.

**5. Décisions → ADR**, statut de l'étape mis à jour, on enchaîne.

## Le format de bilan (handoff)

Imposé, pour une relecture efficace :
1. **Livré** — ce qui a été produit.
2. **Hypothèses & écarts** — rien d'inventé ; toute supposition est citée, base réelle à l'appui.
3. **Validations exécutées** — build, tests, déploiement réel.
4. **Reste à faire**.
5. **Décisions à prendre par l'humain**.

L'auto-critique honnête (« je n'ai pas pu exécuter ceci ») vaut de l'or : elle dirige la relecture.

## Préférences de réponse (à inscrire dans CLAUDE.md)

**C'est une préférence, pas une règle** — fixe la tienne dans `CLAUDE.md`. Certains veulent du **concis, sans blabla, sans script complet sauf demande** (c'est notre cas) ; d'autres préfèrent un Claude détaillé et pédagogue. À toi de voir.

Côté **méthode** (ça, ça ne bouge pas) : **ne pas remettre en cause les choix validés**, donner les **points à corriger** sans tout re-dérouler, et **lire les fichiers en entier avant de conclure**.

## Anti-patterns (les pièges réels)

- ❌ Supposer au lieu de **vérifier le dépôt / introspecter la base**.
- ❌ Mettre un `DROP` / une migration correctrice dans un script de schéma.
- ❌ Mélanger machine de dev et machine de test.
- ❌ Versionner un secret (token, chaîne avec mot de passe, `.bak`).
- ❌ Coder un module métier **avant** les fondations (data, accès, services).
- ❌ Gonfler CLAUDE.md au lieu d'utiliser `docs/`.
- ❌ Laisser une IA juger automatiquement vert/rouge (la machine rejoue, déterministe).
