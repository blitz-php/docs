---
title: Hashage
---

<a name="introduction"></a>
## Introduction

BlitzPHP met à votre disposition le service `hashing` qui fournit un hachage sécurisé `Bcrypt` et `Argon2` pour stocker les mots de passe des utilisateurs.

Bcrypt est un excellent choix pour le hachage de mots de passe car son "facteur de travail" est réglable, ce qui signifie que le temps nécessaire pour générer un hachage peut être augmenté en fonction de la puissance du matériel. Lorsqu'il s'agit de hacher des mots de passe, la lenteur est une bonne chose. Plus un algorithme met de temps à hacher un mot de passe, plus il faut de temps aux utilisateurs malveillants pour générer des "tables arc-en-ciel" de toutes les valeurs de hachage de chaîne possibles, qui peuvent être utilisées dans des attaques par force brute contre des applications.

<a name="configuration"></a>
## Configuration

Le pilote de hachage par défaut de votre application est configuré dans le fichier de configuration `/app/Config/hashing.php` de votre application. Il existe actuellement plusieurs pilotes supportés : <a href="https://fr.wikipedia.org/wiki/Bcrypt" target="_blank">Bcrypt</a> et <a href="https://en.wikipedia.org/wiki/Argon2" target="_blank">Argon2</a> (variantes Argon2i et Argon2id).

<a name="utilisation-de-base"></a>
## Utilisation de base

<a name="hacher-les-mots-de-passe"></a>
### Hacher les mots de passe

Vous pouvez hacher un mot de passe en appelant la méthode `make` du service `hashing` :

```php
<?php

namespace App\Controllers;

use BlitzPHP\Http\Redirection;

class PasswordController extends AppController
{
    /**
     *  Mettre à jour le mot de passe de l'utilisateur.
     */
    public function update(): Redirection
    {
        // Valider la nouvelle longueur du mot de passe...

        auth()->user()->fill([
            'password' => service('hashing')->make($this->request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="ajustement-du-facteur-de-travail-de-bcrypt"></a>
#### Ajustement du facteur de travail de Bcrypt

Si vous utilisez l'algorithme Bcrypt, la méthode `make` vous permet de gérer le facteur de travail de l'algorithme à l'aide de l'option `rounds`; cependant, le facteur de travail par défaut de BlitzPHP est acceptable pour la plupart des applications :

```php
$hashed = service('hashing')->make('password', [
    'rounds' => 12,
]);
```

<a name="ajustement-du-facteur-de-travail-de-argon2"></a>
#### Ajustement du facteur de travail de Argon2

Si vous utilisez l'algorithme Argon2, la méthode `make` vous permet de gérer le facteur de travail de l'algorithme en utilisant les options `memory`, `time` et `threads`; cependant, les valeurs par défaut de BlitzPHP sont acceptables pour la plupart des applications :

```php
$hashed = service('hashing')->make('password', [
    'memory'  => 1024,
    'time'    => 2,
    'threads' => 2,
]);
```

> **Note**  
> Pour plus d'informations sur ces options, veuillez vous référer à l<a href="https://www.php.net/manual/fr/function.password-hash.php" target="_blank">a documentation officielle de PHP concernant le hachage</a>.

<a name="verification-de-la-correspondance-entre-un-mot-de-passe-et-un-hachage"></a>
### Vérification de la correspondance entre un mot de passe et un hachage

La méthode `check` fournie par le service de hashage permet de vérifier qu'une chaîne de texte en clair donnée correspond à un hachage donné :

```php
if (service('hashing')->check('plain-text', $hashedPassword)) {
    // Le mot de passe correspond...
}
```

<a name="determiner-si-un-mot-de-passe-doit-etre-reecrit"></a>
### Déterminer si un mot de passe doit être réécrit

La méthode `needsRehash` permet de déterminer si le facteur de travail utilisé par le hachoir a changé depuis que le mot de passe a été haché. Certaines applications choisissent d'effectuer cette vérification au cours du processus d'authentification de l'application :

```php
if (service('hashing')->needsRehash($hashed)) {
    $hashed = service('hashing')->make('plain-text');
}
```