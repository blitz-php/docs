---
title: Rendu des vues
---

<a name="introduction"></a>
## Introduction

Il n'est pas pratique de renvoyer des chaînes de documents HTML entières directement à partir de vos routes et de vos contrôleurs. Heureusement, les vues offrent un moyen pratique de placer tout notre HTML dans des fichiers séparés.

Une vue est simplement une page web, ou un fragment de page, comme un header, un footer, une sidebar, etc. 

Les vues séparent votre contrôleur / logique d'application de votre logique de présentation et sont stockées dans le dossier `app/Views`. Lorsque l'on utilise BlitzPHP, les vues possèdent généralement une extension `.php` et utilisent la <a href="https://php.net/manual/fr/control-structures.alternative-syntax.php" target="_blank">syntaxe alternative de PHP</a> pour les structures de contrôle et les sorties. Ces fichiers contiennent la logique nécessaire pour servir les données reçues d’un contrôleur dans un format de présentation qui est lisible par votre public.   
  
Une vue simple pourrait ressembler à ceci:

```html
<!-- Vue stockée dans app/Views/greeting.php -->
 
<html>
    <body>
        <h1>Hello, <?= $name ?></h1>
    </body>
</html>
```

Comme cette vue est stockée dans `app/Views/greeting.php`, nous pouvons la renvoyer en utilisant la fonction globale `view()` de la manière suivante :

```php
use BlitzPHP\Facades\Route;

Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

<a name="ecrire-des-vues-dans-react-vue-js"></a>
## Écrire des vues en React / Vue.JS

Au lieu d'écrire leurs vues en PHP, de nombreux développeurs ont commencé à préférer écrire leurs vues en utilisant React ou VueJS. Si vous êtes de ceux là, BlitzPHP vous offre un [adaptateur](/docs/{version}/inertia) pour utiliser <a href="https://inertiajs.com/" target="_blank">Inertia</a> afin de lier votre frontend React / Vue à votre backend BlitzPHP sans les complexités typiques de la construction d'une SPA.

<a name="creation-et-rendu-de-vues"></a>
## Création et rendu de vues

Vous pouvez créer une vue en plaçant un fichier portant l'extension `.php` dans le répertoire `app/Views` de votre application.

Une fois que vous avez créé une vue, vous pouvez la renvoyer depuis l'une des routes ou l'un des contrôleurs de votre application à l'aide de la fonction globale `view()` :

```php
Route::get('/', function () {
    return view('greeting');
});
```

Les vues peuvent également être renvoyées à l'aide de la façade `View` :

```php
use BlitzPHP\Facades\View;
 
return View::make('greeting');
```


Comme vous pouvez le voir, l'argument passé à la fonction `view()` correspond au nom du fichier de vue dans le répertoire `app/Views`. 

<a name="stockage-des-vues-dans-des-sous-dossiers"></a>
### Stockage des vues dans des sous-dossiers

Vos fichiers de vues peuvent également être stockés dans des sous-dossiers si vous préférez ce type d'organisation. Pour ce faire, vous devrez inclure le nom du dossier dans lequel se trouve la vue. Exemple :

```php
return view('directory_name/file_name');
```

<a name="vues-avec-namespace"></a>
### Vues avec namespace

Vous pouvez stocker des vues dans un dossiers `Views` qui est un namespace, et charger cette vue comme si elle était un namespace. Bien que PHP ne supporte pas le chargement de fichiers non-classes à partir d'un namespace, BlitzPHP fournit cette fonctionnalité pour rendre possible l'empaquetage de vos vues à la manière d'un module pour une réutilisation ou une distribution aisée.

Si vous avez un dossier `example/blog` qui a un mappage PSR-4 configuré dans l'autoloader vivant sous le namespace `ExampleBlog`, vous pouvez récupérer les fichiers de vue comme s'ils étaient également dans le namespace.

En suivant cet exemple, vous pourriez charger le fichier `blog_view.php` à partir de `exemple/blog/Views`

```php
<?php

return view('Example\Blog\Views\blog_view');
```

<a name="chargement-de-plusieurs-vues"></a>
### Chargement de plusieurs vues

BlitzPHP va gérer intelligemment les appels multiples à `view()` à partir d'un contrôleur. Si plus d'un appel se produit, ils seront ajoutés l'un à l'autre. Par exemple, vous pouvez souhaiter avoir une vue de l'en-tête, une vue du menu, une vue du contenu et une vue du pied de page. Cela pourrait ressembler à ceci :

```php
<?php

namespace App\Controllers;

class Page extends AppController
{
    public function index()
    {
        $data = [
            'page_title' => 'Your title',
        ];

        return view('header')
            . view('menu')
            . view('content', $data)
            . view('footer');
    }
}
```

Dans l'exemple ci-dessus, nous utilisons des "données ajoutées dynamiquement", que vous verrez ci-dessous.

<a name="determiner-si-une-vue-existe"></a>
### Déterminer si une vue existe

Si vous devez déterminer si une vue existe, vous pouvez utiliser la façade `View`. La méthode `exists()` renvoie la valeur `true` si la vue existe :

```php
use BlitzPHP\Facades\View;
 
if (View::exists('emails/customer')) {
    // ...
}
```

Vous pouvez aussi utiliser la fonction globale `view_exist()` pour obtenir le même résultat :

```php
if (view_exist('emails/customer')) {
    // ...
}
```

<a name="creation-de-la-premiere-vue-disponible"></a>
### Création de la première vue disponible

En utilisant la méthode `first()` de la façade `View`, vous pouvez créer la première vue qui existe dans un tableau de vues donné. Cela peut s'avérer utile si votre application ou votre paquetage permet de personnaliser ou d'écraser les vues :

```php
use BlitzPHP\Facades\View;
 
return View::first(['custom.admin', 'admin'], $data);
```

<a name="transmission-de-donnees-aux-vues"></a>
## Transmission de données aux vues

Comme vous l'avez vu dans les exemples précédents, vous pouvez passer un tableau de données aux vues pour rendre ces données disponibles à la vue :

```php
return view('greetings', ['name' => 'Victoria']);
```

Lorsque vous transmettez des informations de cette manière, les données doivent être un tableau avec des paires clé/valeur. Après avoir fourni des données à une vue, vous pouvez ensuite accéder à chaque valeur dans votre vue en utilisant les clés des données, comme `<?= $name ?>`.

Au lieu de transmettre un tableau complet de données à la fonction `view()`, vous pouvez utiliser la méthode `with()` pour ajouter des éléments de données individuels à la vue. La méthode `with()` renvoie une instance de l'objet `View` afin que vous puissiez continuer à enchaîner les méthodes avant de renvoyer la vue :

```php
return view('greeting')
            ->with('name', 'Victoria')
            ->with('occupation', 'Astronaut');
```

<a name="partager-les-donnees-dans-toutes-les-vues"></a>
### Partager les données dans toutes les vues

Il peut arriver que vous deviez partager des données avec toutes les vues rendues par votre application. Pour ce faire, vous pouvez utiliser la méthode `share()` de la façade `View`. En règle générale, vous devez placer les appels à la méthode `share()` dans [l'écouteur d'évènement](/docs/{version}/evenements) `pre_system` ou dans le [constructeur du contrôleur](/docs/{version}/controleurs#constructeur) `AppController`. 

```php
<?php

namespace App\Events;

use BlitzPHP\Contracts\Event\EventListenerInterface;
use BlitzPHP\Contracts\Event\EventManagerInterface;
use BlitzPHP\Facades\View;

class Listener implements EventListenerInterface
{
    /**
     * {@inheritDoc}
     */
    public function listen(EventManagerInterface $event): void
    {
        $event->attach('pre_system', function () {
             View::share('key', 'value');
        });
    }
}

```

Vous pouvez aussi utiliser le fichier `app/Config/view.php` pour définir les données partagées. C'est une pratique plus **recommandée** :

```php
return [
    // ---
    'shared' => static fn (): array => [
        'key' => 'value',
    ],
    // --
];
```

<a name="l-option-savedata"></a>
### L'option `save_data`

Les données transmises sont conservées pour les appels ultérieurs à `view()`. Si vous appelez la fonction plusieurs fois dans une même requête, vous n'aurez pas à transmettre les données souhaitées à chaque `view()`.

Mais cela n'empêchera peut-être pas les données de "saigner" dans d'autres vues, ce qui pourrait causer des problèmes. Si vous préférez nettoyer les données après un seul appel, vous pouvez passer l'option `save_data` dans le tableau `$option` du troisième paramètre.

```php
$data = [
    'title'   => 'My title',
    'heading' => 'My Heading',
    'message' => 'My Message',
];

return view('greeting', $data, ['save_data' => false]);
```

En outre, si vous souhaitez que la fonctionnalité par défaut de la fonction `view()` soit d'effacer les données entre les appels, vous pouvez définir `$save_data` à `false` dans `app/Config/view.php`.

<a name="options-de-vue"></a>
## Options de vue

<a name="mise-en-cache"></a>
### Mise en cache

Vous pouvez mettre en cache une vue avec la fonction `view()` en passant une option de `cache` avec le nombre de secondes pour la mise en cache de la vue, dans le troisième paramètre :

```php
// Mettre la vue en cache pendant 60 secondes
return view('file_name', $data, ['cache' => 60]) ;
```

Par défaut, la vue sera mise en cache en utilisant le même nom que le fichier de vue lui-même. Vous pouvez personnaliser cela en passant `cache_name` et l'ID de cache que vous souhaitez utiliser :

```php
return view('file_name', $data, ['cache' => 60, 'cache_name' => 'my_cached_view']);
```

<a name="a-l-interieur-d-un-fichier-de-vue-blitzphp"></a>
## A l'intérieur d'un fichier de vue BlitzPHP

<a name="sorties-alternatives"></a>
### Sorties alternatives

Utilisez `echo` ou `print` sur une variable dans votre vue:

```php
<?php echo $variable; ?>
```

Avec le support de « Short Tag »:

```php
<?= $variable; ?>
```

<a name="structures-de-controle-alternatives"></a>
### Structures de contrôle alternatives

Les structures de contrôle tel que `if`, `for`, `foreach`, `switch`, et `while` peuvent être écrites dans un format simplifié. Remarquez l’absence d’accolades. À la place, l’accolade de fin du `foreach` est remplacée par `endforeach`. Chacune des structures de contrôle listées ci-dessous a une syntaxe de fermeture similaire: `endif`, `endfor`, `endforeach`, et `endwhile`. Vous remarquerez aussi qu’à la place du `point-virgule` après chaque structure (à l’exception de la dernière), il y a un `double-point`.

Voici un exemple de `foreach`:

```php
<ul>
<?php foreach ($todo as $item): ?>
    <li><?= $item ?></li>
<?php endforeach; ?>
</ul>
```

Un autre exemple utilisant if/elseif/else. Remarquez les doubles points:

```php
<?php if ($username === 'sally') : ?>
   <h3>Salut Sally</h3>
<?php elseif ($username === 'joe') : ?>
   <h3>Bonjour Joe</h3>
<?php else : ?>
   <h3>Salut utilisateur inconnu</h3>
<?php endif ; ?>
```

Si vous préférez utiliser un moteur de template comme <a href="https://twig.symfony.com/" target="_blank">Twig</a> ou <a href="https://laravel.com/docs/10.x/blade" target="_blank">Blade</a> , vous pourriez le faire facilement en vous référent à la section ci-dessous.

<a name="variables-de-vue"></a>
### Variables de vue

Vous devriez vous rappeler de toujours **échapper** les données d’utilisateur avant de les afficher puisque BlitzPHP n'échappe pas automatiquement la sortie. Vous pouvez échapper le contenu d’utilisateur avec la fonction `h()` ou `esc()`:

```php
<?= esc($user->bio); ?>
```


<a name="creation-de-boucles"></a>
### Création de boucles

Le tableau de données que vous transmettez à vos fichiers de vue n'est pas limité à de simples variables. Vous pouvez transmettre des tableaux multidimensionnels, qui peuvent être bouclés pour générer plusieurs lignes. Par exemple, si vous extrayez des données de votre base de données, elles se présenteront généralement sous la forme d'un tableau multidimensionnel.

Voici un exemple simple. Ajoutez-le à votre contrôleur :

```php
<?php

namespace App\Controllers;

class BlogController extends AppController
{
    public function index()
    {
        $data = [
            'todo_list' => ['Clean House', 'Call Mom', 'Run Errands'],
            'title'     => 'My Real Title',
            'heading'   => 'My Real Heading',
        ];

        return view('blog_view', $data);
    }
}
```

Ouvrez maintenant votre fichier de vue et créez une boucle :

```php
<html>
<head>
    <title><?= esc($title) ?></title>
</head>
<body>
    <h1><?= esc($heading) ?></h1>

    <h2>My Todo List</h2>

    <ul><?php foreach ($todo_list as $item): ?>
        
        <li><?= esc($item) ?></li>
        
    <?php endforeach ?></ul>
</body>
</html>
```

<a name="decorateurs-de-vue"></a>
## Décorateurs de vue

Les décorateurs de vues permettent à votre application de modifier la sortie HTML pendant le processus de rendu. Cela se produit juste avant la mise en cache et vous permet d'appliquer des fonctionnalités personnalisées à vos vues.

La création de vos propres décorateurs de vue nécessite la création d'une nouvelle classe qui implémente `BlitzPHP\Views\ViewDecoratorInterface`. Cela nécessite une méthode unique qui prend la chaîne HTML générée, effectue toutes les modifications nécessaires et renvoie le code HTML résultant.

```php
<?php

namespace App\Views\Decorators;

use BlitzPHP\Views\ViewDecoratorInterface ;

class MyDecorator implements ViewDecoratorInterface
{
    public static function decorate(string $html): string
    {
        // Modifier la sortie ici

        return $html ;
    }
}
```

Une fois créée, la classe doit être enregistrée dans `app/Config/view.php` :

```php
<?php

return [
    //---
    
    /**
     * Les decorateurs de vues sont des méthodes de classe qui seront exécutées en séquence pour avoir la possibilité de modifier la sortie générée juste avant la mise en cache des résultats.
     *
     * Toutes les classes doivent implémenter BlitzPHP\View\ViewDecoratorInterface
     *
     * @var classe-string<ViewDecoratorInterface>[]
      */
    'decorators' => [
        'App\Views\Decorators\MyDecorator',
    ],
    
    //---
];
```


<a name="utilisation-d-un-moteur-de-template"></a>
## Utilisation d'un moteur de template

Quand il s'agit de moteurs de template en PHP, beaucoup de gens vous diront que PHP lui-même est un moteur de template. Mais même si PHP a commencé sa vie comme un langage de template, il n'a pas évolué comme tel ces dernières années. En fait, il ne supporte pas de nombreuses fonctionnalités que les moteurs de template modernes devraient avoir de nos jours.

Si vous êtes fan de moteur de template, et souhaitez l'utiliser dans votre projet, ne vous en faites pas. BlitzPHP est compatible avec 5 des principaux moteurs de template PHP et dans cette section vous verrez comment intégrer chacun d'eux dans votre projet BlitzPHP.

<a name="apercu-des-configurations"></a>
### Aperçu des configurations

Avant de vous plonger dans l'utilisation d'un moteur de template, vous devez savoir que deux principaux paramètres de configurations sont nécessaires pour le faire. A l'intérieur de votre fichier `app/Config/view.php`, vous trouverai la configuration ci-dessous:

```php
<?php 

return [
    /**
     * Adapteur responsable du rendu des vues
     * 
     * voir le tableau `adapters` ci-dessous
     */
    'active_adapter' => 'native',

    // ---
    // ---
    
    'adapters' => [
        /**
         * [Configuration Native]
         */
        'native' => [
            //...
        ],

        /**
         * [Configuration Blade](https://laravel.com/docs/9.x/views)
         * 
         * executer `composer require jenssegers/blade`
         * require jenssegers/blade
         */
        'blade' => [
            //...
        ],

        /**
         * [Configuration Latte]
         * 
         * executer `composer require latte/latte`
         * requiet latte 3.x or higher
         */
        'latte' => [
            //...
        ],

        /**
         * [Configuration Plates](https://platesphp.com/engine/overview/)
         * 
         * executer `composer require league/plates`
         * requiet plates 3.x or higher
         */
        'plates' => [
            //...
        ],

        /**
         * [Configuration Smarty](https://www.smarty.net/docs/en/api.variables.tpl)
         * 
         * executer `composer require smarty/smarty`
         * requiet smarty 3.x or higher
         */
        'smarty' => [
            //...
        ],

        /**
         * [Configuration Twig](https://twig.symfony.com/doc/3.x/api.html#environment-options)
         * 
         * executer `composer require twig/twig`
         * requiet twig 3.x or higher
         */
        'twig' => [
            //...
        ]
    ]
];
```

* La clé `active_adapter` permet de déterminer quel moteur de template doit etre utiliser pour le rendu des vue. En plus du [moteur natif de BlitzPHP](/docs/{version}/vues-natives) (`native`), vous pouvez utiliser <a href="https://laravel.com/docs/10.x/blade" target="_blank">Blade</a>, <a href="https://twig.symfony.com/" target="_blank">Twig</a>, <a href="https://www.smarty.net/" target="_blank">Smarty</a>, <a href="https://latte.nette.org/" target="_blank">Latte</a> ou <a href="http://platesphp.com/" target="_blank">Plates</a>.
* La clé `adapters` est un tableau qui contient la configuration de chacun des moteurs pris en charge.

Pour activer un moteur de template particulier, il suffit donc de changer la valeur du paramètre `active_adapter` et **décommenter** la configuration de celui-ci dans le tableau `adapters`.

<a name="utilisation-de-blade"></a>
### Utilisation de Blade

[Blade](https://laravel.com/docs/master/blade) est sans doute le moteur de template le plus utilisé de nos jour. C'est un langage de modélisation extrêmement léger issu du framework [Laravel](https://laravel.com) qui fournit une syntaxe pratique et courte pour afficher des données, itérer sur des données, etc. :

```blade
<ul>
    @foreach ($users as $user)
        <li>Hello, {{ $user->name }} </li>
    @endforeach
</ul>
```

Pour intégrer Blade à BlitzPHP, commencez par ouvrir votre fichier de configuration `app/Config/View.php` et attribuer la valeur `blade` à la clé `active_adapter`. Par la suite, recherchez la section consacrée à blade dans le tableau `adapters` et décommentez les paramètres qui s'y trouvent. Votre fichier ressemblera donc à ceci:

```php
<?php 

return [
    /**
     * Adapteur responsable du rendu des vues
     * 
     * voir le tableau `adapters` ci-dessous
     */
    'active_adapter' => 'blade',

    // ---
    // ---

    'adapters' => [
        // ---

        /**
         * [Configuration Blade](https://laravel.com/docs/9.x/views)
         * 
         * executer `composer require jenssegers/blade`
         * require jenssegers/blade
         */
        'blade' => [
            /**
             * Répertoire où sont stockés les caches de vues
             * 
             * @var string
             */
            'cache_path'      => VIEW_CACHE_PATH.'blade'.DIRECTORY_SEPARATOR,

            /**
             * Enregistrez un gestionnaire pour les directives personnalisées.
             * 
             * @var array<string, callable>
             */
            'directives' => [],
            
            /**
             * Enregistrez une directive d'instruction "if".
             * 
             * @var array<string, callable>
             */
            'if' => [],

            /**
             * Fonction de configuration manuelle du moteur de template
             */
            'configure' => function (\Jenssegers\Blade\Blade $engine): \Jenssegers\Blade\Blade {

                 return $engine;
            },
        ],
        
        // ---
    ]
];
```

Pour finir, vous devez installer le package <a href="https://packagist.org/packages/jenssegers/blade" target="_blank">jenssegers/blade</a> avant de réellement commencer a écrire vos vues blade.

```bash
composer require beebmx/blade
```

Une fois ceci étant fait, vous pouvez commencer à écrire vos vues blade comme si vous étiez dans Laravel. Au niveau du contrôleur, rien ne change.

<a name="utilisation-de-twig"></a>
### Utilisation de Twig

Issu de l'écosystème <a href="https://symfony.com/" target="_blank">Symfony</a>,  <a href="https://twig.symfony.com/" target="_blank">Twig</a> est un moteur template rapide, sécurisé et flexible.

```twig
<ul>
    {% for user in users %}
        <li>Hello, {{ user.name }} </li>
    {% endfor %}
</ul>
```

Pour intégrer Twig à BlitzPHP, commencez par ouvrir votre fichier de configuration `app/Config/View.php` et attribuer la valeur `twig` à la clé `active_adapter`. Par la suite, recherchez la section consacrée à twig dans le tableau `adapters` et décommentez les paramètres qui s'y trouvent. Votre fichier ressemblera donc à ceci:

```php
<?php 

return [
    /**
     * Adapteur responsable du rendu des vues
     * 
     * voir le tableau `adapters` ci-dessous
     */
    'active_adapter' => 'twig',

    // ---
    // ---

    'adapters' => [
        // ---

        /**
         * [Configuration Twig](https://twig.symfony.com/doc/3.x/api.html#environment-options)
         * 
         * executer `composer require twig/twig`
         * requiet twig 3.x or higher
         */
        'twig' => [
            /**
             * Extension de fichier de vues
             */
            'extension' => 'twig',

            /**
             * Répertoire où sont stockés les caches de modèles
             * 
             * @var string
             */
            'cache_dir'            => VIEW_CACHE_PATH.'twig',

            /**
             * Afficher les noeuds générés ?
             * 
             * @var bool|'auto' Si auto, le systeme affichera les noeuds en phase de developpement mais pas en production
             */
            'debug'            => 'auto',

            /**
             * Le jeu de caractères utilisé par les templates
             * 
             * @var string
             */
            'charset'          => 'UTF-8',

            /**
             * Recompiler le modèle chaque fois que le code source change
             * 
             * @var bool|'auto' Si auto, la recompilation s'effectuera uniquement en phase de developpement
             */
            'auto_reload'      => true,

            /**
             * Si défini sur `FALSE`, ignorera silencieusement les variables invalides, sinon lèvera une exception à la place
             * 
             * @var bool|'auto' Si auto, sera defini automatiquement sur FALSE en phase de production et TRUE en developpement
             */
            'strict_variables' => 'auto',

            /**
             * Variables globales à passer à Twig
             * 
             * @var array<string, mixed>
             */
            'globals' => [
                /*
                'URL'   => new URL,
                'HTML'  => new HTML,
                'Form'  => new Form,
                'Router' => new Router,
                */
            ],

            /**
             * Filtres
             * 
             * @var \Twig\TwigFilter[]
             */
            'filters'   => [],

            /**
             * Fonctions
             * 
             * * @var \Twig\TwigFunction[]
             */
            'functions' => [],

            /**
             * Fonction de configuration manuelle du moteur de template
             */
            'configure'     => function(\Twig\Environment $engine) : \Twig\Environment {
                
                return $engine;
            }
        ],
        
        // ---
    ]
];
```

Pour finir, vous devez installer le package <a href="https://packagist.org/packages/twig/twig" target="_blank">twig/twig</a> avant de réellement commencer a écrire vos vues twig.

```bash
composer require twig/twig
```

Une fois ceci étant fait, vous pouvez commencer à écrire vos vues twig comme si vous étiez dans Symfony. Au niveau du contrôleur, rien ne change.

<a name="utilisation-de-smarty"></a>
### Utilisation de Smarty

<a href="https://smarty-php.github.io/smarty/stable/" target="_blank">Smarty</a> est un moteur de template pour PHP, facilitant la séparation de la présentation (HTML/CSS) de la logique applicative. Il vous permet d'écrire des modèles, en utilisant des variables, des modificateurs, des fonctions et des commentaires.

```smarty
<ul>
    {foreach $users as $user}
        <li>Hello, {$user.name|escape} </li>
    {/foreach}
</ul>
```

Pour intégrer Smarty à BlitzPHP, commencez par ouvrir votre fichier de configuration `app/Config/View.php` et attribuer la valeur `smarty` à la clé `active_adapter`. Par la suite, recherchez la section consacrée à smarty dans le tableau `adapters` et décommentez les paramètres qui s'y trouvent. Votre fichier ressemblera donc à ceci:

```php
<?php 

return [
    /**
     * Adapteur responsable du rendu des vues
     * 
     * voir le tableau `adapters` ci-dessous
     */
    'active_adapter' => 'smarty',

    // ---
    // ---

    'adapters' => [
        // ---

        /**
         * [Configuration Smarty](https://www.smarty.net/docs/en/api.variables.tpl)
         * 
         * executer `composer require smarty/smarty`
         * requiet smarty 3.x or higher
         */
        'smarty' => [
            /**
             * Extension de fichier de vues
             */
            'extension' => 'tpl',

            /**
             * Répertoire ou répertoires utilisés pour stocker les fichiers de configuration utilisés dans les modèles
             * 
             * @var string|string[]
             */
            'config_dir'     => [CONFIG_PATH],
            
            /**
             * Répertoire où sont stockés les caches de vues
             * 
             * @var string
             */
            'cache_dir'      => VIEW_CACHE_PATH.'smarty'.DIRECTORY_SEPARATOR.'cache',

            /**
             * Répertoire où se trouvent les modèles compilés
             * 
             * @var string|string[]
             */
            'compile_dir'    => VIEW_CACHE_PATH.'smarty'.DIRECTORY_SEPARATOR.'compile',

            /**
             * Mise en cache activée ?
             * 
             * @var bool|int
             */
            'caching'        => \Smarty::CACHING_OFF,

            /**
             * Durée de vie du cache en secondes
             * 
             * @var int
             */
            'cache_lifetime' => 5 * MINUTE,

            /**
             * Mettre à jour les modèles de cache à chaque appel ?
             * 
             * @var bool
             */
            'force_cache'    => true,

            /**
             * Mettre à jour les modèles de compilation à chaque appel ?
             * 
             * @var bool
             */
            'force_compile'  => false,

            /**
             * Échappera à toutes les sorties de variable du template ?
             * 
             * @var bool
             */
            'escape_html'    => on_prod(),

            /**
             * Active la console de débogage ?
             * 
             * @var bool
             */
            'debugging'      => on_dev(),

            /**
             * Vérifier le modèle pour les modifications ?
             * 
             * @var bool
             */
            'compile_check'  => on_dev(),

            /**
             * Fonction de configuration manuelle du moteur de template
             */
            'configure'     => function(\Smarty $engine) : \Smarty {

                return $engine;
            }
        ],
        
        // ---
    ]
];
```

Pour finir, vous devez installer le package <a href="https://packagist.org/packages/smarty/smarty" target="_blank">smarty/smarty</a> avant de réellement commencer a écrire vos vues smarty.

```bash
composer require smarty/smarty
```

Une fois ceci étant fait, vous pouvez commencer à écrire vos vues smarty comme vous le feriez d'habitude. Au niveau du contrôleur, rien ne change.

<a name="utilisation-de-plates-et-latte"></a>
### Utilisation de Plates et Latte

L'utilisation de <a href="https://latte.nette.org/" target="_blank">Latte</a> ou de <a href="http://platesphp.com/" target="_blank">Plates</a> se fait de la même manière que précédemment. Deux choses, modification des paramètres, et installation du package adéquat:

```shell
// Pour l'utilisation de Latte
composer require latte/latte

// Pour l'utilisation de Plates
composer require league/plates
```