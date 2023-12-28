---
title: Génération d'URLs
---

<a name="introduction"></a>
## Introduction

BlitzPHP fournit plusieurs helpers pour vous aider à générer des URL pour votre application. Ces helpers sont principalement utiles pour créer des liens dans vos vues et réponses API, ou pour générer des réponses de redirection vers une autre partie de votre application.

<a name="les-bases"></a>
## Les bases

<a name="generer-des-url"></a>
### Générer des URL

Le helper `url` peut être utilisée pour générer des URL arbitraires pour votre application. L'URL générée utilisera automatiquement le schéma (HTTP ou HTTPS) et l'hôte de la requête en cours gérée par l'application :

```php
$post = App\Entities\Post::find(1);
 
echo url("/posts/{$post->id}");
 
// http://example.com/posts/1
```

<a name="acces-a-l-url-actuel"></a>
### Accès à l'URL actuel

Si aucun chemin n'est fourni à la fonction `url()`, une instance de la classe `BlitzPHP\Http\UrlGenerator` est renvoyée, ce qui vous permet d'accéder à des informations sur l'URL actuelle :

```php
// Obtenir l'URL actuelle sans la chaîne de requête...
echo url()->current();
 
// Obtenir l'URL actuelle avec la chaîne de requête...
echo url()->full();
 
// Obtenir l'URL complète de la requête précédente...
echo url()->previous();
```

Chacune de ces méthodes est également accessible via [la façade](/docs/{version}/facades) `Url` :

```php
use BlitzPHP\Facades\Url;
 
echo Url::current();
```

<a name="urls-pour-les-routes-nommees"></a>
## URLs pour les routes nommées

La fonction `route()` peut être utilisé pour générer des URL vers des [routes nommés](/docs/{version}/routage#routes-nommees). Les routes nommés vous permettent de générer des URL sans être liés à l'URL réelle définie sur la route. Par conséquent, si l'URL de la route change, il n'est pas nécessaire de modifier vos appels à la fonction `route()`. Par exemple, imaginons que votre application contienne une route définie comme suit :

```php
Route::name('post.show')->get('/post/(:num)', function ($id) {
    // ...
});
```

Pour générer une URL vers cette route, vous pouvez utiliser la fonction `route()` comme suit :

```php
echo route('post.show', [1]);
 
// http://example.com/post/1
```

Bien entendu, la fonction `route()` peut également être utilisé pour générer des URL pour des routes à paramètres multiples :

```php
Route::name('comment.show')->get('/post/(:num)/comment/(:num)', function ($id_post, $id_comment) {
    // ...
});
 
echo route('comment.show', [1, 3]);
 
// http://example.com/post/1/comment/3
```

<a name="urls-pour-les-actions-de-controleurs"></a>
## URLs pour les actions de contrôleurs

La fonction `action` génère une URL pour l'action du contrôleur donnée :

```php
use App\Controllers\HomeController;
 
$url = action([HomeController::class, 'index']);
```

Si la méthode du contrôleur accepte des paramètres de route, vous pouvez transmettre un tableau contenant les paramètres successifs en tant que deuxième argument de la fonction :

```php  
namespace App\Controllers;

class UserController extends AppController 
{
    public function profile($id, $section)
    {
        ...
    }
}

$url = action([UserController::class, 'profile'], [
    1, // $id
    'contact' // $section
]);
```

<a name="autres-fonctions-d-url"></a>
## Autres fonctions d'URL

Le [helper](/docs/{version}/helpers) `URL` contient des fonctions qui aident à travailler avec les URL. Il est automatiquement chargé par le framework à chaque requête.

Veuillez consulter [l'API du helper URL](/api/{version}/BlitzPHP/Helpers/url.html) pour en savoir plus