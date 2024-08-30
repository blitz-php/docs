---
title: Les contrats
---

<a name="introduction"></a>
## Introduction

Les "contrats" sont un ensemble d'interfaces qui définissent les services de base fournis par le framework. Par exemple, un contrat `BlitzPHP\Contracts\Database\ConnectionInterface` définit les méthodes de manipulation de connexion à une base de données, tandis que le contrat `BlitzPHP\Contracts\Mail\MailerInterface` définit les méthodes nécessaires à l'envoi d'e-mails.

Chaque contrat a une implémentation correspondante fournie par le framework. Par exemple, BlitzPHP fournit une implémentation de connexion à la base de données avec une variété de pilotes, et une implémentation de mailer qui est alimentée principalement par <a href="https://github.com/PHPMailer/PHPMailer" target="_blank">PHPMailer</a>.

Tous les contrats BlitzPHP sont dans <a href="https://github.com/blitz-php/contracts" target="_blank">leur propre dépôt GitHub</a>. Cela fournit un point de référence rapide pour tous les contrats disponibles, ainsi qu'un paquetage unique et découplé qui peut être utilisé lors de la construction de paquetages qui interagissent avec les services BlitzPHP.

<a name="contrats-vs-facades"></a>
## Contrats vs. Façades

[Les façades](/docs/{version}/facades) et les helpers de BlitzPHP fournissent un moyen simple d'utiliser les fonctionnalités de BlitzPHP sans avoir besoin de typer et de résoudre des contrats en dehors du conteneur de service. Dans la plupart des cas, chaque façade a un contrat équivalent.

Contrairement aux façades, qui ne nécessitent pas d'être requises dans le constructeur de votre classe, les contrats vous permettent de définir des dépendances explicites pour vos classes. Certains développeurs préfèrent définir explicitement leurs dépendances de cette manière et préfèrent donc utiliser les contrats, tandis que d'autres développeurs apprécient la commodité des façades. **En général, la plupart des applications peuvent utiliser les façades sans problème pendant le développement**.

<a name="quand-utiliser-les-contrats"></a>
## Quand utiliser les contrats

La décision d'utiliser des contrats ou des façades dépendra de vos goûts personnels et de ceux de votre équipe de développement. Les contrats et les façades peuvent tous deux être utilisés pour créer des applications BlitzPHP robustes et bien testées. Les contrats et les façades ne s'excluent pas mutuellement. Certaines parties de vos applications peuvent utiliser des façades tandis que d'autres dépendent des contrats. Tant que vous gardez les responsabilités de votre classe concentrées, vous remarquerez très peu de différences pratiques entre l'utilisation des contrats et des façades.

En général, la plupart des applications peuvent utiliser les façades sans problème pendant le développement. Si vous construisez un paquetage qui s'intègre avec plusieurs frameworks PHP, vous pouvez utiliser le paquetage `blitz-php/contracts` pour définir votre intégration avec les fonctionnalités de BlitzPHP sans avoir besoin de requérir les implémentations concrètes de BlitzPHP dans le fichier `composer.json` de votre paquetage.

<a name="comment-utiliser-les-contrats"></a>
## Comment utiliser les contrats

Alors, comment obtenir implémentation d'un contrat ? C'est en fait assez simple.

De nombreux types de classes dans BlitzPHP sont résolus par [le conteneur](/docs/{version}/conteneur), y compris les contrôleurs, les événements, les middlewares et même les closures de routes. Ainsi, pour obtenir une implémentation d'un contrat, vous pouvez simplement "indiquer le type" de l'interface dans le constructeur de la classe en cours de résolution.

Par exemple, regardez cet écouteur d'événements :

```php
<?php

namespace BlitzPHP\Wolke\Events;

use BlitzPHP\Contracts\Database\ConnectionResolverInterface;
use BlitzPHP\Contracts\Event\EventListenerInterface;
use BlitzPHP\Contracts\Event\EventManagerInterface;
use BlitzPHP\Database\Connection\BaseConnection;
use BlitzPHP\Utilities\Iterable\Arr;
use BlitzPHP\Wolke\Model;
use BlitzPHP\Wolke\Pagination\AbstractPaginator;
use Psr\Http\Message\ServerRequestInterface;

class Listener implements EventListenerInterface
{
    public function __construct(protected ConnectionResolverInterface $resolver, protected ServerRequestInterface $request)
    {
    }

    /**
     * {@inheritDoc}
     */
    public function listen(EventManagerInterface $manager): void
    {
        $manager->on('pre_system', function () {
            AbstractPaginator::currentPathResolver(fn () => $this->request->getUri()->getPath());
            AbstractPaginator::currentPageResolver(fn ($pageName) => Arr::get($this->request->getQueryParams(), $pageName, 1));
            Model::setConnectionResolver($this->resolver);
        });
    }
}
```

Lorsque l’écouteur d’événements est résolu, le conteneur lit les type-hints sur le constructeur de la classe et injecte la valeur appropriée. Pour en savoir plus sur l’enregistrement d’éléments dans le conteneur, consultez [sa documentation](/docs/{version}/conteneur).

<a name="reference-du-contrat"></a>
## Référence du contrat

Ce tableau fournit une référence rapide à l’ensemble des contrats BlitzPHP et de leurs façades et services équivalentes :

| Contrat                                                                                                                                              | Façade      | Service     |
|------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|-------------|
| [BlitzPHP\Contracts\Autoloader\LocatorInterface](https://github.com/blitz-php/contracts/blob/main/Autoloader/LocatorInterface.php)                   | &nbsp;      | `locator`   |
| [BlitzPHP\Contracts\Cache\CacheInterface](https://github.com/blitz-php/contracts/blob/main/Cache/CacheInterface.php)                                 | `Cache`     | `cache`     |
| [BlitzPHP\Contracts\Container\ContainerInterface](https://github.com/blitz-php/contracts/blob/main/Container/ContainerInterface.php)                 | `Container` | `container` |
| [BlitzPHP\Contracts\Database\BuilderInterface](https://github.com/blitz-php/contracts/blob/main/Database/BuilderInterface.php)                       | &nbsp;      | `builder`   |
| [BlitzPHP\Contracts\Database\ConnectionInterface](https://github.com/blitz-php/contracts/blob/main/Database/ConnectionInterface.php)                 | &nbsp;      | `database`  |
| [BlitzPHP\Contracts\Database\ConnectionResolverInterface](https://github.com/blitz-php/contracts/blob/main/Database/ConnectionResolverInterface.php) | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Database\ResultInterface](https://github.com/blitz-php/contracts/blob/main/Database/ResultInterface.php)                         | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Event\EventInterface](https://github.com/blitz-php/contracts/blob/main/Event/EventInterface.php)                                 | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Event\EventListenerInterface](https://github.com/blitz-php/contracts/blob/main/Event/EventListenerInterface.php)                 | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Event\EventManagerInterface](https://github.com/blitz-php/contracts/blob/main/Event/EventManagerInterface.php)                   | &nbsp;      | `event`     |
| [BlitzPHP\Contracts\Filesystem\FilesystemInterface](https://github.com/blitz-php/contracts/blob/main/Filesystem/FilesystemInterface.php)             | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Http\ResponsableInterface](https://github.com/blitz-php/contracts/blob/main/Http/ResponsableInterface.php)                       | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Mail\MailerInterface](https://github.com/blitz-php/contracts/blob/main/Mail/MailerInterface.php)                                 | &nbsp;      | `mail`      |
| [BlitzPHP\Contracts\Router\AutoRouterInterface](https://github.com/blitz-php/contracts/blob/main/Router/AutoRouterInterface.php)                     | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Router\RouteCollectionInterface](https://github.com/blitz-php/contracts/blob/main/Router/RouteCollectionInterface.php)           | `Route`     | `routes`    |
| [BlitzPHP\Contracts\Router\RouterInterface](https://github.com/blitz-php/contracts/blob/main/Router/RouterInterface.php)                             | &nbsp;      | `router`    |
| [BlitzPHP\Contracts\Security\EncrypterInterface](https://github.com/blitz-php/contracts/blob/main/Security/EncrypterInterface.php)                   | &nbsp;      | `encrypter` |
| [BlitzPHP\Contracts\Security\HasherInterface](https://github.com/blitz-php/contracts/blob/main/Security/HasherInterface.php)                         | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Session\CookieInterface](https://github.com/blitz-php/contracts/blob/main/Session/CookieInterface.php)                           | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Session\CookieManagerInterface](https://github.com/blitz-php/contracts/blob/main/Session/CookieManagerInterface.php)             | &nbsp;      | `cookie`    |
| [BlitzPHP\Contracts\Session\SessionInterface](https://github.com/blitz-php/contracts/blob/main/Session/SessionInterface.php)                         | &nbsp;      | `session`   |
| [BlitzPHP\Contracts\Support\Arrayable](https://github.com/blitz-php/contracts/blob/main/Support/Arrayable.php)                                       | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Support\Enumerable](https://github.com/blitz-php/contracts/blob/main/Support/Enumerable.php)                                     | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\Support\Jsonable](https://github.com/blitz-php/contracts/blob/main/Support/Jsonable.php)                                         | &nbsp;      | &nbsp;      |
| [BlitzPHP\Contracts\View\RendererInterface](https://github.com/blitz-php/contracts/blob/main/View/RendererInterface.php)                             | `View`      | `viewer`    |
| [BlitzPHP\Contracts\View\ViewDecoratorInterface](https://github.com/blitz-php/contracts/blob/main/View/ViewDecoratorInterface.php)                   | &nbsp;      | &nbsp;      |
