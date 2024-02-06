---
title: Gérez vos applications
---

<a name="introduction"></a>
## Introduction

Par défaut, il est supposé que vous n'avez l'intention d'utiliser BlitzPHP que pour gérer une seule application, que vous construirez dans votre répertoire `app`. Il est cependant possible d'avoir plusieurs ensembles d'applications qui partagent une seule installation de BlitzPHP, ou même de renommer ou de déplacer votre répertoire d'applications.

> **Attention**  
> Lorsque vous avez installé BlitzPHP, vous devez supprimer ces lignes, et lancer la commande `composer dump-autoload`.
> ```json  
> {  
>   ...
>   "autoload": {
>       "psr-4": {
>           "App\\": "app", <-- Retirer cette ligne  
>       }
>   },
>   ...
> }
> ```

<a name="renommer-ou-deplacer-le-repertoire-de-l-application"></a>
## Renommer ou déplacer le répertoire de l'application

Si vous souhaitez renommer votre répertoire d'application ou même le déplacer à un autre endroit sur votre serveur, autre que la racine de votre projet, ouvrez votre fichier principal `app/Config/paths.php` et définissez un chemin d'accès complet au serveur dans le paramètre `app` (à la ligne **31** environ) :

```php
return [
    //---
    
    'app' => '/chemin/vers/votre/app',
    
    //---
];
```

Vous devrez modifier un fichiers supplémentaire à la racine de votre projet, afin qu'ils puissent trouver le fichier de configuration `paths` :

* **/public/index.php** est le contrôleur frontal de votre application web.
```php
<?php

$paths_config_file = __DIR__ . '/../app/Config/paths.php';
// ^^^ Changez cette ligne si vous déplacez le dossier de votre application
```

> **Note**  
> En fonction du nouvel emplacement de votre dossier d'application, vous pouvez être amener a modifier également le paramètre `composer` dans le fichier `app/Config/paths.php` (environ à la ligne **63**).

<a name="executer-plusieurs-applications-avec-un-seul-installation-blitzphp"></a>
## Exécuter plusieurs applications avec un seul installation BlitzPHP

Si vous souhaitez partager une installation commune du framework BlitzPHP, pour gérer plusieurs applications différentes, placez simplement tous les répertoires situés dans votre répertoire d'application dans leur propre (sous-)-répertoire.

Par exemple, imaginons que vous souhaitiez créer deux applications, nommées **foo** et **bar**. Vous pourriez structurer les répertoires de vos projets d'application comme suit :

```
foo/
    app/
    public/
    spec/
    storage/
    .env.example
    klinge
bar/
    app/
    public/
    spec/
    storage/
    .env.example
    klinge
vendor/
    autoload.php
    blitz-php/
composer.json
composer.lock
```

Il s'agirait de deux applications, **foo** et **bar**, ayant toutes deux des répertoires d'application standard et un dossier **public**, et partageant un **vendor** commun.

Le paramètre `composer` dans `app/Config/paths.php` à l'intérieur de chacun d'entre eux sera définie pour faire référence au dossier commun partagé **vendor** :

```php
return [
    //---
    
    'composer' => __DIR__ . '/../../../vendor/',
    
    //---
];
```

<a name="gestion-d-environnements-multiples"></a>
## Gestion d'environnements multiples

Les développeurs souhaitent souvent que le système se comporte différemment selon que l'application s'exécute dans un environnement de développement ou de production. Par exemple, une sortie d'erreur verbeuse peut être utile lors du développement d'une application, mais elle peut également poser un problème de sécurité lorsqu'elle est utilisée en production. Dans les environnements de développement, il se peut que vous souhaitiez que des outils supplémentaires soient chargés, ce qui n'est pas le cas dans les environnements de production, etc.

<a name="les-environnements-definis"></a>
### Les environnements définis

Par défaut, BlitzPHP a trois environnements définis.

* `production` pour la production
* `development` pour le développement
* `testing` pour les tests unitaires

> **Attention**  
> L'environnement `testing` sont réservés aux tests unitaires. Il y a des conditions spéciales intégrées dans le framework à différents endroits pour aider à cela. Vous ne pouvez pas l'utiliser pour votre développement.

Si vous souhaitez disposer d'un autre environnement, par exemple pour la mise à l'essai (`stagging`), vous pouvez ajouter des environnements personnalisés.

<a name="definition-de-l-environnement"></a>
### Définition de l'environnement

<a name="la-constante-environment"></a>
#### La constante ENVIRONMENT

Pour définir votre environnement, BlitzPHP est fourni avec la constante `ENVIRONMENT`. Si vous définissez `$_SERVER['ENVIRONMENT']`, la valeur sera utilisée, sinon la valeur par défaut sera `production`.

Cette constante peut être définie de plusieurs manières en fonction de la configuration de votre serveur.

**.env**

La méthode la plus simple pour définir cette variable est de le faire dans votre [fichier .env](/docs/{version}/configuration#le-fichier-dotenv).

```conf
ENVIRONMENT = development
```

> **Note**  
> Vous pouvez modifier la valeur de ENVIRONMENT dans le fichier **.env** à l'aide de la commande `klinge env` :  
> ```bash  
> php klinge env production  
> ```

**Apache**

Cette variable serveur peut être définie dans votre fichier **.htaccess** ou dans la configuration d'Apache à l'aide de <a href="https://httpd.apache.org/docs/2.4/mod/mod_env.html#setenv" target="_blank">SetEnv</a>.

```apache
SetEnv ENVIRONMENT production
```

**Nginx**

Sous nginx, vous devez passer la variable d'environnement par le biais de `fastcgi_params` pour qu'elle apparaisse sous la variable `$_SERVER`. Cela lui permet de fonctionner au niveau de l'hôte virtuel, au lieu d'utiliser env pour la définir pour l'ensemble du serveur, bien que cela fonctionnerait très bien sur un serveur dédié. Vous devez alors modifier la configuration de votre serveur en quelque chose comme :

```nginx
server {
    server_name localhost;
    include     conf/defaults.conf;
    root        /var/www;

    location    ~* \.php$ {
        fastcgi_param ENVIRONMENT "production";
        include conf/fastcgi-php.conf;
    }
}
```

D'autres méthodes sont disponibles pour nginx et d'autres serveurs, ou vous pouvez supprimer entièrement cette logique et définir la constante en fonction de l'adresse IP du serveur (par exemple).

En plus d'affecter le comportement de base du framework (voir la section suivante), vous pouvez utiliser cette constante dans votre propre développement pour différencier l'environnement dans lequel vous travaillez.

<a name="confirmer-l-environnement-actuel"></a>
### Confirmer l'environnement actuel

Pour confirmer l'environnement actuel, il suffit d'afficher la constante `ENVIRONMENT`.

Vous pouvez également vérifier l'environnement actuel à l'aide de la commande `klinge env` :

```bash  
php klinge env
```

<a name="effets-sur-le-comportement-par-defaut-du-framework"></a>
### Effets sur le comportement par défaut du framework

Il y a quelques endroits dans le système BlitzPHP où la constante `ENVIRONMENT` est utilisée. Cette section décrit comment le comportement par défaut du framework est affecté.

<a name="rapport-d-erreur"></a>
#### Rapport d'erreur

Si la constante `ENVIRONMENT` a la valeur `development`, toutes les erreurs PHP seront affichées dans le navigateur lorsqu'elles se produiront. Inversement, mettre la constante à la valeur `production` désactivera toutes les sorties d'erreurs. Désactiver les rapports d'erreur en production est une bonne pratique de sécurité.