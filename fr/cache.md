---
title: Gestion du cache
---

<a name="introduction"></a>
## Introduction

La mise en cache est une technique qui permet d'améliorer considérablement les performances d'une application en stockant les données fréquemment consultées sur un support de stockage plus rapide, tel que la mémoire. Cela réduit la nécessité de recalculer ou d'extraire les données d'une source plus lente, telle qu'une base de données. En mettant en œuvre une couche de mise en cache, l'application peut rapidement récupérer les données dans le cache au lieu de les recalculer ou de les extraire de la source la plus lente, ce qui peut améliorer le temps de réponse global et réduire la charge de la source la plus lente.

BlitzPHP fournit une API expressive et unifiée pour différents gestionnaires de cache, ce qui vous permet de tirer parti de leur rapidité de récupération des données et d'accélérer votre application web. Le système de cache de BlitzPHP implémente <a href="https://www.php-fig.org/psr/psr-16/" target="_blank">l'interface PSR-16</a>, ce qui en fait un choix polyvalent pour les développeurs familiarisés avec les normes PHP.

<a name="configuration"></a>
## Configuration

La configuration du cache de votre application se trouve dans le fichier `app/Config/cache.php`. Dans ce fichier, vous pouvez spécifier le gestionnaire de cache que vous souhaitez utiliser par défaut dans votre application. BlitzPHP supporte les gestionnaires de cache les plus populaires comme <a href="https://memcached.org/" target="_blank">Memcached</a>, <a href="https://redis.io/" target="_blank">Redis</a>, <a href="https://php.net/wincache" target="_blank">Wincache</a> et bien d'autres. En outre, un gestionnaire de cache basé sur les fichiers est disponible, tandis que les gestionnaires de cache `array` fournit un cache pratique pour vos tests automatisés.

Le fichier de configuration du cache contient également d'autres options, qui sont documentées dans le fichier, alors assurez-vous de lire ces options.   

Quelques options globales de configurations pratiques sont:

* `handler`: Il s'agit du nom du gestionnaire qui doit être utilisé comme gestionnaire principal lors du démarrage du moteur. Les noms disponibles sont : `dummy`, `file`, `memcached`, `redis`, `wincache`.
* `fallback_handler`: Le nom du gestionnaire qui sera utilisé si le premier est inaccessible. Souvent, 'file' est utilisé ici puisque le système de fichiers est toujours disponible, même si ce n'est pas toujours pratique pour l'application.
* `prefix`: Si plusieurs applications utilisent la même mémoire cache, vous pouvez ajouter ici un préfixe personnalisé qui sera ajouté à tous les noms de clés.
* `ttl`: Le nombre de secondes par défaut pour enregistrer les éléments si aucun n'est spécifié.  
ATTENTION : Cette valeur n'est pas utilisée par les gestionnaires du framework où `60 secondes` est codé en dur, mais elle peut être utile aux projets et aux modules. Cette valeur remplacera la valeur codée en dur dans une prochaine version.


Par défaut, BlitzPHP est configuré pour utiliser le gestionnaire de cache de fichiers, qui stocke les objets sérialisés et mis en cache sur le système de fichiers du serveur. Pour les applications plus importantes, il est recommandé d'utiliser un gestionnaire plus robuste tel que Memcached ou Redis. 

<a name="gestionnaires-de-cache-disponible"></a>
## Gestionnaires de cache disponible

> **Note**  
> Avant toute chose, sachez que quelqu'en soit le moteur de cache que vous choisirez d’utiliser, votre application interagit avec le cache de manière cohérente. Cela signifie que vous pouvez aisément permuter les moteurs de cache en fonction de l’évolution de votre application.

<a name="cache-base-sur-des-fichiers"></a>
### Cache basé sur des fichiers

La mise en cache basée sur les fichiers est un cache simple qui utilise des fichiers locaux. C’est le moteur de cache le plus lent, et il ne fournit que peu de fonctionnalités pour les opérations atomiques. Cependant, le stockage sur disque est souvent peu consommateur en ressource, le stockage de grands objets ou des éléments qui sont rarement écrits fonctionne bien dans les fichiers.

Utilisez cette possibilité avec précaution et veillez à évaluer votre application, car il peut arriver que les entrées/sorties sur disque annulent les avantages de la mise en cache. Cela nécessite qu'un répertoire de cache soit réellement accessible en écriture par l'application.

<a name="memcached"></a>
### Memcached

L'utilisation du gestionnaire Memcached nécessite l'installation du <a href="https://pecl.php.net/package/memcached" target="_blank">paquet Memcached PECL</a>. Vous pouvez dresser la liste de tous vos serveurs Memcached dans le fichier de configuration config/cache.php. Ce fichier contient déjà une entrée `memcached` pour vous aider à démarrer :

```php
return [
    //---
    
    'memcached' => [
        'host'   => '127.0.0.1',
        'port'   => 11211,
        'weight' => 100,
        'raw'    => false,
    ],
    
    //---
];
```

Si nécessaire, vous pouvez définir l'option `host` sur un chemin de socket UNIX. Dans ce cas, l'option `port` doit être fixée à `0` :

```php
return [
    //---
    
    'memcached' => [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
    ],
    
    //---
];
```

<a name="redis"></a>
### Redis

Redis est un magasin de valeurs clés en mémoire qui peut fonctionner en mode cache LRU. Pour l'utiliser, vous avez besoin d'un <a href="https://github.com/phpredis/phpredis" target="_blank">serveur Redis et de l'extension PHP phpredis</a>.

Les options de configuration pour se connecter au serveur Redis sont stockées dans le fichier de configuration du cache. Les options disponibles sont :

```php
return [
    //---
    
    'redis' => [
        'host'     => '127.0.0.1',
        'password' => false,
        'port'     => 6379,
        'timeout'  => 0,
        'database' => 0,
    ],
    
    //---
];
```

<a name="wincache"></a>
### Wincache

Sous Windows, vous pouvez également utiliser le pilote WinCache.

Pour plus d'informations sur WinCache, veuillez consulter <a href="https://www.php.net/wincache" target="_blank">https://www.php.net/wincache</a>.

<a name="apcu"></a>
### Apcu

Le cache Apcu utilise l’extension PHP <a href="https://php.net/apcu" target="_blank">APCu</a>. Cette extension utilise la mémoire partagée du serveur Web pour stocker les objets. Cela le rend très rapide, et capable de fournir les fonctionnalités atomiques en lecture/écriture.

<a name="utilisation-du-cache"></a>
## Utilisation du cache

<a name="obtention-d-une-instance-de-cache"></a>
### Obtention d'une instance de cache

Pour obtenir une instance de magasin de cache, vous pouvez utiliser la [façade](/docs/{version}/facades) ou le [service](/docs/{version}/services) `Cache`. Dans cette documentation, nous utiliserons la façade mais sachez que toutes les méthodes décrites sont également disponible via le service de cache.

```php
<?php
 
namespace App\Controllers;
 
use BlitzPHP\Facades\Cache;
 
class UserController extends AppController
{
    public function index(): array
    {
        $value = Cache::get('key');
        
        // utilisation du cache via le service
        // $value = service('cache')->get('key');
 
        return [
            // ...
        ];
    }
}
```

<a name="stockage-d-elements-dans-le-cache"></a>
### Stockage d'éléments dans le cache

Vous pouvez utiliser la méthode `write()` pour stocker des éléments dans le cache :

```php
Cache::write('key', 'value', $ttl = 10); // restera dans le cache pour 10 secondes
```

Si la durée de stockage n'est pas communiquée à la méthode `write`, l'élément sera stocké indéfiniment :

```php
Cache::write('key', 'value');
```

Au lieu de transmettre le nombre de secondes sous la forme d'un nombre entier, vous pouvez également transmettre une instance de `DateTime` représentant l'heure d'expiration souhaitée pour l'élément mis en cache :

```php
Cache::write('key', 'value', now()->addMinutes(10));
```

<a name="stocker-plusieurs-elements-d-un-coup"></a>
#### Stocker plusieurs éléments d'un coup

Vous pouvez avoir besoin d’écrire plusieurs clés du cache à la fois. Bien que vous pouvez utiliser plusieurs appels à `write()`, `writeMany()` permet à BlitzPHP l’utilisation d’une API de stockage plus efficace quand cela est possible. Par exemple utiliser `writeMany()` permet de gagner de nombreuses connections réseau lors de l’utilisation de Memcached:

```php
Cache::writeMany([
    'key1' => 'value1',
    'key2' => 'value2'
], $ttl = null);
```

<a name="stocker-si-non-present"></a>
#### Stocker si non présent

La méthode `add` n'ajoutera l'élément au cache que s'il n'existe pas déjà dans le magasin du cache. La méthode renvoie `true` si l'élément est effectivement ajouté au cache. Dans le cas contraire, la méthode renverra `false`. 

```php
Cache::add('key', 'value', $ttl);
```

<a name="incrementation-decrementation-des-valeurs"></a>
#### Incrémentation / Décrémentation des valeurs

Les méthodes `increment()` et `decrement()` peuvent être utilisées pour ajuster la valeur d'éléments entiers dans le cache. Ces deux méthodes acceptent un deuxième argument facultatif indiquant le décalage de l'incrémentation ou de la décrémentation de la valeur de l'élément :

```php
// Initialiser la valeur si elle n'existe pas...
Cache::add('key', 0, now()->addHours(4));
 
// Incrémenter ou décrémenter la valeur...
Cache::increment('key');
Cache::increment('key', $offset);
Cache::decrement('key');
Cache::decrement('key', $offset);
```

<a name="recuperer-et-stocker"></a>
#### Récupérer et stocker

Il peut arriver que vous souhaitiez récupérer un élément du cache, mais aussi stocker une valeur par défaut si l'élément demandé n'existe pas. Par exemple, vous pouvez souhaiter récupérer tous les utilisateurs dans le cache ou, s'ils n'existent pas, les récupérer dans la base de données et les ajouter au cache. Vous pouvez le faire en utilisant la méthode `Cache::remember` :

```php
$value = Cache::remember('users', $seconds, function () {
    return service('database')->table('users')->all();
});
```

Si l'élément n'existe pas dans le cache, la closure transmise à la méthode `remember` sera exécutée et son résultat sera placé dans le cache.

<a name="recuperer-et-supprimer"></a>
#### Récupérer et supprimer

Si vous avez besoin de récupérer un élément du cache et de le supprimer ensuite, vous pouvez utiliser la méthode `pull()`. Comme pour la méthode `read`, `null` sera renvoyé si l'élément n'existe pas dans le cache :

```php
$value = Cache::pull('key');
```

<a name="recuperation-d-elements-dans-le-cache"></a>
### Récupération d'éléments dans le cache

La méthode `read` de la façade `Cache` est utilisée pour récupérer des éléments dans le cache. Si l'élément n'existe pas dans le cache, `null` sera renvoyé. Si vous le souhaitez, vous pouvez passer un second argument à la méthode `read` en spécifiant la valeur par défaut que vous souhaitez voir renvoyée si l'élément n'existe pas :

```php
$value = Cache::read('key');
 
$value = Cache::read('key', 'default');
```

Vous pouvez même passer une closure comme valeur par défaut. Le résultat de la closure sera renvoyé si l'élément spécifié n'existe pas dans le cache. Le passage d'une closure vous permet de différer la récupération des valeurs par défaut à partir d'une base de données ou d'un autre service externe :

```php
$value = Cache::read('key', function () {
    return service('database')->table(/* ... */)->get();
});
```

<a name="recuperer-plusieurs-elements-d-un-coup"></a>
#### Récupérer plusieurs éléments d'un coup

Après avoir écrit plusieurs clés d’un coup, vous voudrez probablement les lire également. Bien que vous pouvez utiliser plusieurs appels à `read()`, `readMany()` permet à BlitzPHP l'utilisation d'une API de stockage plus efficace quand cela est possible. Par exemple utiliser `readMany()` permet de gagner de nombreuses connections réseau lors de l’utilisation de Memcached.

```php
$result = Cache::readMany(['key1', 'key2']);

// $result contiendra
['key1' => '...', 'key2' => '...']
```

<a name="determination-de-l-existence-d-un-element"></a>
#### Détermination de l'existence d'un élément

La méthode `has` peut être utilisée pour déterminer si un élément existe dans le cache. Cette méthode renvoie également `false` si l'élément existe mais que sa valeur est `null` :

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="suppression-d-elements-du-cache"></a>
### Suppression d'éléments du cache

Vous pouvez supprimer des éléments du cache en utilisant la méthode `delete` :

```php
Cache::delete('key');
```

<a name="supprimer-plusieurs-elements-d-un-coup"></a>
#### Supprimer plusieurs éléments d'un coup

Après avoir écrit plusieurs clés d’un coup, vous voudrez probablement les supprimer également. Bien que vous pouvez utiliser plusieurs appels à `delete()`, `deleteMany()` permet à BlitzPHP l’utilisation d’une API de stockage plus efficace quand cela est possible. Par exemple utiliser `deleteMany()` permet de gagner de nombreuses connections réseau lors de l’utilisation de Memcached:

```php
$result = Cache::deleteMany(['key1', 'key2']);

// $result contiendra
['key1' => true, 'key2' => true]
```

Par ailleurs, vous pouvez supprimer plusieurs éléments de la mémoire cache en une seule fois en faisant correspondre leurs clés à un motif de type "glob". Pour ce faire, utilisez la méthode `deleteMatching` qui renvoie le nombre total d'éléments supprimés.

```php
Cache::deleteMatching('prefix_*'); // supprime tous les éléments dont les clés commencent par "prefix_"
Cache::deleteMatching('*_suffix'); // supprime tous les éléments dont les clés se terminent par "_suffix"
```

> **Attention**  
> Cette méthode n'est mise en œuvre que pour les gestionnaires File et Redis. En raison de limitations, elle n'a pas pu être implémentée pour les gestionnaires Memcached et Wincache.

Vous pouvez nettoyer l'ensemble du cache à l'aide de la méthode `clear`. Si la suppression des fichiers du cache échoue, la méthode renverra `false`.

```php
Cache::clear();
```

> **Attention**  
> L'effacement du cache ne respecte pas le "préfixe" configuré et supprime toutes les entrées du cache. Il convient d'en tenir compte lors de l'effacement d'un cache partagé par d'autres applications.

<a name="le-helper-de-mise-en-cache"></a>
### Le helper de mise en cache

Outre l'utilisation du service ou de la façade `Cache`, vous pouvez également utiliser la fonction globale `cache()` pour récupérer et stocker des données via le cache. Lorsque la fonction `cache` est appelée avec un seul argument de type chaîne, elle renvoie la valeur de la clé donnée :

```php
$value = cache('key');
```

Si vous fournissez à la fonction un tableau de paires clé/valeur et un délai d'expiration, elle stockera les valeurs dans le cache pour la durée spécifiée :

```php
cache(['key' => 'value'], $seconds);
 
cache(['key' => 'value'], now()->addMinutes(10));
```

Lorsque la fonction de cache est appelée sans aucun argument, elle renvoie une instance de `BlitzPHP\Cache\Cache`, ce qui vous permet d'appeler d'autres méthodes de mise en cache :

```php
cache()->remember('users', $seconds, function () {
    return service('database')->table('users')->all();
});
```

<a name="ajout-de-gestionnaires-de-cache-personnalises"></a>
## Ajout de gestionnaire de cache personnalisés

<a name="mise-en-oeuvre-du-gestionnaire"></a>
### Mise en œuvre du gestionnaire

Pour créer notre pilote de cache personnalisé, nous devons d'abord mettre en œuvre le [contrat](/docs/{version}/contrats) `BlitzPHP\Cache\CacheInterface`. Par ailleurs, **votre classe DOIT étendre la classe `BlitzPHP\Cache\Handlers\BaseHandler`**. Ainsi, une implémentation de cache MongoDB pourrait ressembler à ceci :

```php
<?php
 
namespace App\Extensions;

use BlitzPHP\Cache\CacheInterface;
use BlitzPHP\Cache\Handlers\BaseHandler;
 
class MongoCacheHandler extends BaseHandler implements CacheInterface
{
    public function get(string $key, mixed $default = null): mixed;
    public function set(string $key, mixed $value, null|DateInterval|int $ttl = null): bool;
    public function increment(string $key, int $offset = 1);
    public function decrement(string $key, int $offset = 1);
    public function delete(string $key): bool;
    public function clear(): bool;
    public function clearGroup(string $group): bool;
}
```

Il nous suffit de mettre en œuvre chacune de ces méthodes en utilisant une connexion MongoDB. Pour un exemple de mise en œuvre de chacune de ces méthodes, jetez un coup d'œil à BlitzPHP\Cache\Handlers\Memcached dans <a href="https://github.com/blitz-php/cache/blob/main/Handlers/Memcached.php" target="_blank">le code source du framework BlitzPHP</a>.

> **Note**  
> Si vous vous demandez où placer le code de votre pilote de cache personnalisé, vous pouvez créer un espace de noms `Extensions` dans le répertoire de votre application. Cependant, gardez à l'esprit que BlitzPHP n'a pas de structure d'application rigide et que vous êtes libre d'organiser votre application selon vos préférences.

<a name="enregistrement-du-gestionnaire"></a>
### Enregistrement du gestionnaire

Une fois votre gestionnaire implémenté, vous êtes prêt à l'enregistrer auprès de BlitzPHP. Pour ajouter des gestionnaire supplémentaires au backend de cache de BlitzPHP il vous suffit de le rajouter au tableau `valid_handlers` de votre fichier de configuration `app/Config/cache.php`:

```php
<?php

use App\Extensions\MongoCacheHandler;

return [
    // ---
    
    'valid_handlers' => [
        // ---
        
        'mongo' => MongoCacheHandler::class,
    ],
    
    // ---
];
```

Une fois votre gestionnaire de cache enregistré, vous pouvez définir le paramètre `handler` à `mongo` dans votre fichier de configuration `app/Config/cache.php` pour qu'il puisse être utilisé.