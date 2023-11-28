---
title: Réponses HTTP
---

<a name="introduction"></a>
## Introduction

La classe `BlitzPHP\Http\Response` est la classe de réponse par défaut dans BlitzPHP. Elle encapsule un nombre de fonctionnalités et de caractéristiques pour la génération de réponses HTTP dans votre application.

<a name="travailler-avec-la-reponse"></a>
## Travailler avec la réponse

Dès le lancement de votre application, une instance de la classe `Response` est créée pour vous et transmise à vos contrôleurs. Elle est accessible via `$this->response`. 

Souvent, vous n'aurez pas besoin de toucher directement la classe, puisque BlitzPHP se charge d'envoyer les en-têtes et le corps pour vous. C'est très bien si la page a réussi à créer le contenu qui lui a été demandé. Cependant, si les choses tournent mal, ou que vous devez renvoyer des codes d'état très spécifiques, ou même profiter de la puissante mise en cache HTTP, l'objet `Response` sera là pour vous.

```php
<?php

class UserController extends AppController
{
    public function index()
    {
    	return $this->response->withStatus(200);
    }
}
```

Si vous n'êtes pas dans un contrôleur et avez besoin d'accéder à l'objet `Response` de l'application, vous pouvez le récupérer en utilisant le [fournisseur de services](/docs/{version}/services) ou le [conteneur d'injection de dépendances](/docs/{version}/conteneur).

```php
$response = service('response');
```

Il est cependant préférable de passer la réponse en tant que dépendance si la classe est autre chose que le contrôleur, où vous pouvez l'enregistrer en tant que propriété de classe.

> **Note**  
> Dans la suite de ce chapitre nous supposerons que la variable `$response` est une instance de la classe `BlitzPHP\Http\Response`. Si vous êtes dans un contrôleur lors de l’exécution de votre code, vous devez utiliser la propriété d'instance `$this->response`.

<a name="creation-de-reponses"></a>
## Création de réponses

<a name="chaines-et-tableaux"></a>
#### Chaînes et tableaux

Toutes les routes et tous les contrôleurs doivent renvoyer une réponse à renvoyer au navigateur de l'utilisateur. BlitzPHP propose plusieurs manières différentes de renvoyer des réponses. La réponse la plus élémentaire consiste à renvoyer une chaîne depuis une route ou un contrôleur. Le framework convertira automatiquement la chaîne en une réponse HTTP complète :

```php
Route::get('/', function () {
    return 'Hello World';
});
```

En plus de renvoyer des chaînes de vos routes et contrôleurs, vous pouvez également renvoyer des tableaux. Le framework convertira automatiquement le tableau en réponse JSON :

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

<a name="l-objet-de-reponse"></a>
#### L'objet de réponse

En règle générale, vous ne renverrez pas simplement de simples chaînes ou tableaux à partir de vos actions de route. Au lieu de cela, vous renverrez des instances complètes `BlitzPHP\Http\Response` ou des [vues](/docs/{version}/vues).

Le renvoi d'une instance de réponse complète vous permet de personnaliser le code d'état HTTP et les en-têtes de la réponse. L'objet `Response` de BlitzPHP implémente <a href="https://github.com/php-fig/http-message/blob/master/src/ResponseInterface.php" target="_blank">l'interface ResponseInterface du psr7</a>, qui fournit diverses méthodes pour créer des réponses HTTP :

```php
use BlitzPHP\Http\Response;

Route::get('/home', function (Response $response) {
    return $response->withStatus(200)
                  ->withBody(to_stream('Hello World'))
                  ->withHeader('Content-Type', 'text/plain');
});
```

<a name="entites-wolke-et-collections"></a>
#### Entités Wolke et collections

Vous pouvez également renvoyer des [entités Wolke ORM](/docs/{version}/wolke) et des collections directement à partir de vos routes et contrôleurs. Lorsque vous le ferez, BlitzPHP convertira automatiquement les modèles et collections en réponses JSON tout en respectant les attributs cachés du modèle :

```php
use App\Entities\User;
 
Route::get('/user/(:num)', function ($id) {
    $user = User::find($id);
    
    return $user;
});
```

<a name="attacher-des-en-tetes-aux-reponses"></a>
### Attacher des en-têtes aux réponses

Gardez à l’esprit que la plupart des méthodes de réponse peuvent être chaînées, ce qui permet la construction fluide d’instances de réponse. Par exemple, vous pouvez utiliser la méthode `withHeader()` pour ajouter une série d'en-têtes à la réponse avant de la renvoyer à l'utilisateur :

```php
return $response
            ->withHeader('Content-Type', $type)
            ->withHeader('X-Header-One', 'Header Value')
            ->withHeader('X-Header-Two', 'Header Value');
```

Vous pouvez également utiliser la méthode `withHeaders()` pour spécifier un tableau d'en-têtes à ajouter à la réponse :

```php
return $response->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

<a name="joindre-des-cookies-aux-reponses"></a>
### Joindre des cookies aux réponses

Vous pouvez attacher un cookie à une instance `BlitzPHP\Http\Response` sortante en utilisant la méthode `cookie()`. Vous devez transmettre le nom, la valeur et le nombre de minutes pendant lesquelles le cookie doit être considéré comme valide à cette méthode :

```php
return $response->cookie('name', 'value', $minutes);
```

La méthode `cookie()` accepte également un tableau d'options supplémentaires qui sont utilisés moins fréquemment. Généralement, ces options ont le même but et la même signification que les arguments qui seraient donnés à la méthode native <a href="https://secure.php.net/manual/en/function.setcookie.php" target="_blank">setcookie</a> de PHP :

```php
return $response->cookie('name', 'value', $minutes, [
    'path' => $path, 
    'domain' => $domain, 
    'secure' => $secure, 
    'http_only' => $httpOnly
]);
```

<a name="generer-des-instances-de-cookies"></a>
#### Générer des instances de cookies

Si vous souhaitez générer une instance `BlitzPHP\Contracts\Session\CookieInterface` qui peut être attachée à une instance de réponse ultérieurement, vous pouvez utiliser le [helper](/docs/{version}/helpers) global `cookie()`. Ce cookie ne sera renvoyé au client que s'il est attaché à une instance de réponse :

```php
$cookie = cookie('name', 'value', $minutes);
 
return $response->cookie($cookie);
```

<a name="cookies-expirant-plus-tot"></a>
#### Cookies expirant plus tôt

Vous pouvez supprimer un cookie en l'expirant via la méthode `withoutCookie()` d'une réponse sortante :

```php
return $response->withoutCookie('name');
```

<a name="cookies-et-cryptage"></a>
### Cookies et cryptage

Par défaut, tous les cookies générés par BlitzPHP sont cryptés et signés afin qu'ils ne puissent pas être modifiés ou lus par le client. Si vous souhaitez désactiver le chiffrement pour un sous-ensemble de cookies générés par votre application, vous pouvez utiliser la propriété `$except` du middleware `App\Middlewares\EncryptCookies`, qui se trouve dans le répertoire `app/Middlewares` :

```php
/**
 * Les noms des cookies qui ne doivent pas être cryptés.
 */
protected array $except = [
    'cookie_name',
];
```

<a name="redirections"></a>
## Redirections

Les réponses de redirection sont des instances de la classe `BlitzPHP\Http\Redirection` et contiennent les en-têtes appropriés nécessaires pour rediriger l'utilisateur vers une autre URL. Il existe plusieurs façons de générer une instance  `Redirection`. La méthode la plus simple consiste à utiliser la fonction globale `redirect()` :

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```

Parfois, vous souhaiterez peut-être rediriger l'utilisateur vers sa page précédente, par exemple lorsqu'un formulaire soumis n'est pas valide. Vous pouvez le faire en utilisant la fonction globale `back()`.

```php
Route::post('/user/profile', function () {
    // Validation de la requete...
 
    return back()->withInput();
});
```

<a name="redirection-vers-des-routes-nommees"></a>
### Redirection vers des routes nommées

Lorsque vous appelez la fonction `redirect()` sans paramètres, une instance de `BlitzPHP\Http\Redirection` est renvoyée, vous permettant d'appeler n'importe quelle méthode sur cette instance afin de contrôler votre processus de redirection. Par exemple, pour générer une `Redirection` vers une route nommée, vous pouvez utiliser la méthode `route()` :

```php
return redirect()->route('login');
```

Si votre route a des paramètres, vous pouvez les transmettre comme deuxième argument à la méthode `route()` :

```php
// Pour une route avec l'URI suivante: /profile/(:num)
 
return redirect()->route('profile', [1]);
```

<a name="redirection-vers-les-actions-du-controleur"></a>
### Redirection vers les actions du contrôleur

Vous pouvez également générer des redirections vers les [actions du contrôleur](/docs/{version/controleurs). Pour ce faire, transmettez le contrôleur et le nom de l'action à la méthode `action()` :

```php
use App\Controllers\UserController;
 
return redirect()->action([UserController::class, 'index']);
```

Si votre route de contrôleur nécessite des paramètres, vous pouvez les transmettre comme deuxième argument à la méthode `action()` :

```php
return redirect()->action(
    [UserController::class, 'profile'], [1]
);
```

<a name="redirection-vers-des-domaines-externes"></a>
### Redirection vers des domaines externes

Parfois, vous devrez peut-être rediriger vers un domaine extérieur à votre application. Vous pouvez le faire en appelant la méthode `away()`, qui crée une `Redirection` sans aucun codage, validation ou vérification d'URL supplémentaire :

```php
return redirect()->away('https://www.google.com');
```

<a name="redirection-avec-des-donnees-de-session-flashees"></a>
### Redirection avec des données de session flashées

La redirection vers une nouvelle URL et [le flashage des données vers la session](/docs/{version}/session#flashage-des-donnees) sont généralement effectués en même temps. En règle générale, cela est effectué après avoir effectué avec succès une action lorsque vous envoyez un message de réussite à la session. Pour plus de commodité, vous pouvez créer une instance `Redirection` et flasher des données sur la session dans une chaîne de méthodes unique et fluide :

```php
Route::post('/user/profile', function () {
    // ...
 
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

Une fois l'utilisateur redirigé, vous pouvez afficher le message flashé de la [session](/docs/{version}/session). Par exemple :

```php
<?php if (session('status')): ?>
    <div class="alert alert-success">
        <?= session('status') ?>
    </div>
<?php endif; ?>
```

<a name="redirection-avec-les-entrees"></a>
#### Redirection avec les entrées

Vous pouvez utiliser la méthode `withInput()` fournie par l'instance `Redirection` pour flasher les données d'entrée de la requête actuelle dans la session avant de rediriger l'utilisateur vers un nouvel emplacement. Cela se produit généralement si l'utilisateur a rencontré une erreur de validation. Une fois l'entrée flashée dans la session, vous pourrez facilement la [récupérer](/docs/{version}/requetes#recupération-d-anciennes-entrees) lors de la prochaine requête pour repeupler le formulaire :

```php
return back()->withInput();
```

<a name="autres-types-de-reponses"></a>
## Autres types de réponses

<a name="reponse-de-vues"></a>
### Réponse de vues

Si vous avez besoin de contrôler le statut et les en-têtes de la réponse, mais également de renvoyer une [vue](/docs/{version}/vues) en tant que contenu de la réponse, vous devez utiliser la méthode `view()` :

```php
return $response
            ->view('hello', $data, 200)
            ->WithHeader('Content-Type', $type);
```

Bien entendu, si vous n'avez pas besoin de transmettre un code d'état HTTP personnalisé ou des en-têtes personnalisés, vous pouvez utiliser la fonction globale `view()`.

<a name="reponse-de-vues"></a>
### Réponse JSON

La méthode `json()` définira automatiquement l'en-tête `Content-Type` sur `application/json`, ainsi que convertira le tableau donné en JSON à l'aide de la fonction PHP `json_encode` :

```php
return $response->json([
    'name' => 'Fabien',
    'state' => 'FR',
]);
```

<a name="telechargements-de-fichiers"></a>
### Téléchargements de fichiers

La méthode `download()` peut être utilisée pour générer une réponse qui oblige le navigateur de l'utilisateur à télécharger le fichier au chemin indiqué. La méthode `download()` accepte un nom de fichier comme deuxième argument, qui déterminera le nom de fichier vu par l'utilisateur qui télécharge le fichier. Enfin, vous pouvez transmettre un tableau d'en-têtes HTTP comme troisième argument de la méthode :

```php
return $response->download($pathToFile);
 
return $response->download($pathToFile, $name, $headers);
```

<a name="telechargements-en-streaming"></a>
#### Téléchargements en streaming

Parfois, vous souhaiterez peut-être transformer la réponse sous forme de chaîne d'une opération donnée en une réponse téléchargeable sans avoir à écrire le contenu de l'opération sur le disque. Vous pouvez utiliser la méthode `streamDownload()` dans ce scénario. Cette méthode accepte un callback, un nom de fichier et un tableau facultatif d'en-têtes comme arguments :

```php
use App\Services\GitLab;
 
return $response->streamDownload(function () {
    echo GitLab::api('repo')
                ->contents()
                ->readme('blitz-php', 'framework')['contents'];
}, 'blitz-php-readme.md');
```

<a name="reponses-de-fichiers"></a>
### Réponses de fichiers

La méthode `file()` peut être utilisée pour afficher un fichier, tel qu'une image ou un PDF, directement dans le navigateur de l'utilisateur au lieu de lancer un téléchargement. Cette méthode accepte le chemin absolu du fichier comme premier argument et un tableau d'en-têtes comme deuxième argument :

```php
return $response->file($pathToFile);
 
return $response->file($pathToFile, $headers);
```

<a name="interagir-avec-le-cache-du-navigateur"></a>
## Interagir avec le cache du navigateur

<a name="desactiver-le-cache"></a>
### Désactiver le cache

Parfois, vous avez besoin de forcer les navigateurs à ne pas mettre en cache les résultats de l’action d’un contrôleur. La méthode `withDisabledCache()` a justement été prévue pour cela:

```php
// Désactive le cache
$response = $response->withDisabledCache();
```

> **Attention**  
> Désactiver le cache à partir de domaines SSL pendant que vous essayez d’envoyer des fichiers à Internet Explorer peut entraîner des erreurs.

<a name="forcer-la-mise-en-cache-d-une-page"></a>
### Forcer la mise en cache d'une page

Vous pouvez aussi dire aux navigateurs que vous voulez qu’ils mettent en cache des réponses. Ceci se fait en utilisant la méthode `withCache()` :

```php
// Autoriser la mise en cache
$response = $response->withCache('-1 minute', '+5 days');
```

Ce qui est au-dessus indiquera aux navigateurs de mettre en cache la réponse résultante pendant 5 jours, espérant ainsi accélérer l’expérience de vos visiteurs. La méthode `withCache()` définit valeur `Last-Modified` en premier argument. L’entête `Expires` et `max-age` sont définis en se basant sur le second paramètre. Le `Cache-Control` est défini aussi à `public`.

<a name="controle-du-cache"></a>
### Contrôle du cache

L'en-tête `Cache-Control` contient de multiples indicateurs qui peuvent changer la façon dont les navigateurs ou les proxies utilisent le contenu mis en cache. Un en-tête `Cache-Control` peut ressembler à ceci:

```http
Cache-Control: private, max-age=3600, must-revalidate
```

La classe `Response` vous aide à configurer cet en-tête avec quelques méthodes utiles qui vont produire un en-tête final `Cache Control` valide. La première est la méthode `withSharable()`, qui indique si une réponse peut être considérée comme partageable pour différents utilisateurs ou clients. Cette méthode contrôle en fait la partie public ou private de cet en-tête. Définir une réponse en private indique que tout ou partie de celle-ci est prévue pour un unique utilisateur. Pour tirer profit des mises en cache partagées, il est nécessaire de définir la directive de contrôle en public.

Le deuxième paramètre de cette méthode est utilisé pour spécifier un `max-age` pour le cache qui est le nombre de secondes après lesquelles la réponse n’est plus considérée comme récente:

```php
// Définit le Cache-Control en public pour 3600 secondes
$response = $response->withSharable(true, 3600);

// Définit le Cache-Control en private pour 3600 secondes
$response = $response->withSharable(false, 3600);
```

`Response` expose des méthodes séparées pour la définition de chaque composant dans l’en-tête de `Cache-Control`.

<a name="L-en-tete-d-expiration"></a>
### L’en-tête d’expiration

Vous pouvez définir l’en-tête `Expires` avec une date et un temps après lesquels la réponse n’est plus considérée comme à jour. Cet en-tête peut être défini en utilisant la méthode `withExpires()`:

```php
$response = $response->withExpires('+5 days');
```

Cette méthode accepte aussi une instance `DateTime` ou toute chaîne de caractère qui peut être parsée par la classe `DateTime`.

<a name="L-en-tete-etag"></a>
### L’en-tête Etag

La validation du Cache dans HTTP est souvent utilisée quand le contenu change constamment et demande à l’application de générer seulement les contenus de la réponse si le cache n’est plus à jour. Sous ce modèle, le client continue de stocker les pages dans le cache, mais au lieu de l’utiliser directement, il demande à l’application à chaque fois si les ressources ont changé ou non. C’est utilisé couramment avec des ressources statiques comme les images et autres ressources.

La méthode `withEtag()` (appelée balise d’entité) est une chaîne de caractère qui identifie de façon unique les ressources requêtées comme le fait un checksum pour un fichier, afin de déterminer si elle correspond à une ressource du cache.

Pour réellement tirer profit de l’utilisation de cet en-tête, vous devez appeler manuellement la méthode `checkNotModified()`

```php
public function index()
{
    $articles = Article::all();

    // Somme de contrôle simple du contenu de l'article.
    // Vous devriez utiliser une implémentation plus efficace
    // dans une application du monde réel.
    $checksum = md5(json_encode($articles));

    $response = $this->response->withEtag($checksum);
    if ($response->checkNotModified($this->request)) {
        return $response;
    }

    $this->response = $response;
}
```

> **Note**  
> La plupart des utilisateurs proxy devront probablement penser à utiliser l’en-tête Last Modified plutôt que Etags pour des raisons de performance et de compatibilité.

<a name="L-en-tete-last-modified"></a>
### L’en-tête Last-Modified

De même, avec la méthode consistant à valider du cache HTTP, vous pouvez définir l’en-tête `Last-Modified` pour indiquer la date et l’heure à laquelle la ressource a été modifiée pour la dernière fois. Définir cet en-tête aide BlitzPHP à indiquer à ces clients si la réponse a été modifiée ou n’est pas basée sur leur cache.

Pour réellement tirer profit de l’utilisation de cet en-tête, vous devez appeler manuellement la méthode `checkNotModified()`

```php
public function view()
{
    $article = Article::first();
    $response = $this->response->withModified($article->modified);
    if ($response->checkNotModified($this->request)) {
        return $response;
    }
    $this->response;
    // ...
}
```

<a name="envoyer-des-reponses-non-modifiees"></a>
### Envoyer des réponses non-modifiées

Compare les en-têtes de cache pour l’objet requêté avec l’en-tête du cache de la réponse et determine s’il peut toujours être considéré comme à jour. Si oui, il supprime le contenu de la réponse et envoie l’en-tête `304 Not Modified`:

```php
// Dans une action de controller.
if ($this->response->checkNotModified($this->request)) {
    return $this->response;
}
```


<a name="erreurs-communes-avec-les-reponses-immutables"></a>
## Erreurs communes avec les réponses immutables

Les objets `Response` offrent de nombreuses méthodes qui traitent les réponses comme des objets immutables. Les objets immutables permettent de prévenir les effets de bord difficiles à repérer. Malgré leurs nombreux avantages, s’habituer aux objets immutables peut prendre un peu de temps. Toutes les méthodes qui commencent par `with` interagiront avec la réponse à la manière immutable et retourneront **toujours** une **nouvelle** instance. L’erreur la plus fréquente quand les développeurs travaillent avec les objets immutables est d’oublier de persister l’instance modifiée:

```php
$response->withHeader('X-BlitzPHP', 'yes!');
```

Dans le code ci-dessus, la réponse ne contiendra pas le header `X-BlitzPHP` car la valeur retournée par `withHeader()` n’a pas été persistée. Pour avoir un code fonctionnel, vous devrez écrire:

```php
$response = $response->withHeader('X-BlitzPHP', 'yes!');
```  
