---
title: Les façades
---

<a name="introduction"></a>
## Introduction

Les façades fournissent une interface "statique" aux classes disponibles dans le conteneur de services de l'application. BlitzPHP est livré avec de nombreuses façades qui permettent d'accéder à la quasi-totalité de ses fonctionnalités.

Les façades servent de "proxies statiques" aux classes sous-jacentes dans le conteneur de service, offrant l'avantage d'une syntaxe laconique et expressive tout en conservant plus de testabilité et de flexibilité que les méthodes statiques traditionnelles.

Toutes les façades de BlitzPHP sont définies dans le namespace `BlitzPHP\Facades`. Nous pouvons donc facilement accéder à une façade de la manière suivante :

```php
use BlitzPHP\Facades\Cache;
use BlitzPHP\Facades\Route;
 
Route::get('/cache', function () {
    return Cache::get('key');
});
```

Dans cette documentation, de nombreux exemples utilisent des façades pour démontrer diverses fonctionnalités du framework.

<a name="quand-utiliser-les-facades"></a>
## Quand utiliser les façades

Les façades présentent de nombreux avantages. Elles fournissent une syntaxe laconique et mémorable qui vous permet d'utiliser les fonctionnalités de BlitzPHP sans avoir à vous souvenir de longs noms de classes qui doivent être injectés ou configurés manuellement. De plus, grâce à leur utilisation unique des méthodes dynamiques de PHP, elles sont faciles à tester.

Toutefois, il convient de faire preuve de prudence lors de l'utilisation des façades. Le principal danger des façades est la "dérive" de la classe. Puisque les façades sont si faciles à utiliser et ne nécessitent pas d'injection, il peut être facile de laisser vos classes continuer à grandir et d'utiliser de nombreuses façades dans une seule classe. En utilisant l'injection de dépendances, ce risque est atténué par le feedback visuel qu'un grand constructeur vous donne pour vous indiquer que votre classe est en train de devenir trop grande. Par conséquent, lorsque vous utilisez des façades, prêtez une attention particulière à la taille de votre classe afin que son champ de responsabilité reste étroit. Si votre classe devient trop grande, envisagez de la diviser en plusieurs classes plus petites.

<a name="facades-et-injection-de-dependance"></a>
### Façades et injection de dépendance

L'un des principaux avantages de l'injection de dépendances est la possibilité d'échanger les implémentations de la classe injectée. Ceci est utile pendant les tests puisque vous pouvez injecter un mock ou un stub et affirmer que diverses méthodes ont été appelées sur le stub.

En règle générale, il n'est pas possible de simuler ou de bloquer une méthode de classe vraiment statique. Cependant, étant donné que les façades utilisent des méthodes dynamiques afin de transmettre les appels de méthode aux objets résolus à partir du conteneur de service, nous pouvons en fait tester les façades de la même manière que nous le ferions pour une instance de classe injectée.

<a name="facades-et-helpers"></a>
### Façades et helpers

En plus des façades, BlitzPHP comprend une variété de fonctions "d'aide" qui peuvent effectuer des tâches courantes comme la génération de vues, le déclenchement d'événements, l'envoi de tâches ou l'envoi de réponses HTTP. La plupart de ces fonctions d'aide exécutent la même fonction que la façade correspondante. Par exemple, cet appel de façade et cet appel d'aide sont équivalents :

```php
return BlitzPHP\Facades\View::make('profile');
 
return view('profile');
```

Il n'y a absolument aucune différence pratique entre les façades et les fonctions d'assistance. Lorsque vous utilisez des fonctions d'assistance, vous pouvez les tester exactement comme vous le feriez avec la façade correspondante. 

<a name="facades-vs-services"></a>
### Façades vs services

Les façades sont résolus en utilisant le [fournisseur de service](/docs/{version}/services). De ce fait vous pouvez utiliser les services ou les façades pour les classes de base de BlitzPHP le résultat sera le même. Par exemple, les trois appels suivants sont identique:

```php
// utilisation de la façade
return BlitzPHP\Facades\Fs::exists('avatar.jpg');
 
 // utilisation de la classe Services
return BlitzPHP\Container\Services::fs()->exists('avatar.jpg');

// utilisation de la fonction service
return service('fs')->exists('avatar.jpg');
```

Les trois méthodes sont produiront le même résultat. Toutefois, Il y a une subtilité majeur que vous devez avoir à l'esprit. 

En utilisant la façade, vous obtiendrait **TOUJOURS une instance partagée** de la classe (confère [comment obtenir un service](/docs/{version}/services#comment-obtenir-un-service)) tandis qu'en passant par le service, vous pouvez décider de charger une nouvelle instance de la classe. 

Par ailleurs, avec les services, vous avez la possibilité de transmettre des configurations personnalisés pour la construction de la classe chargée, ce que n'autorise pas les façade.

<a name="fonctionnement-des-facades"></a>
## Fonctionnement des façades

Dans une application BlitzPHP, une façade est une classe qui permet d'accéder à un objet à partir du conteneur. La machinerie qui permet de réaliser ce travail se trouve dans la classe `Facade`. Les façades de BlitzPHP, et toutes les façades personnalisées que vous créez, étendent la classe de base `BlitzPHP\Facades\Facade`.

La classe de base `Facade` utilise la méthode magique `__callStatic()` pour différer les appels de votre façade vers un objet résolu à partir du conteneur. Dans l'exemple ci-dessous, un appel est fait au système de cache de BlitzPHP. En jetant un coup d'œil à ce code, on pourrait supposer que la méthode statique `get` est appelée sur la classe `Cache` :

```php
<?php
 
namespace App\Controllers;
 
use App\Controllers\AppController;
use BlitzPHP\Facades\Cache;
 
class UserController extends AppController
{
    /**
     * Affiche le profil de l'utilisateur donné.
     */
    public function showProfile(string $id)
    {
        $user = Cache::get('user:'.$id);
 
        return view('profile', ['user' => $user]);
    }
}
```

Remarquez qu'en haut du fichier, nous "importons" la façade `Cache`. Cette façade sert de proxy pour accéder à l'implémentation sous-jacente de l'interface `BlitzPHP\Contracts\Cache\CacheInterface`. Tous les appels que nous faisons en utilisant la façade seront transmis à l'instance sous-jacente du service de cache de BlitzPHP.

Si nous examinons la classe `BlitzPHP\Facades\Cache`, vous verrez qu'il n'y a pas de méthode statique `get()` :

```php
class Cache extends Facade
{
    protected static function accessor(): string
    {
        return 'cache';
    }
}
```

Au lieu de cela, la façade `Cache` étend la classe `Facade` de base et définit la méthode `accessor()`. Cette méthode a pour fonction de renvoyer le nom d'une liaison de service. Lorsqu'un utilisateur fait référence à une méthode statique de la façade `Cache`, BlitzPHP résout la liaison de `cache` du [fournisseur de service](/docs/{version}/services) et exécute la méthode demandée (dans ce cas, `get`) sur cet objet.

<a name="reference-de-la-classe-de-facade"></a>
## Référence de la classe de façade

Vous trouverez ci-dessous chaque façade et sa classe sous-jacente. Il s'agit d'un outil utile pour consulter rapidement la documentation de l'API pour une racine de façade donnée. La clé de liaison du [fournisseur de service](/docs/{version}/services) est également incluse le cas échéant.

<div class="overflow-auto">

Façade  |  Classe |  Clé de service
------------- | ------------- | -------------
Builder  |  [BlitzPHP\Database\Builder\BaseBuilder](/api/{version}/BlitzPHP/Database/Builder/BaseBuilder.html)  |  `builder`
Cache  |  [BlitzPHP\Cache\Cache](/api/{version}/BlitzPHP/Cache/Cache.html)  |  `cache`
Config  |  [BlitzPHP\Config\Config](/api/{version}/BlitzPHP/Config/Config.html)  |  `config`
Cookie  |  [BlitzPHP\Session\Cookie\CookieManager](/api/{version}/BlitzPHP/Session/Cookie/CookieManager.html)  |  `cookie`
Crypt  |  [BlitzPHP\Security\Encryption\Encryption](/api/{version}/BlitzPHP/Security/Encryption/Encryption.html)  |  `encrypter`
Db  |  [BlitzPHP\Database\Connection\BaseConnection](/api/{version}/BlitzPHP/Database/Connection/BaseConnection.html)  |  `database`
Event  |  [BlitzPHP\Event\EventManager](/api/{version}/BlitzPHP/Events/EventManager.html)  |  `event`
Fs  |  [BlitzPHP\Filesystem\Filesystem](/api/{version}/BlitzPHP/Filesystem/Filesystem.html)  |  `fs`
Http  |  [BlitzPHP\HttpClient\Request](/api/{version}/BlitzPHP/HttpClient/Request.html)  |  `httpclient`
Lang  |  [BlitzPHP\Translator\Translate](/api/{version}/BlitzPHP/Translator/Translate.html)  |  `translator`
Log  |  [BlitzPHP\Debug\Logger](/api/{version}/BlitzPHP/Debug/Logger.html)  |  `logger`
Mail  |  [BlitzPHP\Mail\Mail](/api/{version}/BlitzPHP/Mail/Mail.html)  |  `mail`
Password  |  [BlitzPHP\Schild\Authentication\Passwords](/api/{version}/BlitzPHP/Schild/Authentication/Passwords.html)  |  `passwords`
Redirect  |  [BlitzPHP\Http\Redirection](/api/{version}/BlitzPHP/Http/Redirection.html)  |  `redirection`
Request  |  [BlitzPHP\Http\Request](/api/{version}/BlitzPHP/Http/Request.html)  |  `request`
Response |  [BlitzPHP\Http\Response](/api/{version}/BlitzPHP/Http/Response.html)  |  `response`;
Route  |  [BlitzPHP\Router\RouteBuilder](/api/{version}/BlitzPHP/Router/RouteBuilder.html)  |  &nbsp;
Session  |  [BlitzPHP\Session\Store](/api/{version}/BlitzPHP/Session/Store.html)  |  `session`
Storage  |  [BlitzPHP\Filesystem\FilesystemManager](/api/{version}/BlitzPHP/Filesystem/FilesystemManager.html)  |  `storage`
Url  |  [BlitzPHP\Http\UrlGenerator](/api/{version}/BlitzPHP/Http/UrlGenerator.html)  |  &nbsp;
Validator  |  [BlitzPHP\Validation\Validator](/api/{version}/BlitzPHP/Validation/Validator.html)  |  &nbsp;
View  |  [BlitzPHP\View\View](/api/{version}/BlitzPHP/View/View.html)  |  `viewer`

</div>
