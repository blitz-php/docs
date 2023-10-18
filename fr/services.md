---
title: Prestation de services
---

<a name="introduction"></a>
## Introduction

<a name="que-sont-les-services"></a>
### Que sont les services

Chez BlitzPHP, Les services fournissent la fonctionnalité permettant de créer et de partager de nouvelles instances de classe.

La quasi-totalité des classes de base de BlitzPHP sont fournies en tant que « services ». Cela signifie simplement qu'au lieu de coder en dur un nom de classe à charger, les classes à appeler sont définies dans un fichier de configuration très simple. Ce fichier agit comme un type de fabrique pour créer de nouvelles instances de la classe requise.

<a name="pourquoi-utiliser-les-services"></a>
### Pourquoi utiliser les services

Un exemple rapide rendra probablement les choses plus claires, alors imaginez que vous devez extraire une instance de la classe Cache. La méthode la plus simple serait simplement de créer une nouvelle instance de cette classe:

```php
use BlitzPHP\Cache\Cache;

$cache_config = config('cache');

$cache = new Cache($cache_config);
```

Jusqu'ici, tous semble fonctionner très bien. Jusqu'à ce que vous décidiez d'utiliser une autre classe de gestion de cache à sa place. Peut-être que celui-ci a des rapports avancés que le gestionnaire par défaut ne fournit pas. Pour ce faire, vous devez maintenant localiser tous les emplacements de votre application dans lesquels vous avez utilisé la classe `Cache`. Étant donné que vous les avez peut-être mis un peu de partout pour optimiser les performances de votre application constamment en cours d'exécution, cela peut être un moyen long et sujet aux erreurs de gérer cela. C'est là que les services sont utiles.

Au lieu de créer l'instance nous-mêmes, nous laissons une classe centrale créer une instance de la classe pour nous. Cette classe est très simple. Elle ne contient qu'une méthode pour chaque classe que nous voulons utiliser en tant que service. La méthode retourne généralement une instance partagée de cette classe, en lui passant toutes les dépendances qu'elle pourrait avoir. Ensuite, nous remplacerions notre code de création de cache par du code qui appelle cette nouvelle classe:

```php
use BlitzPHP\Container\Services;

$cache = Service::cache();
```

Lorsque vous devez [modifier l'implémentation utilisée](#definition-de-services), vous pouvez modifier le fichier de configuration des services, et la modification se produit automatiquement dans toute votre application sans que vous ayez à faire quoi que ce soit. Il ne vous reste plus qu'à profiter de toute nouvelle fonctionnalité. Très simple et résistant aux erreurs.

> **Note**  
Il est recommandé de créer uniquement des services dans les contrôleurs. Les autres fichiers, comme les modèles et les bibliothèques, [doivent avoir les dépendances](/docs/{version}/conteneur) passées au constructeur ou via une méthode setter.

<a name="comment-obtenir-un-service"></a>
## Comment obtenir un service

Comme de nombreuses classes BlitzPHP sont fournies en tant que services, vous pouvez les obtenir comme suit :

```php
<?php

$cache = \App\Config\Services::cache();
```
`$cache` est une instance de la classe Cache, et si vous appelez à nouveau `\App\Config\Services::cache()`, vous obtiendrez exactement la même instance.

Les services renvoient généralement une **instance partagée** de la classe. Le code suivant crée une instance [HttpClient](/docs/{version}/client-http) au premier appel. Et le deuxième appel renvoie exactement la même instance.

```php
<?php

$options1 = [
    'baseURI' => 'http://example.com/api/v1/',
    'timeout' => 3,
];
$client1 = \App\Config\Services::httpclient($options1);

$options2 = [
    'baseURI' => 'http://another.example.com/api/v2/',
    'timeout' => 10,
];
$client2 = \App\Config\Services::httpclient($options2);

// $options2 ne marche pas.
// $client2 est exactement la meme instance que $client1.
```
Par conséquent, le paramètre `$options2` pour `$client2` ne fonctionne pas. C'est tout simplement ignoré.

<a name="obtenir-une-nouvelle-instance"></a>
### Obtenir une nouvelle instance

Si vous souhaitez obtenir une nouvelle instance de la classe Cache, vous devez passer `false` à l'argument `$shared` :

```php
<?php

$cache = \App\Config\Services::cache(shared: false);
// ou
$cache = \App\Config\Services::cache(null, false);
```

<a name="fonctions-pratiques"></a>
### Fonctions pratiques

Deux fonctions ont été fournies pour obtenir un service. Ces fonctions sont toujours disponibles.

<a name="service"></a>
#### service()

La première est la fonction `service()` qui renvoie une nouvelle instance du service demandé. Le seul paramètre requis est le nom du service à récupérer. Ce nom doit être le nom d'une méthode définie dans [une classe de Service](#etendre-les-services) ou dans le fichier de configurations des [providers](#configurer-les-providers).

> **Note**  
> La fonction `service()` renvoie toujours une instance partagée de la classe. Donc, en appelant cette fonction plusieurs, vous obtiendrez toujours la même instance.

```php
<?php

$logger = service('logger');

// Equivalent à
$logger = \App\Config\Services::logger();
```

Si la méthode de création nécessite des paramètres supplémentaires, ils peuvent être passés après le nom du service:

```php
$translator = service('translator', 'en-US');

// Equivalent à
$translator = \App\Config\Services::translator('en-US');
```

<a name="single_service"></a>
#### single_service()

La deuxième fonction, `single_service()`, fonctionne comme `service()` mais renvoie une nouvelle instance de la classe:

```php
<?php

$logger = single_service('logger');

// Equivalent à
$logger = \App\Config\Services::logger(false);
```  
  
<a name="definition-de-services"></a>
## Définition de services

Pour que les services fonctionnent correctement, vous devez pouvoir compter sur chaque classe disposant d'une API ou d'une interface constante à utiliser. Presque toutes les classes de BlitzPHP fournissent une interface à laquelle elles adhèrent. Lorsque vous souhaitez étendre ou remplacer des classes principales, il vous suffit de vous assurer que vous répondez aux exigences de l'interface et que vous savez que les classes sont compatibles.

Par exemple, la classe `Logger` implémente l'interface `Psr\Log\LoggerInterface`. Lorsque vous souhaitez créer un remplacement offrant une manière différente de gérer les logs, il vous suffit de créer une nouvelle classe qui implémente aussi l'interface `Psr\Log\LoggerInterface`:

```php
<?php

namespace App\Debug;

use Psr\Log\LoggerInterface;

class MyLogger implements LoggerInterface
{
    // Implementez les méthodes requises ici.
}
```

Enfin, ajoutez la méthode `logger()` à **app/Config/Services.php** pour créer une nouvelle instance de `MyLogger` au lieu de `BlitzPHP\Debug\Logger` :

```php
<?php

namespace App\Config;

use BlitzPHP\Container\Services as BaseServices;
use Psr\Log\LoggerInterface;

class Services extends BaseServices
{
    // ...

    public static function logger(): LoggerInterface
    {
        return new \App\Debug\Logger();
    }
}
```

<a name="autoriser-les-parametres"></a>
### Autoriser les paramètres

Dans certains cas, vous souhaiterez avoir la possibilité de transmettre un paramètre à la classe lors de l'instanciation. Puisque le fichier services est une classe très simple, il est facile de faire fonctionner cela.

Un bon exemple est le service de traduction. Par défaut, nous voulons que cette classe puisse traduire en espagnol.

```php
<?php

namespace App\Config;

use BlitzPHP\Container\Services as BaseServices;
use BlitzPHP\Translator\Translate;

class Services extends BaseServices
{
    // ...

    public static function translator(string $locale = 'es')
    {
        return new Translate($locale, static::locator());
    }
}
```

Cela définit la locale par défaut dans le constructeur, mais permet de modifier facilement la locale qu'elle utilise :

```php
<?php

$translator = \App\Config\Services::translator('fr');
```

<a name="instances-partagees"></a>
### Instances partagées

Il peut arriver que vous deviez exiger qu'une seule instance d'un service soit créée. Ceci est facilement géré avec la méthode `singleton()`. Cela gère la vérification si une instance a été créée et enregistrée dans la classe et, sinon, en crée une nouvelle. Toutes les méthodes de services fournissent une valeur `$shared = true` comme dernier paramètre. Vous devez également vous en tenir à la méthode :

```php
<?php

namespace App\Config;

use BlitzPHP\Container\Services as BaseServices;
use Psr\Log\LoggerInterface;

class Services extends BaseServices
{
    // ...

    public static function logger(bool $shared = true): LoggerInterface
    {
        if ($shared) {
            return static::singleton(\App\Debug\Logger::class):
        }
        return new \App\Debug\Logger();
    }
}
```

<a name="decouverte-de-services"></a>
## Découverte de services

BlitzPHP peut découvrir automatiquement tous les fichiers `Config/Services.php` que vous avez créés dans n’importe quel namespace défini. Cela permet une utilisation simple de tous les fichiers de services de module. Pour que les fichiers de services personnalisés soient découverts, ils doivent Répondre aux exigences suivantes :
* Son namespace doit être accessible via Composer (pour des package tiers) ou défini dans `app/Config/Autoload.php`
* À l’intérieur du namespace, le fichier doit se trouver dans `Config/Services.php`
* Il doit étendre `BlitzPHP\Container\Services`

Un petit exemple devrait clarifier cela.

Imaginez que vous avez créé un nouveau répertoire, **Blog** dans le répertoire racine de votre projet. Celui-ci tiendra un [module Blog](/docs/{version}/programmation-modulaire) avec des contrôleurs, modèles, etc., et vous souhaitez rendre certaines des classes disponibles en tant que service. La première étape consiste à créer un nouveau fichier : `Blog/Config/Services.php`. Le squelette du fichier doit être :

```php
<?php

namespace Blog\Config;

use BlitzPHP\Container\Services as BaseServices;

class Services extends BaseServices
{
    public static function postManager()
    {
        // ...
    }
}
```

Vous pouvez maintenant utiliser ce fichier comme décrit ci-dessus. Lorsque vous souhaitez récupérer le service de publications de n’importe quel contrôleur, vous utiliserait simplement la classe du framework pour récupérer votre service: `App\Config\Services`

```php
<?php

$postManager = App\Config\Services::postManager();
// ou
$postManager = service('postManager');
```

> **Note**  
> Si plusieurs fichiers de services ont le même nom de méthode, le premier trouvé sera l’instance renvoyée.