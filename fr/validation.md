---
title: Validation des données
---

<a name="introduction"></a>
## Introduction

> **Note**  
> Le système de validation de BlitzPHP hérite directement du package <a href="https://github.com/dimtrovich/validation" target="_blank">Dimtrovich\Validation</a> qui fourni <a href="https://laravel.com/docs/validation" target="_blank">le système de validation de Laravel</a> de manière autonome. Vous trouverez donc la majeure partie des règles fournis par Laravel et bien d'autres. 

BlitzPHP fournit une classe de validation des données complète qui permet de minimiser la quantité de code que vous écrivez.

BlitzPHP comprend une grande variété de règles de validation pratiques que vous pouvez appliquer aux données, offrant même la possibilité de valider si les valeurs sont uniques dans une table de base de données. Nous allons aborder chacune de ces règles de validation en détail afin de vous familiariser avec toutes nos fonctionnalités de validation.

> **Note**  
> Si vous utilisez habituellement le framework Laravel, vous seriez alaise avec le système de validation de BlitzPHP car il est fortement inspiré de <a href="https://laravel.com/docs/validation" target="_blank">Illuminate\Validation</a>

<a name="demarrage-rapide"></a>
## Démarrage rapide

Pour découvrir les fonctionnalités de validation, examinons un exemple complet de validation d'un formulaire et d'affichage des messages d'erreur à l'intention de l'utilisateur. En lisant cet aperçu de haut niveau, vous serez en mesure d'acquérir une bonne compréhension générale de la façon de valider les données des requêtes entrantes à l'aide de BlitzPHP.

<a name="definir-les-routes"></a>
### Définir les routes

Tout d'abord, supposons que nous ayons défini les routes suivantes dans notre fichier `app/Config/routes.php` :

```php
<?php 
use App\Controllers\PostController;
use BlitzPHP\Facades\Route;
 
Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

La route `GET` affichera un formulaire permettant à l'utilisateur de créer un nouvel article de blog, tandis que la route `POST` enregistrera le nouvel article de blog dans la base de données.

<a name="creation-du-controleur"></a>
### Création du contrôleur

Ensuite, examinons un simple contrôleur qui gère les requêtes entrantes vers ces routes. Nous laisserons la méthode `store` vide pour l'instant :

```php
<?php
 
namespace App\Controllers;
 
class PostController extends Controller
{
    /**
     * Afficher le formulaire de création d'un nouvel article de blog.
     */
    public function create()
    {
        return view('post/create');
    }
 
    /**
     * Enregistrer un nouvel article de blog.
     */
    public function store()
    {
        // Valider et enregistrer l'article de blog...
 
        $post = /** ... */
 
        return redirect()->route('post.show', ['post' => $post->id]);
    }
}
```

<a name="redaction-de-la-logique-de-validation"></a>
### Rédaction de la logique de validation

Nous sommes maintenant prêts à compléter notre méthode store avec la logique de validation du nouvel article de blog. Pour ce faire, nous utiliserons la méthode `validate()` fournie par l'objet `BlitzPHP\Http\Request`. Si les règles de validation sont acceptées, votre code continuera à s'exécuter normalement ; cependant, si la validation échoue, une exception `BlitzPHP\Validation\ValidationException` sera levée et la réponse d'erreur appropriée sera automatiquement renvoyée à l'utilisateur.

Si la validation échoue au cours d'une requête HTTP traditionnelle, une réponse de redirection vers l'URL précédente sera générée. Si la requête entrante est une requête XHR, [une réponse JSON contenant les messages d'erreur de validation](#format-de-reponse-d-erreur-de-validation) sera renvoyée.

Pour mieux comprendre la méthode validate, revenons à la méthode `store` :

```php
/**
 * Enregistrer un nouvel article de blog.
 */
public function store()
{
     $validated = $this->request->validate([
        'title' => 'required|unique:posts|max:255',
        'body'  => 'required',
    ]);
 
    // L'article de blog est valable...
 
    return redirect('/posts');
}
```

Comme vous pouvez le constater, les règles de validation sont transmises à la méthode `validate()`. Ne vous inquiétez pas, toutes les règles de validation disponibles sont documentées. Encore une fois, si la validation échoue, la réponse appropriée sera automatiquement générée. Si la validation réussit, notre contrôleur continuera à s'exécuter normalement.

Les règles de validation peuvent également être spécifiées sous forme de tableaux de règles au lieu d'une seule chaîne de caractères délimitée par `|`:

```php
$validated = $this->request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body'  => ['required'],
]);
```

Si la requête HTTP entrante contient des données de champs "imbriqués", vous pouvez spécifier ces champs dans vos règles de validation en utilisant la syntaxe "point" :

```php
$this->request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

D'autre part, si le nom de votre champ contient un point littéral, vous pouvez explicitement empêcher qu'il soit interprété comme une syntaxe "point" en faisant précéder le point d'une barre oblique inverse :

```php
$this->request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

<a name="affichage-des-erreurs-de-validation"></a>
### Affichage des erreurs de validation

Que se passe-t-il si les champs de la requête entrante ne passent pas les règles de validation données ? Comme indiqué précédemment, BlitzPHP redirigera automatiquement l'utilisateur vers son emplacement précédent. En outre, toutes les erreurs de validation et [les entrées de la requête](/docs/{version}/requetes#ancienne-entree) seront automatiquement transmises à la [session](/docs/{version}/session#donnees-flash).

Une variable `$errors` est partagée avec toutes les vues de votre application par le middleware `BlitzPHP\Middlewares\ShareErrorsFromSession`, qui est fourni par le groupe des middlewares web. Lorsque cet middleware est appliqué, une variable `$errors` sera toujours disponible dans vos vues, ce qui vous permet de supposer que la variable `$errors` est toujours définie et peut être utilisée en toute sécurité. La variable `$errors` sera une instance de la classe `BlitzPHP\Validation\ErrorBag`. Pour plus d'informations sur l'utilisation de cet objet, consultez sa documentation.

Ainsi, dans notre exemple, l'utilisateur sera redirigé vers la méthode `create` de notre contrôleur lorsque la validation échoue, ce qui nous permet d'afficher les messages d'erreur dans la vue :

```php
<!-- /app/Views/post/create.php -->
 
<h1>Creation d'article</h1>
 
<?php if (! $errors->empty()): ?>
    <div class="alert alert-danger">
        <ul>
            <?php foreach ($errors->all() as $error): ?>
                <li><?= $error ?></li>
            <?php endforeach; ?>
        </ul>
    </div>
<?php endif; ?>
 
<!-- Formulaire de creation -->
```

<a name="personnalisation-des-messages-d-erreur"></a>
#### Personnalisation des messages d'erreur

Les règles de validation intégrées de BlitzPHP ont chacune un message d'erreur qui se trouve dans le fichier `app/Translations/en/validation.php` de votre application.

Dans le fichier `app/Translations/en/validation.php`, vous trouverez une entrée de traduction pour chaque règle de validation. Vous êtes libre de changer ou de modifier ces messages en fonction des besoins de votre application.

De plus, vous pouvez copier ce fichier dans un autre répertoire de langue pour traduire les messages dans la langue de votre application. Pour en savoir plus sur l'internalisation de BlitzPHP, consultez [la documentation complète sur l'internalisation](/docs/{version}/internationalisation).

<a name="requetes-xhr-et-validation"></a>
#### Requêtes XHR et validation

Dans cet exemple, nous avons utilisé un formulaire traditionnel pour envoyer des données à l'application. Cependant, de nombreuses applications reçoivent des requêtes XHR à partir d'un frontend fonctionnant avec JavaScript. Lors de l'utilisation de la méthode `validate` pendant une requête XHR, BlitzPHP ne génère pas de réponse de redirection. Au lieu de cela, BlitzPHP génère [une réponse JSON contenant toutes les erreurs de validation](#format-de-reponse-d-erreur-de-validation). Cette réponse JSON sera envoyée avec un code d'état HTTP 422.

<a name="repopulation-des-formulaires"></a>
### Repopulation des formulaires

Lorsque BlitzPHP génère une réponse de redirection en raison d'une erreur de validation, le framework [enregistre automatiquement toutes les données de la requête dans la session](/docs/{version}/session#donnees-flash). Cela permet d'accéder facilement aux données lors de la prochaine requête et de remplir le formulaire que l'utilisateur a tenté de soumettre.

Pour récupérer les données d'entrée flashées de la requête précédente, invoquez la méthode `old()` sur une instance de `BlitzPHP\Http\Request`. La méthode `old()` récupérera les données d'entrée flashées précédemment dans la [session](/docs/{version}/session) :

```php
$title = $request->old('title');
```

BlitzPHP fournit également une fonction globale `old()`. Si vous affichez une ancienne entrée dans une vue, il est plus pratique d'utiliser la fonction `old` pour remplir le formulaire. Si aucune ancienne entrée n'existe pour le champ donné, `null` sera retourné :

```php
<input type="text" name="title" value="<?= old('title') ?>">
```

<a name="format-de-reponse-d-erreur-de-validation"></a>
### Format de réponse d'erreur de validation

Lorsque votre application lance une exception `BlitzPHP\Exception\ValidationException` et que la requête HTTP entrante attend une réponse JSON, BlitzPHP va automatiquement formater les messages d'erreur pour vous et renvoyer une réponse `HTTP 422 Unprocessable Entity`.

Vous trouverez ci-dessous un exemple de format de réponse JSON pour les erreurs de validation. Notez que les clés d'erreur imbriquées sont aplaties au format de notation "point" :

```json
{
    "message" : "Le nom de l'équipe doit être une chaîne de caractères. (et 4 autres erreurs)",
    "errors" : {
        "team_name" : [
            "Le nom de l'équipe doit être une chaîne de caractères",
            "Le nom de l'équipe doit comporter au moins 1 caractère."
        ],
        "authorization.role" : [
            "Le rôle d'autorisation sélectionné n'est pas valide."
        ],
        "users.0.email" : [
            "Le champ users.0.email est obligatoire."
        ],
        "users.2.email" : [
            "Le champ users.2.email doit être une adresse électronique valide."
        ]
    }
}
```

<a name="classe-de-validation"></a>
## Classe de validation

<a name="creation-de-classe-de-validation"></a>
### Création de classe de validation

Pour des scénarios de validation plus complexes, vous pouvez créer une classe de validation. Les classes de validation encapsulent une logique validation complexe permettant d'alléger les contrôleurs. Pour créer une classe de validation, vous pouvez utiliser la commande Klinge `make:validation` :

```shell
php klinge make:validation CreatPost
```

La classe de validation générée sera placée dans le répertoire `app/Validations`. Si ce répertoire n'existe pas, il sera créé lors de l'exécution de la commande `make:validation`. Chaque validation générée par BlitzPHP possède une propriété `$source` et une méthode `rules()`.

Comme vous l'avez peut-être deviné, la propriété `$source` est chargée de déterminer la source des données à valider (POST, GET, COOKIE ou toute donnée de la requête), tandis que la méthode `rules` renvoie les règles de validation qui doivent s'appliquer aux données de la requête :

```php
/**
 * Regles de validation qui s'appliquent à la requête.
 *
 * @return array<string, \BlitzPHP\Validation\Rules\AbstractRule|array|string>
 */
protected function rules(): array
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body'  => 'required',
    ];
}
```

> **Note**  
> Vous pouvez indiquer les dépendances dont vous avez besoin dans le constructeur de votre classe de validation. Elles seront automatiquement résolues par [le conteneur d'injection](/docs/{version}/conteneur) de BlitzPHP.

A présent, au lieu de passer un tableau de règles à la méthode `validate()`, vous pouvez maintenant passer <a href="https://fr.wiktionary.org/wiki/FQCN" target="_blank">le nom complet (FQCN)</a> de votre classe de validation.

```php
/**
 * Enregistrer un nouvel article de blog.
 */
public function store()
{
     $validated = $this->request->validate(\App\Validations\CreatePostValidation::class);
 
    // L'article de blog est valable...
 
    return redirect('/posts');
}

```

Si la validation échoue, une réponse de redirection sera générée pour renvoyer l'utilisateur à son emplacement précédent. Les erreurs seront également transmises à la session afin qu'elles puissent être affichées. Si la requête était une requête XHR, une réponse HTTP avec un code d'état 422 sera renvoyée à l'utilisateur, y compris une [représentation JSON des erreurs de validation](#format-de-reponse-d-erreur-de-validation).

<a name="personnalisations"></a>
### Personnalisations

<a name="personnalisation-des-messages-d-erreur"></a>
#### Personnalisation des messages d'erreur
    
Vous pouvez personnaliser les messages d'erreur utilisés par la classe de validation en surchargeant la méthode `messages`. Cette méthode doit renvoyer un tableau de paires attribut/règle et les messages d'erreur correspondants :

```php
/**
 * Messages d'erreur pour les règles de validation définies.
 *
 * @return array<string, string>
 */
public function messages() : array
{
    return [
        'title:required' => 'Un titre est requis',
        'body:required' => 'Un message est requis',
    ] ;
}
```

<a name="personnalisation-des-attributs-de-validation"></a>
#### Personnalisation des attributs de validation

De nombreux messages d'erreur des règles de validation intégrées de BlitzPHP contiennent un caractère générique `:attribute`. Si vous souhaitez que le caractère générique `:attribute` de votre message de validation soit remplacé par un nom d'attribut personnalisé, vous pouvez spécifier les noms personnalisés en surchargeant la méthode `attributes`. Cette méthode doit renvoyer un tableau de paires attribut / nom :

```php
/**
 * Attributs personnalisés pour les erreurs du validateur.
 *
 * @return array<string, string>
 */
public function attributes() : array
{
    return [
        'email' => 'adresse email',
    ] ;
}
```

<a name="creation-manuelle-de-validateurs"></a>
## Création manuelle de validateurs

Si vous ne souhaitez pas utiliser la méthode `validate()` sur la requête, vous pouvez créer manuellement une instance du validateur à l'aide de la classe `BlitzPHP\Validation\Validator`. La méthode `make()` de cette classe génère une nouvelle instance de validateur :

```php
<?php
 
namespace App\Http\Controllers;
 
use BlitzPHP\Validation\Validator;
 
class PostController extends Controller
{
    /**
     * Enregistrer un nouvel article de blog.
     */
    public function store()
    {
        $validator = Validator::make($this->request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);
 
        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }
 
        // Récupérer les données validées...
        $validated = $validator->validated();
 
        // Récupérer une portion de données validées...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);
 
        // Stocker l'article de blog...
 
        return redirect('/posts');
    }
}
```

Le premier argument transmis à la méthode `make()` est la donnée à valider. Le second argument est un tableau des règles de validation à appliquer aux données.

Après avoir déterminé si la validation de la demande a échoué, vous pouvez utiliser la méthode `withErrors()` pour afficher les messages d'erreur dans la session. Lorsque vous utilisez cette méthode, la variable `$errors` sera automatiquement partagée avec vos vues après la redirection, ce qui vous permettra de les afficher facilement à l'utilisateur. La méthode `withErrors()` accepte un validateur, un `ErrorBag`, une chaîne ou un tableau PHP.

<a name="redirection-automatique"></a>
### Redirection automatique

Si vous souhaitez créer manuellement une instance de validateur tout en profitant de la redirection automatique offerte par la méthode `validate()` de la requête HTTP, vous pouvez appeler la méthode `validate()` sur une instance de validateur existante. Si la validation échoue, l'utilisateur sera automatiquement redirigé ou, dans le cas d'une requête XHR, [une réponse JSON sera renvoyée](#format-de-reponse-d-erreur-de-validation) :

```php
Validator::make($this->request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body'  => 'required',
])->validate();
```

<a name="personnalisation-des-messages-d-erreur-manuelle"></a>
### Personnalisation des messages d'erreur

Si nécessaire, vous pouvez fournir des messages d'erreur personnalisés qu'une instance de validateur doit utiliser à la place des messages d'erreur par défaut fournis par BlitzPHP. Il existe plusieurs façons de spécifier des messages personnalisés. Tout d'abord, vous pouvez passer les messages personnalisés comme troisième argument à la méthode `Validator::make` :

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'Le champ :attribute est requis.',
]);
```

Dans cet exemple, le caractère générique `:attribute` sera remplacé par le nom réel du champ en cours de validation. Vous pouvez également utiliser d'autres caractères génériques dans les messages de validation. Par exemple :

```php
$messages = [
    'same' => ':attribut et :other doivent correspondre.',
    'size' => ':attribut doit être exactement :size.',
    'between' => "La valeur de :attribut n'est pas comprise entre :min - :max.",
    'in' => ":attribut doit être l'un des types suivants: :values",
];
```

<a name="specification-d-un-message-personnalise-pour-un-attribut-donne"></a>
#### Spécification d'un message personnalisé pour un attribut donné

Il peut arriver que vous souhaitiez spécifier un message d'erreur personnalisé uniquement pour un attribut spécifique. Vous pouvez le faire en utilisant la notation "`:`". Spécifiez d'abord le nom de l'attribut, suivi de la règle :

```php
$messages = [
    'email:required' => 'Nous avons besoin de connaître votre adresse email!',
];
```

<a name="specification-de-valeurs-d-attributs-personnalises"></a>
#### Spécification de valeurs d'attributs personnalisés

De nombreux messages d'erreur intégrés à BlitzPHP comprennent un caractère générique :`attribute` qui est remplacé par le nom du champ ou de l'attribut en cours de validation. Pour personnaliser les valeurs utilisées pour remplacer ces espaces réservés pour des champs spécifiques, vous pouvez passer un tableau d'attributs personnalisés comme quatrième argument de la méthode `Validator::make` :

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'adresse email',
]);
```

<a name="travailler-avec-des-donnees-validees"></a>
## Travailler avec des données validées

Après avoir validé les données d'une requête entrante à l'aide d'une classe de validation ou d'une instance de validateur créée manuellement, vous pouvez souhaiter récupérer les données de la requête entrante qui ont effectivement été validées. Cela peut se faire de plusieurs manières. Tout d'abord, vous pouvez appeler la méthode `validated()` sur une instance de validateur (au cas où vous avez fait une validation manuelle). Si vous utiliser la méthode `validate()` de l'objet `$request`, cette méthode sera automatiquement appelée pour vous. Dans tous les cas, vous obtiendrez un tableau des données qui ont été validées :

```php
$validated = $this->request->validate(...);
 
$validated = $validator->validated();
```

* La ligne `Validator::make(...)->validated();` peut être remplacée par `Validator::validate(...);`. La méthode `Validator::validate` prend exactement les mêmes paramètres que `Validator::make`.
* En utilisant la méthode `validate()` de l'objet `$request` vous obtiendrez une instance de la classe `ValidatedInput` tandis qu'avec la méthode `validated()` sur une instance de validateur, vous aurez un tableau PHP classique.
* Si vous voulez obtenir une instance de la classe `ValidatedInput` en passant par une instance du validateur, vous devez utiliser la méthode `safe()`: `$validated = $validator->safe();`

Une instance de `Dimtrovich\Validation\ValidatedInput` expose les méthodes `only`, `except` et `all` pour récupérer un sous-ensemble des données validées ou l'ensemble des données validées :

```php
$validated = $validator->safe()->only(['name', 'email']);
 
$validated = $validator->safe()->except(['name', 'email']);
 
$validated = $validator->safe()->all();
```

En outre, l'instance `Dimtrovich\Validation\ValidatedInput` peut être itérée et consultée comme un tableau :

```php
// Les données validées peuvent être itérées...
foreach ($validator->safe() as $key => $value) {
    // ...
}
 
// Les données validées peuvent être accédées sous forme de tableau...
$validated = $validator->safe() ;
 
$email = $validated['email'] ;
```

Si vous souhaitez ajouter des champs supplémentaires aux données validées, vous pouvez utiliser la méthode de `merge()` :

```php
$validated = $validator->safe()->merge(['name' => 'BlitzPHP']);
```

Si vous souhaitez récupérer les données validées sous la forme d'une instance de [collection](/docs/{version}/collection), vous pouvez appeler la méthode `collect()` :

```php
$collection = $validator->safe()->collect();
```

<a name="travailler-avec-des-messages-d-erreur"></a>
## Travailler avec des messages d'erreur

Après avoir appelé la méthode `errors` sur une instance de validateur, vous recevrez une instance de `BlitzPHP\Validation\ErrorBag`, qui dispose d'une variété de méthodes pratiques pour travailler avec les messages d'erreur. La variable `$errors` qui est automatiquement mise à la disposition de toutes les vues est également une instance de la classe `ErrorBag`.

<a name="recuperation-du-premier-message-d-erreur-pour-un-champ"></a>
#### Récupération du premier message d'erreur pour un champ

Pour récupérer le premier message d'erreur pour un champ donné, utilisez la méthode `first()` :

```php
$errors = $validator->errors();
 
echo $errors->first('email');
```

<a name="recuperation-de-tous-les-messages-d-erreur-pour-un-champ"></a>
#### Récupération de tous les messages d'erreur pour un champ

Si vous souhaitez récupérer un tableau de tous les messages pour un champ donné, utilisez la méthode `get()` :

```php
foreach ($errors->get('email') as $message) {
    // ...
}
```

Vous pouvez également, récupérer tous les messages pour un champ donné sous forme de chaîne en utilisant la méthode `line()` :

```php
echo $errors->line('email');
```

Si vous validez un champ de formulaire sous forme de tableau, vous pouvez récupérer tous les messages pour chacun des éléments du tableau en utilisant le caractère `*` :

```php
foreach ($errors->get('attachments.*') as $message) {
    // ...
}
```

<a name="recuperation-de-tous-les-messages-d-erreur-pour-tous-les-champs"></a>
#### Récupération de tous les messages d'erreur pour tous les champs

Pour récupérer un tableau de tous les messages pour tous les champs, utilisez la méthode `all()` :

```php
foreach ($errors->all() as $message) {
    // ...
}
```

<a name="determination-de-l-existence-de-messages-pour-un-champ"></a>
#### Détermination de l'existence de messages pour un champ

La méthode `has()` peut être utilisée pour déterminer s'il existe des messages d'erreur pour un champ donné :

```php
if ($errors->has('email')) {
    // ...
}
```

<a name="specification-de-messages-personnalises-dans-les-fichiers-de-langue"></a>
### Spécification de messages personnalisés dans les fichiers de langue

Les règles de validation intégrées de BlitzPHP ont chacune un message d'erreur qui se trouve dans le fichier `app/Translations/en/validation.php` de votre application.

Dans le fichier `app/Translations/en/validation.php`, vous trouverez une entrée de traduction pour chaque règle de validation. Vous êtes libre de changer ou de modifier ces messages en fonction des besoins de votre application.

De plus, vous pouvez copier ce fichier dans un autre répertoire de langue pour traduire les messages dans la langue de votre application. Pour en savoir plus sur l'internationalisation de BlitzPHP, consultez [la documentation complète sur l'internationalisation](/docs/{version}/internationalisation).

> **Note**  
> Par défaut, le squelette de l'application BlitzPHP n'inclut pas le répertoire `app/Translations`. Si vous souhaitez personnaliser les fichiers de langue de BlitzPHP, vous pouvez utiliser le package <a href="https://github.com/blitz-php/translations" target="_blank">blitz-php/translations</a>.

<a name="regles-de-validation-disponibles"></a>
## Règles de validation disponibles

Vous trouverez ci-dessous une liste de toutes les règles de validation disponibles et de leur fonction :


<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#regle-accepted)
[Accepted If](#regle-accepted-if)
[Active URL](#regle-active-url)
[After (Date)](#regle-after)
[After Or Equal (Date)](#regle-after-or-equal)
[Alpha](#regle-alpha)
[Alpha Dash](#regle-alpha-dash)
[Alpha Numeric](#regle-alpha-num)
[Array](#regle-array)
[Ascii](#regle-ascii)
[Base64](#regle-base64)
[Before (Date)](#regle-before)
[Before Or Equal (Date)](#regle-before-or-equal)
[Between](#regle-between)
[Bic](#regle-bic)
[Bitcoin Address](#regle-bitcoin-address)
[Boolean](#regle-boolean)
[Callback](#regle-callback)
[Camelcase](#regle-camelcase)
[Capital Char With Number](#regle-capital-char-with-number)
[Car Number](#regle-car-number)
[Cidr](#regle-cidr)
[Confirmed](#regle-confirmed)
[Credit Card](#regle-credit-card)
[Currency](#regle-currency)
[Current Password (Schied)](#regle-current-password)
[Date](#regle-date)
[Date Equals](#regle-date-equals)
[Datetime](#regle-datetime)
[Decimal](#regle-decimal)
[Declined](#regle-declined)
[Declined If](#regle-declined-if)
[Default](#regle-default)
[Different](#regle-different)
[Digits](#regle-digits)
[Digits Between](#regle-digits-between)
[Dimensions (Fichiers Image)](#regle-dimensions)
[Distinct](#regle-distinct)
[Discord Username](#regle-discord-username)
[Doesnt Start With](#regle-doesnt-start-with)
[Doesnt End With](#regle-doesnt-end-with)
[Domain](#regle-domain)
[Duplicate](#regle-duplicate)
[Duplicate Character](#regle-duplicate-character)
[European Article Number (EAN)](#regle-ean)
[Email](#regle-email)
[Ends With](#regle-ends-with)
[Enum](#regle-enum)
[Even Number](#regle-even-number)
[Exists (Base de données)](#regle-exists)
[Ext](#regle-ext)
[File](#regle-file)
[Float](#regle-float)
[Fullname](#regle-fullname)
[Greater Than](#regle-gt)
[Greater Than Or Equal](#regle-gte)
[Global Trade Item Number (GTIN)](#regle-gtin)
[Hash](#regle-hash)
[Hashtag](#regle-hashtag)
[Hex](#regle-hex)
[Hexcolor](#regle-hexcolor)
[Htmlclean](#regle-htmlclean)
[Htmltag](#regle-htmltag)
[International Bank Account Number (IBAN)](#regle-iban)
[Instance Of](#regle-instance-of)
[Image (File)](#regle-image)
[International Mobile Equipment Identity (IMEI)](#regle-imei)
[In](#regle-in)
[In Array](#regle-in-array)
[Integer](#regle-integer)
[IP Address](#regle-ip)
[International Standard Book Number (ISBN)](#regle-isbn)
[International Standard Serial Number (ISSN)](#regle-issn)
[JSON](#regle-json)
[Json Web Token](#regle-jwt)
[Kebabcase](#regle-kebabcase)
[Less Than](#regle-lt)
[Less Than Or Equal](#regle-lte)
[Lowercase](#regle-lowercase)
[MAC Address](#regle-mac)
[Max](#regle-max)
[Max Digits](#regle-max-digits)
[Mime Types](#regle-mimetypes)
[Mimes](#regle-mimes)
[Min](#regle-min)
[Min Digits](#regle-min-digits)
[Missing](#regle-missing)
[Missing If](#regle-missing-if)
[Missing Unless](#regle-missing-unless)
[Missing With](#regle-missing-with)
[Missing With All](#regle-missing-with-all)
[Multiple Of](#regle-multiple-of)
[Not In](#regle-not-in)
[Not In Array](#regle-not-in-array)
[Not Regex](#regle-not-regex)
[Nullable](#regle-nullable)
[Numeric](#regle-numeric)
[Odd Number](#regle-odd-number)
[Pascalcase](#regle-pascalcase)
[Password](#regle-password)
[Pattern](#regle-pattern)
[Phone Number](#regle-phone)
[Port](#regle-port)
[Postal Code](#regle-postalcode)
[Present](#regle-present)
[Present If](#regle-present-if)
[Present Unless](#regle-present-unless)
[Present With](#regle-present-with)
[Present With All](#regle-present-with-all)
[Prohibited](#regle-prohibited)
[Prohibited If](#regle-prohibited-if)
[Prohibited Unless](#regle-prohibited-unless)
[Prohibits](#regle-prohibits)
[Regular Expression](#regle-regex)
[Required](#regle-required)
[Required If](#regle-required-if)
[Required If Accepted](#regle-required-if-accepted)
[Required If Declined](#regle-required-if-declined)
[Required Unless](#regle-required-unless)
[Required With](#regle-required-with)
[Required With All](#regle-required-with-all)
[Required Without](#regle-required-without)
[Required Without All](#regle-required-without-all)
[Same](#regle-same)
[Semantic Version Number](#regle-semver)
[Slash End Of String](#regle-slash-end-of-string)
[SEO-friendly short text (Slug)](#regle-slug)
[Snakecase](#regle-snakecase)
[Size](#regle-size)
[Sometimes](#validation-en-cas-de-presence)
[Starts With](#regle-starts-with)
[String](#regle-string)
[Time](#regle-time)
[Timezone](#regle-timezone)
[Titlecase](#regle-titlecase)
[ULID](#regle-ulid)
[Unique (Base de données)](#regle-unique)
[Uploaded File](#regle-uploaded-file)
[Uppercase](#regle-uppercase)
[URL](#regle-url)
[Username](#regle-username)
[UUID](#regle-uuid)
[VAT ID](#regle-vatid)

</div>

<a name="regle-accepted"></a>
#### accepted

Le champ à valider doit être `"yes"`, `"on"`, `1` ou `true`. Cette fonction est utile pour valider l'acceptation des "conditions d'utilisation" ou d'autres champs similaires.

<a name="regle-accepted-if"></a>
#### accepted_if:anotherfield,value,...

Le champ en cours de validation doit être `"yes"`, `"on"`, `1`, ou `true` si un autre champ en cours de validation est égal à une valeur spécifiée. Ceci est utile pour valider l'acceptation des "Conditions d'utilisation" ou d'autres champs similaires.

<a name="regle-active-url"></a>
#### active_url

Le champ en cours de validation doit avoir un enregistrement A ou AAAA valide selon la fonction PHP `dns_get_record`. Le nom d'hôte de l'URL fournie est extrait en utilisant la fonction PHP `parse_url` avant d'être passé à `dns_get_record`.

<a name="regle-after"></a>
#### after:`date`

Le champ en cours de validation doit être une valeur postérieure à une date donnée. Les dates seront passées dans la fonction PHP `strtotime` afin d'être converties en une instance valide de `DateTime` :

```php
'start_date' => 'required|date|after:tomorrow'
```

Tout ce qui peut être analysé par `strtotime` peut être transmis en tant que paramètre à cette règle. Les exemples valides comprennent :

* after:next week
* after:2016-12-31
* after:2016
* after:2016-12-31 09:56:02

Au lieu de passer une chaîne de date à évaluer par `strtotime`, vous pouvez spécifier un autre champ à comparer à la date :

```php
'finish_date' => 'required|date|after:start_date'
```

<a name="regle-after-or-equal"></a>
#### after\_or\_equal:`date`

Le champ en cours de validation doit être une valeur postérieure ou égale à la date donnée. Pour plus d'informations, voir la règle [after](#regle-after).

<a name="regle-alpha"></a>
#### alpha

Le champ à valider doit être entièrement composé de caractères alphabétiques Unicode contenus dans [`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) et [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=).

Pour restreindre cette règle de validation aux caractères de la gamme ASCII (`a-z` et `A-Z`), vous pouvez fournir l'option `ascii` à la règle de validation :

```php
'username' => 'alpha:ascii',
```

<a name="regle-alpha-dash"></a>
#### alpha_dash

Le champ en cours de validation doit être entièrement composé de caractères alphanumériques Unicode contenus dans [`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=), [`\p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=), ainsi que les tirets ASCII (`-`) et les traits de soulignement ASCII (`_`).

Pour limiter cette règle de validation aux caractères de la plage ASCII (`a-z` et `A-Z`), vous pouvez ajouter l'option `ascii` à la règle de validation :

```php
'username' => 'alpha_dash:ascii',
```

<a name="regle-alpha-num"></a>
#### alpha_num

Le champ en cours de validation doit être entièrement composé de caractères alphanumériques Unicode contenus dans [`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=), et [`\p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=).

Pour limiter cette règle de validation aux caractères de la plage ASCII (`a-z` et `A-Z`), vous pouvez ajouter l'option `ascii` à la règle de validation :

```php
'username' => 'alpha_num:ascii',
```

<a name="regle-array"></a>
#### array

Le champ en cours de validation doit être un tableau PHP.

Lorsque des valeurs supplémentaires sont fournies à la règle `array`, chaque clé du tableau d'entrée doit être présente dans la liste des valeurs fournies à la règle. Dans l'exemple suivant, la clé `admin` du tableau d'entrée est invalide car elle n'est pas contenue dans la liste des valeurs fournies à la règle `array` :

```php
$input = [
    'user' => [
        'name' => 'Dimitri Sitchet',
        'username' => 'dimtrovich',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

En général, vous devez toujours spécifier les clés de tableau qui sont autorisées à être présentes dans votre tableau.

<a name="regle-ascii"></a>
#### ascii

Le champ en cours de validation doit être entièrement constitué de caractères ASCII de 7 bits.

<a name="regle-base64"></a>
#### base64

Le champ en cours de validation doit être <a href="https://en.wikipedia.org/wiki/Base64" target="_blank">encodé en Base64</a>.

<a name="regle-before"></a>
#### before:`date`

Le champ en cours de validation doit être une valeur antérieure à la date donnée. Les dates seront passées à la fonction PHP `strtotime` afin d'être converties en une instance valide de `DateTime`. De plus, comme pour la règle [`after`](#regle-after), le nom d'un autre champ en cours de validation peut être fourni comme valeur de `date`.

<a name="regle-before-or-equal"></a>
#### before\_or\_equal:`date`

Le champ en cours de validation doit être une valeur antérieure ou égale à la date donnée. Les dates seront passées à la fonction PHP `strtotime` afin d'être converties en une instance valide de `DateTime`. De plus, comme pour la règle [`after`](#regle-after), le nom d'un autre champ en cours de validation peut être fourni comme valeur de `date`.

<a name="regle-between"></a>
#### between:`min`,`max`

Le champ en cours de validation doit avoir une taille comprise entre _min_ et _max_ (inclus). Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués de la même manière que la règle [`size`](#regle-size).

<a name="regle-bic"></a>
#### bic

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/ISO_9362" target="_blank">code d'identification de d'entreprise (BIC)</a> valide.

<a name="regle-bitcoin-address"></a>
#### bitcoin\_address

Le champ en cours de validation doit être une adresse bitcoin valide (ex: _1KFHE7w8BhaENAswwryaoccDb6qcT6DbYY_).

<a name="regle-boolean"></a>
#### boolean

Le champ en cours de validation doit pouvoir être converti en booléen. Les entrées acceptées sont `true`, `false`, `1`, `0`, `"1"`, et `"0"`.

<a name="regle-callback"></a>
#### callback

Définir un callback personnalisé pour valider la valeur. Cette règle ne peut pas être enregistrée à l'aide de la syntaxe de chaîne. Pour utiliser cette règle, vous devez utiliser la syntaxe de tableau et soit spécifier explicitement le callback, soit passer la closure :

```php
use BlitzPHP\Validation\Validator;

$validation = Validator::make($data, [
    'even_number' => [
        'required',
        function ($value) {
            // false = invalid
            return (is_numeric($value) AND $value % 2 === 0);
        },
        'callback' => fn ($v) => is_numeric($v) && $v % 2 === 0,
    ]
]);
```

Vous pouvez définir un message personnalisé en renvoyant une chaîne de caractères au lieu de `false` :

```php
use BlitzPHP\Validation\Validator;

$validation = Validator::make($data,  [
    'even_number' => [
        'required',
        function ($value) {
            if (!is_numeric($value)) {
                return ":attribute must be numeric.";
            }
            if ($value % 2 !== 0) {
                return ":attribute is not even number.";
            }
            
            return true; // always return true if validation passes
        }
    ]
]);
```

<a name="regle-camelcase"></a>
#### camelcase

Le champ en cours de validation doit être formaté en <a href="https://en.wikipedia.org/wiki/Camel_case" target="_blank">Camel Case</a>

<a name="regle-capital-char-with-number"></a>
#### capital\_char\_with\_number

Le champ en cours de validation doit être une chaîne en majuscule et comportant au moins un nombre (ex: _BLITZPHP-237_).

<a name="regle-capital-char-with-number"></a>
#### capital\_char\_with\_number

Le champ en cours de validation doit être un numéro de véhicule valide (ex: _KA01AB1234_).

<a name="regle-cidr"></a>
#### cidr

Le champ en cours de validation doit être une valeur ayant une notation <a href="https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing" target="_blank">CIDR (Classless Inter-Domain Routing)</a> valide.

<a name="regle-confirmed"></a>
#### confirmed

Le champ en cours de validation doit avoir un champ correspondant à `{champ}_confirmation`. Par exemple, si le champ à valider est `password`, un champ correspondant `password_confirmation` doit être présent dans l'entrée.

<a name="regle-credit-card"></a>
#### credit_card:`fournisseur`

Le champ en cours de validation doit être un numéro de carte de crédit dont le format correspond à celui utilisé par le `fournisseur` spécifié.  

Les fournisseurs actuellement pris en charge sont les suivants :
- American Express (`amex`),
- China Unionpay (`unionpay`),
- Diners Club CarteBlance (`carteblanche`),
- Diners Club (`dinersclub`),
- Discover Card (`discover`),
- Interpayment (`interpayment`),
- JCB (`jcb`), 
- Maestro (`maestro`),
- Dankort (`dankort`), 
- NSPK MIR (`mir`),
- Troy (`troy`), 
- MasterCard (`mastercard`),
- Visa (`visa`), 
- UATP (`uatp`),
- Verve (`verve`),
- Carte Pratique CIBC (`cibc`),
- Carte client de la Banque Royale du Canada (`rbc`),
- Carte Accès TD Canada Trust (`tdtrust`),
- Carte Scotia de la Banque Scotia (`scotia`),
- Carte de guichet BMO (`bmoabm`),
- Carte HSBC Canada (`hsbc`)

```php
[
    'cc' => ['required', 'credit_card:visa'], //carte visa
    'cc' => ['required', 'credit_card:visa,mastercard'], //carte visa ou mastercard
]
```

> **Note**  
> Le `fournisseur` n'est pas obligatoire pour la regle `credit_card`. Si vous ne specifier pas de fournisseur, la regle utilisera <a href="https://en.wikipedia.org/wiki/Luhn_algorithm" target="_blank">l'algorithme de Luhn</a> pour determiner si le champ est valide.

<a name="regle-currency"></a>
#### currency

Le champ en cours de validation doit être un code monétaire valide (ex: _USD_, _EUR_, _XAF_). 

<a name="regle-current-password"></a>
#### current_password

> **Note**  
> Cette règle nécessite l'installation du module [d'authentification et d'autorisation](/docs/{version}/schild).

Le champ en cours de validation doit correspondre au mot de passe de l'utilisateur authentifié. Vous pouvez spécifier [un gestionnaire d'authentification](/docs/{version}/schild-authentification) en utilisant le premier paramètre de la règle :

```php
'password' => 'current_password:api'
```

> **Attention**  
> Ne pas confondre cette règle avec la règle [password](#regle-password) qui est utilisée pour valider le format d'un mot de passe (dans les pages d'inscription par exemple).

<a name="regle-date"></a>
#### date:`format`

Le champ en cours de validation doit être une date valide, non relative, selon la fonction PHP `strtotime`.

Vous pouvez définir un `format` de date pour vérifier que la date entrée, en plus d’être valide,e correspondant au format donnés. Cette règle de validation supporte tous les formats supportés par la classe [DateTime](https://www.php.net/manual/en/class.datetime.php) de PHP.

<a name="regle-date-equals"></a>
#### date_equals:`date`

Le champ en cours de validation doit être égal à la date donnée. Les dates seront passées à la fonction PHP `strtotime` afin d'être converties en une instance valide de `DateTime`.

<a name="regle-datetime"></a>
#### datetime

Le champ en cours de validation doit être une date valide dont le format est `Y-m-d H:i:s`.  
Cette règle est un raccourcir de la règle `date:Y-m-d H:i:s`.

<a name="regle-decimal"></a>
#### decimal:`min`,`max`

Le champ à valider doit être numérique et doit contenir le nombre de décimales spécifié :

```php
// Doit avoir exactement deux décimales (9.99)...
'price' => 'decimal:2'

// Doit avoir entre 2 et 4 décimales...
'price' => 'decimal:2,4'
```

<a name="regle-declined"></a>
#### declined

Le champ en cours de validation doit être `"no"`, `"off"`, `0`, ou `false`.

<a name="regle-declined-if"></a>
#### declined_if:`autrechamp`,`valeur`

Le champ en cours de validation doit être `"no"`, `"off"`, `0`, ou `false` si un autre champ en cours de validation est égal à une valeur spécifiée.

<a name="regle-default"></a>
#### default

Si l'attribut n'a pas de valeur, cette valeur par défaut sera utilisée à la place dans les données validées.

Par exemple, si vous avez une validation comme celle-ci

```php
use BlitzPHP\Validation\Validator;

$post = [
    'enabled' => null
];

$validation = Validator::make($post, [
    'enabled' => 'default:1|required|in:0,1'
    'published' => 'default:0|required|in:0,1'
]);

$validation->passes(); // true

// Obtenir les données valides/par défaut
$valid_data = $validation->getValidData();

$enabled = $valid_data['enabled'];
$published = $valid_data['published'];s
```

La validation est réussie parce que la valeur par défaut pour `enabled` et `published` est fixée à `1` et `0`, ce qui est valide.

<a name="regle-different"></a>
#### different:`champ`

Le champ en cours de validation doit avoir une valeur différente de `champ`.

<a name="regle-digits"></a>
#### digits:`valeur`

L'entier à valider doit avoir une longueur exacte de `valeur`.

<a name="regle-digits-between"></a>
#### digits\_between:`min`,`max`

L'entier à valider doit avoir une longueur comprise entre les valeurs `min` et `max` indiquées.

<a name="rule-dimensions"></a>
#### dimensions

Le fichier en cours de validation doit être une image répondant aux contraintes de dimension telles que spécifiées par les paramètres de la règle :

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

Les contraintes disponibles sont: `min_width`, `max_width`, `min_height`, `max_height`, `width`, `height`, `ratio`.

Une contrainte de `ratio` doit être représentée par la largeur divisée par la hauteur. Cela peut être spécifié soit par une fraction comme `3/2`, soit par un nombre flottant comme `1.5` :

```php
'avatar' => 'dimensions:ratio=3/2'
```

Comme cette règle nécessite plusieurs arguments, vous pouvez utiliser la méthode `Rule::dimensions` pour construire la règle de manière fluide :

```php
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

<a name="regle-digits-between"></a>
#### digits_between:`min`,`max`

L'entier à valider doit avoir une longueur comprise entre `min` et `max`.

<a name="regle-distinct"></a>
#### distinct

La champ en cours de validation doit être un tableau et que toutes les valeurs qu'il contient sont uniques, c'est-à-dire qu'il n'y a pas de valeurs en double dans le tableau.

```php
use BlitzPHP\Validation\Rule;

$post = [
    'integer' => [81, 3, 15],
    'string'  => ['white', 'green', 'blue'],
    'mixed'   => [81, false, 'orange']
];

$validator = Validation::make($post, [
    'integer' => 'distinct',
    'string'  => 'distinct',
    'mixed'   => 'distinct',
]);

$validator->passes(); // true
```

La règle `distinct` peut aussi être utiliser pour valider les éléments d'un tableau imbriqués.

```php
use BlitzPHP\Validation\Rule;

$post = [
    'foo' => [
        [
            'id' => 5,
            'name' => 'foo',
        ],
        [
            'id' => 6,
            'name' => 'bar',
        ],
        [
            'id' => 4,
            'name' => 'baz',
        ]
    ]
];

$validation = Validator::make($post, [
    'foo.*.id' => 'distinct',
]);

$validation->passes(); // true
```

Vous pouvez ajouter `ignore_case` aux arguments de la règle de validation pour qu'elle ignore les différences de casse :

```php
use BlitzPHP\Validation\Rule;

$post = [
    'color'  => ['white', 'green', 'blue', 'GREEN']
];

$validator = Validation::make($post, [
    'color' => 'distinct',
]);

$validator->passes(); // true

$validator = Validation::make($post, [
    'color' => 'distinct:ignore_case',
]);

$validator->passes(); // false
```

<a name="regle-discord-username"></a>
#### discord_username

Le champ en cours de validation doit être un nom d'utilisateur Discord valide (ex: _Blitz#2134_).

<a name="regle-doesnt-start-with"></a>
#### doesnt\_start\_with:_foo_,_bar_,...

Le champ en cours de validation ne doit pas commencer par l'une des valeurs données.

<a name="regle-doesnt-end-with"></a>
#### doesnt\_end\_with:_foo_,_bar_,...

Le champ en cours de validation ne doit pas se terminer par l'une des valeurs données.

<a name="regle-domain"></a>
#### domain

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/Domain_name" target="_blank">nom de domaine</a> valide.

<a name="regle-duplicate"></a>
#### duplicate

Le champ en cours de validation ne doit pas avoir les caractères ou les nombres dupliqués (ex: _1123456_).

<a name="regle-duplicate-character"></a>
#### duplicate_character

Le champ en cours de validation ne doit pas avoir les caractères dupliqués (ex: _1,2,3,4,5,6,7,8,9_).

<a name="regle-ean"></a>
#### ean:`taille`

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/International_Article_Number" target="_blank">numéro d'article européen (EAN)</a> valide.

Le paramètre `taille` est facultatif et accepte deux valeurs possibles à savoir `8` et `13` qui permettent de valider respectivement les EAN-8 et EAN-13.

<a name="regle-email"></a>
#### email

Le champ en cours de validation doit être formaté comme une adresse électronique. Par défaut, il utilise la fonction <a href="http://php.net/manual/en/function.filter-var.php" target="_blank">filter_var de PHP</a>, et souffre des mêmes restrictions :

```php
'email' => 'email'
```

Vous pouvez également définir le paramètre `dns` pour faire une validation basé sur le contrôle de DNS :

```php
'email' => 'email:dns'
```

> **Attention**  
> L'utilisation du mode `dns` requière l'extension PHP `intl`.

<a name="regle-ends-with"></a>
#### ends\_with:_foo_,_bar_,...

Le champ en cours de validation doit se terminer par l'une des valeurs données.

<a name="regle-enum"></a>
#### enum

La règle `Enum` est une règle basée sur une classe qui valide si le champ en cours de validation contient une valeur d'enum valide. La règle `Enum` accepte le nom de l'enum comme seul argument du constructeur :

```php
use App\Enums\ServerStatus;
use BlitzPHP\Validation\Rule;

$validator = Validation::make([
    'status' => [Rule::enum(ServerStatus::class)],
]);
```

<a name="regle-even-number"></a>
#### even_number

Le champ en cours de validation doit être un nombre pair.

<a name="regle-exists"></a>
#### exists:`table`,`colonne`

> **Note**  
> Cette règle nécessite l'installation du module de [base de données](/docs/{version}/base-de-donnees). Additionnellement, vous pourriez avoir besoin d'installer l'[ORM](/docs/{version}/wolke) pour certaines fonctionnalités liées à cette règle.

Le champ à valider doit exister dans une table de base de données donnée.

**Utilisation de base de la règle `exists`**

```php
'state' => 'exists:states'
```

Si l'option `colonne` n'est pas spécifiée, le nom du champ sera utilisé. Ainsi, dans ce cas, la règle validera que la table de la base de données `states` contient un enregistrement dont la valeur de la colonne `state` correspond à la valeur de l'attribut `state` de la requête.

**Spécification d'un nom de colonne personnalisé**

Vous pouvez spécifier explicitement le nom de la colonne de la base de données qui doit être utilisé par la règle de validation en le plaçant après le nom de la table de la base de données :

```php
'state' => 'exists:states,abbreviation'
```
Il peut arriver que vous deviez spécifier une connexion spécifique à la base de données à utiliser pour la requête d'existence. Vous pouvez le faire en ajoutant le nom de la connexion au nom de la table :

```php
'email' => 'exists:connection.staff,email'
```

Au lieu de spécifier directement le nom de la table, vous pouvez spécifier l’entité [Wolke](/docs/{version}/wolke) qui doit être utilisé pour déterminer le nom de la table :

```php
'user_id' => 'exists:App\Entities\User,id'
```

Si vous souhaitez personnaliser la requête exécutée par la règle de validation, vous pouvez utiliser la classe `Rule` pour définir la règle de manière fluide. Dans cet exemple, nous allons également spécifier les règles de validation sous forme de tableau au lieu d'utiliser le caractère `|` pour les délimiter :

```php
use BlitzPHP\Contracts\Database\BuilderInterface;
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function (BuilderInterface $query) {
            return $query->where('account_id', 1);
        }),
    ],
]);
```

Vous pouvez spécifier explicitement le nom de la colonne de la base de données qui doit être utilisée par la règle `exists` générée par la méthode `Rule::exists` en fournissant le nom de la colonne comme deuxième argument de la méthode :

```php
'state' => Rule::exists('states', 'abbreviation'),
```

<a name="regle-ext"></a>
#### ext:_foo,bar,..._

Le fichier en cours de validation doit avoir une extension attribuée par l'utilisateur correspondant à l'une des extensions énumérées :

```php
'photo' => ['required', 'ext:jpg,png'],
```

<a name="regle-file"></a>
#### file

Le champ en cours de validation doit être un fichier téléchargé avec succès.

```php
'photo' => ['required', 'ext:jpg,png'],
```

<a name="regle-float"></a>
#### float

Le champ en cours de validation doit être un nombre à virgule flottante.

<a name="regle-fullname"></a>
#### fullname

Le champ en cours de validation doit être une chaîne de caractères représentant un nom complet c'est-a-dire ayant au moins 6 caractères, au moins 2 mots, chaque mot devant comporter au moins 2 caractères.

<a name="regle-gt"></a>
#### gt:`champ`|`valeur`

Le champ en cours de validation doit être supérieur au `champ` ou à la `valeur` donné(e). Les deux champs doivent être du même type. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués en utilisant les mêmes conventions que la règle [`size`](#regle-size).

<a name="regle-gte"></a>
#### gte:`champ`|`valeur`

Le champ en cours de validation doit être supérieur ou égal au  `champ` ou à la `valeur` donné(e). Les deux champs doivent être du même type. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués en utilisant les mêmes conventions que la règle [`size`](#regle-size).

<a name="regle-gtin"></a>
#### gtin:`taille`

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/Global_Trade_Item_Number" target="_blank">code article international (GTIN)</a> valide.

Le paramètre `taille` est facultatif et accepte quatre valeurs possibles à savoir `8`, `12`, `13` et `14` qui permettent de valider respectivement les GTIN-8, GTIN-12, GTIN-13 et GTIN-14.

<a name="regle-hash"></a>
#### hash:`algo`,`allow_uppercase`

Le champ en cours de validation doit être un hachage cryptographique valide selon l'algorithme de hachage choisi.   
Le deuxième paramètre peut autoriser les caractères majuscules dans les hachages. Les algorithmes pris en charge sont `MD5`, `SHA1`, `SHA256`, `SHA512` et `CRC32`.

```php
$v1 = Validator::make(['key' => md5('a8jf0a4')], ['key' => 'hash:md5']);
$v2 = Validator::make(['key' => 'a8jf0a4'], ['key' => 'hash:md5']);

$v1->passes(); // true
$v2->passes(); // false
```

Si vous souhaitez autoriser les caractères majuscules, vous devez attribuer la valeur `true` au deuxième paramètre.

```php
$data = ['key' => strtoupper(md5('a8jf0a4'))];

$v1 = Validator::make($data, ['key' => 'hash:md5,true']);
$v2 = Validator::make($data, ['key' => 'hash:md5']);

$v1->passes(); // true
$v2->passes(); // false
```

<a name="regle-hashtag"></a>
#### hashtag

Le champ en cours de validation doit être un hashtag valide (ex: _#blitzphp_).

<a name="regle-hex"></a>
#### hex

Le champ en cours de validation doit être chaîne hexadécimal valide.

<a name="regle-hexcolor"></a>
#### hexcolor

Le champ en cours de validation doit contenir une valeur de couleur valide au format <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/hex-color" target="_blank">hexadécimal</a>.

<a name="regle-htmlclean"></a>
#### htmlclean

Le champ en cours de validation doit être exempt de tout code html.

<a name="regle-htmltag"></a>
#### htmltag

Le champ en cours de validation doit être une balise HTML.

<a name="regle-iban"></a>
#### iban

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/International_Bank_Account_Number" target="_blank">numéro de compte bancaire international (IBAN)</a> valide.

<a name="regle-instance-of"></a>
#### instance_of:`type`

Le champ en cours de validation doit être une instance de la classe ou de l'interface donnée.

```php
class Test {

}

 $post = [
    'obj'      => new Test,
    'obj_name' => Test::class,
];

$v1 = Validator::make($post, [
    'obj' => 'instance_of:Test'
]);
$v1->passes(); // true
        
$v2 = Validator::make($post, [
    'obj_name' => Rule::instanceOf(Test::class),
    'obj'      => Rule::instanceOf('Test'),
]);
$v2->passes(); // true
        
$v3 = Validator::make($post, [
    'obj'      => Rule::instanceOf(new Test),
    'obj_name' => Rule::instanceOf(new Test),
    'std'      => new stdClass,
]);
$v3->passes(); // true

$v4 = Validator::make($post, [
    'obj_name' => Rule::instanceOf()->type(Test::class),
]);
$v5->passes(); // true
    
$v6 = Validator::make($post, [
    'std' => 'instance_of:Test',
]);
$v6->passes(); // false
```

<a name="regle-image"></a>
#### image

Le fichier en cours de validation doit être une image (jpg, jpeg, png, bmp, gif, svg ou webp).

<a name="regle-imei"></a>
#### imei

Le champ en cours de validation doit être une <a href="https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity" target="_blank">identité internationale d'équipement mobile (IMEI)</a> valide.

<a name="regle-in"></a>
#### in:_foo,bar,..._

Le champ en cours de validation doit être inclus dans la liste de valeurs donnée. Comme cette règle nécessite souvent d'`imploser` un tableau, la méthode `Rule::in` peut être utilisée pour construire la règle de manière fluide :

```php
use BlitzPHP\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

Lorsque la règle `in` est combinée avec la règle `array`, chaque valeur du tableau d'entrée doit être présente dans la liste des valeurs fournies à la règle `in`. Dans l'exemple suivant, le code d'aéroport `LAS` dans le tableau d'entrée n'est pas valide puisqu'il n'est pas contenu dans la liste des aéroports fournie à la règle `in` :

```php
use BlitzPHP\Validation\Rule;

$input = [
    'airports' => ['NYC', 'LAS'],
];

Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
```

<a name="regle-in-array"></a>
#### in_array:`autrechamp`

Le champ en cours de validation doit exister dans les valeurs d'un autre champ.

<a name="regle-integer"></a>
#### integer

Le champ à valider doit être un nombre entier.

> **Attention**  
> Cette règle de validation ne vérifie pas que l'entrée est de type "integer", mais seulement que l'entrée est d'un type accepté par la règle `FILTER_VALIDATE_INT` de PHP. Si vous avez besoin de valider l'entrée comme étant un nombre, utilisez cette règle en combinaison avec [la règle de validation `numérique`] (#regle-numeric).

<a name="regle-ip"></a>
#### ip

Le champ en cours de validation doit être une adresse IP.

<a name="ipv4"></a>
#### ipv4

Le champ en cours de validation doit être une adresse IPv4.

<a name="ipv6"></a>
#### ipv6

Le champ en cours de validation doit être une adresse IPv6.

<a name="regle-isbn"></a>
#### isbn:`taille`

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/International_Standard_Book_Number" target="_blank">numéro international normalisé du livre (ISBN)</a> valide.

Le paramètre `taille` est facultatif et accepte deux valeurs possibles à savoir `10` et `13` qui permettent de valider respectivement les ISBN-10 et ISBN-13.

<a name="regle-issn"></a>
#### issn

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/International_Standard_Serial_Number" target="_blank">numéro de série international normalisé (ISSN)</a> valide.

<a name="regle-json"></a>
#### json

Le champ en cours de validation doit être une chaîne JSON valide.

<a name="regle-jwt"></a>
#### jwt

Le champ en cours de validation doit être au format d'un <a href="https://en.wikipedia.org/wiki/JSON_Web_Token" target="_blank">jeton Web JSON</a>.

<a name="regle-kebabcase"></a>
#### kebabcase

Le champ en cours de validation doit être formaté en <a href="https://en.wikipedia.org/wiki/Letter_case#Kebab_case" target="_blank">Kebab Case</a>.

<a name="regle-lt"></a>
#### lt:`champ`|`valeur`

Le champ en cours de validation doit être inférieur au `champ` ou la `valeur` donné. Les deux champs doivent être du même type. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués en utilisant les mêmes conventions que la règle [`size`](#regle-size).

<a name="regle-lte"></a>
#### lte:`champ`|`valeur`

Le champ en cours de validation doit être inférieur ou égal au `champ` ou la `valeur` donné. Les deux champs doivent être du même type. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués en utilisant les mêmes conventions que la règle [`size`](#regle-size).

<a name="regle-lowercase"></a>
#### lowercase

Le champ en cours de validation doit être en minuscules.

<a name="regle-mac"></a>
#### mac_address

Le champ en cours de validation doit être une adresse MAC.

<a name="regle-max"></a>
#### max:`valeur`

Le champ en cours de validation doit être inférieur ou égal à une `valeur` maximale. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués de la même manière que la règle [`size`](#regle-size).

<a name="regle-max-digits"></a>
#### max_digits:`valeur`

L'entier à valider doit avoir une longueur maximale de `valeur`.

<a name="regle-mimes"></a>
#### mimes:_foo_,_bar_...

Le fichier en cours de validation doit correspondre à l'un des types MIME indiqués :

```php
'video' => 'mimes:video/avi,video/mpeg,video/quicktime'
```

Pour déterminer le type MIME du fichier téléchargé, le contenu du fichier est lu et le framework tente de deviner le type MIME, qui peut être différent du type MIME fourni par le client.

<a name="regle-mimetypes"></a>
#### mimes:_foo_,_bar_...

Le fichier en cours de validation doit avoir un type MIME correspondant à l'une des extensions énumérées :

```php
'photo' => 'mimes:jpg,bmp,png'
```

Même si vous ne devez spécifier que les extensions, cette règle valide en fait le type MIME du fichier en lisant le contenu du fichier et en devinant son type MIME. Une liste complète des types MIME et de leurs extensions correspondantes est disponible à l'adresse suivante :

<a href="https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types" target="_blank">https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types</a>

<a name="types-mime-et-extensions"></a>
###### Types MIME et extensions

Cette règle de validation ne vérifie pas la concordance entre le type MIME et l'extension que l'utilisateur a attribuée au fichier. Par exemple, la règle de validation `mimes:png` considère qu'un fichier contenant du contenu PNG valide est une image PNG valide, même si le fichier s'appelle `photo.txt`. Si vous souhaitez valider l'extension du fichier attribuée par l'utilisateur, vous pouvez utiliser la règle `ext`.

<a name="regle-min"></a>
#### min:`valeur`

Le champ en cours de validation doit avoir une `valeur` minimale. Les chaînes de caractères, les nombres, les tableaux et les fichiers sont évalués de la même manière que la règle [`size`](#regle-size).

<a name="regle-min-digits"></a>
#### min_digits:`valeur`

L'entier à valider doit avoir une longueur minimale de `valeur`.

<a name="regle-missing"></a>
#### missing

Le champ en cours de validation ne doit pas être présent dans les données d'entrée.

<a name="regle-missing-if"></a>
#### missing_if:`autrechamp`,`valeur`,...

Le champ en cours de validation ne doit pas être présent si le champ `autrechamp` est égal à une `valeur` quelconque.

<a name="regle-missing-unless"></a>
#### missing_unless:`autrechamp`,`valeur`,...

Le champ en cours de validation ne doit pas être présent à moins que le champ `autrechamp` ne soit égal à une `valeur` quelconque.

<a name="regle-missing-with"></a>
#### missing\_with:_foo_,_bar_,...

Le champ en cours de validation ne doit être présent que si l'un des autres champs spécifiés est présent.

<a name="regle-missing-with-all"></a>
#### missing\_with\_all:_foo_,_bar_,...

Le champ en cours de validation ne doit être présent que si tous les autres champs spécifiés sont présents.

<a name="regle-multiple-of"></a>
#### multiple_of:`valeur`

Le champ en cours de validation doit être un multiple de la `valeur`.

<a name="regle-not-in"></a>
#### not\_in:_foo_,_bar_,...

Le champ en cours de validation ne doit pas être inclus dans la liste de valeurs donnée. La méthode `Rule::notIn` peut être utilisée pour construire la règle de manière fluide :

```php
use BlitzPHP\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

<a name="regle-not-in-array"></a>
#### not\_in\_array:`autrechamp`

Le champ en cours de validation ne doit pas exister dans les valeurs d'un autre champ.

<a name="regle-not-regex"></a>
#### not_regex:`pattern`

Le champ en cours de validation ne doit pas correspondre à l'expression régulière donnée.

En interne, cette règle utilise la fonction PHP `preg_match`. Le `pattern` spécifié doit obéir au même formatage que celui exigé par `preg_match` et donc inclure des délimiteurs valides. Par exemple : `'email' => 'not_regex:/^.+$/i'`.

> **Attention**  
> Lorsque vous utilisez les motifs `regex` / `not_regex`, il peut être nécessaire de spécifier vos règles de validation à l'aide d'un tableau au lieu d'utiliser les délimiteurs `|`, en particulier si l'expression régulière contient un caractère `|`.

<a name="regle-nullable"></a>
#### nullable

Le champ en cours de validation peut être `null`.

<a name="regle-numeric"></a>
#### numeric

Le champ en cours de validation doit être <a href="https://www.php.net/manual/en/function.is-numeric.php" target="_blank">numérique</a>.

<a name="regle-odd-number"></a>
#### odd_number

Le champ en cours de validation doit être un nombre impair.

<a name="regle-pascalcase"></a>
#### pascalcase

Le champ en cours de validation doit être formaté en <a href="https://en.wikipedia.org/wiki/Letter_case#Special_case_styles" target="_blank">Pascal Case</a>.

<a name="regle-password"></a>
#### password

Le champ en cours de validation doit être un mot de passe **valide du point de vue de la forme**. Ceci est principalement utilisé pour la validation des formulaire d'inscription. Consulter la section consacrée à la [validation de mot de passe](#validation-de-mot-de-passe) pour en savoir plus.

<a name="regle-pattern"></a>
#### pattern:`taille`,`separateur`

Le champ en cours de validation doit avoir le même motif correspondant aux paramètres de la règle. Une chaîne de caractères séparée par `separator` et dont chaque bloc a une taille égale à `taille`.

```php
$post = [
    'foo' => 'ABCD-EFGH-4444',
    'bar' => 'ABCD_EFGH_4444',
];

$v1 = Validator::make($post, ['foo' => 'pattern:4']);
$v1->passes(); // true

$v2 = Validator::make($post, ['foo' => 'pattern:5']);
$v2->passes(); // false, car les elements ont 4 caracteres au lieu de 5

$v3 = Validator::make($post, ['foo' => 'pattern:4,_']);
$v3->passes(); // false, cae le separateur ne correspond pas

$v4 = Validator::make($post, ['bar' => 'pattern:4,_']);
$v4->passes(); // true
```

<a name="regle-phone"></a>
#### phone:`countrycode`

Le champ en cours de validation doit être un numéro de téléphone valide. 

```php
$post = [
    'foo' => '+1 650 253 00 00',
    'bar' => '+55 11 91111 1111',
    'baz' => '11 91111 1111',
    'far' => '237677889900',
];

$v1 = Validator::make($post, ['foo' => 'phone']);
$v1->passes(); // true

$v2 = Validator::make($post, ['bar' => 'phone:br']);
$v2->passes(); // true

$v3 = Validator::make($post, ['baz' => 'phone:br']);
$v3->passes(); // false

$v4 = Validator::make($post, ['far' => 'phone:cm']);
$v4->passes(); // true

$v5 = Validator::make($post, ['bar' => 'phone:cm']);
$v5->passes(); // false
```

> **Note**  
> Cette règle utilise un ensemble d'expression régulières pour valider les données.   
> Si le paramètre `countrycode` n'est pas spécifié, la règle tentera de faire une validation sommaire.

<a name="regle-port"></a>
#### port

Le champ en cours de validation doit être un numéro de port valide.

<a name="regle-postalcode"></a>
#### postalcode:_foo_,_bar_

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/Postal_code" target="_blank">code postal</a> correspondant à l'un des pays donnés en paramètres.

```php
$post = [
    'foo' => '44141',
    'bar' => '2240',
    'baz' => '25746',
    'far' => 'A9A 9A9',
    'faz' => '123-4567',
];
    
$v1 = Validator::make($post, ['foo' => 'postalcode:de']);
$v1->passes(); // true

$v2 = Validator::make($post, ['bar' => 'postalcode:de']);
$v2->passes(); // false

$v3 = Validator::make($post, ['baz' => 'postalcode:de,ca']);
$v3->passes(); // true, car est un code postal allemand

$v4 = Validator::make($post, ['far' => 'postalcode:de,ca']);
$v4->passes(); // true, car est un code postal canadien

$v5 = Validator::make($post, ['faz' => 'postalcode:de,ca']);
$v5->passes(); // false, car n'est ni un code postal canadien, ni allemand mais japonais
```

> **Note**  
> Si aucun code n'est transmit, la règle tentera de faire une validation sommaire.

Si de base vous ne connaissez pas le code pays à utiliser et que celui-ci dépend d'un autre champ, vous pouvez utiliser la classe `Rule` pour construire votre règle.

```php
$post = [
    'foo' => '44141',
    'bar' => 'A9A 9A9',
    'country' => 'de',
];
    
$v1 = Validator::make($post, ['foo' => Rule::postalcode()->reference('country')]);
$v1->passes(); // true
    
$v2 = Validator::make($post, ['bar' => Rule::postalcode()->reference('country')]);
$v2->passes(); // false
```

<a name="regle-present"></a>
#### present

Le champ en cours de validation doit exister dans les données d'entrée.

<a name="regle-present-if"></a>
#### present_if:`autrechamp`,`valeur`

Le champ en cours de validation doit être présent si le champ `autrechamp` est égal à une `valeur` quelconque.

<a name="regle-present-unless"></a>
#### present_unless:`autrechamp`,`valeur`

Le champ en cours de validation doit être présent à moins que le champ `autrechamp` ne soit égal à une `valeur` quelconque.

<a name="regle-present-with"></a>
#### present\_with:_foo_,_bar_,...

Le champ en cours de validation ne doit être présent que **si l'un** des autres champs spécifiés est présent.

<a name="regle-present-with-all"></a>
#### present\_with\_all:_foo_,_bar_,...

Le champ en cours de validation ne doit être présent que **si tous** les autres champs spécifiés sont présents.

<a name="regle-prohibited"></a>
#### prohibited

Le champ en cours de validation doit être absent ou vide. Un champ est "vide" s'il répond à l'un des critères suivants :

- La valeur est `null`.
- La valeur est une chaîne vide.
- La valeur est un tableau vide ou un objet `Countable` vide.
- La valeur est un fichier téléchargé dont le chemin d'accès est vide.

<a name="regle-prohibited-if"></a>
#### prohibited_if:`autrechamp`,`valeur`,...

Le champ en cours de validation doit être absent ou vide si le champ `autrechamp` est égal à une `valeur` quelconque. Un champ est "vide" s'il répond à l'un des critères suivants :

- La valeur est `null`.
- La valeur est une chaîne vide.
- La valeur est un tableau vide ou un objet `Countable` vide.
- La valeur est un fichier téléchargé dont le chemin d'accès est vide.

<a name="regle-prohibited-unless"></a>
#### prohibited_unless:`autrechamp`,`valeur`,...

Le champ soumis à la validation doit être absent ou vide, à moins que le champ `autrechamp` ne soit égal à une `valeur` quelconque. Un champ est "vide" s'il répond à l'un des critères suivants :

- La valeur est `null`.
- La valeur est une chaîne vide.
- La valeur est un tableau vide ou un objet `Countable` vide.
- La valeur est un fichier téléchargé dont le chemin d'accès est vide

<a name="regle-prohibits"></a>
#### prohibits:`autreschamps`,...

Si le champ en cours de validation n'est pas manquant ou vide, tous les champs `autreschamps` doivent être manquants ou vides. Un champ est "vide" s'il répond à l'un des critères suivants :

- La valeur est `null`.
- La valeur est une chaîne vide.
- La valeur est un tableau vide ou un objet `Countable` vide.
- La valeur est un fichier téléchargé dont le chemin d'accès est vide

<a name="regle-regex"></a>
#### regex:`pattern`

Le champ en cours de validation doit correspondre à l'expression régulière donnée.

En interne, cette règle utilise la fonction PHP `preg_match`. Le pattern spécifié doit obéir au même formatage que celui exigé par `preg_match` et donc inclure des délimiteurs valides. Par exemple : `'email' => 'regex:/^.+@.+$/i'`.

> **Attention**  
> Lors de l'utilisation des motifs `regex` / `not_regex`, il peut être nécessaire de spécifier les règles dans un tableau au lieu d'utiliser les délimiteurs `|`, en particulier si l'expression régulière contient un caractère `|`.

<a name="regle-required"></a>
#### required

Le champ à valider doit être présent dans les données d'entrée et ne doit pas être vide. Un champ est "vide" s'il répond à l'un des critères suivants :

- La valeur est `null`.
- La valeur est une chaîne vide.
- La valeur est un tableau vide ou un objet `Countable` vide.
- La valeur est un fichier téléchargé dont le chemin d'accès est vide

<a name="regle-required-if"></a>
#### required_if:`autrechamp`,`valeur`,...

Le champ en cours de validation doit être présent et non vide si le champ `autrechamp` est égal à une `valeur` quelconque.

Si vous souhaitez construire une condition plus complexe pour la règle `required_if`, vous pouvez utiliser la méthode `Rule::requiredIf`. Cette méthode accepte un booléen ou une closure. Lorsqu'on lui passe une closure, celle-ci doit renvoyer `true` ou `false` pour indiquer si le champ en cours de validation est obligatoire :

```php
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;

Validator::make($this->request->all(), [
    'role_id' => Rule::requiredIf(auth()->user()->is_admin),
]);

Validator::make($this->request->all(), [
    'role_id' => Rule::requiredIf(fn () => auth()->user()->is_admin),
]);
```

<a name="regle-required-if-accepted"></a>
#### required\_if\_accepted:`autrechamp`

Le champ en cours de validation doit être présent et non vide si le champ `autrechamp` est égal à `yes`, `on`, `1`, `"1"`, `true`, ou `"true"`.

<a name="regle-required-if-declined"></a>
#### required\_if\_declined:`autrechamp`

Le champ en cours de validation doit être présent et non vide si le champ `autrechamp` est égal à `no`, `off`, `0`, `"0"`, `false`, ou `"false"`.

<a name="regle-required-unless"></a>
#### required_unless:`autrechamp`,`valeur`,...

Le champ en cours de validation doit être présent et non vide, sauf si le champ `autrechamp` est égal à n'importe quelle `valeur`. Cela signifie également que `autrechamp` doit être présent dans les données de la requête, sauf si `valeur` est `null`. Si `valeur` est `null` (`required_unless:name,null`), le champ en cours de validation sera obligatoire sauf si le champ de comparaison est `null` ou si le champ de comparaison est absent des données de la requête.

<a name="regle-required-with"></a>
#### required\_with:_foo_,_bar_,...

Le champ en cours de validation doit être présent et non vide **uniquement si** l'un des autres champs spécifiés est présent et non vide.

<a name="regle-required-with-all"></a>
#### required\_with\_all:_foo_,_bar_,...

Le champ en cours de validation ne doit être présent et non vide **que si** tous les autres champs spécifiés sont présents et non vides.

<a name="regle-required-without"></a>
#### required\_without:_foo_,_bar_,...

Le champ en cours de validation doit être présent et non vide **uniquement lorsque** l'un des autres champs spécifiés est vide ou non présent.

<a name="regle-required-without-all"></a>
#### required\_without\_all:_foo_,_bar_,...

Le champ en cours de validation doit être présent et non vide **uniquement lorsque** tous les autres champs spécifiés sont vides ou non présents.

<a name="regle-same"></a>
#### same:`champ`

Le `champ` donné doit correspondre au champ en cours de validation.

<a name="regle-semver"></a>
#### semver

Le champ en cours de validation doit être un numéro de version valide utilisant le <a href="https://semver.org/" target="_blank">Versioning Semantique</a>.

<a name="regle-slash-end-of-string"></a>
#### slash\_end\_of\_string

Le champ en cours de validation doit se terminer par un `slash`.

<a name="regle-slug"></a>
#### slug

Le champ en cours de validation doit être un <a href="https://en.wikipedia.org/wiki/Clean_URL#Slug" target="_blank">texte court adapté à l'utilisateur et au référencement (slug)</a>.

<a name="regle-size"></a>
#### size:`valeur`

Le champ en cours de validation doit avoir une taille correspondant à la `valeur` donnée. Pour les données de type chaîne, la `valeur` correspond au nombre de caractères. Pour les données numériques, la `valeur` correspond à une valeur entière donnée (l'attribut doit également avoir la règle `numeric` ou `integer`). Pour un tableau, la `valeur` correspond au nombre d'éléments du tableau. Pour les fichiers, `valeur` correspond à la taille du fichier en kilo-octets. Voyons quelques exemples :

```php
// Valider qu'une chaîne de caractères contient exactement 12 caractères...
'title' => 'size:12';

// Valider qu'un entier fourni est égal à 10...
'seats' => 'integer|size:10';

// Valider qu'un tableau a exactement 5 éléments...
'tags' => 'array|size:5';

// Valider qu'un fichier téléchargé fait exactement 512 kilo-octets...
'image' => 'file|size:512';
```

<a name="regle-snakecase"></a>
#### snakecase

Le champ en cours de validation doit être formaté en <a href="https://en.wikipedia.org/wiki/Snake_case" target="_blank">Snake Case</a>.

<a name="regle-starts-with"></a>
#### starts\_with:_foo_,_bar_,...

Le champ en cours de validation doit commencer par l'une des valeurs données.

<a name="regle-string"></a>
#### string

Le champ soumis à validation doit être une chaîne de caractères. Si vous souhaitez que le champ puisse également être `null`, vous devez lui attribuer la règle `nullable`.

<a name="regle-time"></a>
#### time:`mode`

Le champ en cours de validation doit être une heure valide dont le format est `H:i`.

Le paramètre `mode` n'est pas obligatoire mais si vous le définissez à `strict`, le format de l'heure devra être `H:i:s`.  
  
Cette règle est un raccourcir de la règle `date:H:i:s`.

<a name="regle-timezone"></a>
#### timezone

Le champ en cours de validation doit être un identifiant de fuseau horaire valide selon la méthode `DateTimeZone::listIdentifiers`.

Les arguments <a href="https://www.php.net/manual/en/datetimezone.listidentifiers.php" target="_blank">acceptés par la méthode `DateTimeZone::listIdentifiers`</a> peuvent également être fournis à cette règle de validation :

```php
'timezone' => 'required|timezone:all';

'timezone' => 'required|timezone:Africa';

'timezone' => 'required|timezone:per_country,US';
```

<a name="regle-titlecase"></a>
#### titlecase

Le champ en cours de validation doit être formaté en <a href="https://en.wikipedia.org/wiki/Title_case" target="_blank">Title Case</a>.

<a name="regle-ulid"></a>
#### ulid

Le champ en cours de validation doit être un <a href="https://github.com/ulid/spec" target="_blank">ULID (Universally Unique Lexicographically Sortable Identifier)</a> valide.

<a name="regle-unique"></a>
#### unique:`table`,`colonne`

> **Note**  
> Cette règle nécessite l'installation du module de [base de données](/docs/{version}/base-de-donnees). Additionnellement, vous pourriez avoir besoin d'installer l'[ORM](/docs/{version}/wolke) pour certaines fonctionnalités liées à cette règle.

Le champ en cours de validation ne doit pas exister dans la table de base de données donnée.

**Spécification d'un nom de table / colonne personnalisé:**

Au lieu de spécifier directement le nom de la table, vous pouvez spécifier l’entité [Wolke](/docs/{version}/wolke) qui doit être utilisé pour déterminer le nom de la table :

```php
'email' => 'unique:App\Entities\User,email_address'
```

L'option `colonne` peut être utilisée pour spécifier la colonne de la base de données correspondant au champ. Si cette option n'est pas spécifiée, le nom du champ en cours de validation sera utilisé.

```php
'email' => 'unique:users,email_address'
```

**Spécification d'une connexion de base de données personnalisée**

Il peut arriver que vous deviez définir une connexion personnalisée pour les requêtes de base de données effectuées par le validateur. Pour ce faire, vous pouvez ajouter le nom de la connexion au nom de la table :

```php
'email' => 'unique:connection.users,email_address'
```

**Forcer une règle unique à ignorer un identifiant donné:**

Il peut arriver que vous souhaitiez ignorer un identifiant donné lors de la validation unique. Prenons l'exemple d'un écran de "mise à jour du profil" qui comprend le nom, l'adresse électronique et la localisation de l'utilisateur. Vous voudrez probablement vérifier que l'adresse électronique est unique. Toutefois, si l'utilisateur ne modifie que le champ du nom et non celui de l'adresse électronique, vous ne voulez pas qu'une erreur de validation soit générée parce que l'utilisateur est déjà le propriétaire de l'adresse électronique en question.

Pour demander au validateur d'ignorer l'identifiant de l'utilisateur, nous utiliserons la classe `Rule` pour définir la règle de manière fluide. Dans cet exemple, nous spécifierons également les règles de validation sous la forme d'un tableau au lieu d'utiliser le caractère `|` pour délimiter les règles :

```php
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> **Attention**  
> Vous ne devez jamais transmettre à la méthode `ignore` une entrée de requête contrôlée par l'utilisateur. Au contraire, vous ne devez transmettre qu'un identifiant unique généré par le système, tel qu'un identifiant auto-incrémenté ou un UUID provenant d'une instance de modèle Wolke. Dans le cas contraire, votre application sera vulnérable à une attaque par injection SQL.

Au lieu de transmettre la valeur de la clé du modèle à la méthode `ignore`, vous pouvez également transmettre l'instance entière du modèle. BlitzPHP extraira automatiquement la clé du modèle :

```php
Rule::unique('users')->ignore($user)
```

Si votre table utilise un nom de colonne de clé primaire différent de id, vous pouvez spécifier le nom de la colonne lorsque vous appelez la méthode `ignore` :

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

Par défaut, la règle `unique` vérifie l'unicité de la colonne correspondant au nom de l'attribut en cours de validation. Toutefois, vous pouvez passer un nom de colonne différent comme deuxième argument de la méthode `unique` :

```php
Rule::unique('users', 'email_address')->ignore($user->id)
```

**Ajout de clauses `Where` supplémentaires:**

Vous pouvez spécifier des conditions de requête supplémentaires en personnalisant la requête à l'aide de la méthode `where`. Par exemple, ajoutons une condition de requête qui limite la recherche aux enregistrements dont la valeur de la colonne `account_id` est `1` :

```php
'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))
```

<a name="regle-uploaded-file"></a>
#### uploaded_file

Le champ en cours de validation doit être un fichier téléchargé avec succès.

<a name="regle-uppercase"></a>
#### uppercase

Le champ en cours de validation doit être en majuscules.

<a name="regle-url"></a>
#### url

Le champ en cours de validation doit être une URL valide.

Si vous souhaitez spécifier les protocoles URL qui doivent être considérés comme valides, vous pouvez les transmettre en tant que paramètres de la règle de validation :

```php
'url' => 'url:http,https',

'game' => 'url:minecraft,steam',
```

<a name="regle-username"></a>
#### username

Le champ en cours de validation doit être un nom d'utilisateur valide. Il doit être composé de caractères alphanumériques, de caractères de soulignement, de caractères moins et commencer par un caractère alphabétique. Les caractères de soulignement et de minoration multiples ne sont pas autorisés. Les caractères de soulignement et de minoration ne sont pas autorisés au début ou à la fin.

<a name="regle-uuid"></a>
#### uuid

Le champ en cours de validation doit être un identifiant universel unique (UUID) RFC 4122 (version 1, 3, 4 ou 5) valide.

<a name="regle-vatid"></a>
#### vatid

Le champ en cours de validation doit être <a href="https://en.wikipedia.org/wiki/VAT_identification_number" target="_blank">un identifiant européen de TVA</a> (ex: _EL123456789123_)

<a name="ajout-conditionnel-de-regles"></a>
## Ajout conditionnel de règles

<a name="validation-en-cas-de-presence"></a>
### Validation en cas de présence

Dans certaines situations, vous pouvez souhaiter exécuter des contrôles de validation sur un champ **uniquement** si ce champ est présent dans les données à valider. Pour y parvenir rapidement, ajoutez la règle `sometimes` à votre liste de règles :

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

Dans l'exemple ci-dessus, le champ `email` ne sera validé que s'il est présent dans le tableau `$data`.

> **Note**  
> Si vous essayez de valider un champ qui devrait toujours être présent mais qui peut être vide, consultez cette [note sur les champs facultatifs](#optional-vs-nullable).

<a name="optional-vs-nullable"></a>
#### Optional Vs Nullable

Il arrive que des attributs soient omis ou qu'ils soient nuls. Ces cas doivent être traités avec précaution et donner lieu à des résultats différents après la validation.

Pour les attributs facultatifs qui peuvent être exclus des données en cours de validation, c'est-à-dire qui ne sont validés que si les données sont présentes, la règle `sometimes` peut être utilisée. Si cette règle est spécifiée, l'attribut peut être complètement omis **OU** il doit répondre aux critères de validation. Cette règle est très utile pour des éléments tels que les filtres de recherche ou les marqueurs de pagination qui ne sont pas toujours nécessaires :

```php
[
    'filters' => 'sometimes|array',
]
```

Dans cet exemple, `filters` est entièrement facultatif, mais s'il est spécifié, il doit s'agir d'un tableau de valeurs. Passer `[filters => '']` ne serait pas valide, il faudrait que ce soit` [filters => []]`.

Parfois, au lieu que l'attribut soit facultatif, il doit être indéfini, c'est-à-dire `null`. En général, il est préférable d'utiliser `sometimes` et d'omettre la valeur, mais il peut arriver que l'attribut soit maintenu avec une valeur `null`. Dans ce cas, utilisez la règle `nullable`. Cette règle permet à l'attribut d'être présent sans aucune valeur. Par exemple : la date d'anniversaire de l'utilisateur peut être nullable ou une date : `nullable|date`.

> **Note**  
> L'utilisation de données nullables peut causer des problèmes car la bibliothèque <a href="https://github.com/dimtrovich/validation" target="_blank">dimtrovich/validation</a> utilise un typage strict. Cela signifie que de nombreuses règles qui testent une chaîne, un tableau ou un nombre sont erronées parce qu'elles reçoivent `null`. Il s'agit d'une ambiguïté dans le processus de définition des règles. Par exemple, la règle : `name => string|max:200` telle qu'elle est définie implique implicitement que le nom doit être une chaîne de 200 caractères maximum - `null` ne devrait pas être valide, mais pour maintenir une compatibilité partielle, elle autorisera null.  
> La prochaine version majeure de cette bibliothèque supprimera cette manipulation et fera en sorte que ce type de définition exige que le champ soit à la fois présent et ait une valeur qui ne soit pas vide (à moins que vide ne soit spécifiquement autorisé). Pour autoriser les valeurs nulles, la règle `nullable` devra être explicitement définie. Il est donc recommandé de toujours utiliser `nullable` ou `sometimes`.

<a name="validation-des-tableaux"></a>
## Validation des tableaux

Le validateur de BlitzPHP peut valider des tableaux de données complexes en utilisant la notation par points (`.`) pour définir la structure du tableau. Il existe quelques variantes et quelques cas limites dont il faut tenir compte pour éviter les problèmes.

La situation la plus courante consiste à vouloir autoriser un tableau d'options similaire.

```php
[
    'skills'              => 'array',
    'skills.*.id'         => 'required|numeric',
    'skills.*.percentage' => 'required|numeric'
],
```

Les règles de l'exemple précédent sont définies pour valider les données relatives à l'utilisateur et comprennent un tableau de compétences. Chaque `skills` possède un identifiant et une valeur en pourcentage. Dans ce cas, la clé parente `skills` doit avoir la règle `array`. Ceci est nécessaire pour s'assurer que les données sont bien un tableau. Chaque propriété de `skills` est ensuite référencée en utilisant `*` pour indiquer qu'il y a plusieurs valeurs dans l'attribut `skills`.

Ces règles valideraient la structure de tableau suivante :

```php
[
    'skills' => [
        [
            'id' => 3,
            'percentage' => 50,
        ],
        [
            'id' => 17,
            'percentage' => 50,
        ],
    ]
]
```

La situation la moins courante est celle d'un tableau de tableaux sans clé parente. Dans ce cas, il n'y a pas de préfixe et chaque sous-clé commence par un `*`. Dans cette situation, vous devez veiller à ne pas mélanger des paires clé -> valeur standard avec les données du tableau.

Par exemple :

```php
[
    '*.id'         => 'required|numeric',
    '*.percentage' => 'required|numeric'
]
```

serait utilisé pour valider la structure de tableau suivante :

```php
[
    [
        'id' => 3,
        'percentage' => 50,
    ],
    [
        'id' => 17,
        'percentage' => 50,
    ],
]
```

Pour éviter tout problème, vous devez vous assurer que les données ne comprennent pas :

```php
[
    'name' => 'foo bar',
    [
        'id' => 3,
        'percentage' => 50,
    ],
    [
        'id' => 17,
        'percentage' => 50,
    ],
]
```

<a name="validation-des-donnees-d-un-tableau-imbrique"></a>
### Règles de validation dépendantes et tableaux de données

Certaines règles sont utilisées pour déterminer la présence ou la nécessité de certaines clés. Elles utilisent généralement le nom de la clé standard, par exemple : `confirm_password` doit être identique au champ `password`, de sorte que la règle s'écrit : `same:password`.

Cependant, pour les données de type tableau, cela ne fonctionnera pas car l'attribut n'est pas le nom de l'attribut mais le chemin d'accès à cet attribut.

En utilisant le même tableau `skills` comme exemple, supposons que nous voulions exiger un label si la compétence est nouvelle. Si l'on spécifie `required_if:id:null`, la validation recherchera un attribut nommé `id` à la racine des données - mais il n'existe pas, ou il se peut qu'elle trouve la mauvaise clé.

Au lieu de cela, nous devons lier explicitement la règle à la même clé de compétence en écrivant la règle sous la forme : `required_if:skills.*.id,null`. Si nous ne le faisons pas, la règle sera ignorée ou échouera. Il en va de même lorsque l'on utilise des tableaux de tableaux : les références à d'autres champs de ce tableau doivent être préfixées par un `*`. par exemple : `required_if:*.id,null`.

Voici des exemples des deux syntaxes :

```php
[
    'skills.*.id'         => 'sometimes|numeric',
    'skills.*.percentage' => 'required|numeric',
    'skills.*.title'      => 'required_if:skills.*.id,null|string',
]
```

Et un tableau de tableaux :

```php
[
    '*.id'         => 'sometimes|numeric',
    '*.percentage' => 'required|numeric',
    '*.title'      => 'required_if:*.id,null|string',
]
```

<a name="validation-des-donnees-d-un-tableau-imbrique"></a>
### Validation des données d'un tableau imbriqué

La validation des champs de saisie d'un formulaire basé sur un tableau imbriqué ne doit pas être une corvée. Vous pouvez utiliser la "notation par points" pour valider les attributs à l'intérieur d'un tableau. Par exemple, si la requête HTTP entrante contient un champ `photos[profil]`, vous pouvez le valider comme suit :

```php
use BlitzPHP\Validation\Validator;

$validator = Validator::make($input, [
    'photos.profile' => 'required|image',
]);
```

Vous pouvez également valider chaque élément d'un tableau. Par exemple, pour valider que chaque courriel dans un champ de saisie d'un tableau donné est unique, vous pouvez faire ce qui suit :

```php
$validator = Validator::make($input, [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

De même, vous pouvez utiliser le caractère `*` pour spécifier des [messages de validation personnalisés dans vos fichiers de langue](#specification-d-un-message-personnalise-pour-un-attribut-donne), ce qui facilite l'utilisation d'un message de validation unique pour les champs basés sur des tableaux :

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Chaque personne doit avoir une adresse électronique unique',
    ]
],
```

<a name="validation-des-fichiers"></a>
## Validation des fichiers

BlitzPHP fournit une variété de règles de validation qui peuvent être utilisées pour valider les fichiers téléchargés, telles que `mimes`, `image`, `min` et `max`. Bien que vous soyez libre de spécifier ces règles individuellement lors de la validation des fichiers, BlitzPHP offre également un constructeur de règles de validation de fichiers fluide que vous pouvez trouver pratique :

```php
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;
 
Validator::validate($input, [
    'attachment' => [
        'required',
        Rule::file()
            ->types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

Si votre application accepte les images téléchargées par vos utilisateurs, vous pouvez utiliser la méthode `image()` de la règle `File` pour indiquer que le fichier téléchargé doit être une image. En outre, la règle `dimensions` peut être utilisée pour limiter les dimensions de l'image :

```php
use BlitzPHP\Validation\Validator;
use BlitzPHP\Validation\Rule;

Validator::validate($input, [
    'photo' => [
        'required',
        Rule::file()
            ->image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

> **Note**  
> Pour plus d'informations sur la validation des dimensions des images, voir [la documentation sur les règles de dimension](#regle-dimensions).

<a name="tailles-des-fichiers"></a>
#### Tailles des fichiers

Par commodité, les tailles minimale et maximale des fichiers peuvent être spécifiées sous la forme d'une chaîne de caractères avec un suffixe indiquant les unités de taille du fichier. Les suffixes `kb`, `mb`, `gb` et `tb` sont pris en charge :

```php
Rule::file()->image()->min('1kb')->max('10mb')
```

<a name="types-des-fichiers"></a>
#### Types des fichiers

Même si vous ne devez spécifier que les extensions lorsque vous invoquez la méthode `types`, cette méthode valide en fait le type MIME du fichier en lisant le contenu du fichier et en devinant son type MIME. Une liste complète des types MIME et de leurs extensions correspondantes est disponible à l'adresse suivante :

<a href="https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types" target="_blank">https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types</a>

<a name="validation-de-mot-de-passe"></a>
## Validation de mot de passe

Pour s'assurer que les mots de passe ont un niveau de complexité adéquat, vous pouvez utiliser la règle `password` de BlitzPHP :

```php
use BlitzPHP\validation\Validator;
use BlitzPHP\Validation\Rule;
 
$validator = Validator::make($this->request->all(), [
    'password' => ['required', 'confirmed', Rule::password()->min(8)],
]);
```

L'objet `Password rule` vous permet de personnaliser facilement les exigences de complexité des mots de passe pour votre application, par exemple en spécifiant que les mots de passe doivent contenir au moins une lettre, un chiffre, un symbole ou des caractères avec une casse mixte :

```php
// Exige au moins 8 caractères...
Rule::password()->min(8)
 
// Exige au moins une lettre...
Rule::password()->min(8)->letters()
 
// Exige au moins une lettre majuscule et une lettre minuscule...
Rule::password()->min(8)->mixedCase()
 
// Exige au moins un chiffre...
Rule::password()->min(8)->numbers()
 
// Requiert au moins un symbole...
Rule::password()->min(8)->symbols()
```

En outre, vous pouvez vous assurer qu'un mot de passe n'a pas été compromis dans le cadre d'une fuite de données de mot de passe publique en utilisant la méthode `uncompromised` :

```php
Rule::password()->min(8)->uncompromised()
```

En interne, l'objet `Password rule` utilise le modèle <a href="https://en.wikipedia.org/wiki/K-anonymity" target="_blank">k-Anonymity</a> pour déterminer si un mot de passe a été divulgué via le service <a href="https://haveibeenpwned.com/" target="_blank">haveibeenpwned.com</a> sans sacrifier la confidentialité ou la sécurité de l'utilisateur.

Par défaut, si un mot de passe apparaît au moins une fois dans une fuite de données, il sera considéré comme compromis. Vous pouvez personnaliser ce seuil en utilisant le premier argument de la méthode `uncompromised` :

```php
// S'assurer que le mot de passe apparaît moins de 3 fois dans la même fuite de données...

Rule::password()->min(8)->uncompromised(3)
```

Bien entendu, vous pouvez enchaîner toutes les méthodes dans les exemples ci-dessus :

```php
Rule::password()
    ->min(8)
    ->letters()
    ->mixedCase()
    ->numbers()
    ->symbols()
    ->uncompromised()
```

<a name="definition-des-regles-de-mot-de-passe-par-defaut"></a>
#### Définition des règles de mot de passe par défaut

Vous pouvez trouver pratique de spécifier les règles de validation par défaut pour les mots de passe à un seul endroit de votre application. Vous pouvez facilement y parvenir en utilisant la méthode `Password::defaults`, qui accepte une closure. La closure donnée à la méthode `defaults` doit renvoyer la configuration par défaut de la règle `Password`. En règle générale, la règle `defaults` doit être appelée à partir de l'événement `app:init` ou `pre_system` d'un écouteur d'événement de votre application :

```php
<?php

namespace App\Listeners;

use BlitzPHP\Contracts\Event\EventListenerInterface;
use BlitzPHP\Contracts\Event\EventManagerInterface;
use Dimtrovich\Validation\Rules\Password;

class AppListener implements EventListenerInterface
{
    /**
     * {@inheritDoc}
     */
    public function listen(EventManagerInterface $event): void
    {
        $event->on('app:init', function () {
            Password::defaults(function () {
                $rule = Password::min(8);
 
                return is_prod()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
            });
        });
    }
}
```

Ensuite, lorsque vous souhaitez appliquer les règles par défaut à un mot de passe particulier en cours de validation, vous pouvez invoquer la méthode `default` :

```php
'password' => ['required', Rule::password()->default()],
```

Il peut arriver que vous souhaitiez ajouter des règles de validation supplémentaires à vos règles de validation de mot de passe par défaut. Pour ce faire, vous pouvez utiliser la méthode `rules` :

```php
use App\Rules\ZxcvbnRule;
use Dimtrovich\Validation\Rules\Password;
 
Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);
 
    // ...
});
```

<a name="regles-de-validation-personnalisees"></a>
## Règles de validation personnalisées

<a name="utilisation-des-objets-de-regles"></a>
### Utilisation des objets de règles

BlitzPHP fournit une variété de règles de validation utiles ; cependant, vous pouvez souhaiter spécifier certaines de vos propres règles. L'une des méthodes pour enregistrer des règles de validation personnalisées consiste à utiliser des objets de règles. Pour générer un nouvel objet règle, vous pouvez utiliser la commande Klinge `make:rule`. Utilisons cette commande pour générer une règle qui vérifie qu'une chaîne de caractères commence par une lettre majuscule. BlitzPHP placera la nouvelle règle dans le répertoire `/app/Rules`. Si ce répertoire n'existe pas, BlitzPHP le créera lorsque vous exécuterez la commande Klinge pour créer votre règle :

```shell
php klinge make:rule StartWithUppercase
```

Une fois la règle créée, nous sommes prêts à définir son comportement. Un objet règle contient une seule méthode : `check`. Cette méthode reçoit la valeur de l'attribut à valider :

```php
<?php
 
namespace App\Rules;
 
use BlitzPHP\Validation\Rules\AbstractRule;
 
class StartWithUppercase extends AbstractRule
{
    /**
     * Run the validation rule.
     *
     * @params mixed $value
     */
    public function check($value): bool
    {
        return strtoupper($value[0]) === $value[0];
    }
}
```

Une fois la règle définie, vous pouvez l'attacher à un validateur avec vos autres règles de validation :

```php
use App\Rules\StartWithUppercase;
use BlitzPHP\Validation\Validator;
 
$validator = Validator::make($this->request->all(), [
    'name' => ['required', 'string', new StartWithUppercase],
]);
```

<a name="decouverte-automatique-des-regles"></a>
#### Découverte automatique des règles

BlitzPHP peut découvrir automatiquement tous les fichiers règle que vous avez créés dans n’importe quel namespace défini. Cela permet une utilisation simple de toutes les règles de validation disponible. Pour qu'un fichier de règle personnalisée soit découvert, il doit répondre aux exigences suivantes :

* Son namespace doit être accessible via Composer (pour des package tiers) ou défini dans `app/Config/Autoload.php`
* À l’intérieur du namespace, le fichier doit se trouver dans le dossier `{namespace}/Rules`
* Il doit étendre `BlitzPHP\Validation\Rules\AbstractRule`

Lorsqu'un règle respecte ces conditions, elle est automatiquement découverte et peut être utilisée sous forme de chaîne ou via la classe `Rule`:

```php
use BlitzPHP\Validation\Rule;
use BlitzPHP\Validation\Validator;
 
$validator = Validator::make($this->request->all(), [
    'name' => 'required|string|start_with_uppercase',
]);

// ou 

$validator = Validator::make($this->request->all(), [
    'name' => ['required', 'string', Rule::startWithUppercase()],
]);
```

> **Note**
> Le nom de la classe est transformé en <a href="https://en.wikipedia.org/wiki/Snake_case" target="_blank">snake case</a> pour que la règle soit utilisée sous forme de chaîne de caractère et en <a href="https://en.wikipedia.org/wiki/Camel_case" target="_blank">camel case</a> lorsque la règle doit être utilisée via la classe `Rule`.

<a name="definition-du-nom-de-la-regle"></a>
#### Définition du nom de la règle

Par défaut, BlitzPHP utilise le nom de la classe pour déterminer le nom de la règle. Si vous voulez donner un nom particulier à votre règle, vous devez ajouter la constante NAME à votre classe.

```php
<?php
 
namespace App\Rules;
 
use BlitzPHP\Validation\Rules\AbstractRule;
 
class StartWithUppercase extends AbstractRule
{
    protected const NAME = 'swu';
 
    // ...
}
```

Une fois ceci étant fait, vous pourriez utiliser votre nom personnalisé pour valider vos données :

```php
use BlitzPHP\Validation\Validator;
 
$validator = Validator::make($this->request->all(), [
    'name' => 'required|string|swu',
]);

// ou 

$validator = Validator::make($this->request->all(), [
    'name' => ['required', 'string', Rule::swu()],
]);
```

<a name="message-d-erreur"></a>
#### Message d'erreur

En utilisant la propriété `message` de votre classe de validation, vous pouvez définir le message d'erreur qui sera afficher en cas d’échec de validation.

```php
<?php
 
namespace App\Rules;
 
use BlitzPHP\Validation\Rules\AbstractRule;
 
class StartWithUppercase extends AbstractRule
{
    protected $message = "The :attribute must be started with uppercase";
 
    // ...
}
```

Au lieu de fournir un message d'erreur littéral à la propriété `$message`, vous pouvez également ignorer cette propriété et mettre les messages de validation directement dans les fichiers de traductions. La clé du tableau doit être le nom de la règle tel que souligné ci-dessus.

```php
<?php
 
/* app/Rules/en/Validation.php */
 
return [
    'swu' => 'The :attribute must be started with uppercase'
];
```

<a name="utilisation-d-une-closure"></a>
### Utilisation d'une closure

Si vous n'avez besoin de la fonctionnalité d'une règle personnalisée qu'une seule fois dans votre application, vous pouvez utiliser une closure au lieu d'un objet règle. Voir [la documentation sur la regle `callback`](#regle-callback) pour plus de détails.

<a name="regles-implicites"></a>
## Règles implicites

Une règle implicite est une règle qui, si elle est invalide, ignore les règles suivantes. Par exemple, si l'attribut n'a pas satisfait aux règles `required*`, les règles suivantes seront invalides. Pour éviter une validation et des messages d'erreur inutiles, nous faisons en sorte que les règles `required*` soient implicites.

Pour rendre votre règle personnalisée implicite, vous pouvez donner à la propriété `$implicit` la valeur `true`. Par exemple :

```php
<?php
use BlitzPHP\Validation\Rules\AbstractRule;

class YourCustomRule extends AbstractRule
{
    protected $implicit = true;
}
```

<a name="modifier-la-valeur"></a>
## Modifier la valeur

Dans certains cas, vous pouvez souhaiter que votre règle personnalisée soit en mesure de modifier la valeur de l'attribut comme la règle `default`. Lors des vérifications de la règle actuelle et de la règle suivante, la valeur modifiée sera utilisée.

Pour ce faire, vous devez implémenter `Rakit\Validation\Rules\Interfaces\ModifyValue` et créer la méthode `modifyValue($value)` sur votre classe de règle personnalisée.

Par exemple :

```php
<?php
use BlitzPHP\Validation\Rules\AbstractRule;
use Rakit\Validation\Rules\Interfaces\ModifyValue;

class YourCustomRule extends AbstractRule implements ModifyValue
{
    /**
     * @param mixed $value
     *
     * @return mixed
     */
    public function modifyValue($value)
    {
        // Faire quelques choses avec $value

        return $value;
    }
}
```

<a name="hook-before-validation"></a>
## Hook Before Validation

Il se peut que vous souhaitiez faire quelques préparatifs avant de lancer la validation. Par exemple, la règle `uploaded_file` résoudra la valeur de l'attribut provenant de la structure de tableau `$_FILES` (indésirable) en un tableau bien organisé.

Pour ce faire, vous devez implémenter `Rakit\Validation\Rules\Interfaces\BeforeValidate` et créer la méthode `beforeValidate()` sur votre classe de règle personnalisée.

Par exemple :

```php
<?php
use BlitzPHP\Validation\Rules\AbstractRule;
use Rakit\Validation\Rules\Interfaces\BeforeValidate;

class YourCustomRule extends AbstractRule implements BeforeValidate
{
    public function beforeValidate()
    {
        $attribute = $this->getAttribute();
        $validation = $this->validation;

        // Faire quelque chose avec $attribut et $validation
        // Par exemple, changer la valeur de l'attribut
        $validation->setValue($attribute->getKey(), "nouvelle valeur");
    }
}
```
