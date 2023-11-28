---
title: Contrôleurs
---

<a name="introduction"></a>
## Introduction

Au lieu de définir toute la logique de gestion de vos requêtes sous forme de closure dans vos fichiers de route, vous souhaiterez peut-être organiser ce comportement à l'aide de classes « contrôleur ». Les contrôleurs peuvent regrouper la logique de traitement des requêtes associée dans une seule classe. Par exemple, une classe `UserController` peut gérer toutes les requêtes entrantes liées aux utilisateurs, y compris l'affichage, la création, la mise à jour et la suppression d'utilisateurs. Par défaut, les contrôleurs sont stockés dans le répertoire `app/Controllers`.

<a name="qu-est-ce-qu-un-controleur"></a>
## Qu'est-ce qu'un contrôleur ?

Un contrôleur est simplement un fichier de classe qui gère une requête HTTP. [Le routage URI](/docs/{version}/routage) associe un URI à un contrôleur.  

Chaque contrôleur que vous créez doit étendre la classe `AppController`. Cette classe fournit plusieurs fonctionnalités disponibles pour tous vos contrôleurs.

<a name="constructeur"></a>
### Constructeur

Le contrôleur de BlitzPHP a un constructeur spécial `initialize()`. Il sera appelé par le framework après l’exécution du constructeur PHP `__construct()`.

Si vous souhaitez remplacer `initialize()`, n'oubliez pas d'ajouter `parent::initialize($request, $response, $logger);` dans la méthode :

```php
<?php

namespace App\Controllers;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Log\LoggerInterface;

class ProductController extends AppController
{
    public function initialize(ServerRequestInterface $request, ResponseInterface $response, LoggerInterface $logger)
    {
        parent::initialize($request, $response, $logger);

        // Ajouter votre code ici
    }

    // ...
}
```

> **Attention**  
> Vous ne pouvez pas utiliser `return` dans le constructeur. Alors retournez `redirect()->to('route');` ne marche pas.

<a name="propriétes-incluses"></a>
### Propriétés incluses

Le contrôleur de BlitzPHP fournit ces propriétés.

* L'objet `Request`: L'instance de la [requête](/docs/{version}/requetes) principale de l'application est toujours disponible en tant que propriété de classe, `$this->request`.
* L'objet `Response`: L'instance de la [reponse](/docs/{version}/reponses) principale de l'application est toujours disponible en tant que propriété de classe, `$this->response`.
* Une instance de la classe [Logger](/docs/{version}/journalisation) est disponible en tant que propriété de classe, `$this->logger`.

En plus de ces propriétés, vous pouvez définir un tableau d'[helpers](/docs/{version}/helpers) en tant que propriété de classe. Chaque fois que le contrôleur est chargé, ces helpers seront automatiquement chargés en mémoire afin que vous puissiez utiliser leurs méthodes n'importe où dans le contrôleur :

```php
<?php

namespace App\Controllers;

class MyController extends AppController
{
    protected array $helpers = ['url', 'scl'];
}
```

<a name="ecriture-de-controleurs"></a>
## Écriture de contrôleurs

<a name="controleurs-de-base"></a>
### Contrôleurs de base

Pour générer rapidement un nouveau contrôleur, vous pouvez exécuter la commande [Klinge](/docs/{version}/klinge) `make:controller`. Par défaut, tous les contrôleurs de votre application sont stockés dans le répertoire `app/Controllers` :

```shell
php klinge make:controller UserController
```

Jetons un coup d'œil à un exemple de contrôleur de base. Un contrôleur peut disposer d'un certain nombre de méthodes publiques qui répondront aux requêtes HTTP entrantes :

```php
<?php
 
namespace App\Controllers;
 
use App\Models\User;
 
class UserController extends AppController
{
    /**
     * Afficher le profil d'un utilisateur donné.
     */
    public function show(string $id)
    {
        return $this->view('profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Une fois que vous avez écrit une classe et une méthode de contrôleur, vous pouvez définir une route vers la méthode de contrôleur comme ceci :

```php
use App\Controllers\UserController;
 
Route::get('/user/(:num)', [UserController::class, 'show']);
```

Lorsqu'une requête entrante correspond à l'URI de route spécifié, la méthode `show` sur la classe `App\Controllers\UserController` sera invoquée et les paramètres de route seront transmis à la méthode.

<a name="controleurs-a-action-unique"></a>
### Contrôleurs à action unique

Si une action de contrôleur est particulièrement complexe, vous trouverez peut-être pratique de consacrer une classe de contrôleur entière à cette action unique. Pour ce faire, vous pouvez définir une seule méthode `__invoke` au sein du contrôleur:

```php
<?php

namespace App\Controllers;

class ProvisionController extends AppController
{
    public function __invoke()
    {
        // ...
    }
}
```

Lors de l'enregistrement des routes pour des contrôleurs à action unique, vous n'avez pas besoin de spécifier une méthode de contrôleur. Au lieu de cela, vous pouvez simplement transmettre le nom du contrôleur au routeur :

```php
<?php
use App\Controllers\ProvisionController;

Route::post('/server', ProvisionController::class);
```

Vous pouvez générer un contrôleur invocable en utilisant l'option `--invokable` de la commande Klinge `make:controller`:

```shell
php klinge make:controller ProvisionController --invokable
```

<a name="methodes-protegees"></a>
## Méthodes des protégées

Dans certains cas, vous souhaiterez peut-être que certaines méthodes soient masquées de l’accès public. Pour y parvenir, déclarez simplement la méthode comme `privée` ou `protégée`. Cela l'empêchera d'être servi par une requête URL.

Par exemple, si vous deviez définir une méthode comme celle-ci pour le contrôleur `Helloworld` :

```php
<?php

namespace App\Controllers;

class HelloworldController extends AppController
{
    protected function utility()
    {
        // un code
    }
}
```

et de définir une route (`helloworld/utility`) pour la méthode. Ensuite, essayer d’y accéder en utilisant l’URL suivante ne fonctionnera pas :

```
example.com/helloworld/utility
```

[Le routage automatique](/docs/{version}/routage#routage-automatique) ne fonctionnera pas non plus.

<a name="validation-des-donnees"></a>
## Validation des données

Pour simplifier la vérification des données, le contrôleur fournit également la méthode pratique `validateData()`.  

La méthode accepte (1) un tableau de données à valider, (2) un tableau de règles, (3) un tableau facultatif de messages d'erreur personnalisés à afficher si les éléments ne sont pas valides.
  
La [section consacrée à la validation](/docs/{version}/validation) contient des détails sur les formats de règles et de tableaux de messages, ainsi que sur les règles disponibles :

```php
<?php

namespace App\Controllers;

use BlitzPHP\Exceptions\ValidationException;

class StoreController extends AppController
{
    public function product(int $id)
    {
        $data = [
            'id'   => $id,
            'name' => $this->request->post('name'),
        ];

        $rule = [
            'id'   => 'integer',
            'name' => 'required|max:255',
        ];

        try {
            $validated = $this->validateData($data, $rule);
        } catch(ValidationException $e) {
            return view('store/product')->withErrors($e->getErrors());
        }

        // ...
    }
}
```

En plus de la méthode `validateData()`, le contrôleur vous fourni également la méthode `validate()` qui a un fonctionnement similaire.

> **Attention**  
> Au lieu de `validate()`, utilisez `validateData()` pour valider uniquement les données `POST`. validate() utilise `$request->all()` qui renvoie les données `$_GET`, `$_POST` et `$_COOKIE` dans cet ordre (selon [l'ordre de la requête](https://www.php.net/manual/en/ini.core.php#ini.request-order) php.ini). Les valeurs les plus récentes remplacent les valeurs plus anciennes. Les valeurs POST peuvent donc être remplacées par les cookies s'ils portent le même nom.

La méthode accepte un tableau de règles dans le premier paramètre et dans le deuxième paramètre facultatif, un tableau de messages d'erreur personnalisés à afficher si les éléments ne sont pas valides.

En interne, cela utilise l'instance `$this->request` du contrôleur pour valider les données.

```php
<?php

namespace App\Controllers;

use BlitzPHP\Exceptions\ValidationException;

class AuthController extends AppController
{
    public function register()
    {
        try {
            $validated = $this->validate([
                'email' => 'required|email|unique:users',
                'name'  => 'required|alpha',
                'pass'  => 'required|password',
            ]);
        } catch(ValidationException $e) {
            return view('auth/register')->withErrors($e->getErrors());
        }

        // ...
    }
}
```

Pour plus de simplicité, vous pouvez placer vos règles de validation dans une classe spécifique et remplacer le tableau `$rules` par le nom complet (FQCN) :

```php
<?php

namespace App\Controllers;

use App\Validations\RegisterValidation;
use BlitzPHP\Exceptions\ValidationException;

class AuthController extends AppController
{
    public function register()
    {
        try {
            $validated = $this->validate(RegisterValidation::class);
        } catch(ValidationException $e) {
            return view('auth/register')->withErrors($e->getErrors());
        }

        // ...
    }
}
```

Nous vous recommandons de NE PAS gérer vous-même l'exception et de laisser BlitzPHP <a href="https://github.com/blitz-php/framework/blob/main/src/Router/Dispatcher.php#L809-L835" target="_blank">convertir l'exception en réponse</a> à l'aide de la négociation de contenu.

Voici une explication du fonctionnement de la [négociation de contenu](/docs/{version}/negociation).

* **Application rendue par le serveur**  
Si vous créez une application Web standard avec des templates côté serveur, nous redirigerons le client vers le formulaire et transmettrons les erreurs sous forme de messages flash de session. Dans votre vue, vous auriez donc accès à l'objet ErrorBag via la variable `$errors`

```php
<?php if($errors->has('username')): ?>
  <p> <?= $errors->line('username') ?></p>
<?php endif; ?>
```

* **Requêtes avec en-tête `Accept=application/json`**  
Les requêtes négociant pour le type de données JSON reçoivent les messages d'erreur sous forme de tableau d'objets. Chaque message d'erreur contient le nom du champ, la règle de validation ayant échoué et le message d'erreur.

```json
{
  errors: [
    {
      field: 'title',
      rule: 'required',
      message: 'required validation failed',
    },
  ]
}
```

<a name="interactions-avec-les-vues"></a>
## Interactions avec les vues

Les vue sont les éléments graphiques qui s'affichent sur l'écran de vos utilisateurs. Les méthodes du contrôleur sont responsables de la conversion des paramètres de la requête dans une réponse pour le navigateur/utilisateur faisant la requête. Les contrôleurs interagissent avec les vues de plusieurs façons.

<a name="rendre-les-vues"></a>
### Rendre les vues

BlitzPHP vous offre la fonction globale `view()` pour rendre une vue. passez simplement le chemin relatif du fichier à charger au premier argument de cette fonction.

```php
class UsersController extends AppController
{
    public function create()
    {
        echo view('users/create');
        // ou
        // return view('users/create');
    }
}
```

Cela rendrait le fichier `/app/Views/users/create.php`. Si ce fichier n'existe pas, une exception sera levée. 

En plus de la fonction globale `view()`, les contrôleurs héritant de la classe `BlitzPHP\Controllers\ApplicationController` possèdent la méthode `$this->view()` qui effectue certains traitements avant de rendre la vue. Le code ci-dessous donnera exactement le même résultat que le précédent.

```php
class UsersController extends AppController
{
    public function create()
    {
        echo $this->view('create');
        // ou
        // return $this->view('create');
    }
}
```

Vous remarquez qu'on ne renseigne plus `users`. BlitzPHP utilisera le nom du contrôleur comme sous dossier dans ce cas.

<a name="definir-les-donnees-de-vue"></a>
### Définir les données de vue

Vos vues afficheront probablement des données dynamique provenant de la base de données ou générées sous certaines conditions. Ce sont vos contrôleurs qui sont chargés d'envoyer des données aux vues et pour ce faire, vous devez transmettre un tableau clé/valeur au **deuxième paramètre** de la méthode `view()`. La clé étant le nom de la variable et la valeur, la valeur associée.
Rendez-vous dans [la section consacrée aux vues](/docs/{version}/vues) pour en savoir plus.  

Si vous avez des données qui serons utilisées par toutes les vues de votre contrôleur, vous pouvez utiliser la propriété `$this->viewDatas`, idéalement dans le constructeur ou la méthode [initialize()](#constructeur) 


```php
class ProductsController extends AppController
{
    public function __construct()
    {
        $this->viewDatas['categories'] = CategoryEntity::all();    
    }
    
    public function create()
    {
        echo $this->view('create');
    }
    
    public function update($id)
    {
        echo $this->view('update');
    }
}
```

La variable `$categories` sera disponible dans les vues `products/create` et `products/update`.

<a name="definir-les-options-de-vue"></a>
### Définir les options de vue

Vous pouvez définir certaines options lors du rendu de vos vues. Ceci se fait en passant un tableau associatif au **troisième paramètre** de la méthode `view()`. Vous pouvez par exemple définir le layout à utiliser, mettre la vue en cache, activé ou non la compression du rendu, etc... Les clés suivante sont utilisées pour définir des options à vos vues:
* `layout`: Chaîne de caractère définissant la mise en page (layout) à utiliser par la vue.
* `compress_ouput`: Booléen spécifiant si on doit compresser le code du rendu final ou pas. Pas défaut elle a la valeur auto, ce qui permet de compresser la sortie uniquement lorsqu'on passe en production.
* `cache_name`: Chaîne de caractère définissant le nom à utiliser pour mettre la vue en cache et pour la récupérer lors des prochaines requêtes.
* `cache_time`: Entier définissant le nombre de minutes qu'une vue doit être conservée en cache.

Un exemple d'utilisation pourrait être:

```php
return $this->view('hello', null, [
    'layout' => 'admin',
    'cache_name' => 'hello_admin',
    'cache_time' => 2
]);
```

<a name="la-methode-render"></a>
### La méthode `render()`

La méthode `render()` utilise le nom de l'action dans laquelle elle est invoquée pour déterminer le fichier de vue à afficher. 

```php
class UsersController extends AppController
{
    public function create()
    {
        return $this->render();
    }
}
```

Le code ci-dessus rendrait le fichier `/app/Views/users/create.php`.   
  
Vous pouvez transmettre les données à la vue ou définir les options en utilisant respectivement les deux premiers paramètres de cette méthode.

```php
class UsersController extends AppController
{
    public function create()
    {
        $data    = [];
        $options = [];
        
        return $this->render($data, $options);
    }
}
```

> **Attention**  
> La méthode `render()` retourne une [réponse HTTP](/docs/{version}/reponses), donc, essayer d'exécuter les méthodes de vues (`with()`, `withErrors()`, `add()`, etc.) ne fonctionnera pas. Par ailleurs, votre contrôleur doit retourner à son tour la réponse générée.

<a name="injection-de-dependance-et-controleurs"></a>
## Injection de dépendance & contrôleurs

<a name="injection-du-constructeur"></a>
### Injection du constructeur

Le [conteneur](/docs/{version}/conteneur) de BlitzPHP est utilisé pour résoudre tous les contrôleurs. En conséquence, vous pouvez indiquer toutes les dépendances dont votre contrôleur pourrait avoir besoin dans son constructeur. Les dépendances déclarées seront automatiquement résolues et injectées dans l'instance du contrôleur :

```php
<?php

namespace App\Controllers;

use App\Repositories\UserRepository;

class UserController extends AppController
{
    public function __construct(protected UserRepository $users) 
    {
        //
    }
}
```

<a name="injection-de-methode"></a>
### Injection de méthode

En plus de l'injection de constructeur, vous pouvez également typer des dépendances sur les méthodes de votre contrôleur. 

```php
<?php

namespace App\Controllers;

use BlitzPHP\Mail\Mail;

class UserController extends AppController
{
    public function store(Mail $mail)
    {
        //...
        
        $mail->to($this->request->email)->send()
        
        //...

        return redirect('/users');
    }
}
```

Si votre méthode de contrôleur attend également une entrée d'un paramètre de route, mettez vos arguments de route après vos autres dépendances. Par exemple, si votre route est définie ainsi :

```php
use App\Controllers\UserController;

Route::put('/user/(:num)', [UserController::class, 'update']);
```

Vous pouvez toujours typer un paramètre `BlitzPHP\Mail\Mail` et accéder à votre paramètre `id` en définissant votre méthode de contrôleur comme suit :

```php
<?php

namespace App\Controllers;

use App\Entities\UserEntity;
use BlitzPHP\Mail\Mail;

class UserController extends AppController
{
    public function update(Mail $mail, $id)
    {
        //...
        
        $user = UserEntity::find($id);
        
        $mail->to($user->email)->send()
        
        //...

        return redirect('/users');
    }
}
```