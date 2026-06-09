# Journal des modifications

Toutes les évolutions notables de ce guide sont consignées ici. Format inspiré de
[Keep a Changelog](https://keepachangelog.com/fr/). Le guide n'a pas de versions
logicielles : on date les lots de changements.

## 2026-06-08

### Ajouté
- Chapitre **[07 — Perspectives](07-perspectives.md)** : remplacer les dernières briques
  PcSoft (télémétrie, identité / OAuth / Groupware, Store privé, états & reporting,
  tâches planifiées, mobile & temps réel) par des équivalents .NET ; encadré
  « voie DevExpress XAF » ; glossaire des sigles.
- Chapitre **[08 — Données : alternatives à HFSQL](08-donnees.md)** : familles de cibles
  (Classic → SQLite, Client/Serveur → PostgreSQL / SQL Server), tableau de correspondance
  des fonctionnalités HFSQL, points sensibles (chiffrement, RGPD, SDD), glossaire.
- `LICENSE` (MIT) et `CHANGELOG.md`.
- `00 — Prérequis` : sur la VM Claude Code, ajout d'un **environnement de dev complet**
  (moteurs de base locaux, connecteurs OLE DB / ODBC, copie locale des données).

### Modifié
- Renommage `03-architecture.md` → **`03-perimetre.md`** (le titre était déjà « Périmètre »),
  liens mis à jour dans le README et le chapitre 06.
- `templates/skill-revue` et `templates/prompt-etape` : les choix d'architecture (.NET en
  couches, async, contexte de session) ne sont plus présentés comme universels mais comme
  des exemples « selon ton projet ».

## Base initiale
- `README.md` + chapitres **00 → 06** (prérequis, principes & rôles, infrastructure,
  périmètre, mémoire projet, boucle de travail, démarrage).
- `templates/` : `CLAUDE.template.md`, `PromptInit.md`, `ADR-000-modele.md`, `passation.md`,
  `prompt-etape.md`, `skill-revue-SKILL.md`.
