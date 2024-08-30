---
title: Vues natives BlitzPHP
---

<a name="introduction"></a>
## Introduction

De nos jours, de nombreux développeurs aiment les moteurs de template car ils offrent des fonctionnalités évoluées par rapport au PHP de base.

BlitzPHP prend en charge un système de mise en page simple, mais très flexible, qui facilite la création de vos vues. Le *moteur de template* natif de BlitzPHP vous offre des fonctionnalités poussées disponible dans les moteurs de template modernes. L'avantage est que vous n'aurez plus besoin d'apprendre une nouvelle syntaxe, vous continuerez à créer vos vues en utilisant du bon vieux PHP. Par ailleurs, votre application gagnera en performance car une étape de traduction du pseudo-code du template ne sera pas nécessaire.

> **Note**  
> Si malgré tout vous aimerez continuer à utiliser votre moteur de template favori, pas de soucis. BlitzPHP vous offre une flexibilité sans pareil. Consultez notre guide dédié pour savoir [comment intégrer votre moteur de template à BlitzPHP](/docs/{version}/vues#utilisation-d-un-moteur-de-template).

<a name="affichage-de-donnees"></a>
## Affichage de données

Vous pouvez afficher les données transmises à vos vues en utilisant les instructions PHP que vous connaissez. Par exemple, étant donné la route suivante :

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

Vous pouvez afficher le contenu de la variable `$name` comme suit :

```php
Hello, <?= $name ?>.
```

> **Attention**  
> Les instructions `echo` ne sont pas automatiquement échappées. Donc, pour empêcher les attaques XSS, vous devez toujours utiliser la fonction `h()` ou `esc()`.

<a name="directives"></a>
## Directives

Le moteur de rendu de BlitzPHP fournit des raccourcis pratiques qui offrent une façon très claire et concise de travailler avec les structures de contrôle PHP.

<a name="classes-et-styles-conditionnels"></a>
### Classes et styles conditionnels

La directive `class` compile conditionnellement une chaîne de classe CSS. La directive accepte un tableau de classes où la clé du tableau contient la ou les classes que vous souhaitez ajouter, tandis que la valeur est une expression booléenne. Si l’élément array a une clé numérique, il sera toujours inclus dans la liste des classes rendues :  

```php
<?php
    $isActive = false;
    $hasError = true;
?>
 
<span <?= $this->class([
    'p-4',
    'font-bold'     => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red'        => $hasError,
]) ?>></span>
```

Le code suivant générera le sortie suivante :

```html
<span class="p-4 text-gray-500 bg-red"></span>
```

De même, la directive `style` peut être utilisée pour ajouter conditionnellement des styles CSS en ligne à un élément HTML :

```php
<?php
    $isActive = true;
?>
 
<span <?= $this->style([
    'background-color: red',
    'font-weight: bold' => $isActive,
]) ?>></span>
```

Le code suivant générera le sortie suivante :

```html
<span style="background-color: red; font-weight: bold;"></span>
```

<a name="attributs-supplementaires"></a>
### Attributs supplémentaires

Pour plus de commodité, vous pouvez utiliser la directive `checked` pour indiquer facilement si une entrée de case à cocher HTML donnée est « cochée ». Cette directive sera répercutée si la condition fournie est évaluée à `true`.

```php
<input type="checkbox"
        name="active"
        value="active"
        <?= $this->checked(old('active', $user->active)) ?> />
```

De même, la directive `selected` peut être utilisée pour indiquer si une option de sélection donnée doit être « sélectionnée » :

```php
<select name="version">
    <?php foreach ($product->versions as $version): ?>
        <option value="<?= $version ?>" <?= $this->selected(old('version') == $version) ?>>
            <?= $version ?>
        </option>
    <?php endforeach; ?>
</select>
```

Par ailleurs, la directive `disabled` peut être utilisée pour indiquer si un élément donné doit être « désactivé » :

```php
<button type="submit" <?= $this->disabled($errors->isNotEmpty()) ?>>Submit</button>
```

Vous avez aussi la directive `readonly` qui peut être utilisée pour indiquer si un élément donné doit être « en lecture seule » :

```php
<input type="email"
        name="email"
        value="contact@blitz-php.com"
        <?= $this->readonly($user->isNotAdmin()) ?> />
```

Enfin, la directive `required` peut être utilisée pour indiquer si un élément donné doit être « requis » :

```php
<input type="text"
        name="title"
        value="title"
        <?= $this->required($user->isAdmin()) ?> />
```

<a name="inclusion-des-sous-vues"></a>
## Inclusion des sous vues

La méthode `include` vous permet d’inclure une vue à partir d’une autre. Toutes les variables disponibles pour la vue parent seront mises à la disposition de la vue incluse :

```php
<div>
    <?= $this->include('errors') ?>
 
    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Même si la vue incluse hérite de toutes les données disponibles dans la vue parente, vous pouvez également transmettre un tableau de données supplémentaires qui doivent être mises à la disposition de la vue incluse :

```php
<?= $this->include('view.name', ['status' => 'complete']) ?>
```

Si vous tentez d’accéder à une vue qui n’existe pas, BlitzPHP lancera une erreur. Si vous souhaitez inclure une vue qui peut être présente ou non, vous devez utiliser la méthode `includeIf`

```php
<?= $this->includeIf('view.name', ['status' => 'complete']) ?>
```

Si vous souhaitez afficher une vue si une expression booléenne donnée est évaluée à `true` ou `false`, vous pouvez utiliser les méthodes `includeWhen` et `includeUnless`

```php
<?= $this->includeWhen($boolean, 'view.name', ['status' => 'complete']) ?>
 
<?= $this->includeUnless($boolean, 'view.name', ['status' => 'complete']) ?>
```

Pour inclure la première vue qui existe à partir d’un tableau donné de vues, vous pouvez utiliser la méthode `includeFirst`:

```php
<?= $this->includeFirst(['custom.admin', 'admin'], ['status' => 'complete']) ?>
```

<a name="construction-des-mises-en-page-layout"></a>
## Construction des mises en page (layout)

Les layouts sont des vues comme les autres. La seule différence réside dans l'utilisation qui en est faite. Les layouts sont les seuls fichiers de vue qui utilisent la méthode `show()`. Cette méthode sert d'espace réservé pour le contenu.

Ex: `app/Views/layouts/default.php`
```php
<!doctype html>
<html>
<head>
    <title>Mon Layout</title>
</head>
<body>
    <?= $this->show('content') ?>
</body>
</html>
```
La méthode `show()` a deux arguments : `$sectionName` et `$preserve`. `$sectionName` est le nom de la section utilisé par toute vue enfant pour nommer la section de contenu. Si l'argument booléen `$preserve` vaut `true`, la méthode enregistre les données pour les appels suivants. Dans le cas contraire, la méthode nettoie les données après avoir affiché le contenu.

Ex: `app/Views/layouts/default.php`
```php
<!doctype html>
<html>
<head>
    <title><?= $this->show('page_title', true) ?></title>
</head>
<body>
    <h1><?= $this->show('page_title') ?></h1>
    <div><?= $this->show('content') ?></div>
</body>
</html>
```

> **Note**  
> Lorsque le nom de votre section est content, vous pouvez utiliser la méthode renderView(). Ainsi, `<?= $this->show('content') ?`> équivaut à `<?= $this->renderView() ?>`

<a name="utilisation-des-layouts-dans-les-vues"></a>
### Utilisation des layouts dans les vues

Lorsqu'une vue veut être insérée dans un layout, elle doit utiliser la méthode `extend()` en tête du fichier :

```php
<?= $this->extend('default') ?>
```

La méthode `extend()` prend le nom de tout fichier de vue que vous souhaitez utiliser. Comme il s'agit de vues standard, elles seront localisées comme une vue. Par défaut, la vue sera recherchée dans le répertoire `app/Views/layouts` de l'application, mais elle pourra également être recherchée dans d'autres namespace définis par PSR-4. Vous pouvez inclure un namespace pour localiser la vue dans le répertoire `Views` d'un namespace particulier :

```php
<?= $this->extend('Blog\Views\default') ?>
```

Tout le contenu d'une vue qui étend un layout doit être inclus dans les appels de méthode `start($name)` et `stop()`. Tout contenu situé entre ces appels sera inséré dans le layout à chaque fois que l'appel `show($name)` correspondant au nom de la section existera.

```php
<?= $this->extend('default') ?>

<?= $this->start('content') ?>
    <h1>Hello World!</h1>
<?= $this->stop() ?>
```

La méthode `stop()` n'a pas besoin du nom de la section. Elle sait automatiquement laquelle fermer.

Les sections peuvent contenir des sections imbriquées :

```php
<?= $this->extend('default') ?>

<?= $this->start('content') ?>
    <h1>Hello World!</h1>
    <?= $this->start('javascript') ?>
       let a = 'a';
    <?= $this->stop() ?>
<?= $this->stop() ?>
```

<a name="rendu-de-vue"></a>
### Rendu de vue

Le rendu de la vue et de son layout se fait exactement comme n'importe quelle autre vue serait affichée dans un contrôleur :

```php
<?php

namespace App\Controllers;

class MyController extends AppController
{
    public function index()
    {
        return view('some_view');
    }
}
```

Il rend la vue `app/Views/some_view.php` et s'il étend `default`, le layout `app/Views/layouts/default.php` est également utilisé automatiquement. Le moteur de rendu est suffisamment intelligent pour détecter si la vue doit être rendue seule ou si elle a besoin d'une présentation.

<a name="inclure-des-vues-partielles"></a>
### Inclure des vues partielles

Les vues partielles sont des fichiers de vues qui n'étendent aucun layout. Elles contiennent généralement du contenu qui peut être réutilisé d'une vue à l'autre. Lorsque vous utilisez des layouts de vues, vous pouvez utiliser `$this->include()` pour inclure les fichiers partiels de vues.

```php
<?= $this->extend('default') ?>

<?= $this->start('content') ?>
    <h1>Hello World!</h1>

    <?= $this->include('sidebar') ?>
<?= $this->stop() ?>
```

La méthode `include()` a plusieurs variantes. Referez-vous à [la section ci-dessus](#inclusion-des-sous-vues) pour en savoir plus.

> **Note**  
> Les vues partielles sont généralement rangées dans le dossier `app/Views/partials`. Toute fois vous pouvez stocker vos parties de vue ailleurs. Ainsi, en faisant `$this->include('sidebar')`, BlitzPHP cherchera tour à tour le fichier `sidebar.php` dans le dossier où se trouve la vue qui essai d'inclure la vue partielle (`{current_dir}`), le sous-dossier partials du dossier courrant `app/Views/{current_dir}/partials`, le dossier `app/Views/partials`, et enfin dans le dossier `app/Views`. Confère <a href="https://github.com/blitz-php/framework/blob/main/src/View/Adapters/NativeAdapter.php#L710-L728" target="_blank">https://github.com/blitz-php/framework/blob/main/src/View/Adapters/NativeAdapter.php#L710-L728</a>


<a name="assets"></a>
### Assets

Vous pouvez utiliser les méthodes `addCss()`, `addJs()`, `addLibCss()` et `addLibJs()` pour ajouter du css ou js à votre layout à partir d'une vue enfant.

En effet, il peut arriver que dans une page spécifique, vous ayez besoin des assets particulier qui ne sont pas chargés par le layout principal. Les méthodes `addCss()` et autres sont justement fait pour ça. 

Par exemple une page de liste d'élève pourrait avoir besoin de DataTable pour afficher les élèves dans un tableau tandis que la page d'ajout d'un élève aura plutôt besoin de Select2 pour avoir une jolie liste déroulante.

De prime à bord, on pourrait s'imaginer charger les 2 bibliothèques directement dans le layout  comme ceci:

```php
<!-- app/Views/layouts/default.php -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?= $this->show('title', true) ?></title>
    <link rel="stylesheet" href="<?= lib_css_url('bootstrap/bootstrap') ?>">
    <link rel="stylesheet" href="<?= lib_css_url('datatable/datatable') ?>">
    <link rel="stylesheet" href="<?= lib_css_url('select2/select2') ?>">
    <link rel="stylesheet" href="<?= css_url('main') ?>">
</head>
<body>
    <h1><?= $this->show('title') ?></h1>

    <div><?= $this->show('content') ?></div>

    <script src="<?= lib_js_url('jquery/jquery') ?>"></script>
    <script src="<?= lib_js_url('bootstrap/bootstrap') ?>"></script>
    <script src="<?= lib_js_url('datatable/datatable') ?>"></script>
    <script src="<?= lib_js_url('select2/select2') ?>"></script>
    <script src="<?= js_url('main') ?>"></script>
    <script src="<?= js_url('init-datatable') ?>"></script>
    <script src="<?= js_url('init-select2') ?>"></script>
</body>
</html>
```

Cette façon de faire a deux principaux problèmes:

1. Les fichiers inutiles seront chargés d'une vue à l'autre. En effet, la liste des élèves n'a pas besoin de `Select2` et la création des élèves n'a pas besoin de `Datatable` mais dans les deux cas, les fichiers JS et CSS de ces bibliothèques seront chargés, ce qui peut augmenter le temps d'affichage de la page par le navigateur.

2. Par ailleurs, on note la présence de 2 fichiers (`init-datatable` et `init-select2.js`). En supposant que ces fichiers ont respectivement les codes `$('.data-table').Datatable();` et `$('.select').Select2();`, la page de la liste des élèves aura une erreur du fait de l'absence d'un élément ayant la classe `select`. De même, la page d'ajout d'élève aura une erreur du fait de l'absence d'un élément ayant la classe `data-table`. 

La manière la plus conviviale de gérer cette situation, c'est d'utiliser les méthodes de piles offert par les vues natives BlitzPHP. Ainsi, vous auriez:


```php
<!-- app/Views/layouts/default.php -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?= $this->show('title', true) ?></title>
    <link rel="stylesheet" href="<?= lib_css_url('bootstrap/bootstrap') ?>">
    <link rel="stylesheet" href="<?= css_url('main') ?>">
    
    <!-- Ajout dynamique des css -->
    <?= $this->stylesBundle() ?>
</head>
<body>
    <h1><?= $this->show('title') ?></h1>

    <div><?= $this->show('content') ?></div>

    <script src="<?= lib_js_url('jquery/jquery') ?>"></script>
    <script src="<?= lib_js_url('bootstrap/bootstrap') ?>"></script>
    <script src="<?= js_url('main') ?>"></script>
    
    <!-- Ajout dynamique des js -->
    <?= $this->scriptsBundle() ?>
</body>
</html>
```

```php
<!-- app/Views/students/create.php -->

<?php $this->section('title', 'Ajouter un élève') ?>

<?php $this->addLibCss('select2/select2')
            ->addLibJs('select2/select2')
            ->addJs('init-select2') ?>
            
<?php $this->start('content') ?>

<form method="post">
    ...
</form>

<?php $this->stop();
```

```php
<!-- app/Views/students/list.php -->

<?php $this->section('title', 'Liste des élèves') ?>

<?php $this->addLibCss('datatable/datatable')
            ->addLibJs('datatable/datatable')
            ->addJs('init-datatable') ?>
            
<?php $this->start('content') ?>

<table class="table data-table">
    ...
</table>

<?php $this->stop(); ?>
```

Ainsi, BlitzPHP chargera les fichiers d'assets uniquement lorsque cela sera nécessaire. 

> **Note**  
> Les méthodes `stylesBundle()` et `scriptsBundle()` utilisent en interne les fonction `lib_css_url()`, `css_url()`, `lib_js_url()` et `js_url()` pour générer les liens des fichiers assets. Vous pouvez vous référer à la page des [helpers BlitzPHP](/docs/{version}/helpers-blitzphp#assets) pour en savoir plus sur leurs fonctionnements.

<a name="formulaires"></a>
## Formulaires

<a name="champ-csrf"></a>
### Champ CSRF

Chaque fois que vous définissez un formulaire HTML dans votre application, vous devez inclure un champ de jeton CSRF caché dans le formulaire afin que [le middleware de protection CSRF](/docs/{version}/csrf) puisse valider la requête. Vous pouvez utiliser la méthode `csrf()` pour générer le champ de jeton :

```php
<form method="POST" action="/profile">
    <?= $this->csrf() ?>
 
    ...
</form>
```

<a name="champ-de-methode"></a>
### Champ de méthode

Comme les formulaires HTML ne peuvent pas effectuer de requêtes `PUT`, `PATCH` ou `DELETE`, vous devrez ajouter un champ caché `_method` pour usurper ces verbes HTTP. La méthode `method()` peut créer ce champ pour vous :

```php
<form action="/foo/bar" method="POST">
    <?= $this->method('PUT') ?>
 
    ...
</form>
```

<a name="erreurs-de-validation"></a>
### Erreurs de validation

BlitzPHP injecte automatiquement la variable `$errors` à l'intérieur de toutes vos vues pour vérifier rapidement s'il existe des [messages d'erreur de validation](/docs/{version}/validation) pour un attribut donné.  :

```php
<!-- /app/Views/post/create.php -->
 
<label for="title">Titre du Post</label>
 
<input name="title" type="text"
    <?= $this->class(['is-invalid' => $errors->has('title')])?>>
 
 <?php if ($errors->has('title')): ?>
    <div class="alert alert-danger"><?= $errors->line('title') ?></div>
<?php endif; ?>
```

<a name="composants"></a>
## Composants

De nombreuses applications comportent de petits fragments de vue qui peuvent être répétés d'une page à l'autre ou à différents endroits sur les pages. Il s'agit souvent de boîtes d'aide, de contrôles de navigation, de publicités, de formulaires de connexion, etc. BlitzPHP vous permet d'encapsuler la logique de ces blocs de présentation dans des **composants de vue**. Ce sont en fait des mini-vues qui peuvent être incluses dans d'autres vues. Elles peuvent avoir une logique intégrée pour gérer toute logique d'affichage spécifique au composant. Elles peuvent être utilisées pour rendre vos vues plus lisibles et plus faciles à maintenir en séparant la logique de chaque composant dans sa propre classe.
Il existe deux approches pour écrire des composants : les composants anonymes et [les composants contrôlés](#composants-controles).

Les composants anonymes sont des classes qui renvoient une chaîne de caractères à partir de la méthode choisie. Un exemple de composant de message d'alerte simple pourrait ressembler à ceci :

```php
<?php

namespace App\Components;

class AlertMessage
{
    public function show(array $params): string
    {
        return "<div class=\"alert alert-{$params['type']}\">{$params['message']}</div>";
    }
}
```

Une fois le composant défini, Vous l'appelleriez à partir d'une vue à travers l'instruction :

```php
// Dans une vue.

<?= component('App\Components\AlertMessage::show', ['type' => 'success', 'message' => 'Connexion réussie.']) ?>
```

Vous pouvez ne pas indiquer le namespace complet de la classe, BlitzPHP supposera qu'elle se trouve dans le namespace `App\Components`. Ainsi, l'exemple ci-dessus est équivalent à:

```php
<?= component('AlertMessage::show', ['type' => 'success', 'message' => 'Connexion réussie.']) ?>
```

Si elle ne s'y trouve pas, tous les namespaces seront parcourus jusqu'à ce qu'elle soit trouvée, en cherchant dans un sous-répertoire **Components** de chaque [namespace reconnu](/docs/{version}/autoloader#namespaces).

Vous pouvez également transmettre les paramètres sous la forme d'une chaîne clé/valeur :

```php
<?= component('AlertMessage::show', 'type=success, message=Connexion réussie.') ?>
```

En outre, vous pouvez utiliser des noms de paramètres qui correspondent aux variables de la méthode pour une meilleure lisibilité. Lorsque vous l'utilisez de cette manière, tous les paramètres doivent toujours être spécifiés dans l'appel du composant :

```php
<?php
namespace App\Components;

use App\Entities\Blog;

class Blog
{
    // ...

    public function recentPosts(string $category, int $limit): string
    {
        $posts = Blog::where('category', $category)
            ->latest('published_on')
            ->limit($limit)
            ->get();

        return view('components/recentPosts', ['posts' => $posts]);
    }
}
```

```php
// Dans une vue.

<?= view_cell('Blog::recentPosts', 'category=sport, limit=5') ?>
```

<a name="composants-controles"></a>
## Composants contrôlés

Les composants contrôlés ont deux objectifs principaux :
* Rendre la construction du composants aussi rapide que possible.
* Fournir une logique et une flexibilité supplémentaires à vos vues, si elles en ont besoin.

La classe du composant doit étendre `BlitzPHP\View\Components\Component`. Elles doivent avoir un fichier de vue dans le même dossier. Par convention, le nom de la classe doit être en PascalCase avec le suffixe `Component` et la vue doit être la version en `kebab_cased` du nom de la classe, sans le suffixe. Par exemple, si vous avez une classe `AlertComponent`, le fichier de vue doit être `alert.php`.

<a name="creation-d-un-composant-controle"></a>
### Création d'un composant contrôlé

Les composants contrôlés doivent être générés à partir d'une classe qui étend la classe `BlitzPHP\View\Components\Component` qui offre des possibilités supplémentaires rendant vos composants de plus flexibles et plus rapides à utiliser.

Au niveau le plus élémentaire, tout ce que vous devez mettre en œuvre dans la classe, ce sont des propriétés publiques. Ces propriétés seront automatiquement mises à la disposition du fichier de vue.

La mise en œuvre de `AlertMessage` ci-dessus en tant que composant contrôlé ressemblerait à ce qui suit :

```php
// Classe de contrôle: app/Components/AlertMessageComponent.php

namespace App\Components;

use BlitzPHP\View\Components\Component;

class AlertMessageComponent extends Component
{
    public $type;
    public $message;
}
```

```php
// Fichier de vue: app/Components/alert-message.php

<div class="alert alert-<?= esc($type, 'attr') ?>">
    <?= esc($message) ?>
</div>
```

```php
// Utilisation dans une vue classique:

<?= component('AlertMessageComponent', 'type=warning, message=Failed.') ?>
```

<a name="generation-d-un-composant-par-commande"></a>
### Génération d'un composant par commande

Vous pouvez également créer un composant contrôlée à l'aide d'une commande intégrée au [CLI de BlitzPHP](/docs/{version}/klinge). La commande est `php klinge make:component`. Elle prend un argument, le nom du composant à créer. Le nom doit être en **PascalCase**, et la classe sera créée dans le répertoire `app/Components`. Le fichier de vue sera également créé dans le répertoire `app/Components`.

```bash
php klinge make:component AlertMessage
```

<a name="utiliser-une-vue-differente"></a>
### Utiliser une vue différente

Vous pouvez spécifier un nom de vue personnalisé en définissant la propriété `view` dans la classe. La vue sera localisée comme n'importe quelle vue :

```php
namespace App\Components;

use BlitzPHP\View\Components\Component;

class AlertMessageComponent extends Component
{
    public $type;
    public $message;
    
    protected string $view = 'my/custom/view';
}
```

<a name="personnaliser-le-rendu"></a>
### Personnaliser le rendu

Si vous avez besoin de plus de contrôle sur le rendu du code HTML, vous pouvez implémenter une méthode `render()`. Cette méthode vous permet d'exécuter une logique supplémentaire et de transmettre des données supplémentaires à la vue, si nécessaire. La méthode `render()` doit renvoyer une chaîne de caractères.

Pour bénéficier de toutes les fonctionnalités des composants contrôlés, vous devez utiliser `$this->view()` au lieu du helper `view()` normal :

```php
namespace App\Components;

use BlitzPHP\View\Components\Component;

class AlertMessageComponent extends Component
{
    public $type;
    public $message;
    
    public function render(): string
    {
        return $this->view('my/custom/view', ['extra' => 'data']);
    }
}
```

<a name="proprietes-calculees"></a>
### Propriétés calculées

Si vous devez exécuter une logique supplémentaire pour une ou plusieurs propriétés, vous pouvez utiliser des propriétés calculées. Pour ce faire, vous devez définir la propriété comme `protégée` ou `privée` et implémenter une méthode publique dont le nom se compose du nom de la propriété entouré de `get` et de `Property` :

```php
// Dans une vue. Initialiser les propriétés protégées.
<?= component('AlertMessageComponent', ['type' => 'note', 'message' => 'test']) ?>
```

```php
namespace App\Components;

use BlitzPHP\View\Components\Component;

class AlertMessageComponent extends Component
{
    protected $type;
    protected $message;
    private $computed;

    public function mount(): void
    {
        $this->computed = sprintf('%s - %s', $this->type, $this->message);
    }

    public function getComputedProperty(): string
    {
        return $this->computed;
    }

    public function getTypeProperty(): string
    {
        return $this->type;
    }

    public function getMessageProperty(): string
    {
        return $this->message;
    }
}
```

```php
// app/Components/alert-message.php
<div>
    <p>type - <?= esc($type) ?></p>
    <p>message - <?= esc($message) ?></p>
    <p>computed: <?= esc($computed) ?></p>
</div>
```

> **Attention**  
> Vous ne pouvez pas définir des propriétés déclarées comme privées lors de l'initialisation de la cellule.

<a name="methodes-de-presentation"></a>
### Méthodes de présentation

Parfois, vous devez exécuter une logique supplémentaire pour la vue, mais vous ne voulez pas la passer en paramètre. Vous pouvez implémenter une méthode qui sera appelée depuis la vue du composant elle-même. Cela peut améliorer la lisibilité de vos vues :

```php
namespace App\Components;

use BlitzPHP\View\Components\Component;

class RecentPostsComponent extends Component
{
    protected $posts;

    public function linkPost($post): string
    {
        return anchor('posts/' . $post->id, $post->title);
    }
}
```

```php
// app/Components/recent-posts.php
<ul>
    <?php foreach ($posts as $post): ?>
        <li><?= $this->linkPost($post) ?></li>
    <?php endforeach ?>
</ul>
```

<a name="execution-de-la-logique-d-installation"></a>
### Exécution de la logique d'installation

Si vous avez besoin d'exécuter une logique supplémentaire avant que la vue ne soit rendue, vous pouvez implémenter une méthode `mount()`. Cette méthode sera appelée juste après l'instanciation de la classe et pourra être utilisée pour définir des propriétés supplémentaires ou effectuer d'autres opérations logiques :

```php
namespace App\Components;

use App\Entities\Post;
use BlitzPHP\View\Components\Component;

class RecentPostsComponent extends Component
{
    protected $posts;

    public function mount(): void
    {
        $this->posts = Post::latest()->take(10)->get();
    }
}
```

Vous pouvez passer des paramètres supplémentaires à la méthode `mount()` en les transmettant sous forme de tableau à la fonction `component()`. Tous les paramètres envoyés qui correspondent à un nom de paramètre de la méthode `mount()` seront transmis :


```php
namespace App\Components;

use App\Entities\Post;
use BlitzPHP\View\Components\Component;

class RecentPostsComponent extends Component
{
    protected $posts;

    public function mount(?int $categorieId): void
    {
        $this->posts = Post::latest()
            ->when(
                $categorieId, 
                static fn ($query, $categoryId) => $query->where('category_id', $categoryId)
            )
            ->take(10)
            ->get();
    }
}
```

```php
// Appelé dans la vue principale : 
<?= component('RecentPostsComponent', ['categoryId' => 5]) ?>
```

<a name="mise-en-cache-des-composants"></a>
### Mise en cache des composants

Vous pouvez mettre en cache les résultats de l'appel du composant en indiquant le nombre de secondes pour lesquelles les données doivent être mises en cache en tant que troisième paramètre. [Le moteur de cache](/docs/{version}/cache) actuellement configuré sera utilisé :

```php
// Mettre le composant en cache pendant 5 minutes
<?= component('App\Components\Blog::recentPosts', 'limit=5', 5 * MINUTE) ?>
```

Vous pouvez fournir un nom personnalisé à utiliser à la place de celui généré automatiquement si vous le souhaitez, en passant le nouveau nom comme quatrième paramètre :

```php
// Mettre le composant en cache pendant 5 minutes
<?= component('App\Components\Blog::recentPosts', 'limit=5', 5 * MINUTE, 'nom_du_cache') ?>
```