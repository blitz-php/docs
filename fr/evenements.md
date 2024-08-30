---
title: Système d'évènements
---

<a name="introduction"></a>
## Introduction

Le système d'événements de BlitzPHP fournit un moyen d'accéder et de modifier le fonctionnement interne du framework sans avoir à pirater les fichiers principaux. Lorsque BlitzPHP s'exécute, il suit un processus d'exécution spécifique. Cependant, il peut arriver que vous souhaitiez déclencher une action à un stade particulier du processus d'exécution. Par exemple, vous pourriez vouloir exécuter un script juste avant que vos contrôleurs ne soient chargés, ou juste après, ou encore déclencher l'un de vos propres scripts à un autre endroit.

Les événements fonctionnent selon un modèle de `publication/abonnement`, où un événement est déclenché à un moment donné au cours de l'exécution du script. D'autres scripts peuvent "s'abonner" à cet événement en s'enregistrant auprès de la classe Événements pour lui faire savoir qu'ils souhaitent effectuer une action lorsque l'événement est déclenché.

<a name="definition-d-un-evenement"></a>
## Définition d'un événement

BlitzPHP peut découvrir automatiquement tous les fichiers `Events/**` que vous avez créés dans n’importe quel namespace défini. Cela permet une utilisation simple de tous les evenements de module. Pour écouter un événement, vous devez créer une classe d'écoute qui répondant aux exigences suivantes :

- Son namespace doit être accessible via Composer (pour des package tiers) ou défini dans `app/Config/Autoload.php`
- À l’intérieur du namespace, le fichier doit se trouver dans le dossier `{namespace}/Events`
- Elle doit implémenter l'interface <a href="https://github.com/blitz-php/contracts/blob/main/Event/EventListenerInterface.php" target="_blank">BlitzPHP\Contracts\Event\EventListenerInterface</a>.

Le squelette de votre classe d'écoute pourrait alors être  :

```php
<?php

namespace App\Events;

use BlitzPHP\Contracts\Event\EventListenerInterface;
use BlitzPHP\Contracts\Event\EventManagerInterface;

class AppListener implements EventListenerInterface
{
    /**
     * {@inheritDoc}
     */
    public function listen(EventManagerInterface $manager): void
    {
        // Ecouter vos evenements ici
    }
}
```

<a name="s-abonner-a-un-evenement"></a>
### S'abonner à 'un événement

Vous pouvez abonner une action à un événement avec la méthode `on()` du gestionnaire d'événement. Le premier paramètre est le nom de l'événement auquel s'abonner. Le second paramètre est un callable qui sera exécuté lorsque l'événement sera déclenché :

```php
public function listen(EventManagerInterface $manager): void
{
    $manager->on('app:init', function() {
        
    });
}
```

Dans cet exemple, chaque fois que l'événement `pre_system` est exécuté, la closure définie est exécutée. Notez que le second paramètre peut être n'importe quelle forme de <a href="https://www.php.net/manual/en/function.is-callable.php" target="_blank">callable</a> que PHP reconnaît :

```php
// Appel d'une fonction autonome
$manager->on('app:init', 'ma_fonction');

// Appel d'une méthode d'instance
$user = new \App\Librairies\User();
$manager->on('app:init', [$user, 'uneMethode']);

// Appel d'une méthode statique
$manager->on('app:init', 'UneClasse::uneMethode');

// Appel d'une closure
$manager->on('app:init', function() {
    // ---       
});
```

<a name="definir-les-priorites"></a>
### Définir les priorités

Étant donné que plusieurs méthodes peuvent être souscrites à un seul événement, vous aurez besoin d'un moyen de définir l'ordre dans lequel ces méthodes seront appelées. Vous pouvez le faire en passant une valeur de priorité comme troisième paramètre de la méthode `on()`. Les valeurs inférieures sont exécutées en premier, la valeur 1 ayant la priorité la plus élevée et les valeurs inférieures n'étant pas limitées :

```php
$manager->on('app:init', 'ma_fonction', 25);
```

Tous les abonnés ayant la même priorité seront exécutés dans l'ordre où ils ont été définis. 
BlitzPHP offre trois constantes de classe, définies pour votre usage, qui fixent des fourchettes utiles pour les valeurs. Vous n'êtes pas obligé de les utiliser, mais vous trouverez peut-être qu'elles facilitent la lecture :

```php
EVENT_PRIORIRY_LOW // 200
EVENT_PRIORITY_NORMAL // 100
EVENT_PRIORITY_HIGH // 10
```

Une fois triés, tous les abonnés sont exécutés dans l'ordre. Si l'un des abonnés renvoie une valeur booléenne `fausse`, l'exécution des abonnés s'arrête.

<a name="declencher-vos-propres-evenements"></a>
## Déclancher vos propres événements

Le gestionnaire d'événements de BlitzPHP vous permet également de créer facilement des événements dans votre propre code. Pour utiliser cette fonctionnalité, il vous suffit d'appeler la méthode `emit()` du service `event` avec le nom de l'événement :

```php
service('event')->emit('un_evenement');

// utilisation de la façade
BlitzPHP\Facades\Event::emit('un_evenement');
```

Accessoirement, la méthode `emit` prend en second paramètre la cible de l'événement. Typiquement, il s'agit de l'objet associé à l'événement. Cet objet est important car les abonnés auront un accès immédiat aux propriétés de l’objet et pourront les inspecter ou les changer à la volée.

```php
$user = User::create([...]);

service('event')->emit('register', $user);
```

Au final, le troisième paramètre est un tableau associatif représentant des données d’événement supplémentaire. Ceci peut être toute donnée que vous considérez utile de passer pour que les écouteurs puissent agir sur eux.

```php
$user = User::create([...]);

// utilisateur créé via le formulaire d'inscription
service('event')->emit('user.create', $user, [
    'via' => 'register-form'
]);

// utilisateur créé via la page d'administration
service('event')->emit('user.create', $user, [
    'via' => 'admin-add-form'
]);
```

Une autre manière d'utiliser la méthode `emit` est de lui passer un objet `Event` comme unique paramètre. 

```php
$user = User::create([...]);

$event = new \BlitzPHP\Event\Event('user.create', $user, [
    'via' => 'register-form'
]);
service('event')->emit($event);
```

Le constructeur de la classe `Event` prend les 3 paramètres tel que définis précédemment (nom, cible, arguments). De ce fait, l'exemple ci-dessus est équivalent au précédent.

<a name="comment-recuperer-la-cible-et-les-arguments-d-un-evenement"></a>
### Comment récupérer la cible et les arguments d'un événement  

Les callables utilisés par les abonnés d'événements reçoivent en argument un objet `Event` ayant tous les données transmis lors du déclenchement d'en événement. Vous pouvez alors utiliser les méthodes `getTarget()` et `getParams()` pour récupérer la cible et les arguments d'un événement.

Si nous voulons s'abonner à l'événement `user.create` créé précédemment, on aura un code semblable à celui-ci:

```php
$manager->on('user.create', function(\BlitzPHP\Event\Event $event) {
    $user = $event->getTarget(); // User
    $args = $event->getParams(); // ['via' => '...']
});
```

<a name="stopper-les-evenements"></a>
## Stopper les événements

Un peu comme les événements DOM, vous voulez peut-être stopper un événement pour éviter aux autres abonnés d’être notifiés.

Afin de stopper les événements, vous pouvez soit retourner `false` dans vos callbacks ou appeler la méthode `stopPropagation()` sur l’objet `Event`:

```php
$manager->on('event', function($event) {
    //---
    $event->stopPropagation();
});

$manager->on('event', function($event) {
    //---
    return false;
});
```

Stopper un événement va éviter à toute callback supplémentaire d’être appelée. En plus, le code attrapant l’événement peut se comporter différemment selon que l’événement est stoppé ou non. Généralement il n’est pas sensé stopper “après” les événements, mais stopper “avant” les événements est souvent utilisé pour empêcher toutes les opérations de se passer.


<a name="evenements-systeme"></a>
## Evénements système

Voici une liste des points d'événement disponibles pour les applications Web invoquées par public/index.php : 
- **pre_system**:  Appelé au début de l'exécution du système. L'URI, la requête et la réponse ont été instanciés, mais la vérification du cache de la page, le routage et l'exécution des middlewares n'ont pas encore eu lieu. 
- **post_controller_constructor**: Appelé immédiatement après l'instanciation de votre contrôleur, mais avant tout appel de méthode. 
- **post_system**: Appelé juste avant que la page rendue finale ne soit envoyée au navigateur, à la fin de l'exécution du système.