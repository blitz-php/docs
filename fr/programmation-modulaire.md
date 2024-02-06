---
title: Programmation modulaire
---

<a name="introduction"></a>
## Introduction

BlitzPHP prend en charge une forme de modularisation du code pour vous aider à créer un code réutilisable. Les modules sont généralement centrés sur un sujet spécifique et peuvent être considérés comme des mini-applications au sein d'une application plus large.

Tous les types de fichiers standard du framework sont pris en charge, comme les contrôleurs, les modèles, les vues, les fichiers de configuration, les helpers, les fichiers de langue, etc. Les modules peuvent contenir aussi peu ou autant de fichiers que vous le souhaitez.

<a name="namespaces"></a>
## Namespaces

L'élément central de la fonctionnalité des modules provient de [l'autoloading compatible PSR-4](/docs/{version}/autoloader) que BlitzPHP utilise. Bien que n'importe quel code puisse utiliser l'autoloader PSR-4 et les espaces de noms, la principale façon de tirer pleinement parti des modules est de créer un espace de noms pour votre code et de l'ajouter à `app/Config/autoload.php`, dans le paramètre `psr4`.

Par exemple, disons que nous voulons conserver un module de blog simple que nous pouvons réutiliser entre les applications. Nous pourrions créer un dossier avec le nom de notre société, `Acme`, pour y stocker tous nos modules. Nous le placerons juste à côté de notre répertoire d'applications dans la racine du projet principal :

```
acme/        // Nouveau répertoire de modules
app/
public/
spec/
storage/
```

Ouvrez le fichier `app/Config/autoload.php` et ajoutez l'espace de noms `Acme\Blog` au tableau `psr4` :

```php
return [
    /**
     * @var array<string, array<int, string>|string>
     */
    'psr4' => [
        APP_NAMESPACE => APP_PATH, // Pour l'espace de noms d'application personnalisé
        'Acme\Blog'   => ROOTPATH . 'acme/Blog',
    ],
    
    //---
];
```

Maintenant que tout est en place, nous pouvons accéder à n'importe quel fichier du dossier `acme/Blog` par l'intermédiaire du namespace `Acme\Blog`. Cela suffit à prendre en charge 80 % de ce qui est nécessaire pour que les modules fonctionnent, vous devez donc vous familiariser avec les namespace et vous sentir à l'aise avec leur utilisation. Plusieurs types de fichiers seront recherchés automatiquement dans tous les espaces de noms définis - un ingrédient crucial pour travailler avec les modules.

Une structure de répertoire commune au sein d'un module imitera le dossier principal de l'application :

```
acme/
    Blog/
        Config/
        Controllers/
        Database/
            Migrations/
            Seeds/
        Entities/
        Helpers/
        Language/
            en/
        Translations/
        Views/
```

Bien sûr, rien ne vous oblige à utiliser cette structure exacte, et vous devez l'organiser de la manière qui convient le mieux à votre module, en supprimant les répertoires dont vous n'avez pas besoin, en créant de nouveaux répertoires pour les modèles, les contrats ou les référentiels, etc.

<a name="chargement-automatique-de-fichiers-non-classe"></a>
## Chargement automatique de fichiers non-classe

Plus souvent qu'autrement, votre module ne contiendra pas seulement des classes PHP, mais aussi d'autres fonctions procédurales, des fichiers d'amorçage, des fichiers de constantes de module, etc. qui ne sont pas normalement chargés de la même manière que les classes. Une approche pour cela est d'utiliser un `require` ou `include` du fichier au début du fichier où il sera utilisé.

Une autre approche fournie par BlitzPHP est de charger automatiquement ces fichiers non-classes comme vous le feriez pour vos classes. Tout ce que nous avons à faire est de fournir la liste des chemins vers ces fichiers et de les inclure dans le paramètre `files` de votre fichier `app/Config/autoload.php`.

```php
return [
    //---
    
    /**
     * @var string[]
     */
    'files' => [
        'path/to/my/functions.php',
        'path/to/my/constants.php',
        'path/to/my/bootstrap.php',
    ],
    
    //---
];
```

<a name="decouverte-automatique"></a>
## Découverte automatique

Souvent, vous devrez spécifier l'espace de noms complet des fichiers que vous souhaitez inclure, mais BlitzPHP peut être configuré pour simplifier l'intégration des modules dans vos applications en découvrant automatiquement de nombreux types de fichiers différents, y compris :

* [Les évènements](/docs/{version}/evenements)
* [Les registres](/docs/{version}/configuration#les-registres)
* [Les fichiers de route](/docs/{version}/routage)
* [Les services](/docs/{version}/services)

Le système d'auto-découverte fonctionne en recherchant des répertoires et des fichiers particuliers dans les namespace psr4 qui ont été définis dans `app/Config/autoload.php` et dans les paquets Composer.

Le processus de découverte recherchera les éléments découvrables sur ce chemin et devrait, par exemple, trouver le fichier routes à l'adresse `acme/Blog/Config/routes.php`.

<a name="decouverte-et-composer"></a>
### Découverte et Composer

Les paquets installés via Composer en utilisant les espaces de noms PSR-4 seront également découverts par défaut. Les paquets utilisant les namespaces PSR-0 ne seront pas détectés.

<a name="spécifier-les-paquets-composer"></a>
#### Spécifier les paquets Composer

Pour éviter de perdre du temps à rechercher des paquets Composer non pertinents, vous pouvez spécifier manuellement les paquets à découvrir en modifiant le paramètre `composer` dans `app/Config/autoload.php` :

```php
return [
    //---
    
    'composer' => [
        //---
        
        /**
         * @var array{only?: list<string>, exclude?: list<string>}
         */
        'packages' => [
            'only' => [
                // Liste de tous les paquets à découvrir automatiquement  
                'blitz-php/schild',
            ],
        ],
    ],
    
    //---
];
```

Vous pouvez également spécifier les paquets à exclure de la découverte.

```php
return [
    //---
    
    'composer' => [
        //---
        
        /**
         * @var array{only?: list<string>, exclude?: list<string>}
         */
        'packages' => [
            'exclude' => [
                // Liste de tous les paquets à exclure
                'filp/whoops',
            ],
        ],
    ],
    
    //---
];
```

<a name="desactiver-la-decouverte-des-paquets-composer"></a>
#### Désactiver la découverte des paquets Composer

Si vous ne souhaitez pas que tous les répertoires connus de Composer soient analysés lors de la recherche de fichiers, vous pouvez désactiver cette fonction en modifiant le paramètre `composer.discover` dans votre fichier `app/Config/autoload.php` :

```php
return [
    //---
    
    'composer' => [
        /**
         * @var bool
         */
        'discover' => false,
        
        //---
    ],
    
    //---
];
```

<a name="travailler-avec-des-fichiers"></a>
## Travailler avec des fichiers

Cette section examine chacun des types de fichiers (contrôleurs, vues, fichiers de langue, etc.) et la manière dont ils peuvent être utilisés dans le module. Certaines de ces informations sont décrites plus en détail dans la partie correspondante de la documentation, mais elles sont reproduites ici afin qu'il soit plus facile de comprendre comment tous les éléments s'imbriquent les uns dans les autres.

<a name="routes"></a>
### Routes

Par défaut, [les routes](/docs/{version}/routage) sont automatiquement recherchés dans les modules.

> **Note**  
> Étant donné que les fichiers sont inclus dans le scope actuel, l'instance `$routes` est déjà définie pour vous. Des erreurs se produiront si vous tentez de redéfinir cette variable.

Lorsque l'on travaille avec des modules, il peut y avoir un problème si les routes de l'application contiennent des caractères génériques. Dans ce cas, voir [Priorité des routes](/docs/{version}/routage#priorite-de-route).

<a name="controleurs"></a>
### Contrôleurs

Les contrôleurs situés en dehors du répertoire principal `app/Controllers` ne peuvent pas être automatiquement acheminés par la détection d'URI, mais doivent être spécifiés dans le fichier `routes.php` lui-même :

```php
<?php

// routes.php
Route::get('blog', '\Acme\Blog\Controllers\BlogController::index');
```

La fonction de routage de groupe est utile pour réduire la quantité de données à saisir :

```php
<?php

// routes.php
Route::prefix('blog')->namespace('Acme\Blog\Controllers')->group(static function () {
    Route::get('/', 'BlogController::index');
});
```

<a name="fichier-de-configuration"></a>
### Fichiers de configuration

Les configurations seront automatiquement trouvées dès lors qu'elles se trouvent dans le répertoire `app/Config`. 

> **Note**  
> Nous vous déconseillons d'utiliser le même nom de fichier de configuration dans les modules. Les modules qui ont besoin de remplacer ou d'ajouter des configurations connues dans `app/Config/` doivent utiliser des [registres implicites](/docs/{version}/configurations#les-registres).

> **Note**  
> Si un module a un fichier de configuration ayant le même nom qu'un fichier de configuration de l'application, les configuration de l'application seront utilisées.  
> Par exemple, si vous avez deux fichiers `app/Config/app.php` et `modules/Acme/Config/app.php`; les configurations du fichier `modules/Acme/Config/app.php` ne seront jamais récupérer. 

<a name="migrations"></a>
### Migrations

Les fichiers de migration seront automatiquement découverts dans les namespaces définis. Toutes les migrations trouvées dans tous les namespacs seront exécutées à chaque fois.

<a name="seeds"></a>
### Seeds

Les fichiers de seeds peuvent être utilisés à la fois à partir du CLI et appelés à partir d'autres fichiers de seeds, à condition que le namespace complet soit fourni. Si vous les appelez à partir du CLI, vous devrez fournir des doubles barres obliques inverses :

Pour Unix:

```shell
php klinge db:seed Acme\\Blog\\Database\\Seeds\\TestPostSeeder
```

Pour Windows:

```shell
php klinge db:seed Acme\Blog\Database\Seeds\TestPostSeeder
```

<a name="helpers"></a>
### Helpers

Les helpers seront automatiquement découvertes dans les namespaces définis lors de l'utilisation de la fonction `helper()`, à condition qu'elles se trouvent dans le répertoire `Helpers` du namespace :

```php
helper('blog');
```

Vous pouvez spécifier des namespaces. Voir [Chargement à partir d'un namespace spécifié](/docs/{version}/helpers#chargement-a-partir-d-emplacements-non-standard) pour plus de détails.

<a name="fichiers-de-traduction"></a>
### Fichiers de traduction

Les fichiers de traductions sont localisés automatiquement à partir des namespaces définis lorsque l'on utilise les fonction `lang()` ou `__()`, à condition que le fichier suive les mêmes structures de répertoire que le répertoire principal de l'application.

<a name="models"></a>
### Models

Si vous instanciez les modèles avec le mot-clé `new` par leur nom de classe pleinement qualifié, aucun accès spécial n'est fourni :

```php
<?php

$model = new \Acme\Blog\Models\PostModel();
```

Les fichiers modèles sont automatiquement découverts à l'aide de la fonction `model()` qui est toujours disponible.

> **Note**  
> Nous vous déconseillons d'utiliser le même nom de classe abrégé dans les modules.

> **Note**  
> `model()` trouve le fichier dans `app/Models/` quand il y a une classe avec le même nom court, même si vous spécifiez un nom de classe pleinement qualifié comme `model(\Acme\Blog\Model\PostModel::class)`. C'est parce que `model()` est une enveloppe pour la classe `FileLoader` qui utilise preferApp par défaut. Voir Chargement des classes pour plus d'informations.

<a name="vues"></a>
### Vues

Les vues peuvent être chargées en utilisant le namespace des classes comme décrit dans la [documentation sur les vues](/docs/{version}/vues#vues-avec-namespace) :

```php
<?php

echo view('Acme\Blog\Views\index');
```