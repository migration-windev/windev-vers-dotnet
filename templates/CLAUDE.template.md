# CLAUDE.md — <App>

**<En 2-3 lignes : ce que fait la solution et pour qui. Remplace une V1 WinDev/HFSQL.>**

> Modèle à adapter. Les **règles non négociables** = la **discipline universelle**. Les **règles spécifiques** (base, intégrations, sécurité, fronts…) se **décident au cadrage** et se complètent plus bas — le playbook n'en impose aucune.

## Carte du dépôt
- `<App>.Data` — accès aux données (connexions, procédures, entités générées). Ne référence rien.
- `<App>.Core` — logique métier + briques mutualisées. → `Data`.
- `<les projets utiles à TON appli : front(s), service, adaptateurs…>` → `Core`.
- `database/` — scripts SQL versionnés. Voir `database/CLAUDE.md`.
- `build/` — générateurs & publication. Voir `build/CLAUDE.md`.
- `docs/` — doc détaillée (pointeurs ci-dessous). `<sources V1 / réf.>` — local, gitignoré.

## Règles non négociables (discipline universelle)
1. **Vérifier la réalité avant de coder** : introspecter la base / le schéma réel, lire l'API réelle des bibliothèques — **ne jamais inventer** un champ, une table/collection, un membre.
2. **Toute modif de structure de données = proposition à l'humain d'abord** ; **aucune suppression destructrice** « correctrice » en silence.
3. **Aucun secret versionné** (licences, chaînes de connexion, tokens → config locale gitignorée / variables d'env).
4. **Style de réponse : à ta convenance** (la concision est une préférence, pas une règle).
5. **Revue en 3 catégories** (bloquant / nommage / métier) ; ne pas remettre en cause les choix validés ; **lire les fichiers en entier**.
6. **Les fichiers générés** (entités, scaffold, designer) ne s'éditent pas à la main → on régénère.

## Règles spécifiques à CE projet (à compléter au cadrage)
> Décidées avec l'humain au démarrage, tracées en ADR. Exemples possibles — **garde uniquement ce qui te concerne** :
> - **architecture & découpage en couches / responsabilités** (présentation ↔ métier ↔ données ; où vivent l'orchestration, le batch…) ;
> - **outillage Git** (forge, ligne de commande, CI/CD) ;
> - gestion du schéma / des données (schéma figé + bundle rejouable, migrations, code-first…) ;
> - intégration d'un système externe et son **mode d'accès** (lecture seule / écritures limitées / aucune) ;
> - organisation des intégrations et des fronts ;
> - sécurité / identité / rôles.

## Commandes utiles
```
dotnet build <App>.slnx -c Release      # objectif : 0 warning
dotnet test  <App>.slnx
<tes commandes : génération d'entités, bundle, publication…>
```

## Pointeurs
- Architecture → `docs/architecture.md` · Conventions SQL → `docs/conventions-sql.md`
- Décisions → `docs/decisions/` · Procédures → `docs/runbooks/`
- **Handoff (dernier bilan) → `docs/runbooks/passation.md`**
- CLAUDE.md locaux : `database/` · `<App>.Data/` · `build/`
