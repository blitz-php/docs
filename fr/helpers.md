---
title: Helpers
---

<a name="introduction"></a>
## Introduction

Les helpers sont des assistants qui vous aident avec les tâches. Chaque fichier d’assistance est simplement un ensemble de fonctions dans une catégorie particulière. Il y a des **helpers d'URL**, qui aident à créer des liens, il y a des **helpers de formulaire** qui aident vous créez des éléments de formulaire, les **helpers de texte** effectuent diverses mises en forme de texte routines, les **helpers de fichiers** pour vous aider traiter des dossiers, etc.

Contrairement à la plupart des autres systèmes de BlitzPHP, les assistants ne sont pas écrits dans un format orienté objet. Ce sont des fonctions simples et procédurales. Chaque fonction d’assistance effectue une tâche spécifique et peut au besoin dépendre d’une autre fonction.

Les helpers ne sont pas chargés par défaut donc, la première étape de l'utilisation d'un helper consiste à le charger. Une fois chargé, il devient disponible dans votre [contrôleur](/docs/{version}/controleurs), votre modèle et vos [vues](/docs/{version}/vues).

Les fichiers d'aide sont généralement stockés dans les répertoire `/vendor/blitz-php/framework/src/Helpers` (assistants natifs du framework) ou `/app/Helpers` (vos propres assistants).

<a name="chargement-d-un-helper"></a>
## Chargement d'un helper

Le chargement d’un fichier d’assistance se fait assez simplement à l’aide de la méthode suivante:

```php
<?php

helper('name');
```

Où `name` est le nom de fichier d’assistance, sans le «**.php**»

> **Attention**  
> Les noms des fichiers d’assistance BlitzPHP sont tous en minuscules. Par conséquent, `helper('Name')` ne fonctionnera pas sur les systèmes de fichiers sensibles à la casse comme Linux.

Par exemple, pour charger le fichier d’assistance aux cookies, nommé `cookies.php`, procédez comme suit :

```php
<?php

helper('cookie');
```

> **Note**  
> La méthode de chargement d’assistance ci-dessus ne renvoie pas de valeur, donc N’essayez pas de l’affecter à une variable. Il suffit de l’utiliser comme indiqué.

<a name="charger-plusieurs-helpers"></a>
### Charger plusieurs helpers

Si vous souhaitez charger plusieurs helpers, vous pourriez être tenté d'appeler la fonction `helper()` plusieurs fois. Bien-sur ça marche mais vous pouvez faire mieux. Transmettez juste un tableau contenant la liste des helpers à charger et BlitzPHP se débrouillera tout seul :

```php
<?php

helper(['cookie', 'date']);
```

<a name="chargement-dans-un-controleur"></a>
### Chargement dans un contrôleur

Un assistant peut être chargé n'importe où dans vos méthodes de contrôleur (ou même dans vos fichiers de vues, bien que ce ne soit pas une bonne pratique), à condition de le charger avant de l'utiliser.

Vous pouvez charger vos assistants dans le constructeur de votre contrôleur afin qu'ils deviennent automatiquement disponibles dans n'importe quelle méthode, ou vous pouvez charger un assistant dans une méthode spécifique qui en a besoin.

<a name="chargement-a-partir-d-emplacements-non-standard"></a>
### Chargement à partir d'emplacements non standard

Les helpers peuvent être chargés à partir de répertoires différents de `app/Helpers` et `vendor/blitz-php/framework/src/Helpers`, à condition que ce chemin puisse être trouvé via un namespace qui a été configuré dans la section PSR-4 du [fichier de configuration de l'autoloader](/docs/{version}/autoloader).

Vous devez préfixer le nom du helper avec le namespace dans lequel il peut se trouver. Dans ce répertoire avec le namespace, l'autoloader s'attend à ce qu'il existe un sous-dossier nommé `Helpers`. Un exemple aidera à comprendre cela.

Pour cet exemple, supposons que nous avons regroupé tout notre code lié au blog dans son propre namespace, `Exemple\Blog`. Les fichiers existent sur notre serveur à `Modules/Blog/`. Nous placerions donc nos helpers pour le module blog dans `Modules/Blog/Helpers/`. Un fichier `blog.php` se trouverait donc dans le dossier `Modules/Blog/Helpers/`. Dans notre contrôleur, nous pourrions utiliser la commande suivante pour charger cet helper :

```php 
<?php

helper('Example\Blog\blog');
```

Vous pouvez également utiliser la manière suivante :

```php 
<?php

helper('Example\Blog\Helpers\blog');
```

> **Note**  
> Les fonctions contenues dans les fichiers chargés de cette manière ne sont pas véritablement dotées d'un namespace. Le namespace est simplement utilisé comme moyen pratique de localiser les fichiers.

<a name="chargement-automatique-des-helpers"></a>
### Chargement automatique des helpers

Si vous vous rendez compte que vous auriez besoin d'un helper particulier dans tous les modules de votre application, vous pouvez dire à BlitzPHP de le charger automatiquement lors de l'initialisation du système. Pour cela, ouvrez le fichier `/app/Config/autoload.php` et ajoutez le helper désiré au tableau `helpers`.

```php 
return [
    // ...
    'helpers' => [
        'cookie', 'date'
    ],
    // ...
];
```

<a name="utilisation-d-un-helper"></a>
## Utilisation d'un helper

Une fois que vous avez chargé le helper contenant la fonction que vous avez l'intention d'utiliser, vous l'appellerez comme vous le feriez pour une fonction PHP standard.   
Par exemple, pour obtenir le pluriel d'un mot à l'aide de la fonction `plural()` dans l'un de vos fichiers d'affichage, vous devez procéder comme suit:

```php
<?php echo plural('member');
```

<a name="creez-votre-propre-helper"></a>
## Créez votre propre helper

Bien que BlitzPHP possède des helpers plus ou moins remarquables, il peut arriver que vous ayez besoin des fonctions spécifiques à votre projet. Vous devez donc créer vos propres helpers pour étendre les fonctionnalités natives du framework.  
Pour créer un helper, créez un fichier dans votre dossier `/app/Helpers` et écrivez-y les fonctions donc vous auriez besoin.   
Dans l'exemple ci-dessous, nous créerons un helper pour les fonctions propres à une application quelconque. Nous appellerons cet assistant `fn_app`; on aura donc un fichier `/app/Helpers/fn_app.php`.  

```php
<?php 
function isFormat(string $str, string $type='id'): bool
{
	$type = strtolower($type);
		
	if (!in_array($type, array('pwd','id'))) {
		return false;
	}
		
	if ($type=='pwd' && strlen($str) >= 6 && strlen($str) <= 30) {
		return true;
	}
	   
	if ($type == 'id' && preg_match('/[a-zA-z0-9]{12}/', $str)) {
		return true;
	}
		
    return false;
}

function simpleTel(string $tel): string 
{
	$tel = htmlspecialchars($tel);
	return str_replace(['+','.',' ','-',',','_'], '', $tel);
}
```

Pour bénéficier l'une de ces fonctions dans votre application, vous n'auriez qu'à charger le helper dans votre contrôleur en faisant `helper('fn_app');` puis utiliser à votre guise.

> **Note**  
> Vos helpers peuvent avoir le même nom que les helpers interne du framework. Dans ce cas les deux fichiers (le fichier système et le votre) seront chargées simultanément. Néanmoins vous devez vous rassurer de n'avoir aucune fonction ayant le même nom qu'une fonction définies par BlitzPHP ou un package Composer car sinon vous obtiendrai une erreur du genre "**Can't redefined function xyz**". Vous pourriez empêcher l'affichage de cet erreur en mettant votre fonction dans un bloc `if (!function_exist('xyz')) { ... }`, dans ce cas, votre fonction sera tout simplement ignorée si elle a déjà étée créer ailleurs.  
  
<a name="etendre-un-helper"></a>
### "Etendre" un helper

Pour « étendre » les helpers, créez un fichier dans votre dossier `app/Helpers/` avec un nom identique à celui du helpers existant.  

Si tout ce que vous avez à faire est d'ajouter des fonctionnalités à un helper existant - peut-être ajouter une fonction ou deux, ou modifier le fonctionnement d'une fonction particulière - alors il est exagéré de remplacer l'intégralité du helper par votre version. Dans ce cas, il est préférable de simplement « étendre » le helper.

> **Note**  
> Le terme « étendre » est utilisé de manière vague puisque les fonctions de helper sont procédurales et discrètes et ne peuvent pas être étendues au sens programmatique traditionnel. Sous le capot, cela vous donne la possibilité d’ajouter ou de remplacer les fonctions fournies par un helper.  
  
Par exemple, pour étendre le helper natif `array`, vous créerez un fichier nommé `app/Helpers/array.php` et ajouterez ou remplacerez des fonctions :

```php
<?php
// any_in_array() n'est pas dans le helper "array", il définit donc une nouvelle fonction
function any_in_array($needle, $haystack)
{
    $needle = is_array($needle) ? $needle : [$needle];

    foreach ($needle as $item) {
        if (in_array($item, $haystack, true)) {
            return true;
        }
    }

    return false;
}

// random_element() est inclus dans le helper "array", il remplace donc la fonction native
function random_element($array)
{
    shuffle($array);

    return array_pop($array);
}
```

> **Attention**  
> Ne spécifiez pas le namespace App\Helpers.

La fonction `helper()` analysera tous les namespaces PSR-4 définis dans `app/Config/autoload.php` et chargera dans TOUS les helpers correspondants du même nom. Cela permet de charger les helpers de n’importe quel module, ainsi que tous ceux que vous avez créés spécifiquement pour cette application. L'ordre de chargement est le suivant :
1. `app/Helpers` - Les fichiers se trouvant ici sont toujours chargés en premier. 
2. `{namespace}/Helpers` - Tous les namespaces sont parcourus en boucle dans l'ordre dans lequel ils sont définis. 
3. `vendor/blitz-php/framework/src/Helpers` - Les fichiers de base sont chargés en dernier
