---
title: Structure des dossiers
---

<a name="introduction"></a>
## Introduction

La structure d’application BlitzPHP par défaut est destinée à fournir un excellent point de départ pour les grandes et les petites applications. Mais vous êtes libre d’organiser votre application comme vous le souhaitez. Néanmoins, pour tirer le meilleur parti de BlitzPHP, vous devez comprendre comment l'application est structurée par défaut et ce que vous pouvez modifier pour répondre à vos besoins spécifiques.

<a name="dossiers-par-defaut"></a>
## Dossiers par défaut

Une nouvelle installation BlitzPHP comporte cinq dossiers: `app`, `public`, `storage`, `spec` et `vendor`. Chacun de ces répertoires a un rôle très spécifique à jouer.

<a name="le-dossier-de-l-application"></a>
### Le dossier de l'application

Le dossier `app` contient le code principal de votre application. Nous explorerons ce répertoire plus en détail bientôt; Toutefois, presque toutes les classes de votre application se trouvent dans ce répertoire.

<a name="le-dossier-public"></a>
### Le dossier public

Le dossier `public` contient la partie accessible par navigateur de votre application Web, empêchant l’accès direct à votre code source. Il contient le fichier `.htaccess` principal, `index.php` et toutes les ressources statiques que vous ajoutez, telles que CSS, Javascript ou images.

Ce dossier est destiné à être la « racine Web » de votre site et de votre serveur Web devra donc être configuré pour pointer vers elle.

<a name="le-dossier-de-stockage"></a>
### Le dossier de stockage

Le dossier `storage` contient vos log, sessions basées sur des fichiers, caches de fichiers et autres fichiers générés par l’infrastructure. Ce répertoire est séparé en 3 dossiers à savoir `app`, `framework` et `logs`. Le dossier `app` peut être utilisé pour stocker tous les fichiers générés par votre application (images uploadées par exemple). Le dossier `framework` est utilisé pour stocker les fichiers générés par le framework et les caches. Enfin, le dossier `logs` contient les fichiers journaux de votre application.

Le dossier `storage/app/public` peut être utilisé pour stocker des fichiers générés par l’utilisateur, tels que des avatars de profil, qui doivent être accessibles au public. Vous devez créer un lien symbolique vers ce dossier. Vous pouvez créer le lien à l’aide de la commande Klinge. `php klinge storage:link`.

<a name="le-dossier-des-tests"></a>
### Le dossier des tests

Les tests unitaires et les tests fonctionnels de votre application sont logés dans le dossier `spec`. BlitzPHP utilise [Kahlan](https://kahlan.github.io/docs/) pour l'écriture des tests.

<a name="le-dossier-des-dependances"></a>
### Le dossier des dépendances

Le dossier `vendor` contient vos dépendances installées via [Composer](https://getcomposer.org/).

<a name="zoom-sur-le-dossier-de-l-application"></a>
## Zoom sur le dossier de l'application

Toute votre application est logée dans le dossier `app`. Par défaut, bien que vous soyez libre de le modifier, tous les fichiers de ce dossier se trouvent sous le namespace `App` et est chargé automatiquement par [Composer](https://getcomposer.org/) à l'aide de la [norme de chargement automatique PSR-4](https://www.php-fig.org/psr/psr-4/).

Le dossier de `app` contient une variété de sous-dossiers supplémentaires tels que `Config`, `Controllers`, `Views`. Plusieurs autres sous-dossiers seront générés dans le `app` lorsque vous utiliserez les commandes Klinge `make` pour générer des classes.

> **Note**  
> La plupart des classes du dossier `app` peuvent être générées par Klinge via des commandes. Pour consulter les commandes disponibles, exécutez la commande `php klinge list make` dans votre terminal.

<a name="les-dossiers-de-base"></a>
### Les dossiers de base

Comme dit précédemment, le dossier `app` est le dossier où vous passerez la majeur partie de votre temps. Par défaut, ce dossier contient la liste des sous-dossiers suivants:
* `Config`: Stocke les fichiers de configuration
* `Controllers`: Les contrôleurs déterminent le déroulement du programme
* `Middlewares`: Stocke les classes de middlewares qui peuvent s'exécuter avant et après le contrôleur
* `Helpers`: Stock des collections de fonctions autonomes
* `Translations`: Le support multilingue lit les chaînes de langue à partir d'ici
* `Views`: Les vues constituent le code HTML affiché au navigateur

Étant donné que le dossier `app` est déjà doté d'un namespace, vous pouvez modifier la structure de ce dossier en fonction des besoins de votre application. Par exemple, vous pouvez décider de commencer à utiliser le modèle `Repository` et les `Entity` pour travailler avec vos données. Dans ce cas, vous pouvez ajouter les dossiers `Entities` et `Repositories`.

> **Note**  
> Cependant, si vous renommez le dossier `Controllers`, vous ne pourrez pas utiliser le routage automatique vers les contrôleurs et devrez [définir toutes vos routes](/docs/{version}/routage) dans le fichier de routes.

En plus des principaux sous-dossiers cités ci-dessus, le dossier `app` peut contenir d'autres sous dossiers que nous aborderons dès les sections suivantes.

<a name="le-dossier-commands"></a>
### Le dossier `Commands`

Le dossier `Commands` contient toutes les commandes Klinge personnalisées pour votre application. Ces commandes peuvent être générées à l'aide de la commande `make:command`.

<a name="le-dossier-events"></a>
### Le dossier `Events`

Ce dossier n'existe pas par défaut, mais sera créé pour vous par la commande Klinge `make:event`. Il héberge [les classes d'événements](/docs/{version}/evenements). Les événements peuvent être utilisés pour alerter d'autres parties de votre application qu'une action donnée s'est produite, offrant ainsi une grande flexibilité et un découplage.

<a name="le-dossier-exceptions"></a>
### Le dossier `Exceptions`

Le dossier `Exceptions` constitue également un bon endroit pour placer toutes les exceptions levées par votre application.

<a name="le-dossier-mail"></a>
### Le dossier `Mail`

Ce dossier n'existe pas par défaut, mais sera créé pour vous si vous exécutez la commande Klinge `make:mail`. C'est un dossier qui contient toutes vos [classes qui représentent les emails](/docs/{version}/email) envoyés par votre application. Les objets Mail vous permettent d'encapsuler toute la logique de création d'un e-mail dans une classe unique et simple qui peut être envoyée à l'aide de la méthode `Mail::envoi`.

<a name="le-dossier-providers"></a>
### Le dossier `Providers`

Bien que n'existant pas par défaut, le dossier `Providers` est fait pour contenir tous les [fournisseurs de services](/docs/{version}/conteneur) de votre application. Les fournisseurs de services amorcent votre application en liant des services dans le conteneur de services, en enregistrant des événements ou en effectuant d’autres tâches pour préparer votre application aux requêtes entrantes.

<a name="le-dossier-rules"></a>
### Les dossiers `Rules` et `Validations`

Ces dossiers n'existent pas par défaut, mais seront créés pour vous si vous exécutez la commande Klinge `make:rule` ou `make:validation`. 

Le dossier `Rules` contient les objets de règles de validation personnalisées pour votre application. Les règles sont utilisées pour encapsuler une logique de validation complexe dans un objet simple. 

Le dossier `Validations` contient les classes de validation pour des scénarios plus complexes. Les classes de validation encapsulent leur propre logique de validation et d’autorisation.

Pour plus d’informations, consultez la [documentation de validation](/docs/{version}/validation).

<a name="le-dossier-database"></a>
### Le dossier `Database`

Par défaut, ce dossier n'existe pas mais il sera automatiquement créé lorsque vous déciderez d'[utiliser une base de données](/docs/{version}/base-de-donnees) dans votre projet. Le dossier de `Database` contient vos migrations de bases de données et vos seeders. Si vous le souhaitez, vous pouvez également utiliser ce répertoire pour contenir une base de données SQLite.

<a name="modification-de-l-emplacement-des-dossiers"></a>
## Modification de l'emplacement des dossiers

Si vous avez déplacé l'un des dossiers principaux, vous pouvez modifier les paramètres de configuration dans `app/Config/paths.php`.

Veuillez lire "[Gérer vos applications](/docs/{version}/gerez-vos-applications)" pour plus d'informations à ce sujet .