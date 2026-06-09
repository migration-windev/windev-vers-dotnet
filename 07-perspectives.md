# 07 — Perspectives : remplacer les dernières briques PcSoft

> **Annexe optionnelle — hors du périmètre garanti du guide.** Le cœur du playbook t'a donné les **outils** (Cowork, Claude Code, Gitea, Velopack, VM) et la **méthode**. Ici, on cartographie comment remplacer **le reste de l'écosystème PcSoft** par des équivalents **.NET**. Ce sont des **pistes** : chaque brique est un **choix de projet** (selon ton contexte), à acter en **ADR** (*Architecture Decision Record* — une fiche de décision : contexte → choix → conséquences, cf. `docs/decisions/`) — le guide n'en impose aucune. Pour chaque brique, ses **caractéristiques** (licence open source ou propriétaire, hébergement chez toi ou service tiers) sont indiquées **sans jugement** : à toi de pondérer selon tes critères. Discipline du guide qui s'applique ici aussi : **aucun secret versionné, on vérifie la réalité avant de câbler.**

> *Tous les sigles (ADR, RAD, IAM, OIDC, FCM…) sont expliqués à leur première apparition **et** regroupés dans le **[Glossaire](#glossaire)** en fin de page.*

## Les briques qui restent après la migration

Migrer le code et les données vers .NET ne remplace pas *tout* PcSoft. Restent des services que WinDev / WebDev offraient « dans la boîte » — **intégrés, clé-en-main** chez PcSoft, donc à reconstruire par assemblage côté .NET :

1. **Supervision / télémétrie** — savoir ce que fait l'appli en production, remonter les erreurs ;
2. **Identité, OAuth & droits** — le *Serveur OAuth* et le *Groupware utilisateur* (authentifier + administrer les habilitations) ;
3. **Distribution** — publier et mettre à jour les postes (le *Store privé*) ;
4. **États & reporting** — l'éditeur d'états et le reporting en libre-service (*États & Requêtes*) ;
5. **Tâches planifiées / batch** — traitements automatiques à heure dite ;
6. **Selon ton profil** — mobile (réplication hors-ligne, notifications push), temps réel, et administration de la base.

Pour chacune : le besoin, la piste, pourquoi elle tient, et **par où commencer**.

> **Cas particulier — tu viens de WinDev : la voie DevExpress XAF.** Pour la **transition la plus douce**, XAF (le framework **RAD** de DevExpress — *Rapid Application Development*, où l'outil génère l'appli ; voir §B) est le .NET le plus proche de l'esprit WinDev : l'interface est **générée depuis ton modèle**, avec **CRUD** (créer / lire / modifier / supprimer), validation, états et **sécurité façon Groupware** intégrés. Plusieurs briques de cette page sont alors **couvertes nativement** (Groupware §B, états §D, interface riche).
>
> **Et le verrouillage éditeur (« lock-in ») ?** Il reste **circonscrit** : (1) **licence par développeur, sans redevance d'exécution** — tu paies tes postes de dev, pas tes utilisateurs déployés ; (2) une version **payée s'utilise et se redistribue indéfiniment** (l'abonnement ne paie que les mises à jour futures) ; (3) surtout, **tu restes en .NET / C#** : si un jour tu quittes XAF, tu refais surtout l'interface, **sans changer ni de langage ni de plateforme** — une sortie bien plus simple qu'un changement complet d'écosystème.
>
> **Le revers** : ça reste **propriétaire** (pas open source) et **structurant** (à adopter dès la conception). Pour garder une sortie peu coûteuse : **découple ta logique métier** du framework et prends **EF Core** (*Entity Framework Core*, l'**ORM** standard de Microsoft — la couche qui relie objets et base) plutôt que **XPO** (l'ORM maison de DevExpress). Un **embranchement** à arbitrer selon ce que tu privilégies — **rapidité de mise en œuvre** ou **indépendance vis-à-vis d'un éditeur** — et à acter en **ADR**.

---

## A. Télémétrie & supervision

**Le besoin** *(≈ la remontée d'anomalies / rapports d'erreur automatiques de WinDev)*. Voir en production ce qui se passe : erreurs, lenteurs, volumétrie, trace d'une requête de bout en bout. *(À ne pas confondre avec le Centre de contrôle HFSQL, qui lui est un outil d'**administration de base** — voir §F.)*

**La piste : OpenTelemetry pour instrumenter, SigNoz pour regarder.**

- **OpenTelemetry (« OTel »)** = le **standard ouvert** d'instrumentation (traces, métriques et journaux — les trois stables, .NET pleinement couvert), sous licence open source Apache, porté par la **CNCF** (*Cloud Native Computing Foundation*, la fondation derrière Kubernetes). Neutre : tu instrumentes une fois, tu peux changer d'outil de visualisation sans retoucher le code. **Aucune dépendance à un éditeur.**
- **SigNoz** = l'**outil où tu regardes** (le « backend »), auto-hébergeable : conçu pour OpenTelemetry, il réunit erreurs, traces et métriques dans **une seule interface** (alternative open source à Datadog / New Relic). Cœur sous licence open source MIT. *L'authentification unique d'entreprise (SSO) et la gestion fine des droits sont dans l'édition payante — le cœur gratuit suffit pour démarrer.*

> **OpenTelemetry n'est pas un concurrent de SigNoz.** OpenTelemetry = comment ton appli **émet** la télémétrie ; SigNoz = **où tu la consultes** (et tu pourras changer d'outil plus tard sans retoucher le code instrumenté).

**Par où commencer.**

1. Ajoute la bibliothèque **OpenTelemetry** à ton/tes projets .NET (instrumentation automatique d'ASP.NET Core, des appels HTTP, du client base de données…).
2. Fais-la pointer vers un **collecteur OpenTelemetry** local (le composant qui reçoit la télémétrie).
3. Monte **SigNoz** en conteneur sur ton infrastructure, branche le collecteur dessus.
4. Vérifie que **la donnée réelle** arrive (une vraie requête tracée de bout en bout) avant d'élargir.

---

## B. Identité, OAuth & droits

Trois besoins PcSoft tombent ici : le **Serveur OAuth**, l'**authentification**, et — le plus dur — l'**administration des droits du Groupware**.

**Le besoin.**

- **Serveur OAuth** *(≈ Serveur OAuth WebDev / PCSCloud)* : un service qui authentifie les utilisateurs et délivre les jetons d'accès, selon les protocoles standard **OAuth 2.0** (autorisation : déléguer un accès sans partager le mot de passe) et **OpenID Connect / OIDC** (authentification, au-dessus d'OAuth 2.0). WinDev sait déjà *consommer* ces protocoles (fonction `AuthIdentifie`, depuis la v24) ; il te manque le **serveur**.
- **Groupware utilisateur** *(≈ module intégré WinDev)* : **bien plus que du login.** Son cœur — et **le vrai défi** — c'est que **l'utilisateur final habilité administre lui-même les droits, depuis l'appli, à la maille du bouton** : qui a le droit de cliquer sur quoi, qui voit quel écran/champ — **sans dev, sans recompilation, effet immédiat.**

**La piste.**

- **Serveur OAuth + authentification → Keycloak.** Un **IAM** (*Identity and Access Management* — gestion des identités et des accès) open source, porté par la **CNCF** ; serveur complet **OpenID Connect / OAuth 2.0 / SAML** (*SAML* = un autre protocole d'authentification, courant en entreprise). C'est l'alternative à Auth0 ou Okta, **sans coût par utilisateur ni dépendance éditeur** : **SSO** (*Single Sign-On* — authentification unique : un seul login pour plusieurs applis), rattachement d'annuaires existants (Microsoft Entra ID, Okta…), clés d'accès (passkeys), **MFA** (*Multi-Factor Authentication* — double authentification). Ton appli .NET **et** ton portail de distribution (§C) délèguent l'authentification à Keycloak. Remplacement quasi direct.
- **Administration des droits du Groupware → deux voies :**
  - **Clé-en-main, si tu bâtis avec DevExpress XAF.** XAF (*eXpressApp Framework*) est un **framework RAD .NET** (développement rapide : l'outil génère l'interface depuis ton modèle métier) — **l'esprit le plus proche de WinDev** (licence et réversibilité : voir l'encadré en tête de page). Son **Security System** offre **nativement** : permissions **type / objet / membre** (le niveau *membre* **cache ou désactive l'élément dans l'interface**) + permissions de **navigation** + un **éditeur de rôles administrable depuis l'appli** par un utilisateur habilité → l'**équivalent direct du Groupware**. **Mais** : framework **structurant** (à adopter **dès la conception**) et **propriétaire**. Choix d'architecture majeur, à acter en **ADR**.
  - **À coder, sinon (appli non-XAF).** Le pattern : (1) un **catalogue des contrôles** avec une **clé stable** (`Écran.Contrôle`, ex. `FrmMain.btOuvrir` — préférable à un identifiant numérique fragile ; rempli **statiquement** à la compilation, ou **dynamiquement** à la 1ʳᵉ ouverture de chaque écran, façon WinDev) ; (2) une **table de droits** `(utilisateur ou rôle × contrôle × action : voir / activer / éditer)`, éditée par un **écran d'admin** ; (3) l'**application** à l'ouverture de l'écran (`Visible / Enabled / ReadOnly`) **+ un contrôle côté serveur** (masquer un bouton ≠ sécuriser l'action). Moteur de décision : Keycloak / **OpenFGA** / **Casbin** (moteurs de droits open source), ou le produit commercial **Visual Guard** (se greffe sur une appli WinForms / WPF / ASP.NET existante).

> **Le morceau dur — mais il a une issue.** Avec **DevExpress XAF**, l'admin des droits est quasi clé-en-main (au prix du framework imposé et du propriétaire). **Sans XAF**, c'est un **module à coder** (le pattern ci-dessus) : l'éditeur de droits en libre-service, à la maille du bouton, modifiable à chaud — la signature du Groupware — reste alors une vraie fonctionnalité à **chiffrer comme un module à part entière**.

**Par où commencer.**

1. Monte **Keycloak** (authentification + OAuth) en conteneur ; crée un espace (« realm »), un client **OpenID Connect** par appli/portail ; branche **ASP.NET Core** dessus.
2. **Tranche l'admin des droits** : framework **XAF** (clé-en-main, propriétaire, dès la conception) **ou** **pattern à coder** (catalogue + table + application). Acte le choix en **ADR**.
3. Si pattern à coder : modélise les **permissions fines** (par contrôle/action), construis l'**écran d'admin** et le **câblage de l'interface**, double d'un **contrôle côté serveur**.
4. **Aucun secret en dur** : le secret du client en variable d'environnement / config locale non versionnée.

---

## C. Distribution & mises à jour — le « Store privé »

**Le besoin** *(≈ Store privé WinDev)*. Un point d'entrée interne où l'utilisateur **autorisé** voit les applis qui le concernent, clique « **télécharger / installer** », puis reçoit les **mises à jour automatiquement**. Chez PC SOFT, le Store privé s'installe sur un serveur d'entreprise (ou dans le cloud), propose la mise à jour au lancement, garde l'**historique des versions** (retour arrière possible) et gère les **droits utilisateurs**.

**La piste : portail web .NET (vitrine) + Velopack (moteur) + Keycloak (droits).** Les trois ensemble = de quoi reconstituer le Store privé :

- **Velopack** *(`02`, déjà acté)* : le moteur d'installation + mise à jour automatique, son fichier de référence par canal (`prod` / `recette`), l'historique des versions.
- **Portail web .NET dédié** : la **vitrine** — liste des applis, **bouton télécharger**, et hébergement du fichier de mises à jour (Velopack sait le lire depuis n'importe quel serveur de fichiers).
- **Keycloak** *(§B)* : **qui a le droit de voir / télécharger quoi**. Le portail exige un **login** ; les rôles/groupes Keycloak décident des applis et canaux visibles — exactement la « gestion des droits » du Store WinDev.

> **Une nuance honnête.** Protège **la vitrine** (accès humain, premier téléchargement) par le login Keycloak. La **mise à jour automatique**, elle, est récupérée par le client *sans* navigateur → garde-la derrière le **jeton en lecture seule** de Velopack (`02`), pas derrière un login interactif.

**Par où commencer.**

1. Garde **Velopack + Gitea** comme socle.
2. Monte un **site .NET minimal** : page vitrine (applis + bouton télécharger) + hébergement du fichier de mises à jour.
3. Mets le portail **derrière Keycloak** ; relie rôles/groupes → applis & canaux visibles.
4. Teste une **vraie** montée de version (publier → le client se met à jour seul) sur `recette` avant `prod`.

---

## D. États & reporting

**Le besoin** *(≈ l'éditeur d'états WinDev + l'outil « États & Requêtes »)*. Produire des **éditions** (impression, **PDF**, étiquettes code-barres, états composites, graphes) — et, très souvent, laisser **l'utilisateur final créer ou personnaliser ses propres états** (le reporting en libre-service). Quasi tout logiciel de gestion WinDev en dépend.

**Deux pistes, selon ton curseur.**

- **Génération par code : QuestPDF.** Bibliothèque .NET (open source pour la majorité des cas) pour générer des **PDF** par programme, proprement. Idéale pour les éditions « définies par le dev ». **Limite** : pas de **concepteur d'états pour l'utilisateur final** → le libre-service reste un module à construire (même logique que le Groupware, §B).
- **Clé-en-main : DevExpress ou Syncfusion.** Ces suites fournissent un **concepteur d'états intégrable dans ton appli, utilisable par le client final** (l'équivalent direct d'*États & Requêtes* : l'utilisateur dessine son état), **plus** des **composants d'interface riches** (tableaux, grilles, éditeurs) pour reconstruire des écrans denses façon WinDev. Elles **bouchent vite les trous dans la raquette**. À noter : produits **propriétaires** (licence par développeur, **sans redevance d'exécution** ; *pas* open source → dépendance éditeur ; Syncfusion propose, sous conditions de taille d'entreprise, une licence Community gratuite — à vérifier). Elles **s'exécutent dans ton appli** (rien chez un tiers).

> **Le compromis.** QuestPDF = open source, mais le concepteur en libre-service est à coder. DevExpress / Syncfusion = concepteur + interface riches « offerts », au prix du propriétaire et de la dépendance éditeur. À arbitrer selon ton curseur **rapidité ↔ indépendance**.

**Par où commencer.**

1. Liste tes états réels (combien, lesquels figés, lesquels « à la carte » par l'utilisateur).
2. États figés → **QuestPDF**. Besoin d'un concepteur pour l'utilisateur → évalue **DevExpress / Syncfusion** (essai gratuit) et tranche en **ADR**.
3. Vérifie sur un **vrai** état complexe existant (rupture, sous-total, graphe) avant de généraliser.

---

## E. Tâches planifiées & traitements batch

**Le besoin** *(≈ tâches planifiées du Serveur d'application WEBDEV, tâches planifiées HFSQL, automate)*. Lancer des traitements **à heure dite** ou en file d'attente : imports, calculs nocturnes, envois, purges.

**La piste : un service .NET de fond + un planificateur.**

- **Quartz.NET** — planification riche (style *cron*, la planification horaire d'Unix), pour des horaires précis et récurrents.
- **Hangfire** — files de tâches + **tableau de bord** de suivi (tâches réussies/échouées, relances), pratique à superviser.
- Hébergés dans un **Worker Service .NET** (programme de fond) tournant sur ta VM/serveur — l'équivalent d'un service Windows.

**Par où commencer.**

1. Recense les traitements planifiés actuels (quoi, quand, rejouable sans dégât ou non).
2. Choisis **Quartz.NET** (planning pur) ou **Hangfire** (file + tableau de bord) ; héberge-les dans un Worker Service.
3. Rends chaque tâche **rejouable sans dégât** (idempotente) et **journalisée** (cf. télémétrie, §A).

---

## F. Selon ton profil — mobile, temps réel, admin base

Briques **conditionnelles** : à ne traiter que si elles te concernent.

- **Mobile — réplication / mode déconnecté** *(≈ réplication universelle HFSQL, base embarquée)*. Pour le terrain (travail hors-ligne puis synchronisation). **Pas d'équivalent clé-en-main ⚠️** : c'est un **moteur de synchronisation à construire** (base locale type SQLite + service de synchro + gestion des conflits). Gros chantier, à chiffrer sérieusement.
- **Mobile — notifications push.** Le push mobile **passe forcément** par **FCM** (*Firebase Cloud Messaging*, le service de Google) et **APNs** (*Apple Push Notification service*, celui d'Apple) : incontournables, donc **hors de ton contrôle** côté transport. Pour le **web / desktop**, des serveurs auto-hébergeables existent : **ntfy** ou **Gotify**.
- **Temps réel & hébergement web** *(≈ Serveur d'application WEBDEV, webservices, WebSockets)*. Côté .NET, c'est quasi natif : **ASP.NET Core** (héberge sites et **API** — interfaces de programmation) + **SignalR** (la techno **temps réel** de .NET : notifications navigateur, écrans qui se rafraîchissent seuls). Faible effort.
- **Administration de la base** *(≈ Centre de contrôle HFSQL, qui est l'équivalent de SSMS)*. Après migration, tu administres ta base cible avec **son** outil natif : **SSMS** (*SQL Server Management Studio*, l'outil d'admin de SQL Server), **Azure Data Studio**, **DBeaver** (multi-bases, gratuit), **pgAdmin** (pour PostgreSQL)… Ce n'est pas du dev, juste un outil à installer côté administrateur.

---

## Récapitulatif

Pour chaque brique, l'équivalent .NET et ses **caractéristiques** (sans jugement) :

| Brique PcSoft | Équivalent .NET | Licence | Hébergement |
|---|---|---|---|
| Remontée d'anomalies / erreurs | OpenTelemetry + SigNoz | open source | chez toi |
| Serveur OAuth + authentification | Keycloak | open source | chez toi |
| Admin des droits (Groupware) | DevExpress XAF · ou Keycloak / Visual Guard + module maison | XAF & Visual Guard propriétaires ; Keycloak open source | chez toi *(module à développer hors XAF)* |
| Store privé | Velopack + portail .NET | open source | chez toi |
| États (figés) | QuestPDF | open source | chez toi |
| États + concepteur utilisateur | DevExpress / Syncfusion | propriétaire | chez toi |
| Tâches planifiées / batch | Quartz.NET / Hangfire | open source | chez toi |
| Réplication / hors-ligne (mobile) | moteur de synchro maison | — | chez toi *(à développer)* |
| Notifications push | FCM·APNs (mobile) · ntfy/Gotify (web) | propriétaire (Google/Apple) · open source | service Google/Apple · ou chez toi |
| Temps réel / hébergement web | ASP.NET Core + SignalR | open source | chez toi |
| Admin de la base (≈ SSMS) | outil natif du SGBD (SSMS, DBeaver…) | selon l'outil | chez toi |

> **Comment lire ce tableau.** La **licence** (open source / propriétaire) et l'**hébergement** (chez toi / service tiers) sont indiqués comme des **faits** : **le guide ne privilégie aucun de ces critères** — pondère selon les tiens. **Aucune brique n'est obligatoire** : prends ce dont ton projet a besoin, trace le choix en **ADR**, **vérifie la réalité** avant de câbler, et **ne versionne aucun secret**.

## Glossaire

- **ADR** (*Architecture Decision Record*) : fiche qui acte une décision d'architecture — contexte, choix, conséquences.
- **API** : interface de programmation — le point d'entrée par lequel un programme en appelle un autre.
- **APNs** (*Apple Push Notification service*) : service de notifications push d'Apple.
- **CNCF** (*Cloud Native Computing Foundation*) : fondation qui héberge des projets comme Kubernetes, OpenTelemetry, Keycloak.
- **CRUD** (*Create, Read, Update, Delete*) : créer / lire / modifier / supprimer — les opérations de base sur une donnée.
- **EF Core** (*Entity Framework Core*) : l'**ORM** standard de Microsoft pour .NET.
- **FCM** (*Firebase Cloud Messaging*) : service de notifications push de Google.
- **IAM** (*Identity and Access Management*) : gestion des identités et des accès.
- **IHM / UI** : interface utilisateur (Interface Homme-Machine).
- **MFA** (*Multi-Factor Authentication*) : double authentification (ex. mot de passe + code à usage unique).
- **OAuth 2.0** : protocole standard d'autorisation (déléguer un accès sans partager le mot de passe).
- **OIDC** (*OpenID Connect*) : protocole standard d'authentification, bâti au-dessus d'OAuth 2.0.
- **ORM** (*Object-Relational Mapping*) : couche qui relie les objets du code aux tables de la base.
- **RAD** (*Rapid Application Development*) : développement rapide où l'outil génère une grande partie de l'appli (le modèle WinDev / XAF).
- **SAML** : protocole d'authentification / SSO, courant en entreprise.
- **SGBD** : système de gestion de base de données (SQL Server, PostgreSQL, MySQL…).
- **SSMS** (*SQL Server Management Studio*) : l'outil d'administration de SQL Server.
- **SSO** (*Single Sign-On*) : authentification unique — un seul login pour plusieurs applis.
- **WinForms / WPF / Blazor / .NET MAUI** : les principales technos d'interface .NET (desktop classique / desktop moderne / web / multiplateforme).
- **XPO** (*eXpress Persistent Objects*) : l'ORM propriétaire de DevExpress.

## Liens

- **OpenTelemetry .NET** — https://opentelemetry.io/docs/languages/dotnet/ · **SigNoz** — https://signoz.io/
- **Keycloak** — https://www.keycloak.org/ · *Authorization Services* — https://www.keycloak.org/docs/latest/authorization_services/ · **OpenFGA** — https://openfga.dev/ · **Casbin** — https://casbin.org/
- **DevExpress XAF — Security System** — https://docs.devexpress.com/eXpressAppFramework/113366/data-security-and-safety/security-system · **DevExpress — licences (FAQ)** — https://www.devexpress.com/support/eulas/ · **Visual Guard** — https://www.visual-guard.com/
- **Velopack** — https://docs.velopack.io/distributing/overview
- **QuestPDF** — https://www.questpdf.com/ · **DevExpress Reporting** — https://www.devexpress.com/products/net/reporting/ · **Syncfusion** — https://www.syncfusion.com/
- **Quartz.NET** — https://www.quartz-scheduler.net/ · **Hangfire** — https://www.hangfire.io/ · **ASP.NET Core SignalR** — https://learn.microsoft.com/aspnet/core/signalr/introduction · **ntfy** — https://ntfy.sh/ · **Gotify** — https://gotify.net/
- **Références WinDev** — Store privé — https://doc.pcsoft.fr/fr-FR/?name=store_prive_des_applications_windev · Groupware utilisateur — https://doc.windev.com/en-US/?name=Tips_for_user_groupware&lf=fr · Serveur OAuth — https://doc.windev.com/en-US/?name=Server_OAuth_WEBDEV&lf=fr
