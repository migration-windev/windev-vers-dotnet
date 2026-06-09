# Template — prompt d'une étape (pour Claude Code)

> Copier-coller, remplir le cœur, garder **le préambule** et **la clôture** à l'identique.

```
Avant toute chose, lis docs/runbooks/passation.md (dernier bilan d'étape) puis le CLAUDE.md racine.
Respecte les règles non négociables du `CLAUDE.md` (vérifier avant de supposer, aucun secret versionné, + les choix actés du projet).

Étape « <nom de l'étape> » (<projet concerné>) :

- <objectif 1, précis>
- <objectif 2, précis>
- <contraintes : introspecter la base / l'API réelle avant de coder ; aucun secret en dur ;
  + les contraintes propres à TON archi actée (ex. async + CancellationToken ; sens des couches ; idempotence) — cf. CLAUDE.md / ADR>

Validations attendues : build Release 0 warning, tests xUnit (préciser les cas), déploiement/smoke test réel.

À la fin : commit + push, et METS À JOUR docs/runbooks/passation.md (bilan d'étape au format imposé :
Livré / Hypothèses & écarts / Validations exécutées / Reste à faire / Décisions à prendre).
```

## Rappels de rédaction
- Toujours le **préambule** (lire le handoff + CLAUDE.md) et la **clôture** (bilan).
- Donner les objectifs **précis** + les contraintes, pas le « comment » détaillé : Claude Code choisit la mise en œuvre, l'humain arbitre à la relecture.
- Si une décision a été prise depuis la dernière session : la rappeler dans le prompt (ou pointer son ADR).

## Exemples de consignes réutilisables (selon tes choix de projet)

> À piocher et coller dans le cœur du prompt, **selon les choix actés** pour ton projet — ce sont des exemples, pas des règles du guide.

- **IHM « WinDev-like » (designer-first)** : « Pour l'IHM, construis les écrans avec le **concepteur visuel** (fichiers *designer* générés par l'éditeur) et mets la logique dans le **code-behind** des événements. **Ne crée pas les contrôles par code.** Objectif : un projet *graphique qui porte du code*, proche de l'expérience WinDev. » *(suppose une techno à concepteur — typiquement WinForms ; à adapter pour WPF/MVVM, Blazor…)*
