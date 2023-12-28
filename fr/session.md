---
title: Session HTTP
---

<a name="introduction"></a>
## Introduction

Étant donné que les applications HTTP sont sans état, les sessions permettent de stocker des informations sur l'utilisateur à travers plusieurs requêtes. Ces informations sur l'utilisateur sont généralement placées dans une base de données persistante à laquelle il est possible d'accéder lors de requêtes ultérieures.

BlitzPHP est livré avec une variété de gestionnaires de session qui sont accessibles via une API expressive et unifiée. La prise en charge de gestionnaires populaires tels que <a href="https://memcached.org/" target="_blank">Memcached</a>, <a href="https://redis.io/" target="_blank">Redis</a> et les bases de données est incluse.

<a name="utilisation-de-la-session"></a>
## Utilisation de la session

<a name="acceder-a-la-session"></a>
### Accéder à la session

Les sessions sont généralement exécutées globalement à chaque chargement de page, de sorte que la classe Session doit être initialisée comme par magie.

Pour accéder à la session et l'initialiser :

```php
<?php 
use BlitzPHP\Container\Services;

$session = Services::session(); // Accès via le prestataire de service
// ...
$session = service('session'); // Accès via le helper service
// ...
$session = session(); // Accès à l'aide de la fonction dédiée
// ...
$session = $request->session(); // Accès à partir de l'objet Request
```

<a name="comment-fonctionnent-les-sessions"></a>
### Comment fonctionnent les sessions ?

Lorsqu'une page est chargée, la classe de session vérifie si un cookie de session valide est envoyé par le navigateur de l'utilisateur. Si un cookie de session n'existe pas (ou s'il ne correspond pas à un cookie stocké sur le serveur ou s'il a expiré), une nouvelle session est créée et sauvegardée.

Si une session valide existe, ses informations seront mises à jour. À chaque mise à jour, l'identifiant de la session peut être régénéré s'il est configuré à cet effet.

Il est important de comprendre qu'une fois initialisée, la classe Session s'exécute automatiquement. Il n'y a rien à faire pour que le comportement ci-dessus se produise. Vous pouvez, comme vous le verrez plus loin, travailler avec des données de session, mais le processus de lecture, d'écriture et de mise à jour d'une session est automatique.

<a name="qu-est-ce-que-les-donnees-de-session"></a>
### Qu'est-ce que les données de session ?

Les données de session sont simplement un tableau associé à un identifiant de session particulier (cookie).

Si vous avez déjà utilisé des sessions en PHP, vous devez être familier avec la <a href="https://www.php.net/manual/en/reserved.variables.session.php" target="_blank">superglobale $_SESSION</a> de PHP (si ce n'est pas le cas, veuillez lire le contenu de ce lien).

BlitzPHP donne accès à ses données de session par les mêmes moyens, en utilisant le mécanisme des gestionnaires de session fourni par PHP. L'utilisation des données de session est aussi simple que la manipulation (lecture, définition et suppression des valeurs) du tableau `$_SESSION`.

> **Note**  
> En général, l'utilisation de variables globales est une mauvaise pratique. Il n'est donc pas recommandé d'utiliser directement la variable superglobale `$_SESSION`.

<a name="interagir-avec-la-session"></a>
## Interagir avec la session

<a name="recuperation-des-donnees"></a>
### Récupération des données

Comme présenté précédemment, il existe quatre façon d'accéder à la session. Cependant, la plupart du temps, seules la fonction globale `session()` et l'accès via une instance de requête seront utilisés. 

Tout d'abord, examinons l'accès à la session via une instance de requête, qui peut être indiquée sur une closure de route ou une méthode de contrôleur.

```php
<?php
 
namespace App\Controllers;
 
class UserController extends Controller
{
    /**
     * Affiche le profil de l'utilisateur donné.
     */
    public function show(string $id)
    {
        $value = $this->request->session()->get('key');
 
        // ...
 
        $user = $this->users->find($id);
 
        return view('user/profile', ['user' => $user]);
    }
}
```

Lorsque vous récupérez un élément de la session, vous pouvez également transmettre une valeur par défaut en tant que deuxième argument de la méthode `get()`. Cette valeur par défaut sera renvoyée si la clé spécifiée n'existe pas dans la session. Si vous passez une closure comme valeur par défaut à la méthode `get()` et que la clé demandée n'existe pas, la closure sera exécutée et son résultat sera renvoyé :

```php
$value = $request->session()->get('key', 'default');
 
$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="la-fonction-globale-session"></a>
#### La fonction globale `session()`

Vous pouvez également utiliser la fonction globale `session()` pour récupérer et stocker des données dans la session. Lorsque cette fonction est appelée avec un seul argument de type chaîne, elle renvoie la valeur de cette clé de session. Lorsqu'elle est appelée avec un tableau de paires clé/valeur, ces valeurs seront stockées dans la session :

```php
Route::get('/home', function () {
    // Récupère un élément de données de la session...
    $value = session('key') ;
 
    // Spécification d'une valeur par défaut...
    $value = session('key', 'default') ;
 
    // Stocker une donnée dans la session...
    session(['key' => 'value']) ;
});
```

> **Note**  
> Dans la suite de cette documentation, on supposera que la variable $session est une instance de la session courante.  
> Vous pouvez la définir avec la ligne `$session = session();` ou `$session = $this->request->session();`

<a name="recuperation-de-toutes-les-donnees-de-la-session"></a>
#### Récupération de toutes les données de la session

Si vous souhaitez récupérer toutes les données de la session, vous pouvez utiliser la méthode `all()` :

```php
$data = $session->all();
```

<a name="determiner-si-un-element-existe-dans-la-session"></a>
#### Déterminer si un élément existe dans la session

Pour déterminer si un élément est présent dans la session, vous pouvez utiliser la méthode `has()`. Elle renvoie `true` si l'élément est présent et n'est pas nul :

```php
if ($session->has('users')) {
    // ...
}
```

Pour déterminer si un élément est présent dans la session, même si sa valeur est `nulle`, vous pouvez utiliser la méthode `exists()` :

```php
if ($session->exists('users')) {
    // ...
}
```

Pour déterminer si un élément n'est pas présent dans la session, vous pouvez utiliser la méthode `missing()` qui renvoie la valeur `true` si l'élément n'est pas présent :

```php
if ($session->missing('users')) {
    // ...
}
```

<a name="stockage-des-donnees"></a>
### Stockage des données

Pour stocker des données dans la session, vous utiliserez généralement la méthode `put()` de l'instance de requête ou la fonction globale `session()` :

```php
// Via une instance de requête...
$request->session()->put('key', 'value');
 
// Via la fonction "session"...
session(['key' => 'value']);
```

<a name="poussee-vers-les-valeurs-de-tableau-dans-la-session"></a>
#### Poussée vers les valeurs de tableau dans la session
    
La méthode `push()` peut être utilisée pour ajouter une nouvelle valeur à une valeur de session qui est un tableau. Par exemple, si la clé `user.teams` contient un tableau de noms d'équipes, vous pouvez ajouter une nouvelle valeur au tableau comme suit :

```php
$session->push('user.teams', 'developers');
```

<a name="recuperation-et-suppression-d-un-element"></a>
#### Récupération et suppression d'un élément
    
La méthode `pull()` permet de récupérer et de supprimer un élément de la session en une seule instruction :

```php
$value = $session->pull('key', 'default');
```

<a name="incrementation-et-decrementation-des-valeurs-de-session"></a>
#### Incrémentation et décrémentation des valeurs de session

Si les données de votre session contiennent un nombre entier que vous souhaitez incrémenter ou décrémenter, vous pouvez utiliser les méthodes `increment()` et `decrement()` :

```php
$session->increment('count');
 
$session->increment('count', $incrementBy = 2);
 
$session->decrement('count');
 
$session->decrement('count', $decrementBy = 2);
```

<a name="donnees-flash"></a>
### Données Flash

Il peut arriver que vous souhaitiez stocker des éléments dans la session en vue de la prochaine requête. Vous pouvez le faire en utilisant la méthode `flash()`. Les données stockées dans la session à l'aide de cette méthode seront disponibles immédiatement et lors de la requête HTTP suivante. Après la requête HTTP suivante, les données flashées seront supprimées. Les données flash sont principalement utiles pour les messages d'état de courte durée :

```php
$session->flash('status', 'La tâche a réussi !');
```

Si vous devez conserver vos données flash pour plusieurs requêtes, vous pouvez utiliser la méthode `reflash()`, qui conservera toutes les données flash pour une requête supplémentaire. Si vous ne devez conserver que des données flash spécifiques, vous pouvez utiliser la méthode `keep()` :

```php
$session->reflash();
 
$session->keep(['username', 'email']);
```

Pour récupérer une valeur flash, vous pouvez utiliser la méthode `getFlashdata()`

```php
$value = $session->getFlashdata('item'); // obtenir un seul element flash

$values = $session->getFlashdata(); // obtenir tous les elements flash
```

<a name="donnees-temporaires"></a>
### Données temporaires

BlitzPHP prend également en charge les données de session avec un délai d’expiration. Après l’expiration de la valeur, ou l’expiration de la session ou est deleted, la valeur est automatiquement supprimée. La définition des données temporaires se fait par la méthode `temp()` comme suit:


```php
$session->temp('item', 'value', 300); // durée de vie: 300 secondes

// ou en utilisant un tableau

$tempdata = ['newuser' => true, 'message' => 'Bienvenue chez nous'];
$session->temp($tempdata, null, $expire);

```

> **Note**  
> Si l’expiration est omise ou définie sur 0, la valeur par défaut Une durée de vie de 300 secondes (ou 5 minutes) sera utilisée.

Pour récupérer une valeur temporaire, vous pouvez utiliser la méthode `getTempdata()`

```php
$value = $session->getTempdata('item'); // obtenir un seul element temporaire

$values = $session->getTempdata(); // obtenir tous les elements temporaires
```

Si vous avez besoin de supprimer une valeur temporaire avant son expiration, vous pouvez directement Désactivez-le du tableau `$_SESSION`:

```php
unset($_SESSION['item']);
```

Cependant, cela ne supprimera pas le marqueur qui fait de cet élément spécifique un élément temporaire (il sera invalidé lors de la prochaine requête HTTP), donc si vous Si vous avez l’intention de réutiliser cette même clé dans la même requête, vous devez utiliser `removeTempdata()` :

```php
$session->removeTempdata('item');
```

<a name="suppression-des-donnees"></a>
### Suppression des données

La méthode `remove()` supprime un élément de données de la session :

```php
// Supprimer une seule clé...
$session->remove('name');
 
// Supprimer plusieurs clés...
$session->remove(['name', 'status']);
```

<a name="fin-de-session"></a>
### Fin de session

<a name="cloture-d-une-session"></a>
#### Clôture d'une session

Pour fermer manuellement la session en cours lorsque vous n'en avez plus besoin, utilisez la méthode `close()` :

```php
$session->close();
```

Vous n'avez pas besoin de fermer la session manuellement, PHP la fermera automatiquement après la fin de votre script. Mais comme les données de session sont verrouillées pour empêcher les écritures concurrentes, une seule requête peut opérer sur une session à tout moment. Vous pouvez améliorer les performances de votre site en fermant la session dès que toutes les modifications apportées aux données de la session sont terminées.

Cette méthode fonctionne exactement de la même manière que la fonction PHP <a href="https://www.php.net/session_write_close" target="_blank">session_write_close()</a>.

<a name="destruction-d-une-session"></a>
#### Destruction d'une session

Pour effacer la session en cours (par exemple, lors d'une déconnexion), il suffit d'utiliser la méthode `destroy()` :

```php
$session->destroy();
```

Cette méthode fonctionne exactement de la même manière que la fonction <a href="https://www.php.net/session_destroy" target="_blank">session_destroy()</a> de PHP.

Il doit s'agir de la dernière opération liée à la session que vous effectuez au cours de la même requête. Toutes les données de la session (y compris les données flash et temporaires) seront détruites de manière permanente.

> **Note**  
> Il n'est pas nécessaire d'appeler cette méthode à partir du code habituel. Nettoyer les données de la session plutôt que de la détruire.

<a name="configuration"></a>
## Configuration

Généralement, BlitzPHP fait en sorte que tout fonctionne dans la boîte. Cependant, les sessions sont un élément très sensible de toute application, c'est pourquoi une configuration minutieuse doit être effectuée. Prenez le temps d'examiner toutes les options et leurs effets.

Vous trouverez les préférences suivantes liées à la session dans votre fichier `app/Config/session.php` :

<div class="overflow-auto">

<table>
<thead>
	<tr>
		<th>Préférence</th>
		<th>Description</th>
		<th>Options</th>
		<th>Valeur par défaut</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>handler</td>
		<td>Le gestionnaire de stockage de session à utiliser.</td>
		<td>
-file<br>- database<br>- redis<br>- memcached<br>- array
</td>
		<td>file</td>
	</tr>
	<tr>
		<td>cookie_name</td>
		<td>Le nom utilisé pour le cookie de session.</td>
		<td>Caractères <strong>[A-Za-z_-]</strong> uniquement</td>
		<td>blitz_session</td>
	</tr>
	<tr>
		<td>expiration</td>
		<td>Le nombre de secondes que vous souhaitez que la session dure.<br>Si vous souhaitez une session qui n'expire pas (jusqu'à ce que le navigateur soit fermé), réglez la valeur sur zéro : 0</td>
		<td>Entier représentant la durée en seconde</td>
		<td>7200 (2h)</td>
	</tr>
	<tr>
		<td>savePath</td>
		<td>Spécifie l'emplacement de stockage, en fonction du pilote utilisé.</td>
		<td>Dossier existant</td>
		<td>/storage/framework/session</td>
	</tr>
	<tr>
		<td>matchIP</td>
		<td>Validation ou non de l'adresse IP de l'utilisateur lors de la lecture du cookie de session.<br>Notez que certains fournisseurs d'accès changent dynamiquement l'adresse IP.<br>vous mettrez probablement cette valeur à false.</td>
		<td>booleen</td>
		<td>false</td>
	</tr>
	<tr>
		<td>time_to_update</td>
		<td>Cette option détermine la fréquence à laquelle la classe de session se régénère et crée un nouvel identifiant de session.<br>identifiant de session. La valeur 0 désactive la régénération de l'identifiant de session.</td>
		<td>Entier représentant la durée en seconde</td>
		<td>300</td>
	</tr>
	<tr>
		<td>regenerate_destroy</td>
		<td>Détruire ou non les données de session associées à l'ancien identifiant de session lors de la régénération automatique de l'identifiant de session.<br>l'ID de session. Si la valeur est false, les données seront supprimées ultérieurement par le ramasse-miettes.</td>
		<td>booleen</td>
		<td>false</td>
	</tr>
</tbody>
</table>

</div>

> **Note**  
> Si l'expiration est fixée à `0`, le paramètre `session.gc_maxlifetime` défini par PHP dans la gestion des sessions sera utilisé tel quel (souvent la valeur par défaut de 1440). Cette valeur doit être modifiée dans `php.ini` ou via `ini_set()` si nécessaire.

En plus des valeurs ci-dessus, le cookie de session utilise les valeurs de configuration suivantes dans votre fichier `app/Config/cookie.php` :


| Préférence | Valeur par défaut | Description                                                                                      |
|------------|---------|--------------------------------------------------------------------------------------------------|
| domain     | ''      | Le domaine pour lequel la session est applicable                                                 |
| path       | /       | Le chemin auquel la session est applicable                                                       |
| secure     | false   | Indiquer si le cookie de session doit être créé uniquement pour les connexions cryptées (HTTPS). |
| samesite   | Lax     | Le paramètre SameSite pour le cookie de session                                                  |

> **Note**  
> Le paramètre `httponly` n'a pas d'effet sur les sessions. Au lieu de cela, le paramètre HttpOnly est toujours activé, pour des raisons de sécurité. De plus, le paramètre `cookie.prefix` est complètement ignoré.

<a name="gestionnaires-de-session"></a>
## Gestionnaires de session

Comme nous l'avons déjà mentionné, la bibliothèque Session est livrée avec 4 gestionnaires, ou moteurs de stockage, que vous pouvez utiliser :

* `file` - les sessions sont stockées dans /storage/framework/sessions.
* `database` - les sessions sont stockées dans une base de données relationnelle.
* `memcached / redis` - les sessions sont stockées dans l'une de ces mémoires rapides, basées sur le cache.
* `array` - les sessions sont stockées dans un tableau PHP et ne sont pas conservées.

Par défaut, le gestionnaire par fichier (`file`) est utilisé lors de l'initialisation d'une session, car il s'agit du choix le plus sûr et qu'il est censé fonctionner partout (pratiquement tous les environnements disposent d'un système de fichiers).

Cependant, tout autre gestionnaire peut être sélectionné via le paramètre `handler` dans votre fichier `app/Config/session.php`, si vous le souhaitez. Gardez cependant à l'esprit que chaque pilote a ses propres inconvénients, alors familiarisez-vous avec eux (voir ci-dessous) avant de faire votre choix.

> **Note**  
> Le gestionnaire `array` est utilisé lors des tests et stocke toutes les données dans un tableau PHP, tout en empêchant la persistance des données.

<a name="fichier"></a>
### Fichier

Le gestionnaire de session par fichier utilise votre système de fichiers pour stocker les données de la session.

On peut dire qu'il fonctionne exactement comme l'implémentation de session par défaut de PHP, mais si ce détail est important pour vous, sachez qu'il ne s'agit pas du même code et qu'il a quelques limitations (et avantages).

Plus précisément, il ne supporte pas l<a href="https://www.php.net/manual/en/session.configuration.php#ini.session.save-path" target="_blank">es formats de niveau de répertoire et de mode de PHP utilisés dans session.save_path</a>, et la plupart des options sont codées en dur pour des raisons de sécurité. Au lieu de cela, seuls les chemins absolus sont supportés pour le paramètre `savePath`.

Une autre chose importante à savoir est de s'assurer que vous n'utilisez pas un répertoire accessible au public ou partagé pour stocker vos fichiers de session. Assurez-vous que vous êtes le seul à avoir accès au contenu du répertoire savePath que vous avez choisi. Sinon, quiconque peut le faire, peut également voler n'importe quelle session en cours (également connue sous le nom d'attaque par "fixation de session").

Sur les systèmes d'exploitation de type UNIX, on y parvient généralement en définissant les autorisations en mode `0700` sur ce répertoire via la commande `chmod`, qui permet uniquement au propriétaire du répertoire d'effectuer des opérations de lecture et d'écriture sur celui-ci. Mais attention, l'utilisateur système qui exécute le script n'est généralement pas le vôtre, mais quelque chose comme 'www-data', donc le seul fait de définir ces permissions va probablement casser votre application.

Au lieu de cela, vous devriez faire quelque chose comme ceci, en fonction de votre environnement :

```shell
chmod 0700 /storage/framework/session/
chown www-data /storage/framework/session/
```

<a name="base-de-donnees"></a>
### Base de données

> **Note**  
> Pour utiliser le gestionnaire `database`, vous devez au préalable installer le module de base de données. Rendez-vous sur [le guide de la base de données](/docs/{version}/base-de-donnees) pour en savoir plus.

> **Attention**  
> Seules les bases de données MySQL et PostgreSQL sont officiellement prises en charge, en raison de l'absence de mécanismes de verrouillage consultatifs sur les autres plateformes. L'utilisation de sessions sans verrou peut causer toutes sortes de problèmes, en particulier en cas d'utilisation intensive d'AJAX, et nous ne prendrons pas en charge de tels cas. Utilisez la méthode [close()](#clôture-d-une-session) après avoir traité les données de la session si vous avez des problèmes de performance.

Le gestionnaire `database` utilise une base de données relationnelle telle que MySQL ou PostgreSQL pour stocker les sessions. C'est un choix populaire parmi de nombreux utilisateurs, car il permet au développeur d'accéder facilement aux données de session au sein d'une application - il s'agit simplement d'une table supplémentaire dans votre base de données.

Cependant, certaines conditions doivent être remplies :

* Vous ne pouvez PAS utiliser de connexion persistante.

<a name="bd-config"></a>
#### Configuration pour le gestionnaire `database`

<a name="bd-config-definition-de-la-table"></a>
##### Définition de la table

Pour utiliser le gestionnaire `database`, vous devez également créer la table que nous avons déjà mentionnée et la définir comme valeur de `session.savePath`. Par exemple, si vous souhaitez utiliser '`blitz_sessions`' comme nom de table, vous devez faire ceci:

```php
<?php

// app/Config/session.php

return [
    'handler' => 'database',
    
    // ---
    
    'savePath' => 'blitz_sessions',
    
    // ---
];
```

<a name="bd-config-creation-de-la-table"></a>
##### Création de la table

Et puis bien sûr, créer la table de la base de données ...

Pour MySQL :

```sql
CREATE TABLE IF NOT EXISTS `blitz_sessions` (
    `id` varchar(128) NOT null,
    `ip_address` varchar(45) NOT null,
    `timestamp` timestamp DEFAULT CURRENT_TIMESTAMP NOT null,
    `data` blob NOT null,
    KEY `blitz_sessions_timestamp` (`timestamp`)
);
```

Pour PostgreSQL :

```sql
CREATE TABLE "blitz_sessions" (
    "id" varchar(128) NOT NULL,
    "ip_address" inet NOT NULL,
    "timestamp" timestamptz DEFAULT CURRENT_TIMESTAMP NOT NULL,
    "data" bytea DEFAULT '' NOT NULL
);

CREATE INDEX "blitz_sessions_timestamp" ON "blitz_sessions" ("timestamp");
```

> **Note**  
> La valeur `id` contient le nom du cookie de session (`config > session.cookie_name`), l'identifiant de session et un délimiteur. Elle doit être augmentée si nécessaire, par exemple lors de l'utilisation de longs identifiants de session.

<a name="bd-config-ajout-d-une-cle-primaire"></a>
##### Ajout d'une clé primaire

Vous devrez également ajouter une CLÉ PRIMAIRE en fonction de votre paramètre $matchIP. Les exemples ci-dessous fonctionnent à la fois sur MySQL et PostgreSQL :

```sql
-- Lorsque $matchIP = true
ALTER TABLE blitz_sessions ADD PRIMARY KEY (id, ip_address) ;

-- Lorsque $matchIP = false
ALTER TABLE blitz_sessions ADD PRIMARY KEY (id) ;

-- Pour supprimer une clé primaire précédemment créée (à utiliser lorsque l'on modifie le paramètre)
ALTER TABLE blitz_sessions DROP PRIMARY KEY ;
```

> **Attention**  
> Si vous n'ajoutez pas la clé primaire correcte, l'erreur suivante peut se produire :  
> ```Uncaught mysqli_sql_exception: Duplicate entry 'blitz_session:***' for key 'blitz_sessions.PRIMARY'```

<a name="bd-config-modification-du-groupe-de-bd"></a>
##### Modification du groupe de base de données

Le groupe de base de données par défaut est utilisé par défaut. Vous pouvez modifier le groupe de base de données à utiliser en remplaçant le paramètre `group` du fichier `app/Config/session.php` par le nom du groupe à utiliser :

```php
<?php

// app/Config/session.php

return [
    // ---
    
    'group' => 'nom_du_groupe',
];
```

<a name="bd-config-configuration-d-une-table-via-klinge"></a>
##### Configuration d'une table de base de données à via Klinge

Si vous préférez ne pas faire tout cela à la main, vous pouvez utiliser la commande `session:table` du client pour générer un fichier de migration pour vous. Pour en savoir plus sur les migrations de bases de données, vous pouvez consulter [la documentation complète sur la migration](/docs/{version}/migrations) :

```bash
php klinge session:table
 
php klinge migrate
```

Cette commande prend en compte les configurations `session.savePath` et `session.matchIP` lorsqu'elle génère le code.

<a name="redis"></a>
### Redis

> **Note**  
> Comme Redis n'a pas de mécanisme de verrouillage exposé, les verrous pour ce gestionnaire sont émulés par une valeur séparée qui est conservée jusqu'à 300 secondes. Par ailleurs, vous pouvez connecter Redis avec le protocole TLS.

Redis est un moteur de stockage typiquement utilisé pour la mise en cache et populaire en raison de ses hautes performances, ce qui est probablement la raison pour laquelle vous utilisez le gestionnaire de session `redis`.

L'inconvénient est qu'il n'est pas aussi omniprésent que les bases de données relationnelles et qu'il nécessite l'installation de l'extension PHP <a href="https://github.com/phpredis/phpredis" target="_blank">phpredis</a> sur votre système, qui n'est pas fournie avec PHP. Il y a de fortes chances que vous n'utilisiez le gestionnaire redis que si vous êtes déjà familier avec Redis et que vous l'utilisez à d'autres fins.

<a name="redis-config"></a>
#### Configuration de pour le gestionnaire `redis`

Tout comme pour les gestionnaires `file` et `database`, vous devez également configurer l'emplacement de stockage de vos sessions via le paramètre `savePath`. Le format ici est un peu différent et compliqué à la fois. Il est mieux expliqué par le fichier README de l'extension phpredis, nous allons donc simplement vous y renvoyer :

* <a href="https://github.com/phpredis/phpredis" target="_blank">https://github.com/phpredis/phpredis</a>

Dans la plupart des cas, une simple paire `hôte/port` devrait suffire :

```php
<?php

// app/Config/session.php

return [
    'handler' => 'redis',
    
    // ---
    
    'savePath' =>  'tcp://localhost:6379',
    
    // ---
];
```

<a name="memcached"></a>
### Memcached

> **Note**  
> Memcached n'ayant pas de mécanisme de verrouillage exposé, les verrous pour ce pilote sont émulés par une valeur séparée qui est conservée jusqu'à 300 secondes.

Le gestionnaire `memcached` est très similaire au gestionnaire `redis` dans toutes ses propriétés, sauf peut-être pour la disponibilité, car l'extension <a href="https://www.php.net/memcached" target="_blank">Memcached</a> de PHP est distribuée via PECL et certaines distributions Linux la rendent disponible sous la forme d'un paquetage facile à installer.

En dehors de cela, et sans aucun parti pris intentionnel pour Redis, il n'y a pas grand chose de différent à dire sur Memcached - c'est aussi un produit populaire qui est généralement utilisé pour la mise en cache et réputé pour sa rapidité.

Toutefois, il convient de noter que la seule garantie donnée par Memcached est que le fait de définir la valeur X pour qu'elle expire après Y secondes entraînera sa suppression après Y secondes (mais pas nécessairement qu'elle n'expirera pas avant ce délai). Cela se produit très rarement, mais doit être pris en compte car cela peut entraîner la perte de sessions.

<a name="memcached-config"></a>
#### Configuration de pour le gestionnaire `memcached`

Le format de `savePath` est assez simple, puisqu'il s'agit simplement d'une paire `hôte/port` :

```php
<?php

// app/Config/session.php

return [
    'handler' => 'memcached',
    
    // ---
    
    'savePath' =>  'localhost:11211',
    
    // ---
];
```

<a name="ajout-de-gestionnaires-de-session-personnalises"></a>
## Ajout de gestionnaires de session personnalisés

<a name="mise-en-oeuvre-du-gestionnaire"></a>
### Mise en œuvre du gestionnaire

Si aucun des pilotes de session existants ne correspond aux besoins de votre application, BlitzPHP vous permet d'écrire votre propre gestionnaire de session. Votre gestionnaire de session personnalisé doit implémenter l'interface intégrée `SessionHandlerInterface` de PHP. Cette interface ne contient que quelques méthodes simples. Par ailleurs, **votre classe DOIT étendre la classe `BlitzPHP\Session\Handlers\BaseHandler`**. Une implémentation de MongoDB ressemble à ce qui suit :

```php
<?php
 
namespace App\Extensions;

use BlitzPHP\Session\Handlers\BaseHandler;
use SessionHandlerInterface;
 
class MongoSessionHandler extends BaseHandler implements SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

> **Note**  
> BlitzPHP n'est pas livré avec un répertoire pour contenir vos extensions. Vous êtes libre de les placer où vous le souhaitez. Dans cet exemple, nous avons créé un répertoire Extensions pour héberger le MongoSessionHandler.

Étant donné que l'objectif de ces méthodes n'est pas facilement compréhensible, examinons rapidement ce que fait chacune d'entre elles :

* La méthode `open` est généralement utilisée dans les systèmes de stockage de session basés sur des fichiers. Comme BlitzPHP est livré avec un gestionnaire de session fichier, vous aurez rarement besoin de mettre quoi que ce soit dans cette méthode. Vous pouvez simplement laisser cette méthode vide.
* La méthode `close`, comme la méthode `open`, peut également être ignorée. Pour la plupart des pilotes, elle n'est pas nécessaire.
* La méthode `read` doit renvoyer la version chaîne des données de session associées à l'identifiant `$sessionId` donné. Vous devez, au besoin, effectuer une sérialisation ou un autre encodage lors de la récupération ou du stockage des données de session dans votre gestionnaire, car BlitzPHP ne le fera pas automatiquement.
* La méthode `write` doit écrire la chaîne `$data` associée à `$sessionId` dans un système de stockage persistant, tel que MongoDB ou un autre système de stockage de votre choix. Là encore, le processus de sérialisation est à votre charge.
* La méthode `destroy` doit supprimer les données associées au `$sessionId` du système de stockage persistant.
La méthode `gc` doit détruire toutes les données de la session qui sont plus anciennes que la durée de vie donnée `$lifetime`, qui est un timestamp UNIX. Pour les systèmes à expiration automatique comme Memcached et Redis, cette méthode peut être laissée vide.

<a name="enregistrement-du-gestionnaire"></a>
### Enregistrement du gestionnaire

Une fois votre gestionnaire implémenté, vous êtes prêt à l'enregistrer auprès de BlitzPHP. Pour ajouter des gestionnaire supplémentaires au backend de session de BlitzPHP, vous pouvez utiliser la méthode `extend` fournie par la [façade](/docs/{version}/facades) `Session`. Vous devez appeler la méthode `extend` à partir de l'événement `app:init` d'un écouteur d'événement

```php
<?php

namespace App\Events;

use App\Extensions\MongoSessionHandler;
use BlitzPHP\Contracts\Event\EventListenerInterface;
use BlitzPHP\Contracts\Event\EventManagerInterface;
use BlitzPHP\Facades\Session;

class Listener implements EventListenerInterface
{
    /**
     * {@inheritDoc}
     */
    public function listen(EventManagerInterface $event): void
    {
        $event->attach('app:init', function () {
            Session::extend('mongo', MongoSessionHandler::class);
        });
    }
}
```

Une fois votre gestionnaire de session enregistré, vous pouvez utiliser définir le paramètre `handler` à `mongo` dans votre fichier de configuration `app/Config/session.php`.