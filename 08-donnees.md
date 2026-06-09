# 08 — Données : alternatives à HFSQL

> Le guide **ne choisit pas ta base** (SQL, NoSQL, lequel — c'est **ton** choix, à acter en **ADR**). Ce chapitre t'aide à **choisir en connaissance de cause** : HFSQL offre beaucoup « dans la boîte » ; avant de migrer, repère **ce que TON appli utilise vraiment**, puis regarde ce que la cible te donne nativement et ce qui est à recâbler.
>
> *Sigles expliqués au fil du texte et regroupés dans le **Glossaire** en fin de page.*

## D'abord : quel HFSQL ?

HFSQL existe en deux modes — ta cible en découle :

- **HFSQL Classic** (fichiers `.FIC` / `.NDX` / `.FTX`, en local / réseau / embarqué) → cible naturelle : **SQLite** (embarqué, un seul fichier, zéro serveur — l'alternative la plus simple). Variantes : **LiteDB** (embarqué, orienté documents) ou **SQL Server Express LocalDB** si tu restes côté Microsoft.
- **HFSQL Client/Serveur** (un service TCP/IP, multi-utilisateurs, scalable) → cible naturelle : **PostgreSQL** (open source, très complet), **SQL Server** ou **MariaDB / MySQL**.

> Règle simple : **fichier / embarqué → SQLite** ; **serveur multi-postes → PostgreSQL / SQL Server**. Le code d'accès, lui, passe par un **ORM** (*Object-Relational Mapping* — la couche qui relie objets et tables) comme **EF Core**, ou un micro-ORM comme **Dapper**.

## Ce que tu retrouves, ce qui est à recâbler

Correspondance des fonctionnalités HFSQL — ✅ natif ailleurs · 🟧 via extension/effort · ⚠️ à reconstruire :

| Fonctionnalité HFSQL | Équivalent côté cible | Statut |
|---|---|---|
| SQL, index, **transactions ACID**, vues | PostgreSQL / SQL Server (natif) ; SQLite (oui) | ✅ |
| **Intégrité référentielle**, suppression en cascade | Clés étrangères (FK) — *à activer explicitement dans SQLite* | ✅ / 🟧 |
| **Triggers** | PostgreSQL / SQL Server / SQLite | ✅ |
| **Procédures stockées** (en WLangage) | PostgreSQL (PL/pgSQL) / SQL Server (T-SQL) ; **SQLite : non** | ✅ / ⚠️ |
| **Chiffrement au repos** (AES) | SQLite → **SQLCipher** ; SQL Server → **TDE / Always Encrypted** ; PostgreSQL → pgcrypto / TDE selon édition | 🟧 |
| Type **Mot de passe** (hash salé natif) | À coder : hachage **Argon2 / bcrypt / PBKDF2** en .NET | ⚠️ |
| **RGPD** (rubrique « donnée perso » + audit) | Pas d'équivalent natif → convention/attributs + *data masking* (SQL Server) | ⚠️ |
| **SDD** (modif. auto du schéma à chaud) | **Migrations EF Core** (ou Flyway / DbUp) — proche, mais pas « auto à chaud » | 🟧 |
| **Journalisation** (avant/après, restauration datée) | *Temporal tables* (SQL Server) / CDC ; **PITR** pour la restauration à une date | 🟧 |
| **Ordonnanceur** (tâches dans la base) | SQL Server Agent ; **pg_cron** ; ou Quartz.NET / Hangfire (cf. `07`) | ✅ / 🟧 |
| **Réplication / mode déconnecté** (mobile) | Réplication native du SGBD ; offline = **moteur de synchro maison** (cf. `07 §F`) | 🟧 / ⚠️ |
| **Sauvegarde à chaud / différentielle** | Native (pg_dump / pg_basebackup, sauvegardes SQL Server) | ✅ |
| **Full-text** (recherche plein texte) | SQLite **FTS5** ; PostgreSQL / SQL Server full-text | ✅ |
| « **Injection SQL impossible** » | **Requêtes paramétrées** / ORM (discipline, pas magie) | ✅ |
| **Table inaltérable** (append-only) | SQL Server **ledger tables** ; sinon append-only applicatif | 🟧 |
| Multi-OS, **Unicode**, OLE DB / ODBC | Standard partout | ✅ |

## Les points qui méritent vraiment ton attention

La majorité se retrouve sans douleur. Trois sujets demandent une vraie décision :

- **Chiffrement.** Si tes fichiers HFSQL sont chiffrés (AES), prévois l'équivalent dès le départ : **SQLCipher** pour SQLite, **TDE / Always Encrypted** pour SQL Server. À ne pas découvrir en fin de projet.
- **RGPD.** Le marquage natif HFSQL n'a pas d'équivalent clé-en-main : reconstitue-le par **convention** (attributs/annotations sur tes entités) + un inventaire des données personnelles, éventuellement du *data masking* côté base.
- **Mises à jour de schéma.** Le confort du **SDD** (modif auto à chaud) se remplace par des **migrations EF Core** versionnées — moins « magique », mais traçable et rejouable, et ça colle à l'esprit du guide (versionné, vérifiable).

## Par où commencer

1. **Introspecte l'existant** : quelles fonctionnalités HFSQL ton appli utilise *réellement* (triggers ? procédures stockées ? chiffrement ? journal ? réplication ?). Beaucoup d'applis n'en utilisent qu'une fraction.
2. Choisis la famille : **Classic → SQLite**, **Client/Serveur → PostgreSQL / SQL Server**.
3. Pour chaque fonctionnalité **réellement utilisée**, repère sa ligne dans le tableau et tranche (natif / extension / à coder). **Acte le choix de base en ADR.**
4. Valide sur un **vrai sous-ensemble de données** (volumétrie, types, encodage) avant de généraliser.

## Glossaire

- **ACID** : atomicité, cohérence, isolation, durabilité — les garanties d'une transaction fiable.
- **CDC** (*Change Data Capture*) : capture des modifications d'une table (qui / quoi / quand).
- **EF Core** (*Entity Framework Core*) : l'ORM standard de Microsoft pour .NET.
- **FK** (*Foreign Key*) : clé étrangère — lien d'intégrité entre deux tables.
- **FTS** (*Full-Text Search*) : recherche plein texte.
- **ORM** (*Object-Relational Mapping*) : couche qui relie les objets du code et les tables de la base (ex. EF Core, Dapper).
- **PITR** (*Point-In-Time Recovery*) : restauration de la base à une date/heure précise.
- **SDD** (*Synchronisation du schéma des Données Déployées*) : la mise à jour automatique du schéma propre à HFSQL.
- **TDE** (*Transparent Data Encryption*) : chiffrement du stockage, transparent pour l'application.
- **UDF** (*User-Defined Function*) : autre nom des procédures stockées côté HFSQL.

## Liens

- **HFSQL — fonctionnalités** — https://pcsoft.fr/hfsql/fonctionnalites.htm · **sécurité** — https://pcsoft.fr/hfsql/securite.htm · **RGPD (WINDEV)** — https://pcsoft.fr/pcsoft/rgpd-windev-vous-aide.htm
- **SQLite** — https://www.sqlite.org/ · **SQLCipher** — https://www.zetetic.net/sqlcipher/ · **chiffrement SQLite en .NET** — https://learn.microsoft.com/dotnet/standard/data/sqlite/encryption
- **PostgreSQL** — https://www.postgresql.org/ · **EF Core — migrations** — https://learn.microsoft.com/ef/core/managing-schemas/migrations/ · **Dapper** — https://github.com/DapperLib/Dapper
