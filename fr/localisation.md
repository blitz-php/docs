---
title: Localisation
---

<a name="introduction"></a>
## Introduction

L’une des meilleures façons pour qu’une application ait une audience plus large est de gérer plusieurs langues. Cela peut souvent se révéler être une tâche gigantesque, mais les fonctionnalités de localisation dans BlitzPHP rendront cela plus facile.

Les fonctions de localisation de BlitzPHP offrent un moyen pratique de récupérer des chaînes dans différentes langues, ce qui vous permet de prendre facilement en charge plusieurs langues dans votre application. Les chaînes de langues peuvent être stockées dans des fichiers situés dans le répertoire `app/Translations` de l'application. Dans ce répertoire, il peut y avoir des sous-répertoires pour chaque langue prise en charge par l'application :

```
/app/
    Translations/
        en/
            messages.php
        fr/
            messages.php
```

<a name="travailler-avec-les-parametres-regionaux"></a>
## Travailler avec les paramètres régionaux

BlitzPHP fournit plusieurs outils pour vous aider à localiser votre application pour différentes langues. Bien que la localisation complète d'une application soit un sujet complexe, il est simple d'échanger des chaînes de caractères dans votre application avec les différentes langues supportées.

<a name="configuration-de-la-langue"></a>
### Configuration de la langue

<a name="reglage-de-la-langue-par-defaut"></a>
#### Réglage de la langue par défaut

Chaque site a une langue/locale par défaut dans laquelle il fonctionne. Celle-ci peut être définie dans le fichier `app/Config/app.php` :

```php
return [
    // ...
    'language' => 'en',
    // ...
];
```

La valeur peut être n'importe quelle chaîne que votre application utilise pour gérer les chaînes de texte et d'autres formats. Il est recommandé d'utiliser un code de langue <a href="http://www.rfc-editor.org/rfc/bcp/bcp47.txt" target="_blank">BCP 47</a>. Il en résulte des codes de langue tels que en-US pour l'anglais américain, ou fr-FR, pour le français. Une introduction plus lisible à ce sujet est disponible sur <a href="https://www.w3.org/International/articles/language-tags/" target="_blank">le site du W3C</a>.

Le système est suffisamment intelligent pour se rabattre sur des codes linguistiques plus génériques si une correspondance exacte ne peut être trouvée. Si le code local est `en-US` et que nous n'avons que des fichiers de langue pour `en`, ceux-ci seront utilisés car il n'existe rien pour la langue plus spécifique `en-US`. Toutefois, si un répertoire de langues existe dans le répertoire `app/Translations/en-US`, il sera utilisé en premier.

<a name="detection-de-la-langue"></a>
### Détection de la langue

> **Attention**  
> La détection de la langue ne fonctionne que pour les requêtes web qui utilisent la classe `ServerRequest`. Les requêtes en ligne de commande ne disposent pas de ces fonctionnalités.

Il existe deux méthodes pour détecter la locale correcte lors de la requête.

1. [Négociation de contenu](#negociation-de-contenu) : La première méthode est une méthode "`set and forget`" qui effectue automatiquement [la négociation de contenu](/docs/{version}/negociation) pour vous afin de déterminer la langue à utiliser. 
2. [Dans les routes](#dans-les-routes) : La deuxième méthode vous permet de spécifier un segment dans vos routes qui sera utilisé pour définir la langue.

Si vous avez besoin de définir directement les paramètres linguistiques, consultez la section [Réglage de la langue actuelle](#reglage-de-la-langue-actuelle).

<a name="negociation-de-contenu"></a>
#### Négociation de contenu

Vous pouvez configurer la négociation de contenu pour qu'elle se fasse automatiquement en définissant deux paramètres supplémentaires dans `app/Config/app.php`. La première valeur indique à la ServerRequest que nous voulons négocier une locale, il suffit donc de lui donner la valeur `true` :

```php
return [
    // ...
    'negotiate_locale' => true,
    // ...
];
```

Lorsque cette option est activée, le système négocie automatiquement la langue correcte sur la base d'un tableau de langues que vous avez défini dans `supported_locales`. Si aucune correspondance n'est trouvée entre les langues que vous prenez en charge et la langue demandée, le premier élément de `supported_locales` sera utilisé. Dans l'exemple suivant, la langue `en` sera utilisée si aucune correspondance n'est trouvée :

```php
return [
    // ...
    'supported_locales' => ['en', 'fr', 'es'],
    // ...
];
```

<a name="dans-les-routes"></a>
#### Dans les routes

La seconde méthode utilise un caractère générique personnalisé pour détecter la locale souhaitée et la définir dans la requête. Le caractère générique `{locale}` peut être placé comme un segment dans votre route. S'il est présent, le contenu du segment correspondant sera votre locale :

```php
Route::get('{locale}/books', 'App\Controllers\BooksController::index');
```

Dans cet exemple, si l'utilisateur tente de se rendre sur le site `http://example.com/fr/books`, la locale sera définie comme `fr`, à condition qu'elle soit configurée comme une locale valide.

Si la valeur ne correspond pas à une locale valide telle que définie dans `$supported_locales` dans `app/Config/app.php`, la locale par défaut sera utilisée à sa place, à moins que vous n'ayez choisi de n'utiliser que les locales supportées définies dans le fichier de configuration du `app/Config/routing.php` :

```php
return [
    // ...
    'use_supported_locales_only' => true,
    // ...
];
```

<a name="reglage-de-la-langue-actuelle"></a>
### Réglage de la langue actuelle

<a name="langue-de-la-requete-entrante"></a>
#### Langue de la requête entrante

Si vous souhaitez définir la langue directement, vous pouvez utiliser la méthode `setLocale()` de la classe [Request](/docs/{version}/requetes) :

```php
/** @var \BlitzPHP\Http\ServerRequest $request */
$request->setLocale('ja');
```

Avant de définir la langue, vous devez définir les langues valides. En effet, toute tentative de définition de langue non valide entraînera la définition de la langue par défaut.

Par défaut, les langues valides sont définis à travers la variable `supported_locales` dans le fichier `app/Config/app.php` :

```php
return [
    // ...
    'supported_locales' => ['en', 'fr', 'es'],
    // ...
];
```

<a name="langue-du-traducteur"></a>
#### Langue du traducteur

La classe `Translate` utilisée dans la fonction `lang()` contient également la langue actuelle, qui est fixée à la langue de la requête entrante lors de l'instanciation. Si vous souhaitez modifier la langue après l'instanciation de la classe de traduction, utilisez la méthode `Translate::setLocale()`.

```php
/** @var \BlitzPHP\Translator\Translate $translator */
$translator = service('translator');
$translator->setLocale('ja');
```

<a name="recuperation-de-la-langue-actuelle"></a>
### Récupération de la langue actuelle

La langue actuelle peut toujours être récupérée à partir de l'objet Request, via la méthode `getLocale()`. Si votre contrôleur hérite de `BlitzPHP\Controllers\BaseController`, ceci sera disponible via `$this->request` :

```php
namespace App\Controllers;

class UserController extends AppController
{
    public function index()
    {
        $locale = $this->request->getLocale();
    }
}
```

Vous pouvez également utiliser la classe [Services](/docs/{version}/services) pour récupérer la requête en cours :

```php
$locale = service('request')->getLocale();
```

<a name="localisation-des-langues"></a>
## Localisation des langues

<a name="Création de fichiers de langue"></a>
### Création de fichiers de langue

Les chaînes de langues sont stockées dans le répertoire `app/Translations`, avec un sous-répertoire pour chaque langue prise en charge (locale) :

```
/app/
    Translations/
        en/
            app.php
        fr/
            app.php
```

> **Note**  
> Les fichiers de langue n'ont pas d'espace de noms.

Les langues n'ont pas de convention de nommage spécifique à respecter. Le fichier doit être nommé logiquement pour décrire le type de contenu qu'il contient. Par exemple, imaginons que vous souhaitiez créer un fichier contenant des messages d'erreur. Vous pouvez le nommer simplement : **`errors.php`**

Dans le fichier, vous devez renvoyer un tableau, où chaque élément du tableau a un code langue et peut avoir une chaîne à renvoyer :

```php
return [
     'languageKey' => 'Le message à afficher.',
];
```

> **Note**  
> Vous ne pouvez pas utiliser de points (`.`) au début et à la fin des clés de langue.

Il prend également en charge les définitions imbriquées :

```php
return [
    'languageKey' => [
        'nested' => [
            'key' => 'Le message à afficher.',
        ],
    ],
];
```

```php
return [
    'emailMissing' => 'Vous devez soumettre une adresse e-mail', 
    'urlMissing' => 'Vous devez soumettre une URL',
    'usernameMissing' => 'Vous devez soumettre un nom d\'utilisateur', 
    'nested' => [
        'error' => [
            'message' => 'Un message d\'erreur spécifique',
        ],
    ],
];
```

<a name="utilisation-de-base"></a>
### Utilisation de base

Vous pouvez utiliser la fonction d'aide `lang()` pour extraire du texte de n'importe quel fichier de langue, en passant le nom du fichier et le code de la langue comme premier paramètre, séparés par un point (`.`).

Par exemple, pour charger la chaîne `emailMissing` à partir du fichier de langue `errors.php`, vous devez procéder comme suit :

```php
echo lang('errors.emailMissing');
```

Pour une définition imbriquée, vous devez procéder comme suit :

```php
echo lang('errors.nested.error.message');
```

Si la clé de langue demandée n'existe pas dans le fichier pour la langue actuelle (après la [langue de secours](#langue-de-secours)), la chaîne sera renvoyée, inchangée. Dans cet exemple, elle renverrait `errors.emailMissing` ou `errors.nested.error.message` si elle n'existait pas.

<a name="remplacement-des-parametres"></a>
#### Remplacement des parametres

> **Note**  
> Les fonctions suivantes requièrent toutes que l'extension <a href="https://www.php.net/manual/en/book.intl.php" target="_blank">intl</a> soit chargée sur votre système pour fonctionner. Si l'extension n'est pas chargée, aucun remplacement ne sera tenté. Une excellente vue d'ensemble est disponible sur le site <a href="https://www.sitepoint.com/localization-demystified-understanding-php-intl/" target="_blank">Sitepoint</a>.

Vous pouvez passer un tableau de valeurs pour remplacer les caractères de remplacement dans la chaîne de la langue en tant que deuxième paramètre de la fonction `lang()`. Cela permet d'effectuer des traductions de nombres et des mises en forme très simples :

```php
// Le fichier de langue, tests.php : 

return [
    'apples' => "J'ai {0, number} pommes", 
    'men' => 'Les {1, number} premiers hommes ont surpassé les {0, number} restants', 
    'namedApples' => "J'ai {nombre_de_pommes, number, integer} pommes", 
]; 

// Affiche "J'ai 3 pommes" 
echo lang('tests.apples', [3]);
```

Le premier élément de l'espace réservé correspond à l'indice de l'élément dans le tableau, s'il est numérique :

```php
// Affiche "Les 23 premiers hommes ont surpassé les 20 restants"

echo lang('tests.men', [20, 23]);
```

Si vous le souhaitez, vous pouvez également utiliser des clés nommées pour faciliter l'organisation :

```php
// Affiche "J'ai 3 pommes" 
echo lang('tests.namedApples', ['nombre_de_pommes' => 3]);
```

Il est évident que vous pouvez faire plus qu'un simple remplacement de nombres. Selon <a href="https://unicode-org.github.io/icu-docs/apidoc/released/icu4c/classMessageFormat.html#details" target="_blank">la documentation officielle de l'ICU</a> pour la bibliothèque sous-jacente, les types de données suivants peuvent être remplacés :

* numbers - integer, currency, percent
* dates - short, medium, long, full
* time - short, medium, long, full
* spellout - spells out numbers (ex., 35 devient treinte cinq)
* ordinal
* duration

Voici quelques exemples :

```php
// Le fichier de langue, tests.php 

return [
    'shortTime' => "Il est maintenant {0, time, short}.", 
    'mediumTime' => "Il est maintenant {0, time, medium}.", 
    'longTime' => "Il est maintenant {0, time, long}.", 
    'fullTime' => "Il est maintenant {0, time, full}.", 
    'shortDate' => 'La date est maintenant {0, date, short}.', 
    'mediumDate' => 'La date est maintenant {0, date, medium}.', 
    'longDate' => 'La date est maintenant {0, date, long}.', 
    'fullDate' => 'La date est maintenant {0, date, full}.', 
    'spelledOut' => '35 est {0, spellout}', 
    'ordinal' => "L'ordinal est {0, ordinal}",
    'duration' => 'Il a été {0, duration}', 
];

// Affiche "Il est maintenant 11:18 PM" 
echo lang('tests.shortTime', [time()]); 

// Affiche "Il est maintenant 11:18:50 PM" 
echo lang('tests.mediumTime', [time()]); 

// Affiche "Il est maintenant 11:19:09 PM CDT" 
echo lang('tests.longTime', [time()]); 

// Affiche "Il est maintenant 11:19:26 PM Central Daylight Time" 
echo lang('tests.fullTime', [time()]);

//---------------------------

// Affiche "La date est maintenant 10/15/24" 
echo lang('tests.shortDate', [time()]); 

// Affiche "La date est maintenant 15 oct 2024" 
echo lang('tests.mediumDate', [time()]); 

// Affiche "La date est maintenant 15 octobre 2024" 
echo lang('tests.longDate', [time()]); 

// Affiche "La date est maintenant mardi 15 octobre 2024" 
echo lang('tests.fullDate', [time()]) ;

//---------------------------

// Affiche "35 est trente-cinq" 
echo lang('tests.spelledOut', [35]); 

// Affiche "Il a été 408,676:24:35" 
echo lang('Tests.ordinal', [time()]) ;
```

Vous devriez vous renseigner sur la classe MessageFormatter et sur le formatage ICU sous-jacent afin d'avoir une meilleure idée de ses capacités, comme le remplacement conditionnel, la pluralisation, etc. Les deux liens mentionnés plus haut vous donneront une excellente idée des options disponibles.

<a name="specification-de-la-langue"></a>
#### Spécification de la langue

Pour spécifier une langue différente à utiliser lors du remplacement des paramètres, vous pouvez passer la langue en tant que troisième paramètre de la fonction `lang()`.

```php
// Affiche "Il est 23:21:28 GMT-5"
echo lang('tests.longTime', [time()], 'ru-RU');

// Affiche "£7.41"
echo lang('{price, number, currency}', ['price' => 7.41], 'en-GB');

// Affiche "$7.41"
echo lang('{price, number, currency}', ['price' => 7.41], 'en-US');
```

Si vous souhaitez modifier la langue actuelle, consultez la section [langue du traducteur](#langue-du-traducteur).

<a name="tableaux-imbriques"></a>
#### Tableaux imbriqués

Les fichiers de langue permettent également d'imbriquer les tableaux pour faciliter le travail avec les listes, etc...

```php

// Translations/en/fruit.php

return [
    'list' => [
        'Pommes',
        'Bananes',
        'Ananas',
        'Citrons',
        'Oranges',
        'Fraises',
    ],
];

// Affiche "Pommes, Bananes, Ananas, Citrons, Oranges, Fraises"
echo implode(', ', lang('fruit.list'));
```

<a name="langue-de-secours"></a>
### Langue de secours

Si vous disposez d'un ensemble de messages pour une langue donnée, par exemple `app/Translations/en/app.php`, vous pouvez ajouter des variantes linguistiques pour cette langue, chacune dans son propre dossier, par exemple `app/Translations/en-US/app.php`.

Vous ne devez fournir des valeurs que pour les messages qui seraient localisés différemment pour cette variante de locale. Toute définition de message manquante sera automatiquement extraite de la locale principale.

Mieux encore : la localisation peut revenir à l'anglais (**en**), au cas où de nouveaux messages seraient ajoutés au framework et que vous n'auriez pas encore eu l'occasion de les traduire pour votre région.

Ainsi, si vous utilisez la locale `fr-CA`, un message localisé sera d'abord recherché dans le répertoire `app/Translations/fr-CA`, puis dans le répertoire `app/Translations/fr`, et enfin dans le répertoire `app/Translations/en`.

<a name="traductions-des-messages-systeme"></a>
### Traductions des messages système

Nous disposons d'un ensemble "officiel" de traductions des messages du système dans <a href="https://github.com/blitz-php/translations" target="_blank">leur propre dépôt GitHub</a>.

Vous pouvez télécharger ce dépôt et copier son dossier `Translations` dans votre dossier `app`. Les traductions incorporées seront automatiquement récupérées.

Une autre solution (la meilleure) consisterait à exécuter la commande suivante dans votre projet : 

```shell
composer require blitz-php/translations`
```

Les messages traduits seront automatiquement pris en charge, car les dossiers de traduction sont mappés de manière appropriée.

<a name="remplacement-des-traductions-des-messages-systeme"></a>
### Remplacement des traductions des messages système

Le framework fournit les [traductions des messages du système](#traductions-des-messages-systeme), et les paquets que vous avez installés peuvent également fournir les traductions des messages.

Si vous souhaitez remplacer certains messages de langue, créez des fichiers de langue dans le répertoire `app/Translations`. Ensuite, ne renvoyez dans le fichier que le tableau que vous souhaitez remplacer.

<a name="generer-des-fichiers-de-traduction-par-commande"></a>
### Générer des fichiers de traduction par commande

Vous pouvez générer et mettre à jour automatiquement des fichiers de traduction dans le dossier de votre application. La commande recherchera l'utilisation de la fonction `lang()`, combinera les clés de traduction actuelles dans `app/Translations` en définissant la locale `language` à partir de `app/Config/app.php`. Après l'opération, vous devez traduire vous-même les clés de langue. La commande est capable de reconnaître les clés imbriquées normalement `File.array.nested.text`. Les clés précédemment enregistrées ne changent pas.

```shell
php klinge lang:find
```

```php
<?php

// app/Controllers/Students/Sample.php
$message  = lang('Text.info.success');
$message2 = lang('Text.paragraph');

// Ce qui suit sera enregistré dans app/Translations/en/Text.php
return [
    'info' => [
        'success' => 'Text.info.success',
    ],
    'paragraph' => 'Text.paragraph',
];
```

> **Note**  
> Lorsque la commande analyse les dossiers, `app/Translations` est ignoré.

Les fichiers de langue générés ne seront probablement pas conformes à vos normes de codage. Il est recommandé de les formater. Par exemple, exécuter `vendor/bin/php-cs-fixer fix ./app/Translations` si `php-cs-fixer` est installé.

Avant la mise à jour, il est possible de prévisualiser les traductions trouvées par la commande :

```shell
php klinge lang:find --verbose --show-new
```

La sortie détaillée de `--verbose` affiche également une liste des clés non valides. Par exemple :

```
...

Files found: 10
New translates found: 30
Bad translates found: 5
+------------------------+---------------------------------+
| Bad Key                | Filepath                        |
+------------------------+---------------------------------+
| ..invalid_nested_key.. | app/Controllers/Translation.php |
| .invalid_key           | app/Controllers/Translation.php |
| TranslationBad         | app/Controllers/Translation.php |
| TranslationBad.        | app/Controllers/Translation.php |
| TranslationBad...      | app/Controllers/Translation.php |
+------------------------+---------------------------------+

All operations done!
```

Pour une recherche plus précise, indiquez le lieu ou le répertoire à analyser.

```shell
php klinge lang:find --dir Controllers/Students --locale en --show-new
```

Des informations détaillées peuvent être obtenues en exécutant la commande :

```shell
php klinge lang:find --help
```