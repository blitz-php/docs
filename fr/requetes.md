---
title: Requêtes HTTP
---

<a name="introduction"></a>
## Introduction

La classe `BlitzPHP\Http\Request` de BlitzPHP fournit un moyen orienté objet d'interagir avec la requête HTTP actuelle traitée par votre application ainsi que de récupérer les données, les cookies et les fichiers soumis avec la requête.

<a name="acceder-a-la-requete"></a>
## Accéder à la requête

Une instance de la classe de `Request` est déjà remplie pour vous si la classe actuelle est un descendant de la classe `BlitzPHP\Controllers\BaseController` et est accessible en tant que propriété de classe: 

```php
<?php

class UserController extends AppController
{
    public function index()
    {
    	if ($this->request->isAjax()) {
        	// ...
        }
    }
}
```

Si vous n'êtes pas dans un contrôleur et avez besoin d'accéder à l'objet `Request` de l'application, vous pouvez le récupérer en utilisant le [fournisseur de services](/docs/{version}/services) ou le [conteneur d'injection de dépendances](/docs/{version}/conteneur).

```php
$request = service('request');
```

Il est cependant préférable de passer la requête en tant que dépendance si la classe est autre chose que le contrôleur, où vous pouvez l'enregistrer en tant que propriété de classe.

> **Note**  
> Dans la suite de ce chapitre nous supposerons que la variable `$request` est une instance de la classe `BlitzPHP\Http\Request`. Si vous êtes dans un contrôleur lors de l’exécution de votre code, vous devez utiliser la propriété d'instance `$this->request`.

<a name="interagir-avec-la-requete"></a>
## Interagir avec la requête

<a name="chemin-de-requete-hote-et-methode"></a>
### Chemin de requête, hôte et méthode

<a name="recuperation-du-chemin-de-la-requete"></a>
#### Récupération du chemin de la requête

La méthode `path()` renvoie les informations de chemin de la requête. Ainsi, si la requête entrante cible `http://example.com/foo/bar`, la méthode `path()` renverra `foo/bar` :

```php
$uri = $request->path();
```

<a name="inspection-du-chemin-route-de-la-requete"></a>
#### Inspection du chemin/route de la requête

La méthode `pathIs()` vous permet de vérifier que le chemin de la requête entrante correspond à un modèle donné. Vous pouvez utiliser le caractère `*` comme caractère générique lorsque vous utilisez cette méthode :

```php
if ($request->pathIs('admin/*')) {
    // ...
}
```

À l'aide de la méthode `routeIs()`, vous pouvez déterminer si la requête entrante correspond à une route nommée :

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

<a name="recuperation-de-l-url-de-la-requete"></a>
#### Récupération de l'URL de la requête

Pour récupérer l'URL complète de la demande entrante, vous pouvez utiliser les méthodes `url()` ou `fullUrl()`. La méthode `url()` renverra l'URL sans la chaîne de paramètres de requête, tandis que la méthode `fullUrl()` inclut la chaîne de paramètres de requête :

```php
$url = $request->url();
 
$urlWithQueryString = $request->fullUrl();
```

Si vous souhaitez ajouter des paramètres de chaîne de requête à l'URL actuelle, vous pouvez appeler la méthode `fullUrlWithQuery()`. Cette méthode fusionne le tableau donné de variables de chaîne de requête avec la chaîne de requête actuelle :

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

Si vous souhaitez obtenir l'URL actuelle sans paramètre de chaîne de requête donné, vous pouvez utiliser la méthode `fullUrlWithoutQuery()` :

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="recuperation-de-l-hote-de-la-requete"></a>
#### Récupération de l'hôte de la requête

Vous pouvez récupérer "l'hôte" de la requête entrante via les méthodes `host()`, `httpHost()` et `SchemeAndHttpHost()` :

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

<a name="recuperation-de-l-hote-de-la-requete"></a>
#### Récupération de l'hôte de la requête

La méthode `method()` renverra le verbe HTTP pour la requête. Vous pouvez utiliser la méthode `isMethod()` pour vérifier que le verbe HTTP correspond à une chaîne donnée :

```php
$method = $request->method();
 
if ($request->isMethod('post')) {
    // ...
}
```

<a name="entetes-de-la-requete"></a>
### Entêtes de la requête

Vous pouvez récupérer un en-tête de requête à partir de l'instance `BlitzPHP\Http\Request` à l'aide de la méthode `header()`. Si l'en-tête n'est pas présent sur la requête, `null` sera renvoyé. Cependant, la méthode `header()` accepte un deuxième argument facultatif qui sera renvoyé si l'en-tête n'est pas présent sur la requête :

```php
$value = $request->header('X-Header-Name');
 
$value = $request->header('X-Header-Name', 'default');
```

La méthode `hasHeader()` peut être utilisée pour déterminer si la requête contient un en-tête donné :

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

Pour plus de commodité, la méthode `bearerToken()` peut être utilisée pour récupérer un token à partir de l'en-tête `Authorization`. Si aucun en-tête de ce type n'est présent, une chaîne vide sera renvoyée :
  
```php
$token = $request->bearerToken();
```

<a name="adresse-ip-de-la-requete"></a>
### Adresse IP de la requête

La méthode `ip()` peut être utilisée pour récupérer l'adresse IP du client qui a fait la requête à votre application :

```php
$ipAddress = $request->ip();
```

<a name="negociation-de-contenu"></a>
### Négociation de contenu

BlitzPHP fournit plusieurs moyens pour inspecter les types de contenu demandés par la requête entrante via l'en-tête `Accept`. Tout d'abord, la méthode `getAcceptableContentTypes()` renverra un tableau contenant tous les types de contenu acceptés par la requête :

```php
$contentTypes = $request->getAcceptableContentTypes();
```

La méthode `accepts()` accepte un tableau de types de contenu et renvoie `true` si l'un des types de contenu est accepté par la requête. Sinon, `false` sera renvoyé :

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

Vous pouvez utiliser la méthode `prefers()` pour déterminer quel type de contenu parmi un tableau donné de types de contenu est le plus préféré par la demande. Si aucun des types de contenu fournis n'est accepté par la requête, `null` sera renvoyé :

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Étant donné que de nombreuses applications ne servent que du HTML ou du JSON, vous pouvez utiliser la méthode `expectsJson()` pour déterminer rapidement si la requête entrante attend une réponse JSON :

```php
if ($request->expectsJson()) {
    // ...
}
```

<a name="requetes-psr7"></a>
### Requêtes PSR7

L’objet `Request` de BlitzPHP implémente <a href="https://www.php-fig.org/psr/psr-7/" target="_blank">l’interface PSR-7 ServerRequestInterface</a> facilitant l’utilisation des librairies en-dehors de BlitzPHP. Ainsi, en plus de toutes les méthodes présentées dans cette documentation, vous pouvez utiliser toutes <a href="https://github.com/php-fig/http-message/blob/master/src/ServerRequestInterface.php" target="_blank">les méthodes de l'interface ServerRequestInterface</a>

<a name="donnees"></a>
## Données

<a name="recuperation-des-donnees"></a>
### Récupération des données

<a name="recuperation-de-toutes-les-donnees-d-entree"></a>
#### Récupération de toutes les données d'entrée

Vous pouvez récupérer toutes les données d'entrée de la requête entrante sous forme de `tableau` en utilisant la méthode `all()`. Cette méthode peut être utilisée indépendamment du fait que la requête entrante provienne d'un formulaire HTML ou d'une requête XHR :

```php
$input = $request->all();
```

À l'aide de la méthode `collect()`, vous pouvez récupérer toutes les données d'entrée de la requête entrante sous forme de [collection](/docs/{version}/collection) :

```php
$input = $request->collect();
```

La méthode `collect()` vous permet également de récupérer un sous-ensemble des entrées de la requête entrante sous forme de collection :

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

<a name="recuperation-d-une-valeur-d-entree"></a>
#### Récupération d'une valeur d'entrée

À l’aide de quelques méthodes simples, vous pouvez accéder à toutes les entrées utilisateur de votre instance `BlitzPHP\Http\Request` sans vous soucier du verbe HTTP utilisé pour la requête. Quel que soit le verbe HTTP, la méthode de `input()` peut être utilisée pour récupérer les entrées de l'utilisateur :

```php
$name = $request->input('name');
```

Vous pouvez transmettre une valeur par défaut comme deuxième argument à la méthode de `input()`. Cette valeur sera renvoyée si la valeur d'entrée demandée n'est pas présente sur la requête :

```php
$name = $request->input('name', 'Sally');
```
  
Lorsque vous travaillez avec des formulaires contenant des entrées de tableau, utilisez la notation « point » pour accéder aux tableaux :

```php
$name = $request->input('products.0.name');
 
$names = $request->input('products.*.name');
```

Vous pouvez appeler la méthode `input()` sans aucun argument afin de récupérer toutes les valeurs d'entrée sous forme de tableau associatif :

```php
$input = $request->input();
```

<a name="recuperation-des-entrees-de-la-chaine-de-requete"></a>
#### Récupération des entrées de la chaîne de requête

Alors que la méthode `input()` récupère toutes les données de la requête (y compris la chaîne de requête), la méthode de `query()` récupère uniquement les valeurs de la chaîne de requête :

```php
$name = $request->query('name');
```

Si les données de valeur de chaîne de requête demandées ne sont pas présentes, le deuxième argument de cette méthode sera renvoyé :

```php
$name = $request->query('name', 'Helen');
```

Vous pouvez appeler la méthode `query()` sans aucun argument afin de récupérer toutes les valeurs de la chaîne de requête sous forme de tableau associatif :

```php
$query = $request->query();
```

<a name="recuperation-des-valeurs-d-entree-json"></a>
#### Récupération des valeurs d'entrée JSON

Lors de l'envoi de requêtes JSON à votre application, vous pouvez accéder aux données JSON via la méthode de `input()` tant que l'en-tête `Content-Type` de la requête est correctement défini sur `application/json`. Vous pouvez même utiliser la syntaxe « point » pour récupérer les valeurs imbriquées dans des tableaux/objets JSON :

```php
$name = $request->input('user.name');
```

<a name="recuperation-des-valeurs-d-entree-pouvant-etre-une-chaine"></a>
#### Récupération de valeurs d'entrée pouvant être une chaîne

Au lieu de récupérer les données d'entrée de la requête sous forme de chaîne primitive, vous pouvez utiliser la méthode `string()` pour récupérer les données de requête en tant qu'instance de `BlitzPHP\Utilities\String\Stringable` :

```php
$name = $request->string('name')->trim();
```

<a name="recuperation-des-valeurs-d-entree-booleennes"></a>
#### Récupération des valeurs d'entrée booléennes

Lorsqu'elle traite des éléments HTML tels que des cases à cocher, votre application peut recevoir des valeurs « véridiques » qui sont en réalité des chaînes. Par exemple, "true" ou "on". Pour plus de commodité, vous pouvez utiliser la méthode `boolean()` pour récupérer ces valeurs sous forme booléenne. La méthode `boolean()` renvoie `true` pour 1, "1", true, "true", "on" et "yes". Toutes les autres valeurs renverront `false` :

```php
$remember = $request->boolean('remember');
```

<a name="recuperation-des-valeurs-d-entree-de-date"></a>
#### Récupération des valeurs d'entrée de date

Pour plus de commodité, les valeurs d'entrée contenant des dates/heures peuvent être récupérées sous forme d'instances `BlitzPHP\Utilities\Date` à l'aide de la méthode de `date()`. Si la requête ne contient pas de valeur d'entrée avec le nom donné, `null` sera renvoyé :

```php
$birthday = $request->date('birthday');
```

Les deuxième et troisième arguments acceptés par la méthode `date()` peuvent être utilisés pour spécifier respectivement le format de la date et le fuseau horaire :

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

Si la valeur d'entrée est présente mais a un format non valide, une `InvalidArgumentException` sera levée ; par conséquent, il est recommandé de [valider](/docs/{version}/validation) l'entrée avant d'appeler la méthode `date()`.

<a name="recuperation-des-valeurs-d-entree-enum"></a>
#### Récupération des valeurs d'entrée Enum

Les valeurs d'entrée qui correspondent aux <a href="https://www.php.net/manual/fr/language.types.enumerations.php" target="_blank">énumérations PHP</a> peuvent également être récupérées à partir de la requête. Si la requête ne contient pas de valeur d'entrée avec le nom donné ou si l'énumération n'a pas de valeur de support qui correspond à la valeur d'entrée, `null` sera renvoyé. La méthode `enum()` accepte le nom de la valeur d'entrée et la classe enum comme premier et deuxième arguments :

```php
use App\Enums\Status;
 
$status = $request->enum('status', Status::class);
```

<a name="recuperation-d-entree-via-proprietes-dynamiques"></a>
#### Récupération d'entrée via des propriétés dynamiques

Vous pouvez également accéder aux entrées utilisateur à l'aide de propriétés dynamiques sur l'instance `BlitzPHP\Http\Request`. Par exemple, si l'un des formulaires de votre requête contient un champ `name`, vous pouvez accéder à la valeur du champ comme suit :

```php
$name = $request->name;
```

<a name="recuperation-d-une partie-des-donnees-d-entree"></a>
#### Récupération d'une partie des données d'entrée

Si vous devez récupérer un sous-ensemble des données d'entrée, vous pouvez utiliser les méthodes `only()` et `except()`. Ces deux méthodes acceptent un seul tableau ou une liste dynamique d'arguments :

```php
$input = $request->only(['username', 'password']);
 
$input = $request->only('username', 'password');
 
$input = $request->except(['credit_card']);
 
$input = $request->except('credit_card');
```

> **Note**  
> La méthode `only()` renvoie toutes les paires clé/valeur que vous demandez ; cependant, il ne renverra pas les paires clé/valeur qui ne sont pas présentes sur la requête.

<a name="determiner-si-une-entree-est-presente"></a>
### Déterminer si une entrée est présente

Vous pouvez utiliser la méthode `has()` pour déterminer si une valeur est présente dans la requête. La méthode `has()` renvoie `true` si la valeur est présente sur la requête :

```php
if ($request->has('name')) {
    // ...
}
```

Lorsqu'on lui donne un tableau, la méthode `has()` déterminera si toutes les valeurs spécifiées sont présentes :

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

La méthode `hasAny()` renvoie `true` si l'une des valeurs spécifiées est présente :

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

La méthode `whenHas()` exécutera la closure donnée si une valeur est présente sur la requête :

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

Une seconde closure peut être passée à la méthode `whenHas()` qui sera exécutée si la valeur spécifiée n'est pas présente sur la requête :

```php
$request->whenHas('name', function (string $input) {
    // La valeur "name" est presente...
}, function () {
    // La valeur "name" n'est pas presente...
});
```

Si vous souhaitez déterminer si une valeur est présente sur la requête et n'est pas une chaîne vide, vous pouvez utiliser la méthode `filled()` :

```php
if ($request->filled('name')) {
    // ...
}
```

La méthode `anyFilled()` renvoie `true` si l'une des valeurs spécifiées n'est pas une chaîne vide :

```php
if ($request->anyFilled(['name', 'email'])) {
    // ...
}
```

La méthode `whenFilled()` exécutera la closure donnée si une valeur est présente sur la requête et n'est pas une chaîne vide :

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

Une seconde closure peut être passée à la méthode `whenFilled()` qui sera exécutée si la valeur spécifiée n'est pas "remplie" :

```php
$request->whenFilled('name', function (string $input) {
    // La valeur "name" est remplie...
}, function () {
    // La valeur "name" n'est pas remplie...
});
```

Pour déterminer si une clé donnée est absente de la requête, vous pouvez utiliser les méthodes `missing()` et `whenMissing()` :

```php
if ($request->missing('name')) {
    // ...
}
 
$request->whenMissing('name', function (array $input) {
    // La valeur "name" est absente...
}, function () {
    // La valeur "name" est presente...
});
```

<a name="ancienne-entree"></a>
### Ancienne entrée

BlitzPHP vous permet de conserver les entrées d'une requête lors de la requête suivante. Cette fonctionnalité est particulièrement utile pour remplir à nouveau les formulaires après avoir détecté des erreurs de validation. Cependant, si vous utilisez [les fonctionnalités de validation](/docs/{version}/validation) incluses de BlitzPHP, il est possible que vous n'ayez pas besoin d'utiliser manuellement ces méthodes de flashage d'entrée de session directement, car certaines des fonctionnalités de validation intégrées de BlitzPHP les appelleront automatiquement.

<a name="flasher-une-entree-a-la-session"></a>
#### Flasher une entrée à la session

La méthode `flash()` flashera l'entrée actuelle dans la [session](/docs/[version}/session) afin qu'elle soit disponible lors de la prochaine requête de l'utilisateur à l'application :

```php
$request->flash();
```

Vous pouvez également utiliser les méthodes `flashOnly()` et `flashExcept()` pour flasher un sous-ensemble des données de la demande dans la session. Ces méthodes sont utiles pour garder les informations sensibles telles que les mots de passe hors de la session :

```php
$request->flashOnly(['username', 'email']);
 
$request->flashExcept('password');
```

<a name="flasher-une-entree-et-rediriger"></a>
#### Flasher une entrée et rediriger

Puisque vous souhaiterez souvent flasher une entrée dans la session, puis rediriger vers la page précédente, vous pouvez facilement enchaîner l'entrée flashée sur une redirection en utilisant la méthode `withInput()` :

```php
return redirect('form')->withInput();
 
return redirect()->route('user.create')->withInput();
 
return redirect('form')->withInput(
    $request->except('password')
);
```

<a name="recupération-d-anciennes-entrees"></a>
#### Récupération d'anciennes entrées

Pour récupérer l’entrée flashée de la requête précédente, appelez la méthode `old()` sur une instance de `BlitzPHP\Http\Request`. La méthode `old()` extraira les données d'entrée précédemment flashées de la [session](/docs/[version}/session) :

```php
$username = $request->old('username');
```

BlitzPHP fournit également une fonction globale `old()`. Si vous affichez une ancienne entrée dans une [vue](/docs/{version}/vues), il est plus pratique d'utiliser la fonction `old()` pour remplir à nouveau le formulaire. Si aucune ancienne entrée n'existe pour le champ donné, `null` sera renvoyé :

```php
<input type="text" name="username" value="<?= old('username') ?>">
```

<a name="cookies"></a>
### Cookies

Tous les cookies créés par le framework BlitzPHP sont cryptés et signés avec un code d'authentification, ce qui signifie qu'ils seront considérés comme invalides s'ils ont été modifiés par le client. Pour récupérer une valeur de cookie de la requête, utilisez la méthode `cookie()` sur une instance `BlitzPHP\Http\Request` : 

```php
$value = $request->cookie('name');
```

<a name="fichiers"></a>
## Fichiers

<a name="recuperation-des-fichiers-telecharges"></a>
### Récupération des fichiers téléchargés

Vous pouvez récupérer les fichiers téléchargés à partir d'une instance `BlitzPHP\Http\Request` en utilisant la méthode `file()` ou en utilisant des propriétés dynamiques. La méthode `file()` renvoie une instance de la classe `BlitzPHP\Filesystem\Files\UploadedFile`, qui étend la classe PHP SplFileInfo et fournit diverses méthodes pour interagir avec le fichier :

```php
$file = $request->file('photo');
 
$file = $request->photo;
```

Vous pouvez déterminer si un fichier est présent sur la requête en utilisant la méthode `hasFile()` :

```php
if ($request->hasFile('photo')) {
    // ...
}
```

<a name="validation-des-telechargements-reussis"></a>
### Validation des téléchargements réussis

En plus de vérifier si le fichier est présent, vous pouvez vérifier qu'il n'y a eu aucun problème lors du téléchargement du fichier via la méthode `isValid()` :

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

<a name="chemins-de-fichiers-et-extensions"></a>
### Chemins de fichiers et extensions

La classe `UploadedFile` contient également des méthodes permettant d'accéder au chemin complet du fichier et à son extension. La méthode `extension()` tentera de deviner l'extension du fichier en fonction de son contenu. Cette extension peut être différente de l'extension fournie par le client :

```php
$path = $request->photo->path();
 
$extension = $request->photo->extension();
```

<a name="stockage-des-fichiers-telecharges"></a>
### Stockage des fichiers téléchargés

Pour stocker un fichier téléchargé, vous utiliserez généralement l'un de [vos systèmes de fichiers](/docs/{version}/fichiers) configurés. La classe `UploadedFile` possède une méthode de `store()` qui déplacera un fichier téléchargé vers l'un de vos disques, qui peut être un emplacement sur votre système de fichiers local ou un emplacement de stockage cloud comme Amazon S3.

La méthode `store()` accepte le chemin où le fichier doit être stocké par rapport au répertoire racine configuré du système de fichiers. Ce chemin ne doit pas contenir de nom de fichier, car un identifiant unique sera automatiquement généré pour servir de nom de fichier.

La méthode `store()` accepte également un deuxième argument facultatif pour le nom du disque qui doit être utilisé pour stocker le fichier. La méthode renverra le chemin du fichier par rapport à la racine du disque :

```php
$path = $request->photo->store('images');
 
$path = $request->photo->store('images', 's3');
```

Si vous ne souhaitez pas qu'un nom de fichier soit généré automatiquement, vous pouvez utiliser la méthode `storeAs()`, qui accepte le chemin, le nom du fichier et le nom du disque comme arguments :

```php
$path = $request->photo->storeAs('images', 'filename.jpg');
 
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

> **Note**  
> Pour plus d'informations sur le stockage de fichiers dans BlitzPHP, consultez [la documentation complète sur le stockage de fichiers](/docs/{version}/fichiers).