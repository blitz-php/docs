---
title: Journalisation
---

<a name="introduction"></a>
## Introduction

Pour vous aider à en savoir plus sur ce qui se passe dans votre application, BlitzPHP fournit des services de journalisation robustes qui vous permettent d'enregistrer des messages dans des fichiers, dans le journal des erreurs du système et même dans Slack pour en informer toute votre équipe.

Sous le capot, BlitzPHP utilise la bibliothèque <a href="https://github.com/Seldaek/monolog" target="_blank">Monolog</a>, qui fournit un support pour une variété de gestionnaires de logs puissants. BlitzPHP facilite la configuration de ces gestionnaires, ce qui vous permet de les combiner pour personnaliser le traitement des journaux de votre application.

<a name="configuration"></a>
## Configuration

Toutes les options de configuration du comportement de votre application en matière de journalisation se trouvent dans le fichier de configuration `app/Config/log.php`. Ce fichier vous permet de configurer les gestionnaires de journalisation de votre application ; veillez donc à passer en revue chacun des gestionnaires disponibles et leurs options. Nous passerons en revue quelques options courantes ci-dessous.

<a name="configuration-du-nom-des-journaux"></a>
### Configuration du nom des journaux

Par défaut, Monolog est instancié avec le nom `application`. Pour modifier cette valeur, vous pouvez modifier la variable `name` dans votre fichier de configuration :

```php
<?php

/**
 * ------------------------------------------------- -------------------------
 * Configuration des log
 * ------------------------------------------------- -------------------------
 *
 * Ce fichier vous permet de definir comment votre application doit traiter les log
 */

return [
    /**
     * @var string
     */
    'name' => 'my-super-application',
    
    //---
];
```

<a name="format-des-dates"></a>
### Format des dates

Chaque élément enregistré a une date associée. Vous pouvez définir le format à utiliser pour toutes les date de journalisation.

```php
<?php

return [
    //---
    
    /**
     * @var string
     */
    'date_format' => 'Y-m-d H:i:s',
    
    //---
];
```

<a name="processeurs"></a>
### Processeurs

Les processeurs permettent d'ajouter des données supplémentaires pour tous les enregistrements de vos journaux. Vous pouvez utiliser la variable `processors` pour activer ou désactiver des processeurs à votre application.

```php
<?php

return [
    //---
    
    /**
     * @var string[]
     */
     'processors' => [
        'hostname',
        'web',
    ],
    
    //---
];
```

Les processeurs disponibles sont :


<table>
<thead>
	<tr>
		<th>Processeur</th>
		<th>Description</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>web</td>
		<td>Ajoute l'URI de la requête actuelle, la méthode de requête et l'IP du client à un enregistrement de journal.</td>
	</tr>
	<tr>
		<td>introspection</td>
		<td>Ajoute la ligne/le fichier/la classe/la méthode à l'origine de l'appel de journal.</td>
	</tr>
	<tr>
		<td>hostname</td>
		<td>Ajoute le nom d'hôte actuel à un enregistrement de journal.</td>
	</tr>
	<tr>
		<td>process_id</td>
		<td>Ajoute l'ID de processus à un enregistrement de journal.</td>
	</tr>
	<tr>
		<td>uid</td>
		<td>Ajoute un identifiant unique à un enregistrement de journal.</td>
	</tr>
	<tr>
		<td>memory_usage</td>
		<td>Ajoute l'utilisation actuelle de la mémoire à un enregistrement de journal.</td>
	</tr>
	<tr>
		<td>psr</td>
		<td>Traite le message d'un enregistrement de journal conformément aux règles PSR-3, en le remplaçant {foo} par la valeur de $context['foo']</td>
	</tr>
</tbody>
</table>

<a name="gestionnaires-de-journalisation"></a>
### Gestionnaires de journalisation

BlitzPHP propose plusieurs gestionnaires permettant d'enregistrer vous journaux. Le gestionnaire détermine comment et où le message de journal est réellement enregistré. Les gestionnaires de journalisation suivants sont disponibles dans chaque application BlitzPHP. Certains de ces gestionnaires sont commentés dans votre fichier `app/Config/log.php` de votre application, vous pourriez donc les décommenter pour les activer.

**Enregistrement**

* `file`: Enregistre les données de logs dans les fichiers.
* `error`: Enregistre les données de logs dans la fonction `error_log()` de PHP.

**Notification**

* `email`: Envoie les données de logs par mail.
* `telegram`: Envoie les données de logs par <a href="https://core.telegram.org/bots/api" target="_blank">un bot Telegram</a>.

**Débogage**

* `chrome`: Envoi les log dans la console Chrome pour le débogage. Nécessite l'extension ChromeLogger installée dans votre navigateur.
* `firebug`: Envoi les log dans la console Firebug pour le débogage. Nécessite l'extension Firebug installée dans votre navigateur.
* `browser`: Envoi les log dans la console du navigateur pour le débogage.

Tous ces gestionnaires sont présent dans votre fichier `app/Config/log.php`, nous vous invitons donc à le parcourir pour voir les options de configuration de chacun d'eux.

<a name="ecriture-des-messages-du-journal"></a>
## Écriture des messages du journal

Vous pouvez écrire des informations dans les journaux en utilisant la [façade](/docs/{version}/facades) Log. Le logger de BlitzPHP fournit les huit niveaux de journalisation définis dans la <a href="https://tools.ietf.org/html/rfc5424" target="_blank">spécification RFC 5424</a> : **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** et **debug** :

```php
use BlitzPHP\Facades\Log;
 
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

Vous pouvez appeler n'importe laquelle de ces méthodes pour enregistrer un message pour le niveau correspondant. Le message sera écrit via tous les canaux (gestionnaires) de journalisation spécifiés dans votre fichier de configuration.

```php
<?php
 
namespace App\Controllers;
 
use App\Entities\User;
use BlitzPHP\Facades\Log;
 
class UserController extends AppController
{
    /**
     * Affiche le profil de l'utilisateur donné.
     */
    public function show(string $id)
    {
        Log::info("Afficher le profil de l'utilisateur: " . $id);
 
        return view('user/profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

<a name="informations-contextuelles"></a>
### Informations contextuelles

Un tableau de données contextuelles peut être transmis aux méthodes d'enregistrement. Ces données contextuelles seront formatées et affichées avec le message du journal :

```php
use BlitzPHP\Facades\Log;
 
Log::info("L'utilisateur {id} n'a pas réussi à se connecter.", ['id' => $user->id]);
```

<a name="note-sur-les-niveaux-de-journalisation"></a>
### Note sur les niveaux de journalisation

Tout d'abord, considérons les configurations de gestionnaires suivantes:

```php
// ---

'handlers' => [
    'file' => [
        'level' => \Psr\Log\LogLevel::DEBUG,

        //...
    ],

    'email' => [
        'level' => \Psr\Log\LogLevel::CRITICAL,
        
        //...
    ],
]
```

Notez l'option de configuration `level` présente dans toutes les configurations des gestionnaires de votre fichier `app/Config/log.php`. Cette option détermine le "niveau" minimum qu'un message doit avoir pour être enregistré par le gestionnaire. Monolog, qui alimente les services de journalisation de BlitzPHP, offre tous les niveaux de journalisation définis dans la <a href="https://tools.ietf.org/html/rfc5424" target="_blank">spécification RFC 5424</a>. Par ordre décroissant de gravité, ces niveaux de journalisation sont : `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info` et `debug`.

Imaginons donc que nous enregistrons un message à l'aide de la méthode `debug` :

```php
Log::debug("Un message d'information.");
```

Compte tenu de notre configuration, le gestionnaire `file` écrira le message dans le journal; cependant, comme le message d'erreur n'est pas critique ou supérieur, il ne sera pas envoyé par `email`. Cependant, si nous enregistrons un message d'urgence, il sera à la fois enregistré au fichier journal et envoyé par email puisque le niveau d'urgence est supérieur à notre seuil minimum pour les deux canaux :

```php
Log::emergency("Le système est en panne !");
```