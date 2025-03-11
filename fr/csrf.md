---
title: Protection CSRF
---

<a name="introduction"></a>
## Introduction

Les Cross-Site Request Forgeries (CSRF) sont un type d'exploitation malveillante par laquelle des commandes non autorisées sont exécutées au nom d'un utilisateur authentifié à son insu ou sans son consentement. Heureusement, BlitzPHP permet de protéger facilement votre application contre ces types d'attaques.

<a name="explication-de-la-vulnerabilite"></a>
#### Explication de la vulnérabilité

Au cas où vous ne seriez pas familier avec les contrefaçons de requêtes intersites (CSRF), examinons un exemple de la manière dont cette vulnérabilité peut être exploitée. Imaginez que votre application dispose d'une route `/user/email` qui accepte une requête `POST` pour modifier l'adresse électronique de l'utilisateur authentifié. Très probablement, cette route s'attend à ce qu'un champ de saisie `email` contienne l'adresse électronique que l'utilisateur souhaite commencer à utiliser.

Sans protection CSRF, un site web malveillant pourrait créer un formulaire HTML qui pointe vers la route `/user/email` de votre application et soumet l'adresse électronique de l'utilisateur malveillant :

```html
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Si le site web malveillant soumet automatiquement le formulaire lorsque la page est chargée, l'utilisateur malveillant n'a qu'à inciter un utilisateur peu méfiant de votre application à visiter son site web et son adresse électronique sera modifiée dans votre application.

Pour éviter cette vulnérabilité, nous devons inspecter chaque requête `POST`, `PUT`, `PATCH` ou `DELETE` entrante à la recherche d'une valeur de session secrète à laquelle l'application malveillante ne peut accéder.

<a name="prevention-des-requetes-csrf"></a>
## Prévention des requêtes CSRF

BlitzPHP génère automatiquement un "jeton" CSRF pour chaque [session utilisateur](/docs/{version}/session) active gérée par l'application. Ce jeton est utilisé pour vérifier que l'utilisateur authentifié est bien la personne qui effectue les requêtes auprès de l'application. Comme ce jeton est stocké dans la session de l'utilisateur et change chaque fois que la session est régénérée, une application malveillante ne peut pas y accéder.

Le jeton CSRF de la session en cours est accessible via la session de la requête ou via la fonction `csrf_token()` :

```php
Route::get('/token', function () {
    $token = session()->token();

    $token = csrf_token();

    // ...
});
```

Chaque fois que vous définissez un formulaire HTML "POST", "PUT", "PATCH" ou "DELETE" dans votre application, vous devez inclure un champ CSRF `_token` caché dans le formulaire afin que le middleware de protection CSRF puisse valider la requête. Par commodité, vous pouvez utiliser la [méthode de vue `csrf`](/docs/{version}/vues-natives#champ-csrf) pour générer le champ de saisie du jeton caché :

```php
<form method="POST" action="/profile">
    <?= $this->csrf() ?>
 
    <!-- Equivalent à... -->
    <input type="hidden" name="_token" value="<?= csrf_token() ?>" />
</form>
```

Le [middleware](/docs/{version}/middleware) `App\Middlewares\VerifyCsrfToken`, vérifiera automatiquement que le jeton dans les données de la requête correspond au jeton stocké dans la session. Lorsque ces deux jetons correspondent, nous savons que l'utilisateur authentifié est celui qui a initié la requête.

<a name="exclusion-des-uri-de-la-protection-csrf"></a>
### Exclusion des URI de la protection CSRF

Il peut arriver que vous souhaitiez exclure un ensemble d'URI de la protection CSRF. Par exemple, si vous utilisez <a href="https://stripe.com/" target="_blank">Stripe</a> pour traiter les paiements et que vous utilisez leur système de webhook, vous devrez exclure votre route du gestionnaire du webhook Stripe de la protection CSRF car Stripe ne saura pas quel jeton CSRF envoyer à vos routes.

Vous pouvez exclure les routes en ajoutant leurs URI à la propriété `$except` du middleware `VerifyCsrfToken` :

```php
<?php

namespace App\Middlewares;

use BlitzPHP\Middlewares\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * Les URI qui doivent être exclus de la vérification CSRF.
     */
    protected array $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ];
}
```

<a name="x-csrf-token"></a>
## X-CSRF-TOKEN

Outre la vérification du jeton CSRF en tant que paramètre POST, le middleware `App\Middlewares\VerifyCsrfToken` vérifiera également l'en-tête de requête X-CSRF-TOKEN. Vous pouvez, par exemple, stocker le jeton dans une balise méta  :

```php
<meta name="csrf-token" content="<?= csrf_token() ?>">
```

Vous pouvez ensuite demander à une bibliothèque comme jQuery d'ajouter automatiquement le jeton à tous les en-têtes de requête. Vous obtenez ainsi une protection CSRF simple et pratique pour vos applications AJAX utilisant la technologie JavaScript traditionnelle :

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="x-xsrf-token"></a>
## X-XSRF-TOKEN


BlitzPHP stocke le jeton CSRF actuel dans un cookie XSRF-TOKEN crypté qui est inclus dans chaque réponse générée par le framework. Vous pouvez utiliser la valeur du cookie pour définir l'en-tête de requête `X-XSRF-TOKEN`.

Ce cookie est principalement envoyé par commodité pour les développeurs, car certains framework et bibliothèques JavaScript, comme Angular et Axios, placent automatiquement sa valeur dans l'en-tête `X-XSRF-TOKEN` sur les requêtes de même origine.