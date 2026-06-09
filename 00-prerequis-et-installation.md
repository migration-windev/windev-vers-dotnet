# 00 — Prérequis & installation (de zéro)

Tu viens de **WinDev / PcSoft** et tu pars sur **.NET piloté par Claude** ? Voici **tout** ce qu'il te faut, en partant de rien. À faire **une seule fois** — ensuite, chaque projet réutilise cette base.

## A. Les comptes

1. **Compte Claude** — sur [claude.com](https://claude.com). Prends un **abonnement qui donne accès à Claude Code** (vérifie les offres du moment sur le site). Ce compte sert aux **deux** IA : **Cowork** (le cerveau) et **Claude Code** (les mains).
2. **Compte Microsoft** — gratuit, sur [account.microsoft.com](https://account.microsoft.com). Il sert pour Windows, Visual Studio (gratuit), et — si tu veux — une VM dans le cloud Azure.

## B. Les logiciels (sur ton poste)

1. **L'application Claude (desktop)** — c'est là que tourne **Cowork**, ton copilote de cadrage. Télécharge-la depuis [claude.com](https://claude.com).
2. **Visual Studio Community** (gratuit) — l'éditeur .NET ; installe la charge de travail « .NET ». (Ou **VS Code** si tu préfères plus léger.)
3. **Git** — pour cloner / récupérer / publier le code ([git-scm.com](https://git-scm.com)).

## C. Le serveur Git (où vit le code) — une fois

Il te faut un endroit central où le code est **versionné** et **partagé** entre ton poste et la VM Claude Code. **C'est l'équivalent de ton GDS WinDev** (Gestionnaire de Sources) : git/Gitea **remplace ton GDS**, en standard et universel.

- **Recommandé : Gitea auto-hébergé** (privé, gratuit). Le plus simple : sur un **NAS Synology** (paquet « Git Server » ou, mieux, **Container Manager**). Suis un how-to dédié :
  - doc officielle Gitea : [docs.gitea.com](https://docs.gitea.com) ;
  - tuto NAS : cherche « installer Gitea Synology » (ex. *Marius Hosting*).
- **Alternative** : **GitHub**, si tu ne veux rien héberger (mais le code part chez un tiers).

> Le serveur Git s'installe **une fois** ; ensuite tu y crées **un dépôt par projet**.

## D. La VM Claude Code — et pourquoi c'est important

**Claude Code agit comme un humain sur une machine** : il ouvre des terminaux, **lance des programmes**, installe des outils, exécute des commandes, **prend la main** sur la session. Trois raisons de lui dédier **sa propre machine** (une VM) plutôt que de le laisser tourner sur ton PC :

1. **Il ne te dérange pas.** Sur ton PC, il te **prendrait la main** (focus, fenêtres, terminal) pendant que tu travailles, et pourrait perturber ton environnement. Sur sa VM, il bosse dans son coin.
2. **Elle tourne même quand ton PC est éteint.** Une VM hébergée sur un serveur / hyperviseur (ou dans le cloud) **reste allumée** : Claude Code peut **continuer à travailler** sur des tâches longues sans dépendre de ton poste.
3. **Tu cloisonnes ses accès.** Sur une VM (serveur / cloud), il n'a accès **qu'à ce que tu lui donnes** — tu maîtrises son périmètre. Sur ton PC, il hérite de **tout ce à quoi TOI tu as accès** (tes fichiers, tes partages réseau, tes identifiants enregistrés), donc bien plus large et bien plus risqué. La VM = un **bac à sable** que tu contrôles.

**Comment** : une VM **Windows** sur ton hyperviseur (Proxmox, Hyper-V, un petit serveur…) **ou** dans le cloud (Azure, via ton compte Microsoft). Installe dessus : **Claude Code**, **Visual Studio + SDK .NET**, **Git** — plus un **environnement de dev complet et autonome** (ci-dessous).

**Un environnement de dev COMPLET, en local sur la VM** *(selon ton projet — n'installe que l'utile)*. Objectif : Claude Code doit pouvoir **se connecter aux données et tout exécuter en local**, sans jamais toucher la prod ni sortir du **bac à sable**. Installe donc ce dont **ta source** et **ta cible** ont besoin, par exemple :

- un ou plusieurs **moteurs de base, en local** : **SQL Server Express**, un **serveur HFSQL** (pour relire / rejouer l'existant PC SOFT), **MySQL / MariaDB**… ;
- les **connecteurs d'accès aux données** : **OLE DB** et **ODBC** (+ les pilotes spécifiques, ex. *Native Access / pilote HFSQL*) — pour que le code .NET **et** les outils d'analyse atteignent chaque source ;
- une **copie locale des données** (base restaurée / importée depuis `origine/`), **jamais la prod**.

Ainsi Claude Code développe et teste **dans un environnement cloisonné** et autosuffisant, sans accès au réseau de prod.

> *(Optionnel mais conseillé)* une **2ᵉ VM de test** pour rejouer les tests chaque nuit — voir [02 — Infrastructure](02-infrastructure.md).

---

Une fois **A → D** en place, tu es prêt : passe au **Démarrage rapide** du [README](README.md) (créer le dépôt du projet + déposer les templates + coller le prompt d'init).
