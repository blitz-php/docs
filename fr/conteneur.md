---
title: Injection de dépendances
---

<a name="introduction"></a>
## Introduction

L'injection de dépendances (dependency injection en anglais) est un mécanisme qui permet d'implémenter le principe de l'inversion de contrôle. Il consiste à créer dynamiquement (injecter) les dépendances entre les différents objets en s'appuyant sur une description (fichier de configuration ou métadonnées) ou de manière programmatique. Ainsi les dépendances entre composants logiciels ne sont plus exprimées dans le code de manière statique mais déterminées dynamiquement à l'exécution.

BlitzPHP utilise le conteneur de [PHP-DI](https://php-di.org/) pour vous permettre de gérer les dépendances de classes de vos services applicatifs par l’injection de dépendance. L’injection de dépendance « injecte » automatiquement les dépendances d’un objet dans son constructeur, sans qu’il soit besoin de les instancier manuellement.

BlitzPHP utilisera le conteneur de services lors de l’appel d’actions dans vos contrôleurs et l’invocation de commandes dans la console. Vous pouvez aussi avoir des dépendances qui soient injectées dans les constructeurs de vos contrôleurs.

<a name="concept"></a>
## Concept

L'injection de dépendance est une expression fantaisiste qui signifie essentiellement ceci : *les dépendances de classe sont "injectées" dans la classe via le constructeur ou, dans certains cas, des méthodes "setter"*.

Regardons un exemple simple :

```php
<?php
namespace App\Controllers;

use App\Repositories\UserRepository;

class UserController extends AppController
{
    /**
     * Constructeur.
     *
     * @param  UserRepository $userRepository
     */
    public function __construct(protected UserRepository $users)
    {
    }

    /**
     * Profile d'un utilisateur.
     *
     * @param  int  $id
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return $this->view('profile')->with('user', $user);
    }
}
```

Dans cet exemple, la classe `UserController` a besoin de récupérer des utilisateurs à partir d'une source de données. Nous allons donc **injecter** un service capable de récupérer les utilisateurs. Dans ce contexte, notre UserRepository utilise probablement l'ORM pour récupérer les informations utilisateur de la base de données. Cependant, comme le service est injecté, nous pouvons facilement l'échanger avec une autre implémentation. Nous sommes également capables de facilement « simuler » ou de créer une implémentation fictive de UserRepository lors du [test de notre application](/docs/{version}/tests).

Une compréhension approfondie du conteneur de service BlitzPHP est essentielle pour construire une application puissante et volumineuse, ainsi que pour contribuer au noyau BlitzPHP lui-même.

<a name="resolution-de-configuration-nulle"></a>
### Résolution de configuration nulle

Si une classe n’a pas de dépendances ou ne dépend que d’autres classes concrètes (pas d’interfaces), le conteneur n’a pas besoin d’être instruit sur la façon de résoudre cette classe. Par exemple, vous pouvez placer le code suivant dans votre fichier `app/Config/routes.php`

```php
<?php
 
class Service
{
    // ...
}
 
$routes->get('/', function (Service $service) {
    die(get_class($service));
});
```

Dans cet exemple, l’accès à la route `/` de votre application résoudra automatiquement la classe `Service` et l’injectera dans le gestionnaire de votre gamme. Cela change la donne et signifie que vous pouvez développer votre application et tirer parti de l’injection de dépendances sans vous soucier des fichiers de configuration gonflés.

La plupart des classes que vous écrirez lors de la création d'une application BlitzPHP reçoivent automatiquement leurs dépendances via le conteneur, notamment [les contrôleurs](/docs/{version}/controleurs), [les écouteurs d'événements](/docs/{version}/evenements), [les middlewares](/docs/{version}/middlewares), etc. Une fois que vous avez goûté à la puissance de l’injection automatique de dépendances sans configuration, il semble impossible de se développer sans elle.

<a name="quand-utiliser-le-conteneur"></a>
### Quand utiliser le conteneur

Grâce à une résolution de configuration nulle, vous typerez souvent des dépendances sur routes, les contrôleurs, les écouteurs d'événements et ailleurs sans jamais interagir manuellement avec le conteneur. Par exemple, vous pouvez saisir l'objet `BlitzPHP\Http\Request` sur votre définition de route afin de pouvoir accéder facilement à la requête en cours. Même si nous n'avons jamais besoin d'interagir avec le conteneur pour écrire ce code, il gère l'injection de ces dépendances en coulisses :

```php
<?php 

use BlitzPHP\Http\Request;
 
$routes->get('/', function (Request $request) {
    // ...
});
```

Dans de nombreux cas, grâce à l'injection automatique de dépendances et aux [façades](/docs/{version}/facades), vous pouvez créer des applications BlitzPHP sans jamais lier ou résoudre manuellement quoi que ce soit à partir du conteneur. **Alors, quand interagiriez-vous manuellement avec le conteneur** ? Examinons deux situations.

Premièrement, si vous écrivez une classe qui implémente une interface et que vous souhaitez indiquer cette interface sur un constructeur de route ou de classe, vous devez [indiquer au conteneur comment résoudre cette interface](#liaison-d-interfaces-aux-implementations). Deuxièmement, si vous écrivez un package BlitzPHP que vous envisagez de partager avec d'autres développeurs BlitzPHP, vous devrez peut-être lier les services de votre package dans le conteneur.

<a name="definir-les-dependances"></a>
## Définir les dépendances

Jusqu'ici, BlitzPHP s'est débrouillé tant bien que mal à résoudre et à injecter vos dépendances. Cependant, si vous avez des dépendances complexes (comprenant des valeurs statiques non résolvable) ou abstraites (interfaces), il faudra indiquer à BlitzPHP comment résoudre les choses.

Les définitions de dépendances du conteneur peuvent se faire dans tous fichiers se trouvant dans `app/Providers`. La classe créée doit étendre la classe `BlitzPHP\Container\AbstractProvider`

<a name="liaisons-simples"></a>
### Liaisons simples

Au sein de votre classe vous avez accès à l'instance courante du conteneur via la propriété `container`. Nous pouvons donc enregistrer une liaison à l’aide de la méthode `add()`, en passant le nom de classe ou d’interface que nous souhaitons enregistrer avec une closure qui retourne une instance de la classe. Ceci se fait à l'intérieur de la méthode `register` comme le montre l'exemple suivant:

```php
<?php 

namespace App\Providers;

use App\Services\Transistor;
use App\Services\PodcastParser;
use BlitzPHP\Container\AbstractProvider;
use BlitzPHP\Container\Container;

class AppProvider extends AbstractProvider
{
    public function register(): void
    {
        $this->container->add(Transistor::class, function (Container $app) {
            return new Transistor($app->make(PodcastParser::class));
        });       
    }
}
```

Notez que nous recevons le conteneur lui-même comme argument du résolveur. Nous pouvons ensuite utiliser le conteneur pour résoudre les sous-dépendances de l’objet que nous construisons.

> **Note**  
> Le résolveur (deuxième argument de la méthode `add`) ne reçoit pas obligatoirement le Container comme argument. Vous pouvez injecter n'importe quoi d'autre. 
<br>
```php
$this->container->add(Axios::class, function (Request $request) {
    return new Axios($request);
});
```

> **Note**  
> La définition des valeurs primitives (statiques) se fait directement avec la méthode `set`) 
<br>
```php
$this->container->set('database.host', 'localhost');
$this->container->set('report.recipient', [
    'bob@example.com',
    'alice@example.com',
]);
```

En principe, vous interagirez généralement avec le conteneur à l'intérieur d'une classe `Provider`. Toutefois, si vous souhaitez interagir avec le conteneur à l’extérieur de cette fonction, vous pouvez le faire via [le service](/docs/{version}/services) `container` ou [la façade](/docs/{version}/facades) `Container`:

```php
use App\Services\Transistor;

// Via la facade

use BlitzPHP\Facades\Container;
 
Container::add(Transistor::class, function () {
    // ...
});

// Via le service

service('container')->add(Transistor::class, function () {
    // ...
});
```

Vous pouvez utiliser la méthode `addIf` pour enregistrer une liaison de conteneur uniquement si aucune liaison n’a pas déjà été enregistrée pour le type donné:

```php
$this->container->addIf(Transistor::class, function (Container $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

> **Note**  
> Il n’est pas nécessaire de lier des classes dans le conteneur si elles ne dépendent d’aucune interface. Le conteneur n’a pas besoin d’être instruit sur la façon de construire ces objets, car il peut résoudre automatiquement ces objets à l’aide de la réflexion.

<a name="liaison-d-interfaces-aux-implementations"></a>
### Liaison d’interfaces aux implémentations

Une fonctionnalité très puissante du conteneur de services est sa capacité à lier une interface à une implémentation donnée. Par exemple, supposons que nous ayons une interface `EventPusher` et une implémentation `RedisEventPusher`. Une fois que nous avons codé notre implémentation `RedisEventPusher` de cette interface, nous pouvons l’enregistrer avec le conteneur de service comme ceci:

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;
 
$this->container->set(EventPusher::class, RedisEventPusher::class);
```

Cette instruction indique au conteneur qu'il doit injecter `RedisEventPusher` lorsqu'une classe a besoin d'une implémentation de `EventPusher`. Nous pouvons maintenant typer un indice sur l'interface `EventPusher` dans le constructeur d'une classe résolue par le conteneur. N'oubliez pas que les contrôleurs, les écouteurs d'événements, les middlewares et divers autres types de classes au sein des applications BlitzPHP sont toujours résolus à l'aide du conteneur :

```php
use App\Contracts\EventPusher;
 
/**
 * Constructeur.
 */
public function __construct(protected EventPusher $pusher) 
{
}
```

<a name="liaison-par-definitions"></a>
### Liaison par définitions

Jusqu'ici, vous ajoutez des éléments au conteneur via la méthode `register` de votre classe Provider en utilisant les méthode `add` et `set` du conteneur. **Cette façon de faire est DECONSEILLEE car elle ne permet pas la compilation du conteneur.** Ainsi, toutes les dépendances liées par cette approche dite "**dynamique**" ne seront pas compilées et impactera donc sur les performances de votre application.

> **Note**  
> Pour en savoir plus sur la compilation du conteneur et les performance, nous vous invitons à lire le [guide de perfomances de PHP-DI](https://php-di.org/doc/performances.html#compiling-the-container) et leur [mise en garde concernant la définition dynamique des éléments du conteneur](https://php-di.org/doc/php-definitions.html#setting-definitions-in-the-container-directly)

Vous pouvez lier les éléments dans le conteneur en utilisant les définitions PHP. Les définitions PHP de BlitzPHP fonctionnent exactement comme [celles de PHP-DI](https://php-di.org/doc/php-definitions.html). Ces définitions se font à travers la méthode statique `definitions` de votre classe Provider. Ci-dessous, un exemple de définition.

```php
<?php 

namespace App\Providers;

use BlitzPHP\Container\AbstractProvider;

class AppProvider extends AbstractProvider
{
    public function definitions(): array
    {
        return [
             'LoggerInterface' => \DI\create('MyLogger'),
             'Foo' => function (LoggerInterface $logger) {
                return new Foo($logger);
            },
        ];       
    }
}
```

> **Note**  
> La liaison par définition est FORTEMENT RECOMMANDEE par BlitzPHP car permet de meilleures performances en production. Lire le [guide de perfomances de PHP-DI](https://php-di.org/doc/performances.html#compiling-the-container)

<a name="ordre-de-priorite-des-liaisons"></a>
### Ordre de priorité des liaisons

Le conteneur de BlitzPHP utilise PHP-DI sous le capot et a activé l'[autowiring](https://php-di.org/doc/autowiring.html) et l'utilisation des [attributs PHP](https://php-di.org/doc/attributes.html). Ainsi, il est important que vous sachiez le niveau de priorité des injections.

De la priorité la plus basse à la plus élevée, nous avons :
* Le câblage automatique(autowiring). cf [résolution de configuration nulle](#resolution-de-configuration-nulle)
* Les attributs
* Les définitions PHP dans l’ordre dans lequel elles ont été ajoutées (les définitions de l'application `App` sont prioritaires par rapports à celles de BlitzPHP, qui à leur tour sont prioritaires par rapports à d'autres packages installés via Composer)
* Les définitions ajoutées directement dans le conteneur avec `$container->set()` ou `$container->add()`


<a name="decouverte-de-providers"></a>
## Découverte de providers

BlitzPHP peut découvrir automatiquement tous les fichiers `Providers/**` que vous avez créés dans n’importe quel namespace défini. Cela permet une utilisation simple de tous les providers de module. Pour qu'un provider personnalisé soit découvert, il doit Répondre aux exigences suivantes :
* Son namespace doit être accessible via Composer (pour des package tiers) ou défini dans `app/Config/Autoload.php`
* À l’intérieur du namespace, le fichier doit se trouver dans le dossier `{namespace}/Providers`
* Il doit étendre `BlitzPHP\Container\AbstractProvider`

Un petit exemple devrait clarifier cela.

Imaginez que vous avez créé un nouveau répertoire, **Blog** dans le répertoire racine de votre projet. Celui-ci tiendra un [module Blog](/docs/{version}/programmation-modulaire) avec des contrôleurs, modèles, etc., et vous souhaitez définir certaines classes dans le conteneur. La première étape consiste à créer un nouveau fichier : `Blog/Providers/BlogProvider.php` ou `Blog/Providers/ProviderA.php`, `Blog/Providers/ProviderB.php` (si vous prévoyez avoir plusieurs Providers). Le squelette du fichier doit être :

```php
<?php

namespace Blog\Providers;

use BlitzPHP\Container\AbstractProvider;

class Providers extends AbstractProvider
{
    public static function definitions(): array
    {
        // ...
    }
    
    public function register(): void
    {
        // ...
    }
}
```

Vous pouvez maintenant utiliser ce fichier comme décrit ci-dessus. 

<a name="interagir-avec-le-conteneur"></a>
## Interagir avec le conteneur

<a name="la-methode-make"></a>
### La méthode `make`

Vous pouvez utiliser la méthode `make` pour résoudre une instance de classe à partir du conteneur. La méthode `make` accepte le nom de la classe ou de l'interface que vous souhaitez résoudre :

```php
<?php 
use App\Services\Transistor;
 
$transistor = $this->container->make(Transistor::class);
```

Si certaines dépendances de votre classe ne peuvent pas être résolues via le conteneur, vous pouvez les injecter en les passant sous forme de tableau associatif en utilisant le deuxième argument de la méthode `make`. Par exemple, nous pouvons transmettre manuellement l'argument `$id` du constructeur requis par le service `Transistor` :

```php
<?php 
use App\Services\Transistor;
 
$transistor = $this->container->make(Transistor::class, ['id' => 1]);
```

La méthode `make()` fonctionne comme `get()` sauf qu'elle résoudra l'entrée à chaque fois qu'elle est appelée. Selon le type d'entrée, cela signifie :
* si l'entrée est un objet, une nouvelle instance sera créée à chaque fois 
* si l'entrée est une factory, la factory sera appelée à chaque fois 
* si l'entrée est un alias, l'alias sera résolu à chaque fois

Veuillez noter que seule l'entrée que vous demandez sera résolue à chaque fois : toutes les dépendances de l'entrée ne le seront pas ! Cela signifie que si l’entrée est un alias, l’entrée vers laquelle pointe l’alias *ne sera résolue qu’une seule fois*.

Si vous fournissez des paramètres à la méthode `make()` dans son deuxième argument, et si l'entrée à résoudre est un objet à créer, les paramètres fournis seront utilisés pour le constructeur de l'objet, et les paramètres manquants seront résolus à partir du conteneur.

> **Note**  
> La méthode `make()` est utile pour créer des objets qui ne doivent pas être stockés à l'intérieur du conteneur (c'est-à-dire qui ne sont pas des services ou qui ne sont pas [sans état](https://igor.io/2013/03/31/stateless-services.html)), mais qui ont des dépendances. C'est également utile si vous souhaitez remplacer certains paramètres du constructeur d'un objet.    
<br>
> Si vous devez utiliser la méthode `make()` dans un service, un contrôleur ou autre, il est recommandé de d'injecter l'interface `\DI\FactoryInterface`. Cela évite de coupler votre code au conteneur. `DI\FactoryInterface` est automatiquement lié au conteneur afin que vous puissiez l'injecter sans aucune configuration

Si vous ne n'êtes pas dans une classe Provider et qu'à  un moment de votre code vous n'avez pas accès à la variable `$container`, vous pouvez utiliser [la façade](/docs/{version}/facades) `Container` ou [le service](/docs/{version}/services) `container` pour résoudre une instance de classe à partir du conteneur :

```php
<?php 

use App\Services\Transistor;
use BlitzPHP\Facades\Container;
 
$transistor = Container::make(Transistor::class);
 
$transistor = service('container')->make(Transistor::class);

$transistor = service(Transistor::class);
```

Si vous souhaitez que l'instance du conteneur BlitzPHP elle-même soit injectée dans une classe en cours de résolution par le conteneur, vous pouvez typer un paramètre du constructeur de votre classe sur `\BlitzPHP\Container\Container` :

```php
<?php 

use BlitzPHP\Container\Container;
 
/**
 * Constructeur.
 */
public function __construct(protected Container $container) 
{
}
```

<a name="bound-et-injectOn"></a>
### `bound` et `injectOn`

La méthode `bound` peut être utilisée pour déterminer si une classe ou une interface a été explicitement liée dans le conteneur :

```php
<?php 

if ($this->container->bound(Transistor::class)) {
    // ...
}
```

Parfois, vous souhaitez injecter des dépendances sur un objet déjà créé.  
  
Par exemple, certains package ne vous permettent pas de contrôler la façon dont les classes sont créés. Avec `injectOn`, vous pouvez demander au conteneur de remplir les dépendances après la création de l'objet.


```php
<?php 

class UserService extends \Some\Package\Service
{
    #[Inject]
    private SomeService $someService;

    public function __construct()
    {
        // Le package ne nous permet pas de contrôler la façon dont le service est créé, donc
         // nous ne pouvons pas utiliser le conteneur pour créer ce service
         // On demande donc au conteneur d'injecter des dépendances
        service('container')->injectOn($this);

        // Maintenant les dépendances sont injectées
        $this->someService->doSomething();
    }
}
```

Comme vous l'avez peut-être deviné, vous ne pouvez pas utiliser l'injection de constructeur avec cette méthode. Mais d'autres types d'injections (propriété ou setter) fonctionneront, que vous utilisiez des attributs ou que vous ayez configuré votre objet dans [les définitions](#liaison-par-definitions).

> **Note**  
> Gardez à l’esprit qu’il est généralement toujours préférable d’utiliser `get()` ou `make()` au lieu de `injectOn()`, utilisez-le uniquement là où vous en avez vraiment besoin.

<a name="invocation-de-methode-et-injection"></a>
### Invocation de méthode et injection

Parfois, vous souhaiterez peut-être invoquer une méthode sur une instance d'objet tout en permettant au conteneur d'injecter automatiquement les dépendances de cette méthode. Par exemple, étant donné la classe suivante :

```php
<?php
 
namespace App;
 
use App\Repositories\UserRepository;
 
class UserReport
{
    /**
     * Génère un nouveau rapport d'utilisateur.
     */
    public function generate(UserRepository $repository): array
    {
        return [
            // ...
        ];
    }
}
```

Vous pouvez appeler la méthode `generate` via le conteneur comme ceci 

```php
use App\UserReport;
use BlitzPHP\Facades\Container;
 
$report = Container::call([new UserReport, 'generate']);
```

La méthode `call` accepte tout *callable* PHP. La méthode `call` du conteneur peut même être utilisée pour invoquer une closure tout en injectant automatiquement ses dépendances :

```php
use App\Repositories\UserRepository;
use BlitzPHP\Facades\Container;
 
$result = Container::call(function (UserRepository $repository) {
    // ...
});
```

<a name="compatibilite-psr-11"></a>
### Compatibilité PSR 11

Le conteneur de services de BlitzPHP implémente l'interface [PSR-11](http://www.php-fig.org/psr/psr-11/). Par conséquent, vous pouvez type un paramètre sur l'interface du conteneur PSR-11 (`Psr\Container\ContainerInterface`) pour obtenir une instance du conteneur BlitzPHP :

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;
 
$routes->get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);
 
    // ...
});
```

Une exception est levée si l'identifiant donné ne peut pas être résolu. L'exception sera une instance de `Psr\Container\NotFoundExceptionInterface` si l'identifiant n'a jamais été lié. Si l'identifiant était lié mais n'a pas pu être résolu, une instance de `Psr\Container\ContainerExceptionInterface` sera levée.

<a name="les-alias-de-classes"></a>
## Les alias de classes

<a name="definition-des-alias"></a>
### Définition des alias

En plus de l'injection de dépendances et [les services](/docs/{version}/services), Une autre fonctionnalité offerte par BlitzPHP est la possibilité pour vous de définir des alias à toutes les classes de votre projet (que ce soit vos propres classes ou les classes du framework).

Ceci vous permet notamment d'éviter d'écrire le nom de classe long dans votre code métier ou l'utilisation du mot clé `use`. Cette fonctionnalité est particulièrement utile au niveau [des vues](/docs/{version}/vues).

Supposons par exemple que vous souhaitez vérifier qu'une chaîne de caractère commence par un mot précis. Pour ce faire, vous utiliserez la méthode `\BlitzPHP\Utilities\String\Text::startsWith` comme ceci

```php
<?php 
// Verifie si la texte contenu dans variable $string commence par "test"
if (\BlitzPHP\Utilities\String\Text::startsWith($string, 'test')) {
    // Faire le traitement approprié
}
```

Le problème est que vous écrirez un nom de classe long et si vous devez faire ça à plusieurs endroit ça deviendra vite fastidieux. Certes vous pouvez opter pour l'utilisation du mot clé `use` pour pallier à cela mais au niveau [des fichiers de vues](/docs/{version}/vues), ce n'est pas très agréable surtout si vous avez plusieurs classes à utiliser.

BlitzPHP vous permet de créer un fichier `/app/Config/aliases.php` et définir un tableau associatif contenant vos raccourcis. Un exemple serait alors

```php
<?php 

return [
    'str' => \BlitzPHP\Utilities\String\Text::class,
];
```

Le fichier crée doit retourné un tableau associatif ayant en clé l'alias que l'on souhaite, et en valeur le nom complet de la classe que l'on veut raccourcir. Ainsi, le traitement précédent pourrait être réduit à ceci:

```php
<?php 

// Verifie si la texte contenu dans variable $string commence par "test"
if (str::startsWith($string, 'test')) {
    // Faire le traitement approprié
}
```

Comme la classe `str` n'existe pas rééllement, BlitzPHP chargera la classe `\BlitzPHP\Utilities\String\Text` à la place. **Vous DEVEZ alors RESPECTER LA CASSE de l'alias**.

La définition des alias est valable pour tout type de classe. Que ce soit vos classes, les classes de BlitzPHP ou celles installées via Composer, vous pouvez définir des alias à condition expresse de bien renseigné son nom complet.

```php
<?php 

// Définitions des alias de classes

return [
    // Classes applicatif
    'Table' => '\App\Dto\Factories\Table',
    'Payment' => \App\Services\PaiementService::class,
	
    // Classes système
    'Date' => \BlitzPHP\Utilities\Date::class,
	
    // Dépendances
    'Comparator' => \Symfony\Component\Finder\Comparator::class
];
```


Si vous avez [définir les dépendances](#definir-les-dependances), vous pouvez directement utiliser vos classes Provider pour définir vos alias sans avoir à créer un fichier `app/Config/aliases.php`. Ajouter juste une méthode statique `aliases` à votre classe et BlitzPHP fera le reste.

```php
<?php

namespace App\Providers;

use BlitzPHP\Container\AbstractProvider;

class Providers extends AbstractProvider
{
    public static function aliases(): array
    {
        return [
            // Classes applicatif
            'Table' => '\App\Dto\Factories\Table',
            'Payment' => \App\Services\PaiementService::class,
	
            // Classes système
            'Date' => \BlitzPHP\Utilities\Date::class,
	
            // Dépendances
            'Comparator' => \Symfony\Component\Finder\Comparator::class
        ];
    }
    
    // ...
}
```

> **Note**  
> Si vous utilisez simultanément le fichier `app/Config/aliases.php` et une classe Provider pour définir vos alias, les alias de la classe Provider seront remplacés par ceux du fichier de configuration ayant la même clé.

<a name="decouverte-des-alias"></a>
### Découverte des alias

BlitzPHP peut découvrir automatiquement tous les fichiers Config/aliases.php que vous avez créés dans n’importe quel namespace défini. Cela permet une utilisation simple de tous les fichiers de définition d'alias de module. Pour que les fichiers d'alias soient découverts, ils doivent Répondre aux exigences suivantes :
* Son namespace doit être accessible via Composer (pour des package tiers) ou défini dans app/Config/Autoload.php
* À l’intérieur du namespace, le fichier doit être  `Config/aliases.php`.

Les alias sont enregistrés suivant un ordre de priorité bien établi. De la priorité la plus élevée à la moins importante, nous avons :

1. `app/Config/aliases.php` - Les alias de l'application remplaceront tous les autres s'ils ont les mêmes clés.
2. `{namespace}/Config/aliases.php` - Tous les namespaces sont parcourus en boucle dans l'ordre dans lequel ils sont définis.
3. `\App\Providers\**::aliases()` - Les alias définis dans les classes Provider de l'application seront écrasés par ceux des fichiers de configuration de l'application ou des packages tiers.
4. `{namespace}\Providers\**::aliases()` - Les alias définis dans les classes Provider des packages tiers ou des sous modules seront écrasés par tous les autres.