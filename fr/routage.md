---
title: Routage
---

<a name="qu-est-ce-que-le-routage-uri"></a>
## Qu’est-ce que le routage URI ?

Le routage d’URI associe un URI à la méthode d’un contrôleur.

BlitzPHP a deux types de routage. L’un est le routage explicite (définition de routes) et l’autre est le routage automatique. Avec le routage explicite, vous pouvez définir des routes manuellement. Il permet une URL flexible. Le routage automatique achemine automatiquement les requêtes HTTP en fonction des conventions et exécute les méthodes de contrôleur correspondantes. Il n’est pas nécessaire de définir des routes manuellement.

Tout d’abord, regardons le routage explicite. Si vous souhaitez utiliser le routage automatique, consultez [routage automatique](#routage-automatique).

<a name="definition-des-regles-de-routage"></a>
## Définition des règles de routage

Les règles de routage sont définies dans le fichier `app/Config/routes.php`. Dans ce fichier, vous accès à une instance de la classe RouteCollection via la variable `$routes` qui vous permettra de spécifier vos propres critères de routage. Les sorties peuvent être spécifiées à l'aide d'espaces réservés ou d'expressions régulières.

Lorsque vous spécifiez une route, vous choisissez une méthode correspondant aux verbes HTTP (méthode de requête). Si vous attendez une requête GET, vous utilisez la méthode `get()` :

```php
<?php

$routes->get('/', 'HomeController::index');
```

Une route prend le chemin de route (chemin URI relatif à BaseURL. `/`) à gauche et le mappe au gestionnaire de route (contrôleur et méthode `HomeController::index`) à droite, ainsi que tous les paramètres qui doivent être transmis au contrôleur.

Le contrôleur et la méthode doivent être répertoriés de la même manière que vous utiliseriez une méthode statique, en séparant la classe et sa méthode par un double deux-points, comme `UsersController::list`.

Si cette méthode nécessite que des paramètres lui soient transmis, ils seront alors répertoriés après le nom de la méthode, séparés par des barres obliques :

```php
<?php

// Exécute $UsersController->list()
$routes->get('users', 'UsersController::list');

// Exécute $UsersController->list(1, 23)
$routes->get('users/1/23', 'UsersController::list/1/23');
```

<a name="exemples"></a>
### Exemples

Voici quelques exemples de routage de base.

Une URL contenant le mot `blog` dans le premier segment sera mappée à la classe `\App\Controllers\BlogsController` et à la méthode par défaut, qui est généralement `index()` :

```php
<?php 

$routes->get('blog', 'BlogsController');
```

Une URL contenant les segments `blog/joe` sera mappée à la classe `\App\Controllers\BlogsController` et à la méthode `users()`. L'ID sera défini sur `34`

```php
<?php

$routes->get('blog/joe', 'BlogsController::users/34');
```

Une URL avec `product` comme premier segment et tout autre chose dans le second sera mappé à la classe `\App\Controllers\CatalogController` et à la méthode `productLookup()` :

```php
<?php

$routes->get('product/(:segment)', 'CatalogController::productLookup');
```

Une URL avec `product` comme premier segment et un numéro dans le second sera mappée à la classe `\App\Controllers\CatalogController` et à la méthode `productLookupByID()` en transmettant la correspondance en tant que variable à la méthode :

```php
<?php

$routes->get('product/(:num)', 'CatalogController::productLookupByID/$1');
```

<a name="methodes-de-routeur-disponibles"></a>
### Méthodes de routeur disponibles

Le routeur vous permet d’enregistrer des routes qui répondent à n’importe quel verbe HTTP :

```php
<?php 

$routes->get($uri, $handler);
$routes->post($uri, $handler);
$routes->put($uri, $handler);
$routes->patch($uri, $handler);
$routes->delete($uri, $handler);
$routes->options($uri, $handler);
```

Parfois, vous devrez peut-être enregistrer une route qui répond à plusieurs verbes HTTP. Vous pouvez le faire en utilisant la méthode `match()`. Ou, vous pouvez même enregistrer une route qui répond à tous les verbes HTTP en utilisant la méthode `any()`

```php
<?php

$routes->match(['get', 'post'], $uri, $handler);

$routes->any($uri, $handler);
```

> **Note**  
> Lors de la définition de plusieurs routes qui partagent le même URI, les routes utilisant les méthodes `get`, `post`, `put`, `patch`, `delete`, et `options` doivent être définis avant les routes utilisant les méthodes `any`, `match`, et `redirect`. Cela garantit que la requête entrante est mise en correspondance avec la route correcte.

<a name="injection-de-dependance"></a>
### Injection de dépendance

Vous pouvez typer toutes les dépendances requises par votre route dans la signature du callback de votre route. Les dépendances déclarées seront automatiquement résolues et injectées dans le callback par le [conteneur de BlitzPHP](/docs/{version}/conteneur). Par exemple, vous pouvez typer un paramètre de la classe `BlitzPHP\Http\Request` pour que la requête HTTP actuelle soit automatiquement injectée dans votre callback de route :

```php
<?php

use BlitzPHP\Http\Request;

$routes->get('/users', function (Request $request) {
    // ...
});
```

<a name="specification-des-gestionnaires-de-routes"></a>
### Spécification des gestionnaires de routes

<a name="namespace-du-controleur"></a>
#### Namespace du contrôleur

Lorsque vous spécifiez un nom de contrôleur et de méthode sous forme de chaîne, si un contrôleur est écrit sans le backslash `\` en début de chaîne, le namespace par défaut sera ajouté :

```php
<?php

// Route vers \App\Controllers\Api\UsersController::update()
$routes->post('api/users', 'Api\UsersController::update');
```

Si vous mettez `\` au début, il est traité comme un nom de classe complet :

```php
<?php

// Route vers \Acme\Blog\Controllers\HomeController::list()
$routes->get('blog', '\Acme\Blog\Controllers\HomeController::list');
```

Vous pouvez également spécifier le namespace avec l'option `namespace` :

```php
<?php

// Route vers \Admin\UsersController::index()
$routes->get('admin/users', 'UsersController::index', ['namespace' => 'Admin']);
```

Voir [attribution d'un namespace](#attribution-d-un-namespace) pour plus de détails.

<a name="utiliser-un-tableau-comme-gestionnaire"></a>
#### Utiliser un tableau comme gestionnaire

Vous pouvez utiliser la syntaxe de callback par tableau pour spécifier le contrôleur :

```php
$routes->get('/', [\App\Controllers\HomeController::class, 'index']);
```

Ou en utilisant le mot-clé `use` :

```php
use App\Controllers\HomeController;

$routes->get('/', [HomeController::class, 'index']);
```

> **Note**  
> Lorsque vous utilisez la syntaxe de callback par tableau, le nom de classe est toujours interprété comme un nom de classe complet. Le [namespace par défaut](#namespace-par-defaut) et [l'option namespace](#attribution-d-un-namespace) n'ont donc aucun effet.

S'il y a des espaces réservés, il définira automatiquement les paramètres dans l'ordre spécifié :

```php
use App\Controllers\ProductController;

$routes->get('product/(:num)/(:num)', [ProductController::class, 'index']);

// Le code ci-dessus est le même que le suivant :
$routes->get('product/(:num)/(:num)', 'ProductController::index/$1/$2');
```

<a name="utilisation-de-closures"></a>
#### Utilisation de closure

Vous pouvez utiliser une fonction anonyme, ou Closure, comme destination vers laquelle une route est mappée. Cette fonction sera exécutée lorsque l'utilisateur visitera cet URI. C'est pratique pour exécuter rapidement de petites tâches, ou même simplement afficher une vue simple :

```php
<?php

$routes->get('feed', static function () {
    $rss = new RSSFeeder();

    return $rss->feed('general');
});
```

<a name="specification-des-chemins-de-routes"></a>
### Spécification des chemins de routes

<a name="espaces-reserves"></a>
#### Espaces réservés

Une route typique pourrait ressembler à ceci :

```php
<?php

$routes->get('product/(:num)', 'CatalogController::productLookup');
```

Dans une route, le premier paramètre contient l'URI à mettre en correspondance, tandis que le deuxième paramètre contient la destination vers laquelle il doit être acheminé. Dans l'exemple ci-dessus, si le mot littéral « product » est trouvé dans le premier segment du chemin de l'URL et qu'un nombre est trouvé dans le deuxième segment, la classe `CatalogController` et la méthode `productLookup` sont utilisées à la place.

Les espaces réservés sont simplement des chaînes qui représentent un modèle d'expression régulière. Pendant le processus de routage, ces espaces réservés sont remplacés par la valeur de l'expression régulière. Ils sont principalement utilisés pour la lisibilité.

Les espaces réservés suivants sont disponibles pour que vous puissiez les utiliser dans vos routes :

<div class="overflow-auto">

| Espaces réservés | Description                                                                                                                  | Expression régulières                                        |
|------------------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| **(:any)**           | Correspondra à tous les caractères à partir de ce point jusqu'à la fin de l'URI. Cela peut inclure plusieurs segments d'URI. | `.*`                                                           |
| **(:segment)**       | Correspondra à n’importe quel caractère à l’exception d’une barre oblique (`/`) limitant le résultat à un seul segment.        | `[^/]+`                                                        |
| **(:num)**           | Correspondra à n’importe quel entier.                                                                                        | `[0-9]+`                                                       |
| **(:alpha)**         | Correspondra à n'importe quelle chaîne de caractères alphabétiques                                                           | `[a-zA-Z]+`                                                    |
| **(:alphanum)**      | Correspondra à n’importe quelle chaîne de caractères alphabétiques ou d’entiers, ou à toute combinaison des deux.            | `[a-zA-Z0-9]+`                                                 |
| **(:hash)**          | Est identique à `(:segment)`, mais peut être utilisé pour voir facilement quelles routes utilisent des identifiants hachés.    | `[^/]+`                                                        |
| **(:slug)**          | Correspondra à un segment contenant des lettres minuscules, des chiffres et des tirets                                       | `[a-z0-9-]+`                                                   |
| **(:uuid)**          | correspondra à un segment contenant un UUID valide                                                                           | `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` |

</div>

> **Note**  
> `{locale}` ne peut pas être utilisé comme espace réservé ou autre partie de la route, car il est réservé à une utilisation dans l'[internationalisation](/docs/{version}/internationalisation).

Notez qu'un seul `(:any)` correspondra à plusieurs segments de l'URL s'il est présent. Par exemple la route :

```php
<?php

$routes->get('product/(:any)', 'ProductController::productLookup/$1');
```

correspondra à `product/123`, à `product/123/456`, à `product/123/456/789` et ainsi de suite. L'implémentation dans le contrôleur doit prendre en compte les paramètres maximaux :

```php
<?php

namespace App\Controllers;

class ProductController extends BaseController
{
    public function productLookup($seg1 = false, $seg2 = false, $seg3 = false)
    {
        echo $seg1; // Sera 123 dans tous les exemples
        echo $seg2; // faux dans le premier, 456 dans les deuxième et troisième exemples
        echo $seg3; // faux en premier et en deuxième, 789 en troisième
    }
}
```

> **Attention**  
> Ne mettez aucun espace réservé après `(:any)`. Parce que le nombre de paramètres transmis à la méthode du contrôleur peut changer.

Si la correspondance de plusieurs segments n'est pas le comportement prévu, `(:segment)` doit être utilisé lors de la définition des routes. Avec les exemples d'URL ci-dessus :

```php
<?php

$routes->get('product/(:segment)', 'CatalogController::productLookup/$1');
```

ne correspondra qu'à `produit/123` et générera des erreurs 404 pour un autre exemple.

<a name="espaces-reserves-personnalises"></a>
#### Espaces réservés personnalisés

Vous pouvez créer vos propres espaces réservés qui peuvent être utilisés dans vos routes pour personnaliser entièrement l'expérience et la lisibilité.

Vous ajoutez de nouveaux espaces réservés avec la méthode `placeholder()`. Le premier paramètre est la chaîne à utiliser comme espace réservé. Le deuxième paramètre est le modèle d'expression régulière par lequel il doit être remplacé. Ceci doit être appelé avant d'ajouter la route :

```php
<?php

$routes->placeholder('uid', '(?=[a-f\d]{24}$)(\d+[a-f]|[a-f]+\d)');

$routes->get('users/(:uid)', 'UsersController::show/$1');
```

<a name="expressions-regulieres"></a>
#### Expressions régulières

Si vous préférez, vous pouvez utiliser des expressions régulières pour définir vos règles de routage. Toute expression régulière valide est autorisée, tout comme les références arrière.

> **Attention**  
> Si vous utilisez des références arrière, vous devez utiliser la syntaxe dollar plutôt que la syntaxe à double barre oblique inverse. Une route RegEx typique pourrait ressembler à ceci :  

```php
<?php

$routes->get('products/([a-z]+)/(\d+)', 'ProductsController::show/$1/id_$2');
```

Dans l'exemple ci-dessus, un URI similaire à **products/shirts/123** appellerait à la place la méthode `show()` de la classe du contrôleur `ProductsController`, avec le premier et le deuxième segment d'origine qui lui seraient transmis comme arguments.

Avec les expressions régulières, vous pouvez également intercepter un segment contenant une barre oblique (`/`), qui représente généralement le délimiteur entre plusieurs segments.

Par exemple, si un utilisateur accède à une zone protégée par mot de passe de votre application Web et que vous souhaitez pouvoir le rediriger vers la même page après s'être connecté, cet exemple peut vous être utile :

```php
<?php

$routes->get('login/(.+)', 'AuthController::login/$1');
```

Pour ceux d’entre vous qui ne connaissent pas les expressions régulières et souhaitent en savoir plus, [regular-expressions.info](https://www.regular-expressions.info/) pourrait être un bon point de départ.

> **Note**  
> Vous pouvez également mélanger et faire correspondre des espaces réservés avec des expressions régulières.

<a name="routes-de-redirection"></a>
### Routes de redirection

Tout site qui vit assez longtemps aura forcément des pages qui bougent. Vous pouvez spécifier des routes qui doivent rediriger vers d'autres routes avec la méthode `redirect()`. Le premier paramètre est le modèle URI de l’ancienne route. Le deuxième paramètre est soit le nouvel URI vers lequel rediriger, soit le nom d'une route nommée. 

```php
<?php

$routes->get('users/profile', 'UsersController::profile', ['as' => 'profile']);

// Rediriger vers une route nommée
$routes->redirect('users/about', 'profile');
// Redirection vers un URI
$routes->redirect('users/about', 'users/profile');

// Redirection avec espace réservé
$routes->get('post/(:num)/comment/(:num)', 'PostsCommentsController::index', ['as' => 'post.comment']);

// Redirection vers un URI
$routes->redirect('article/(:num)/(:num)', 'post/$1/comment/$2');
// Rediriger vers une route nommée
$routes->redirect('article/(:num)/(:num)', 'post.comment');
```

Par défaut, la méthode `redirect()` renvoie un code d'état `302` qui est une redirection temporaire et est recommandée dans la plupart des cas. Vous pouvez personnaliser le code d'état à l'aide du troisième paramètre facultatif :

```php
<?php

$routes->redirect('/here', '/there', 301);
```

Vous pouvez également utiliser la méthode `permanentRedirect()` pour renvoyer un code d'état `301` :

```php
<?php

$routes->permanentRedirect('/here', '/there');
```

> **Note**  
> Si une route de redirection correspond lors du chargement d'une page, l'utilisateur sera immédiatement redirigé vers la nouvelle page avant qu'un contrôleur puisse être chargé.

<a name="routes-de-vues"></a>
### Routes de vues

Si vous souhaitez simplement afficher une [vue](/docs/{version}/vues) sans logique associée, vous pouvez utiliser la méthode `view()`. Ceci est toujours traité comme une requête GET. La méthode `view` accepte un URI comme premier argument et un nom de vue comme deuxième argument.

```php
<?php

// Affiche la vue /app/Views/pages/about.php
$routes->view('about', 'pages/about');
```

De plus, vous pouvez fournir un tableau de données à transmettre à la vue comme troisième argument facultatif.

```php
<?php

// Affiche la vue /app/Views/pages/about.php
$routes->view('about', 'pages/about', ['author' => 'Dimitri']);
```

Si vous utilisez des espaces réservés dans votre route, vous pouvez y accéder dans la vue dans la variable spéciale `$segments`. Ils sont disponibles sous forme de tableau, indexés dans l'ordre dans lequel ils apparaissent dans la route.

```php
<?php

// Affiche la vue /app/Views/map.php
$routes->view('map/(:segment)/(:segment)', 'map');

// Dans la vue, vous pouvez accéder aux segments avec 
// $segments[0] et $segments[1] respectivement.
```

<a name="restrictions-d-environnements"></a>
### Restrictions d'environnements

Vous pouvez créer un ensemble de routes qui ne seront visibles que dans un certain environnement. Cela vous permet de créer des outils que seul le développeur peut utiliser sur ses machines locales qui ne sont pas accessibles sur les serveurs de test ou de production. Cela peut être fait avec la méthode `environment()`. Le premier paramètre est le nom de l'environnement. Tous les routes définies dans cette closure ne sont accessibles qu'à partir de l'environnement donné:

```php
<?php

$routes->environment('development', static function ($routes) {
    $routes->get('builder', 'Tools\BuilderController::index');
});
```


<a name="routes-avec-n-importe-quel-verbe-http"></a>
### Routes avec n’importe quel verbe HTTP

> **Attention**  
> Bien que la méthode `add()` semble pratique, il est recommandé de toujours utiliser les verbes HTTP, décrits ci-dessus, car ils sont plus sûrs. Si vous utilisez la protection CSRF, elle ne protège pas les requêtes GET. Si l’URI spécifié dans la méthode `add()` est accessible par la méthode GET, la protection CSRF ne fonctionnera pas.

Il est possible de définir une route avec n’importe quel verbe HTTP. Vous pouvez utiliser la méthode `add()`

```php
<?php

$routes->add('products', 'Product::feature');
```

> **Note**  
> L’utilisation des routes basées sur des verbes HTTP entraînera également une légère augmentation des performances, car seules les routes qui correspondent à la méthode de requête actuelle sont stockés, ce qui réduit le nombre de routes à analyser lorsque vous essayez de trouver une correspondance.

<a name="mapper-de-plusieurs-routes"></a>
### Mapper de plusieurs routes

> **Attention**  
> Tout comme la méthode `add()`, la méthode `map()` n’est pas recommandée parce qu’elle appelle la méthode `add()` en interne.

Bien que la méthode `add()` soit simple à utiliser, il est souvent plus pratique de travailler avec plusieurs routes à la fois, en utilisant la méthode `map()`. Au lieu d’appeler la méthode `add()` pour chaque route que vous devez ajouter, vous pouvez définir un tableau de routes, puis passez-le comme premier paramètre à la méthode `map()`.

```php
<?php

$multipleRoutes = [
    'product/(:num)'      => 'CatalogController::productLookupById',
    'product/(:alphanum)' => 'CatalogController::productLookupByName',
];

$routes->map($multipleRoutes);
```


<a name="options-globales"></a>
## Options globales

Toutes les méthodes de création d’une route (`get()`, `post()`, `resource()` etc.) peuvent prendre un éventail d’options qui peut modifier les routes générées ou les restreindre davantage. Le tableau `$options` est toujours le dernier paramètre.

```php
<?php

$routes->add('from', 'to', $options);
$routes->get('from', 'to', $options);
$routes->post('from', 'to', $options);
$routes->put('from', 'to', $options);
$routes->head('from', 'to', $options);
$routes->options('from', 'to', $options);
$routes->delete('from', 'to', $options);
$routes->patch('from', 'to', $options);
$routes->match(['get', 'put'], 'from', 'to', $options);
$routes->resource('photos', $options);
$routes->presenter('photos', $options);
$routes->map($array, $options);
$routes->group('name', $options, static function () {});
```

<a name="application-des-middlewares"></a>
### Application des middlewares

Vous pouvez modifier le comportement d’une route spécifique en fournissant des [middlewares](/docs/{version}/middleware) à exécuter avant ou après le contrôleur. Ceci est particulièrement pratique lors de [l’authentification](/docs/{version}/schild) ou de [la journalisation](/docs/{version}/journalisation) des API. La valeur du middleware peut être une chaîne ou un tableau de chaînes :

* Correspondant aux alias définis dans `app/Config/middlewares.php`.
* Le nom d'une classe de middleware

Voir [le chapitre sur les middlewares](/docs/{version}/middleware) pour plus d’informations sur la configuration.

> **Attention**  
> Si vous définissez des middlewares sur les routes dans `app/Config/routes.php` (pas dans `app/Config/middlewares.php`), il est recommandé de désactiver [le routage automatique](#routage-automatique). Lorsque le routage automatique est activé, il est possible qu’un contrôleur soit accessible via une URL différente de lq route configurée, auquel cas le middleware que vous avez spécifié à la route ne sera pas appliqué. Voir [utiliser des routes définies uniquement](#utiliser-des-routes-definies-uniquement) pour désactiver le routage automatique.

<a name="alias-de-middleware"></a>
#### Alias de middleware

Vous spécifiez un alias défini dans `app/Config/middlewares.php` pour la valeur du middleware :

```php
<?php

$routes->get('admin', ' AdminController::index', ['middleware' => 'admin-auth']);
```

Vous pouvez également fournir des arguments à transmettre aux middlewares :

```php
<?php

$routes->get('admin', ' AdminController::index', ['middleware' => 'role:admin,moderator']);
```

<a name="nom-de-classe-de-middleware"></a>
#### Nom de classe de middleware

Vous spécifiez un nom de classe de middleware pour la valeur de middleware

```php
<?php

$routes->get('admin', ' AdminController::index', ['middleware' => \App\Middlewares\SomeMiddleware::class]);
```

<a name="tableau-de-middleware"></a>
#### Tableau de middleware

Vous spécifiez un tableau pour la valeur de middleware :

```php
<?php

$routes->get('admin', ' AdminController::index', ['middleware' => ['admin-auth', \App\Middlewares\SomeMiddleware::class]]);
```

<a name="affectation-d-un-namespace"></a>
### Affectation d’un namespace

Bien qu’un [namespace par défaut](#namespace-par-defaut) soit ajouté aux contrôleurs générés, vous pouvez également spécifier un namespace différent à utiliser dans n’importe quel tableau d’options, avec l’option `namespace`. La valeur doit être le namespace que vous souhaitez modifier :

```php
<?php

// Route vers \Admin\UsersController::index()
$routes->get('admin/users', 'UsersController::index', ['namespace' => 'Admin']);
```

Le nouveau namespace n'est appliqué que lors de cet appel pour toutes les méthodes qui créent une seule route, comme get, post, etc. Pour toutes les méthodes qui créent plusieurs routes, le nouveau namespace est attaché à toutes les routes générées par cette fonction ou, dans le cas de `group()`, à toutes les routes générées dans la closure.

<a name="limite-au-nom-d-hote"></a>
### Limite au nom d’hôte

Vous pouvez restreindre les groupes de routes à fonctionner uniquement dans certains domaines ou sous-domaines de votre application en transmettant l'option `hostname` avec le domaine souhaité pour l'autoriser au tableau d'options :

```php
<?php

$routes->get('from', 'to', ['hostname' => 'accounts.example.com']);
```

Cet exemple permettrait aux hôtes spécifiés de fonctionner uniquement si le domaine correspondait exactement à **accounts.exemple.com**. Cela ne fonctionnerait pas sur le site principal **exemple.com**.

<a name="limite-aux-sous-domaines"></a>
### Limite aux sous domaines

Lorsque l'option de `subdomain` est présente, le système limitera les routes pour qu'elles soient uniquement disponibles sur ce sous-domaine. La route ne sera mise en correspondance que si le sous-domaine est celui via lequel l'application est consultée.

```php
<?php

// Limité à media.example.com
$routes->get('from', 'to', ['subdomain' => 'media']);
```

Vous pouvez le restreindre à n'importe quel sous-domaine en définissant la valeur sur un astérisque (`*`). Si vous consultez à partir d'une URL qui ne contient aucun sous-domaine, ceci ne sera pas mis en correspondance :

```php
<?php

// Limité à n'importe quel sous domaine
$routes->get('from', 'to', ['subdomain' => '*']);
```

> **Attention**  
> Le système n'est pas parfait et doit être testé pour votre domaine spécifique avant d'être utilisé en production. La plupart des domaines devraient fonctionner correctement, mais certains cas extrêmes, en particulier avec un point dans le domaine lui-même (non utilisé pour séparer les suffixes ou www), peuvent potentiellement conduire à des faux positifs.

<a name="compensation-des-parametres-correspondants"></a>
### Compensation des paramètres correspondants

Vous pouvez décaler les paramètres correspondants dans votre route de n'importe quelle valeur numérique avec l'option `offset`, la valeur étant le nombre de segments à décaler.

Cela peut être utile lors du développement d'API, le premier segment URI étant le numéro de version. Il peut également être utilisé lorsque le premier paramètre est une chaîne de langue :

```php
<?php

$routes->get('users/(:num)', 'users/show/$1', ['offset' => 1]);

// Créé:
$routes['users/(:num)'] = 'users/show/$2';
```

<a name="routage-inverse"></a>
## Routage inversé

Le routage inversé vous permet de définir le contrôleur et la méthode, ainsi que tous les paramètres vers lesquels un lien doit aller, et de demander au routeur de rechercher la route menant vers celui-ci. Cela permet aux définitions de routes de changer sans que vous ayez à mettre à jour le code de votre application. Ceci est généralement utilisé dans les vues pour créer des liens.

Par exemple, si vous souhaitez créer un lien vers une galerie de photos vers une route, vous pouvez utiliser la fonction d'assistance `link_to()` pour obtenir la route à utiliser. Le premier paramètre est le nom complet du contrôleur (FQCN) et la méthode, séparés par un double deux-points (`::`), un peu comme vous l'utiliseriez lors de l'écriture de la route initiale elle-même. Tous les paramètres qui doivent être transmis à la route le sont ensuite :

```php
<?php

// La route est définie comme :
$routes->get('users/(:num)/gallery/(:num)', 'GalleriesController::showUserGallery/$1/$2');

?>

<!-- Générez l'URI pour créer un lien vers l'ID utilisateur 15, galerie 12: -->
<a href="<?= link_to('GalleriesController::showUserGallery', 15, 12) ?>">Voir la gallerie</a>
<!-- Résultat: 'http://example.com/users/15/gallery/12' -->
```

<a name="routes-nommees"></a>
## Routes nommées

Vous pouvez nommer les routes pour rendre votre application moins fragile. Cela applique un nom à une route qui peut être appelée plus tard, et même si la définition de la route change, tous les liens de votre application construits avec `link_to()` fonctionneront toujours sans que vous ayez à apporter de modifications. Une route est nommée en passant l'option `as` avec le nom qu'on souhaite lui attribué :

```php
<?php

// La route est définie comme :
$routes->get('users/(:num)/gallery/(:num)', 'GalleriesController::showUserGallery/$1/$2', ['as' => 'user_gallery']);

?>

<!-- Générez l'URI pour créer un lien vers l'ID utilisateur 15, galerie 12: -->
<a href="<?= link_to('user_gallery', 15, 12) ?>">Voir la gallerie</a>
<!-- Résultat: 'http://example.com/users/15/gallery/12' -->
```

Cela présente également l’avantage supplémentaire de rendre les vues plus lisibles.

<a name="regroupement-de-routes"></a>
## Regroupement de routes

Vous pouvez regrouper vos routes sous un nom commun avec la méthode `group()`. Le nom du groupe devient un segment qui apparaît avant les routes définies à l'intérieur du groupe. Cela vous permet de réduire la saisie nécessaire pour créer un ensemble complet de routes partageant toutes un préfixe commun, comme lors de la création d'une zone d'administration :

```php
<?php

$routes->group('admin', static function ($routes) {
    $routes->get('users', 'AdminController\Users::index');
    $routes->get('blog', 'AdminController\Blog::index');
});
```

Cela préfixerait les URI `users` et des `blog` avec `admin`, gérant ainsi les URL telles que `admin/users` et `admin/blog`.

<a name="definition-du-namespace"></a>
### Définition du namespace

Si vous devez attribuer des options à un groupe, comme [affecter un namespace](#affectation-d-un-namespace), vous pouvez le faire avant le callback :

```php
<?php

$routes->group('api', ['namespace' => 'App\API\v1'], static function ($routes) {
    $routes->resource('users');
});
```

Cela gérerait une route de ressources vers le contrôleur `App\API\v1\UsersController` avec l'URI `api/users`.

<a name="definition-des-middlewares"></a>
### Définition des middlewares

Vous pouvez également utiliser un [middeware](/docs/{version}/middleware) spécifique pour un groupe de routes. Cela exécutera toujours le middleware avant ou après le contrôleur. Ceci est particulièrement pratique lors de l'authentification ou de la journalisation de l'API :

```php
<?php

$routes->group('api', ['middleware' => 'api-auth'], static function ($routes) {
    $routes->resource('users');
});
```

<a name="definition-d-autres-options"></a>
### Définition d'autres options

À un moment donné, vous souhaiterez peut-être regrouper les routes dans le but d'appliquer des middlewares ou d'autres options de configuration de route comme le namespace, le sous-domaine, etc. Sans nécessairement avoir besoin d'ajouter un préfixe au groupe, vous pouvez transmettre une chaîne vide à la place du préfixe. et les routes du groupe seront acheminées comme si le groupe n'avait jamais existé mais avec les options de configuration de route données :

```php
<?php

$routes->group('', ['namespace' => 'Schild\Controllers'], static function ($routes) {
    $routes->get('login', 'AuthController::login', ['as' => 'login']);
    $routes->post('login', 'AuthController::attemptLogin');
    $routes->get('logout', 'AuthController::logout');
});
```

<a name="groupes-imbriqués"></a>
### Groupes imbriqués

Il est possible d'imbriquer des groupes au sein de groupes pour une organisation plus fine si vous en avez besoin :

```php
<?php

$routes->group('admin', static function ($routes) {
    $routes->group('users', static function ($routes) {
        $routes->get('list', 'AdminController\Users::list');
    });
});
```

Cela gérerait l'URL vers `admin/users/list`.

<a name="priorite-de-route"></a>
## Priorité de route

Les routes sont enregistrées dans la table de routage dans l'ordre dans lequel elles sont définies. Cela signifie que lorsqu'on accède à un URI, la première route correspondante est exécutée.

> **Attention**  
> Si une route est définie plusieurs fois avec des gestionnaires différents, seul la première route définie est enregistrée.

Vous pouvez vérifier les routes enregistrées dans la table de routage en exécutant la commande [klinge route:list](#listing-des-routes).

<a name="modification-de-la-priorite-de-la-route"></a>
### Modification de la priorité de la route

Lorsque l'on travaille avec des modules, il peut y avoir un problème si les routes de l'application contiennent des caractères génériques. Dans ce cas, les routes du module ne seront pas traités correctement. Vous pouvez résoudre ce problème en réduisant la priorité du traitement des routes à l'aide de l'option `priority`. Ce paramètre accepte des nombres entiers positifs et zéro. Plus le nombre spécifié dans l'option `priority` est élevé, plus la priorité de la route dans la file d'attente de traitement est faible :

```php
<?php

// Il faut d'abord activer le traitement de la file d'attente des routes par priorité.
$routes->setPrioritize();

// Configurer les routes
$routes->get('(.*)', 'PostsController::index', ['priority' => 1]);

// Modules\Acme\Config\routes
$routes->get('admin', 'AdminController::index');

// La route "admin" sera désormais traitée avant la route générique.
```

Pour désactiver cette fonctionnalité, vous devez appeler la méthode avec le paramètre `false` :

```php
$routes->setPrioritize(false);
```

> **Note**  
> Par défaut, toutes les routes ont une priorité de 0. Les nombres entiers négatifs seront convertis en valeur absolue.

<a name="options-de-configuration-des-routes"></a>
## Options de configuration des routes

La classe RoutesCollection fournit plusieurs options qui affectent toutes les routes et peuvent être modifiées pour répondre aux besoins de votre application. Ces options sont disponibles dans le fichier `app/Config/routing.php`.

<a name="namespace-par-defaut"></a>
### Namespace par défaut

Lors de la mise en correspondance d'un contrôleur avec une route, le routeur ajoutera la valeur du namespace par défaut à l'avant du contrôleur spécifié par la route. Par défaut, cette valeur est `App\Controllers`.

Si vous définissez la valeur chaîne vide (`''`), chaque route est laissée pour spécifier le contrôleur entièrement (avec son namespace) :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'default_namespace' => '',
    // ...
];

// Le contrôleur sera \UsersController
$routes->get('users', 'UsersController::index');

// Le contrôleur sera \Admin\UsersController
$routes->get('users', 'Admin\UsersController::index');
```

Si vos contrôleurs ne sont pas explicitement dotés d'un namespace, il n'est pas nécessaire de modifier cela, vous pouvez donc laisser la chaîne vide. Si vous nommez vos contrôleurs, vous pouvez modifier cette valeur pour économiser la saisie :

```php
<?php

// Cela peut être remplacé dans le fichier routes.php
$routes->setDefaultNamespace('App');

// Le contrôleur sera \App\Users
$routes->get('users', 'UsersController::index');

// Le contrôleur sera \App\Admin\Users
$routes->get('users', 'Admin\UsersController::index');
```

<a name="traduire-les-tirets-uri"></a>
### Traduire les tirets URI

Cette option vous permet de remplacer automatiquement les tirets (`-`) par des traits de soulignement dans les segments URI du contrôleur et de la méthode lorsqu'ils sont utilisés dans le routage automatique, vous permettant ainsi d'économiser des entrées de route supplémentaires si vous devez le faire. Ceci est nécessaire car le tiret n'est pas un caractère de nom de classe ou de méthode valide et provoquerait une erreur fatale si vous essayez de l'utiliser :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'translate_uri_dashes' => true,
    // ...
];

// Cela peut être remplacé dans le fichier routes.php
$routes->setTranslateURIDashes(true);
```

> **Note**  
> Lors de l'utilisation du routage automatique, si `translate_uri_dashes` est activé, deux URI peuvent correspondre à une seule méthode de contrôleur, un URI pour les tirets (par exemple, `foo-bar`) et un URI pour les traits de soulignement (par exemple, `foo_bar`). C'est un comportement incorrect c'est pourquoi avec BlitzPHP, l'URI des traits de soulignement (`foo_bar`) ne sera pas accessible.

<a name="utiliser-uniquement-les-routes-definies"></a>
### Utiliser uniquement les routes définies

Lorsqu'aucune route définie correspondant à l'URI n'est trouvée, le système tentera de faire correspondre cet URI aux contrôleurs et aux méthodes lorsque le routage automatique est activé.

Vous pouvez désactiver cette correspondance automatique et restreindre les routes à celles que vous avez définies en définissant la clé `auto_route` sur `false` :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'auto_route' => false,
    // ...
];

// Cela peut être remplacé dans le fichier routes.php
$routes->setAutoRoute(false);
```

> **Attention**  
> Si vous utilisez la protection CSRF, elle ne protège pas les requêtes GET. Si l'URI est accessible par la méthode GET, la protection CSRF ne fonctionnera pas.

<a name="routes-de-secours"></a>
### Routes de secours

Lorsqu'aucune page correspondant à l'URI actuel n'est trouvée, le système affichera une vue 404 générique. Vous pouvez modifier ce qui se passe en spécifiant une action à effectuer avec la méthode `fallback()`. La valeur peut être soit une paire classe/méthode valide, comme vous le feriez pour n'importe quelle route, soit une closure :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'fallback' => 'App\Errors::show404',
    // ...
];

// Cela peut être remplacé dans le fichier routes.php

// Exécuterait la méthode show404 de la classe App\Errors
$routes->fallback('App\Errors::show404');

// Affichera une vue personnalisée
$routes->fallback(static function () {
    echo view('my_errors/not_found.html');
});
```

> **Note**  
> La méthode `fallback()` ne modifie pas le code d'état de réponse en `404`. Si vous ne définissez pas le code d'état dans le contrôleur que vous avez défini, le code d'état par défaut `200` sera renvoyé. Voir [BlitzPHP\Http\Response::withStatusCode()](/docs/{version}/reponses#modification-du-code-d-etat) pour plus d'informations sur la façon de définir le code d'état.

<a name="la-facade-route"></a>
## La façade Route

La définitions des options de route peut devenir très fastidieuse si on a plusieurs options à configurer. Prenons l'exemple ci-dessous

```php
<?php 

$routes->group('admin', [
    'middleware' => ['session', 'verified', 'role:admin'],
    'controller' => '\App\Controllers\AdminController',
    'subdomain' => 'app'
    ], static function($routes) {
        $routes->get('to', 'from');
        // ...
});
```

On constate à travers cet exemple que la définition de la route est un peu touffu du fait du nombre important d'options à définir. Si on a plusieurs routes à définir de la sorte, le fichier de route pourra devenir vite illisible.

BlitzPHP introduit [la façade](/docs/{version}/facades) `Route` pour vous permettre de définir les options de routes plus aisément. La syntaxe de cette approche est plus conviviale et est inspirée du système de <a href="https://laravel.com/docs/10.x/routing" target="_blank">routage de Laravel</a>

En utilisant la façade Route, l'exemple ci dessus devient le suivant:

```php
<?php 

use BlitzPHP\Facades\Route;
use BlitzPHP\Router\RouteBuilder;

Route::prefix('admin')->middleware(['session', 'verified', 'role:admin'])->controller('\App\Controllers\AdminController')->subdomain('app')->group(function(RouteBuilder $route) {
    $route->get('to', 'from');
    // ...
});
```

<a name="facade-definition-des-routes"></a>
### Définition des routes

Les routes sont définies normalement comme décrit précédemment. Sauf que, au lieu d'utiliser la variable `$routes` (instance de la classe `RouteCollection`), on utilisera la façade `Route`:

```php
<?php 
use BlitzPHP\Facades\Route;

Route::add('from', 'to', $options);
Route::get('from', 'to', $options);
Route::post('from', 'to', $options);
Route::put('from', 'to', $options);
Route::head('from', 'to', $options);
Route::options('from', 'to', $options);
Route::delete('from', 'to', $options);
Route::patch('from', 'to', $options);
Route::match(['get', 'put'], 'from', 'to', $options);
Route::form('from', 'to', $options);
Route::view('from', $view, $options);
Route::resource('photos', $options);
Route::presenter('photos', $options);
Route::map($array, $options);
```

<a name="facade-definition-des-options"></a>
### Définition des options

Chaque options de route évoquée ci-dessus possède une méthode permettant de la définir plus facilement:


```php
<?php 
use BlitzPHP\Facades\Route;

Route::name('string'); // Nom de la route
Route::controller('string'); // Contrôleur à utiliser
Route::namespace('string'); // Namespace par défaut
Route::hostname('string'); // Limitation au nom d'hôte
Route::subdomain('string'); // Limitation au sous domaine
Route::prefix('string'); // Prefixe des routes
Route::middleware('string|array'); // Middlewares à exécuter
Route::priority('int'); // Priorité de la route
```

> **Attention**  
> Toutes les options de routes doivent être définies avec l'appel d'une méthode de définition de route (`get`, `post`, `put`, etc)

<a name="facade-regroupement-des-routes"></a>
### Regroupement des routes

Le regroupement de route se fait avec la méthode `group()`. Contrairement au [regroupement classique](#regroupement-de-routes), le regroupement via la façade Route n'a qu'un seul argument: **le callback de définition des routes**. 

Si vous avez des options à définir pour un groupe, ils doivent être définir avant l'appel de la méthode `group()`:

```php
<?php 
use BlitzPHP\Facades\Route;
use BlitzPHP\Router\RouteBuilder;

Route::namespace('App\API\v1')->prefix('api')->group(static function (RouteBuilder $route) {
    $route->resource('users');
});
```

> **Note**  
> Contrairement au regroupement classique, le paramètre de la fonction de callback de la méthode `group()` n'est pas une instance de la classe `RouteCollection` mais une instance de la classe `RouteBuilder`. Vous pouvez donc réutiliser toutes les méthode de la façade.

<a name="routage-automatique"></a>
## Routage automatique

Lorsqu'aucune route définie correspondant à l'URI n'est trouvée, le système tentera de faire correspondre cet URI aux contrôleurs et aux méthodes lorsque le routage automatique est activé.

> **Attention**  
> Pour des raisons de sécurité, si un contrôleur est utilisé dans les routes définies, le routage automatique n'effectue pas de route vers le contrôleur.

Le routage automatique peut acheminer automatiquement les requêtes HTTP en fonction de conventions et exécuter les méthodes de contrôleur correspondantes.

> **Note**  
> Le routage automatique est désactivé par défaut. Pour l'utiliser, voir ci-dessous.

<a name="activer-le-routage-automatique"></a>
### Activer le routage automatique

Pour l'utiliser, vous devez modifier l'option de paramètre `auto_route` sur true dans le fichier `app/Config/routing.php` : 

```php
return [
    'auto_route' => true,
    // ...
];
```

<a name="segments-d-uri"></a>
### Segments d'URI

Les segments de l'URL, conformément à l'approche Modèle-Vue-Contrôleur, représentent généralement : 

```
example.com/classe/méthode/paramètres
```

1. Le premier segment représente la **classe** de contrôleur qui doit être invoquée.
2. Le deuxième segment représente la **méthode** de classe qui doit être appelée.
3. Le troisième segment, ainsi que tous les segments supplémentaires, représentent toutes les **variables** qui seront transmises au contrôleur.

Considérons cet URI :

```
example.com/helloworld/hello/1
```

Dans l'exemple ci-dessus, lorsque vous envoyez une requête HTTP avec la méthode GET, le routage automatique tente de trouver un contrôleur nommé `App\Controllers\HelloworldController` et exécute la méthode `getHello()` en passant `« 1 »` comme premier argument.

> **Note**  
> Une méthode de contrôleur qui sera exécutée par le routage automatique nécessite un préfixe de verbe HTTP (`get`, `post`, `put`, etc.) comme `getIndex()`, `postCreate()`.

<a name="options-de-configuration"></a>
### Options de configuration

Ces options sont disponibles au début du fichier `app/Config/routing.php`.

<a name="controleur-par-defaut"></a>
#### Contrôleur par défaut

<a name="pour-l-uri-racine-du-site"></a>
##### Pour l'URI racine du site

Lorsqu'un utilisateur visite la racine de votre site (c'est-à-dire exemple.com), le contrôleur à utiliser est déterminé par la valeur définie par la clé `default_controller`, à moins qu'une route n'existe explicitement pour lui.

La valeur par défaut est `HomeController` qui correspond au contrôleur dans `app/Controllers/HomeController.php` :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'default_controller' => 'HomeController',
    // ...
];
```

<a name="pour-l-uri-du-repertoire"></a>
##### Pour l'URI du répertoire

Le contrôleur par défaut est également utilisé lorsqu'aucune route correspondante n'a été trouvée et que l'URI pointe vers un répertoire dans le répertoire des contrôleurs. Par exemple, si l'utilisateur visite **example.com/admin**, si un contrôleur a été trouvé dans **app/Controllers/Admin/HomeController.php**, il sera utilisé.

> **Attention**  
> Vous ne pouvez pas accéder au contrôleur par défaut avec l'URI du nom du contrôleur. Lorsque le contrôleur par défaut est `HomeController`, vous pouvez accéder à **example.com/**, mais si vous accédez à **example.com/home**, il ne sera pas trouvé.

<a name="methode-par-defaut"></a>
#### Méthode par défaut

Cela fonctionne de manière similaire au paramètre de contrôleur par défaut, mais est utilisé pour déterminer la méthode par défaut utilisée lorsqu'un contrôleur correspondant à l'URI est trouvé, mais qu'aucun segment n'existe pour la méthode. La valeur par défaut est `index`.

Dans cet exemple, si l'utilisateur visitait **example.com/products** et qu'un contrôleur `ProductsController` existait, la méthode `ProductsController::listAll()` serait exécutée :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'default_method' => 'listAll',
    // ...
];
```

> **Attention**  
> Vous ne pouvez pas accéder au contrôleur avec l'URI du nom de méthode par défaut. Dans l'exemple ci-dessus, vous pouvez accéder à **example.com/products**, mais si vous accédez à **example.com/products/listall**, il ne sera pas trouvé.

<a name="routage-des-modules"></a>
### Routage des modules

Vous pouvez utiliser le routage automatique même si vous utilisez des [modules de code](/docs/{version}/programmation-modulaire) et placez les contrôleurs dans un namespace différent.

Pour router vers un module, la clé `module_routes` dans `app/Config/routing.php` doit être définie :

```php
<?php

// Dans le fichier app/Config/routing.php
return [
    'module_routes' => [
        'blog' => 'Acme\Blog\Controllers',
    ],
    // ...
];
```

La clé est le premier segment URI du module et la valeur est le namespace du contrôleur. Dans la configuration ci-dessus, **http://localhost:8080/blog/foo/bar** sera acheminé vers `Acme\Blog\Controllers\Foo::getBar()`.

> **Note**
> Si vous définissez `module_routes`, le routage du module est prioritaire. Dans l'exemple ci-dessus, même si vous disposez du contrôleur `App\Controllers\BlogController`, **http://localhost:8080/blog** sera acheminé vers le contrôleur par défaut `Acme\Blog\Controllers\HomeController`.

<a name="Confirmation des routes"></a>
## Confirmation des routes

BlitzPHP met à votre disposition une [commande](/docs/version/klinge) pour afficher toutes les routes.

<a name="listing-des-routes"></a>
### Listing des routes

Affiche toutes les routes et middlewares :

```bash
php klinge route:list
```

Le résultat ressemble à ce qui suit :

```
| Méthode | Route | Nom | Gestionnaire                           | Middlewares |
|---------|-------|-----|----------------------------------------|-------------|
| GET     | /     | »   | \App\Controllers\HomeController::index | toolbar     |
| GET     | feed  | »   | (Closure)                              | toolbar     |
```

- La colonne *Méthode* affiche la méthode HTTP que la route écoute. 
- La colonne *Route* affiche le chemin de la route à correspondre. Le chemin d'une route définie est exprimé sous forme d'expression régulière.
- La colonne *Nom* affiche le nom de la route. `»` indique que le nom est le même que le chemin de la route (généralement quand aucun nom n'a explicitement été défini).

> **Attention**  
> Le système n'est pas parfait. Si vous utilisez des [espaces réservés personnalisés](#espaces-reserves-personnalises), les middlewares peuvent ne pas être corrects. Si vous souhaitez vérifier les middlewares pour une route, vous pouvez utiliser la commande [klinge middleware:check](/docs/{version}/middleware#klinge-middleware-check).

<a name="autorouting"></a>
### Autorouting

Lorsque vous utilisez le routage automatique, le résultat ressemble à ce qui suit :
```
| Méthode   | Route                   | Nom | Gestionnaire                                | Middlewares |
|-----------|-------------------------|-----|---------------------------------------------|-------------|
| GET(auto) | product/list/../..[/..] |     | \App\Controllers\ProductController::getList | toolbar     |

```

- La méthode sera comme `GET(auto)`.
- `/..` dans la colonne Route indique un segment. `[/..]` indique qu'il est facultatif.

> **Note**  
> Lorsque le routage automatique est activé et que vous disposez de la route `home`, il est également accessible par `Home`, ou peut-être par `hOme`, `home`, `HOME`, etc., mais la commande n'affichera que `home`.

Si vous voyez une route commençant par `x` comme ci-dessous, cela indique une route non valide qui ne sera pas acheminée, mais le contrôleur dispose d'une méthode publique d'acheminement.

```
| Méthode   | Route      | Nom | Gestionnaire                            | Middlewares |
|-----------|------------|-----|-----------------------------------------|-------------|
| GET(auto) | x home/foo |     | \App\Controllers\HomeController::getFoo | <unknown>   |

```

L'exemple ci-dessus montre que vous disposez de la méthode `\App\Controllers\HomeController::getFoo()`, mais elle n'est pas acheminée car il s'agit du contrôleur par défaut (`HomeController` par défaut) et le nom du contrôleur par défaut doit être omis dans l'URI. Vous devez supprimer la méthode `getFoo()`.

<a name="trier-par-gestionnaire"></a>
### Trier par gestionnaire

Vous pouvez trier les routes par gestionnaire :

```bash
php klinge route:list -h
```

<a name="specifier-l-hote"></a>
### Spécifier l'hôte

Vous pouvez spécifier l'hôte dans l'URL de la requête avec l'option `--host` :

```bash
php klinge route:list --host=accounts.example.com
```