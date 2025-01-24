---
title: Planification des tâches
---

<a name="introduction"></a>
## Introduction

Dans le passé, vous avez peut-être écrit une entrée de configuration cron pour chaque tâche que vous deviez planifier sur votre serveur. Cependant, cela peut rapidement devenir pénible car votre programme de tâches n'est plus dans le contrôle des sources et vous devez vous connecter en SSH à votre serveur pour voir vos entrées cron existantes ou ajouter des entrées supplémentaires.

Le planificateur de tâches de BlitzPHP offre une nouvelle approche de la gestion des tâches planifiées sur votre serveur. Le planificateur vous permet de définir de manière fluide et expressive votre programme de commandes au sein même de votre application. Lorsque vous utilisez le scheduler, une seule entrée cron est nécessaire sur votre serveur. La planification des tâches est définie dans le fichier `app/Config/tasks.php` de votre application.

<a name="installation"></a>
## Installation

L'application de base par défaut de BlitzPHP n'est pas livrée avec le planificateur de tâche. Vous pouvez l'ajouter via Composer en exécutant la commande suivante :

```bash
composer require blitz-php/tasks
```

Une fois installé, vous devez publier les fichiers de configurations (si ça n'a pas été fait automatiquement) en exécutant la commande ci-dessous :

```bash
php klinge publish
```

<a name="utilisation-d-une-base-de-donnees"></a>
### Utilisation d'une base de données

Le planificateur de tâche utilise le package [blitz-php/parametres](/docs/{version}/parametres) pour sauvegarder les données liées à l'exécution des tâches. Par défaut, ce package stocke ces données dans un fichier JSON.  
Si vous souhaitez les stocker dans une base de données, nous vous invitons à vous rendre dans la [section dédié à la configuration de cet autre package](/docs/{version}/parametres) pour savoir comment le faire.

<a name="definition-des-plannings"></a>
## Définition des plannings

Après avoir installé le package `blitz-php/tasks` et publier ses fichiers de configurations, vous pourrez définir toutes vos tâches programmées dans le fichier `app/Config/tasks.php` de votre application. 

Pour commencer, voyons un exemple. Dans cet exemple, nous allons programmer une closure qui sera appelée tous les jours à minuit. A l'intérieur de cette closure, nous exécuterons une requête de base de données pour effacer une table :

```php
<?php
use BlitzPHP\Tasks\Scheduler;

return [
    'init' => function (Scheduler $schedule) {
        $schedule->call(function() {
           service('database')->table('recent_users')->delete(); 
        })->daily();
    },
];
```

> **Note**  
> Dans la suite de cette documentation, pour plus de commodité, nous ferons abstraction de la clé de configuration `'init' => function (Scheduler $schedule)`. On considérera dès à présent que tous les codes que nous écrirons en rapport avec la définition des plannings seront mis à l'intérieur de cette clé comme dans l'exemple précédent.

En plus de la planification à l'aide des closures, vous pouvez également planifier des <a href="https://www.php.net/manual/fr/language.oop5.magic.php#object.invoke" target="_blank">objets invocables</a> ou tout éléments de <a href="https://www.php.net/manual/fr/language.types.callable.php" target="_blank">type callable</a>. Les objets invocables sont de simples classes PHP qui contiennent une méthode `__invoke` :

```php
$schedule->call(new DeleteRecentUsers())->daily();
```

Si vous souhaitez obtenir une vue d'ensemble de vos tâches programmées et savoir quand elles seront exécutées, vous pouvez utiliser la commande Klinge `tasks:list` :

```bash
php klinge tasks:list
```

<a name="planification-des-commandes-klinge"></a>
### Planification des commandes Klinge

Outre la planification des closure, vous pouvez également planifier des [commandes Klinge](/docs/{version}/klinge). Par exemple, vous pouvez utiliser la méthode `command` pour planifier une commande Klinge en utilisant le nom ou la classe de la commande.

Lorsque vous planifiez des commandes Klinge en utilisant le nom de classe de la commande, vous pouvez passer un tableau d'arguments de ligne de commande supplémentaires qui doivent être fournis à la commande lorsqu'elle est invoquée :

```php
use App\Commands\SendEmailsCommand;
 
$schedule->command('emails:send Dimitri --force')->daily();
 
$schedule->command(SendEmailsCommand::class, ['Dimitri', '--force'])->daily();
```

<a name="planification-des-commandes-shell"></a>
### Planification des commandes Shell

La méthode `shell` peut être utilisée pour envoyer une commande au système d'exploitation. 
Il suffit de fournir la commande à appeler et les arguments éventuels, et elle sera exécutée à l'aide de la fonction `exec()` de PHP.

```php
$schedule->shell('node /path/to/script.js')->daily();
```

> **Note**  
> De nombreux serveurs mutualisés désactivent l'accès à la fonction `exec()` pour des raisons de sécurité. Si vous travaillez sur un serveur partagé, vérifiez que vous pouvez utiliser la fonction exec avant d'utiliser cette fonctionnalité.

<a name="planification-des-evenements"></a>
### Planification des evenements

Si vous souhaitez déclencher un [événement](/docs/{version}/evenements), vous pouvez utiliser la méthode `event` pour le faire, en indiquant le nom de l'événement à déclencher.

```php
$schedule->event('Foo')->hourly();
```

<a name="planification-d-appels-d-url"></a>
### Planification d'appels d'URL

> **Note**  
> Cette fonctionnalité a besoin du [Client HTTP](/docs/{version}/client-http) pour fonctionner. Veuillez vous rassurez d'avoir installer ce dernier avant d'utiliser cette fonctionnalité.  
> Vous pouvez installer le package via la commande suivante.  
> ```bash  
> composer require blitz-php/http-client  
> ```  

Si vous avez besoin d'envoyer régulièrement une requête à une URL, vous pouvez utiliser la méthode `url` pour effectuer une simple requête GET à l'aide de cURL à l'URL que vous avez fournie. Si vous avez besoin de plus de dynamisme qu'une simple chaîne d'URL, vous pouvez utiliser une closure ou une commande à la place.

```php
$schedule->url('https://my-status-cloud.com?site=foo.com')->everyFiveMinutes();
```

<a name="options-de-frequence-de-planification"></a>
### Options de fréquence de planification

Nous avons déjà vu quelques exemples de la manière dont vous pouvez configurer une tâche pour qu'elle s'exécute à des intervalles spécifiques. Cependant, il existe de nombreuses autres fréquences de planification des tâches que vous pouvez assigner à une tâche :

<div class="overflow-auto">

Méthode  | Description
------------- | -------------
`->cron('* * * * *');`  | Exécute la tâche selon une planification cron personnalisée
`->everyMinute(); / ->everyMinute(17);`  | Exécute la tâche toutes les minutes ou toutes les dix-sept minutes
`->betweenMinutes(0, 30);`  |  Exécute la tâche entre les minutes 0 et 30.
`->minutes([0, 20, 46]);`  |  Exécute la tâche à des minutes précises (0, 20 et 46)
`->everyTwoMinutes();`  |  Exécute la tâche toutes les deux minutes
`->everyThreeMinutes();`  |  Exécute la tâche toutes les trois minutes
`->everyFourMinutes();`  |  Exécute la tâche toutes les quatre minutes
`->everyFiveMinutes();`  |  Exécute la tâche toutes les cinq minutes
`->everyTenMinutes();`  |  Exécute la tâche toutes les dix minutes
`->everyFifteenMinutes();`  |  Exécute la tâche toutes les quinze minutes
`->everyThirtyMinutes();`  |  Exécute la tâche toutes les trente minutes
`->hourly(); / ->hourly(17);`  |  Exécute la tâche toutes les heures ou toutes les heures à 17 minutes.
`->everyHour(3, 15);`  |  Exécute la tâche toutes les 3 heures à XX:15 heures
`->betweenHours(6, 12);`  |  Exécute la tâche entre 6h et 12 heures
`->hours([3, 10, 22]);`  |  Exécute la tâche à des heures précises (3, 10 et 22)
`->everyTwoHours($minutes = 0);`  |  Exécute la tâche toutes les deux heures
`->everyThreeHours($minutes = 0);`  |  Exécute la tâche toutes les trois heures
`->everyFourHours($minutes = 0);`  |  Exécute la tâche toutes les quatre heures
`->everySixHours($minutes = 0);`  |  Exécute la tâche toutes les six heures
`->everyOddHour($minutes = 0);`  |  Exécute la tâche toutes les heures impaires
`->daily(); / ->daily('13:00');`  | Exécute la tâche tous les jours à minuit ou à 13:00
`->daysOfMonth([1, 15]);`  |  Exécute la tâche uniquement aux dates spécifiées (le 1er et le 15 de chaque mois)
`->weekdays(); / ->weekdays('8:00');`  |  Exécute la tâche du lundi au vendredi à minuit ou à 8:00
`->weekends(); / ->weekends('13:25');`  |  Exécute la tâche les samedi et dimanche à minuit ou  à 13:25
`->monthly(); / ->monthly('4:20');`  |  Exécute la tâche le premier jour de chaque mois à 00:00 ou à 04:20
`->everyMonth(); / ->everyMonth(9);`  | Exécute la tâche chaque mois ou tous les neuf mois
`->monthlyOn(4, '15:00');`  |  Exécuter la tâche tous les mois, le 4 à 15:00
`->lastDayOfMonth(); / ->lastDayOfMonth('15:00');` | Exécute la tâche le dernier jour du mois à minuit ou à 15:00
`->quarterly(); / ->quarterly('10:02')` |  Exécute la tâche le premier jour de chaque trimestre à 00:00 ou à 10:02
`->quarterlyOn(4, '14:00');` |  Exécuter la tâche chaque trimestre, le 4 à 14:00
`->yearly(); / ->yearly('19:32');`  |  Exécute la tâche le premier jour de chaque année à 00:00 ou à 19:32
`->yearlyOn(6, 2, '17:48');`  |  Exécute la tâche chaque année, le 2 juin à 17h48
`->timezone('America/New_York');` | Défini le fuseau horaire de la tâche

</div>

Ces méthodes peuvent être combinées avec des contraintes supplémentaires pour créer des programmes encore plus précis qui ne s'exécutent que certains jours de la semaine. Par exemple, vous pouvez programmer l'exécution hebdomadaire d'une commande le lundi :

```php
// S'execute une fois par semaine le lundi à 13h...
$schedule->call(function () {
    // ...
})->weekly()->mondays()->at('13:00');
 
// S'execute toutes les heures de 8h à 17h en semaine...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
```

Une liste de contraintes horaires supplémentaires figure ci-dessous :

<div class="overflow-auto">

Méthode  | Description
------------- | -------------
`->sundays();`  |  Limite l'exécution de la tâche aux dimanche
`->mondays();`  |  Limite l'exécution de la tâche aux lundi
`->tuesdays();`  |  Limite l'exécution de la tâche aux mardi
`->wednesdays();`  |  Limite l'exécution de la tâche aux mercredi
`->thursdays();`  |  Limite l'exécution de la tâche aux jeudi
`->fridays();`  |  Limite l'exécution de la tâche aux vendredi
`->saturdays();`  |  Limite l'exécution de la tâche aux samedi
`->days(array);`  |  Limite l'exécution de la tâche à des jours spécifiés (0 = dimanche, 6 = samedi)
`->between($startTime, $endTime);`  |  Limite l'exécution de la tâche entre un temps (H:m) de début et de fin
`->unlessBetween($startTime, $endTime);`  |  Limite l'exécution de la tâche en dehors d'un temps (H:m) de début et de fin
`->when(Closure);`  |  Limite l'exécution de la tâche lorsqu'une condition est vérifiée
`->environments($env);`  |  Limite l'exécution de la tâche dans un environnement spécifique

</div>

<a name="contraintes-journalieres"></a>
#### Contraintes journalières

La méthode `days` peut être utilisée pour limiter l'exécution d'une tâche à des jours spécifiques de la semaine. Par exemple, vous pouvez planifier une commande pour qu'elle s'exécute toutes les heures le dimanche et le mercredi :

```php
$schedule->command('emails:send')
        ->hourly()
        ->days([0, 3]);
```

Vous pouvez également utiliser les constantes disponibles dans la classe `BlitzPHP\Tasks\Scheduler` pour définir les jours d'exécution d'une tâche :

```php
use BlitzPHP\Tasks\Scheduler;

$schedule->command('emails:send')
        ->hourly()
        ->days([Scheduler::SUNDAY, Scheduler::WEDNESDAY]);
```

<a name="contraintes-de-temps"></a>
#### Contraintes de temps

La méthode `between` peut être utilisée pour limiter l'exécution d'une tâche en fonction de l'heure de la journée :

```php
$schedule->command('emails:send')
        ->hourly()
        ->between('7:00', '22:00');
```

<a name="contraintes-liees-a-l-environnement"></a>
#### Contraintes liées à l'environnement

La méthode `environments` peut être utilisée pour exécuter des tâches uniquement dans les environnements donnés (tels que définis par la [variable d'environnement](/docs/{version}/configuration#configuration-d-environnement) `ENVIRONMENT`) :

```php
$schedule->command('emails:send')
        ->daily()
        ->environments('staging', 'production');
```

<a name="fuseaux-horaires"></a>
### Fuseaux horaires

La méthode `timezone` permet de spécifier que l'heure d'une tâche programmée doit être interprétée dans un fuseau horaire donné :

```php
$schedule->command('report:generate')
         ->timezone('Africa/Douala')
         ->at('2:00');
```

Si vous assignez régulièrement le même fuseau horaire à toutes vos tâches planifiées, vous pouvez définir une clé `timezone` dans votre fichier de configuration `app/Config/tasks.php`. Cette clé doit fournir le fuseau horaire par défaut qui doit être attribué à toutes les tâches planifiées :

```php
// app/Config/tasks.php
<?php 

return [
    // ...
    
    /**
     * Définir le fuseau horaire à utiliser par défaut pour les tâches programmées.
     *
     * @var string
     */
    'timezone' => 'America/Chicago',
    
    // ...
];
```

> **Attention**  
> N'oubliez pas que certains fuseaux horaires utilisent l'heure d'été. Lorsque l'heure d'été change, votre tâche programmée peut s'exécuter deux fois, voire ne pas s'exécuter du tout. C'est pourquoi nous vous recommandons d'éviter, dans la mesure du possible, de planifier des fuseaux horaires.

<a name="nommer-des-taches"></a>
### Nommer des tâches 

Vous pouvez nommer des tâches afin qu'elles puissent être facilement référencées ultérieurement, ceci à l'aide de la méthode `named` :

```php
$schedule->command('foo')->hourly()->named('foo-task');
```

<a name="execution-du-planificateur"></a>
## Exécution du planificateur

Maintenant que nous avons appris à définir les tâches planifiées, voyons comment les exécuter sur notre serveur. La commande Klinge `tasks:run` évaluera toutes les tâches programmées et déterminera si elles doivent être exécutées en fonction de l'heure actuelle du serveur.

Ainsi, lorsque nous utilisons le planificateur de BlitzPHP, il nous suffit d'ajouter une seule entrée de configuration cron à notre serveur qui exécutera la commande `tasks:run` toutes les minutes :

```shell
* * * * * cd /path-to-your-project && php klinge tasks:run >> /dev/null 2>&1
```

<a name="executer-le-planificateur-localement"></a>
### Exécuter le planificateur localement

> **Note**  
> Cette fonctionnalité a besoin du package `symfony/process` pour fonctionner.. Veuillez vous rassurez d'avoir installer ce dernier avant d'utiliser cette fonctionnalité.  
> Vous pouvez installer le package via la commande suivante. 
> ```bash  
> composer require --dev symfony/process  
> ```  
  
En règle générale, vous n'ajoutez pas d'entrée cron à votre machine de développement locale. Au lieu de cela, vous pouvez utiliser la commande Klinge `tasks:work`. Cette commande s'exécutera au premier plan et invoquera le planificateur toutes les minutes jusqu'à ce que vous mettiez fin à la commande :

```bash
php klinge tasks:work
```

<a name="resultats-de-la-tache"></a>
## Résultats de la tâche

Le planificateur BlitzPHP fournit plusieurs méthodes pratiques pour travailler avec la sortie générée par les tâches planifiées. Tout d'abord, la méthode `sendOutputTo` permet d'envoyer la sortie dans un fichier pour une inspection ultérieure :

```php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
 ```
 
 Si vous souhaitez ajouter la sortie à la suite d'un fichier donné, vous pouvez utiliser la méthode appendOutputTo :
 
 ```php
 $schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
 ```
 
 En utilisant la méthode `emailOutputTo`, vous pouvez envoyer le résultat par email à l'adresse de votre choix. Avant d'envoyer le résultat d'une tâche par mail, vous devez configurer [le services d'envoi d'email](/docs/{version}/email) de BlitzPHP :
 
 ```php
 $schedule->command('report:generate')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('dimtrovich@example.com');
 ```
 
 Si vous ne voulez envoyer le résultat par mail que si la commande programmée se termine par un code de sortie non nul, utilisez la méthode `emailOutputOnFailure` :
 
 ```php
 $schedule->command('report:generate')
         ->daily()
         ->emailOutputOnFailure('dimtrovich@example.com');
 ```
 
> **Attention**  
> Les méthodes `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo` et `appendOutputTo` sont exclusives aux méthodes `command` et `shell`. 

<a name="hooks-de-taches"></a>
## Hooks de tâches

En utilisant les méthodes `before` et `after`, vous pouvez spécifier le code à exécuter avant et après l'exécution de la tâche planifiée :

```php
$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // La tâche est sur le point d'être exécutée...
         })
         ->after(function () {
             // La tâche a été exécutée...
         });
```

Les méthodes `onSuccess` et `onFailure` vous permettent de spécifier le code à exécuter en cas de réussite ou d'échec de la tâche programmée. Un échec indique que la commande Klinge ou système programmée s'est terminée avec un code de sortie différent de zéro :

```php
$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function () {
             // La tâche a réussie...
         })
         ->onFailure(function () {
             // La tâche a échoué...
         });
```

Si une sortie (`buffer`) est disponible dans votre commande, vous pouvez y accéder dans vos hooks `after`, `onSuccess` ou `onFailure` en indiquant une instance de `BlitzPHP\Utilities\String\Stringable` en tant qu'argument `$output` de la définition de closure de votre hook :

```php
use BlitzPHP\Utilities\String\Stringable;
 
$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function (Stringable $output) {
            // La tâche a réussie...
         })
         ->onFailure(function (Stringable $output) {
             // La tâche a échoué...
         });
```