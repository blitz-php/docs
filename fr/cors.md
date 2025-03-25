---
title: Cross-Origin Resource Sharing (CORS)
---

<a name="introduction"></a>
## Introduction

Le partage de ressources inter-origines (CORS) est un mécanisme de sécurité basé sur un en-tête HTTP qui permet à un serveur d'indiquer toutes les origines (domaine, schéma ou port) autres que la sienne à partir desquelles un navigateur doit autoriser le chargement des ressources.

CORS fonctionne en ajoutant des en-têtes aux requêtes et réponses HTTP pour indiquer si la ressource demandée peut être partagée entre différentes origines, contribuant ainsi à prévenir les attaques malveillantes telles que la falsification de requêtes intersites (CSRF) et le vol de données.

Si vous n'êtes pas familier avec CORS et les en-têtes CORS, veuillez consulter <a href="https://developer.mozilla.org/fr/docs/Web/HTTP/Guides/CORS#en-têtes_de_réponse_http" target="_blank">la documentation MDN sur CORS</a>. 

<a name="configuration"></a>
## Configuration

Par défaut, votre application n'a pas de fichier de configuration pour Cors. vous devez donc créer un fichier `app/Config/cors.php` et y insérer le code ci-dessous:

```php
<?php

return [
    /**
     * @var list<string>
     */
    'allowed_headers' => ['*'],

    /**
     * @var list<string>
     */
    'allowed_methods' => ['*'],

    /**
     * @var list<string>
     */
    'allowed_origins' => ['*'],

    /**
     * @var list<string>
     */
    'allowed_origins_patterns' => [],

    /**
     * @var list<string>
     */
    'exposed_headers' => [],

    /**
     * @var int
     */
    'max_age' => 7200,

    /**
     * @var bool
     */
    'supports_credentials' => false,
];

```

Au minimum, les éléments suivants doivent être définis :

* `allowed_origins`: Liste explicite des origines que vous souhaitez autoriser.
* `allowed_headers`: Liste explicite des en-têtes HTTP que vous souhaitez autoriser.
* `allowed_methods`: Liste explicite des méthodes HTTP que vous souhaitez autoriser.

> **Attention**  
> Sur la base du principe du moindre privilège, seuls l'origine, les méthodes et les en-têtes minimaux nécessaires doivent être autorisés.

Si vous envoyez des informations d'identification (par exemple, des cookies) avec une  requête d'origine croisée, définissez `supports_credentials` sur `true`.

<a name="activation-de-cors"></a>
## Activation de CORS

Pour activer CORS, vous devez faire deux choses :

1. Spécifier le middleware `cors` aux routes qui autorisent CORS.
2. Ajouter des routes **OPTIONS** pour les requêtes CORS Preflight.

<a name="activation-via-les-routes"></a>
### Activation via les routes

Vous pouvez définir le middleware `cors` pour les routes dans votre fichier `app/Config/routes.php`. Par exemple:

```php
<?php
use BlitzPHP\Facades\Route;
use BlitzPHP\Http\Response;

Route::middleware('cors')->group(static function() {
    Route::resource('product');

    Route::options('product', static function (Response $response) {
        // Implémenter le traitement des requêtes OPTIONS normales non préliminaires, si nécessaire.
        
        return $response->withStatusCode(204)->withHeader('Allow', 'OPTIONS, GET, POST, PUT, PATCH, DELETE');
    });
    
    Route::options('product/(:any)', static function () {});
});
```

N'oubliez pas d'ajouter des routes OPTIONS pour les demandes de contrôle en amont. En effet, **les middlewares du contrôleur ne fonctionnent pas si la route n'existe pas**.

Le middleware CORS traite toutes les demandes de contrôle en amont, de sorte que les contrôleurs sous forme de closure des routes OPTIONS ne sont normalement pas appelés.

<a name="activation-via-le-fichier-de-configuration-des-middlewares"></a>
### Activation via le fichier de configuration des middlewares

Alternativement, si vous voulez activer CORS sur toutes vos routes, vous pouvez ajouter le middleware `cors` dans le tableau `globals` du fichier `app/Config/middlewares.php`.

```php
return [
    // ...
    /**
     * @var array<class-string|Closure|string>
     */
    'globals' => [
        // ...
        'cors',
        // ...
    ],
    // ...
];
```

N'oubliez pas d'ajouter des routes OPTIONS pour les demandes de contrôle en amont. En effet, **les middlewares du contrôleur ne fonctionnent pas si la route n'existe pas**.

```php
<?php
use BlitzPHP\Facades\Route;

Route::options('(:any)', static function () {});
```

Le middleware CORS traite toutes les demandes de contrôle en amont, de sorte que les contrôleurs sous forme de closure des routes OPTIONS ne sont normalement pas appelés.

<a name="verification-des-routes-et-des-middlewares"></a>
## Vérification des routes et des middlewares

Après la configuration, vous pouvez vérifier les routes et les middlewares à l'aide de la commande [klinge route:list](/docs/{version}/routage#listing-des-routes).

```bash
php klinge route:list -v
```