---
title: Frontend
---

<a name="introduction"></a>
## Introduction

BlitzPHP est un framework backend qui fournit toutes les fonctionnalités dont vous avez besoin pour créer des applications Web modernes, telles que [le routage](/docs/{version}/routage), [la validation](/docs/{version}/validation), [la mise en cache](/docs/{version}/cache), [le stockage de fichiers](/docs/{version}/fichiers), et plus encore. Cependant, nous pensons qu’il est important d’offrir aux développeurs une belle expérience full-stack, y compris des approches puissantes pour créer le frontend de votre application.

La construction de votre frontend peut se faire de 2 façons. Vous pouvez optez pour un rendu côté serveur (SSR) en tirant parti de PHP ou faire un rendu côté client (CSR) en utilisant des frameworks JavaScript tels que Vue et React. Dans tous les cas BlitzPHP est flexible et vous aidera au mieux pour réaliser votre application.

> **Note**  
> Vous pouvez [consulter cette vidéo](https://www.youtube.com/watch?v=sqYfmWixI5A) pour en savoir plus sur les différentes forme de rendu et faire votre choix entre le rendu côté serveur et le rendu côté client avant de vous lancer

<a name="utilisation-de-php"></a>
## Utilisation de PHP

Si vous décidez d'utiliser une approche SSR, BlitzPHP vous laisse le choix entre l'utilisation de PHP classique ou l'utilisation d'un moteur de template pour rendre vos vues. 5 principaux moteurs de template sont supportés par BlitzPHP.

<a name="le-moteur-de-template-natif-de-blitzphp"></a>
### Le moteur de template natif de BlitzPHP

Par défaut, BlitzPHP dispose d'un sémi moteur de template basé sur PHP classique. C'est le moteur activé par défaut mais vous pouvez le changer aisément.

Il offre quelques fonctions puissantes pour faciliter l'écriture de vos vues tout en continuant à écrire du PHP traditionnel c'est-à-dire en continuant d'utiliser les instructions `echo`, `if/else`, `for`.

```php
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?= $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

L'apprentissage de ce système sera étudié en détail dans [une section qui lui est spécialement dédiée](/docs/{version}/vues-natives).

<a name="blade"></a>
### Blade

[Blade](https://laravel.com/docs/master/blade) est sans doute le moteur de template le plus utilisé de nos jour. C'est un langage de modélisation extrêmement léger issu du framework [Laravel](https://laravel.com) qui fournit une syntaxe pratique et courte pour afficher des données, itérer sur des données, etc. :

```blade
<ul>
    @foreach ($users as $user)
        <li>Hello, {{ $user->name }} </li>
    @endforeach
</ul>
```

L'intégration de Blade à BlitzPHP se fait en **une minute**. Vous n'avez qu'à installer le package [beebmx/blade](https://packagist.org/packages/beebmx/blade) et modifier votre fichier `app/Config/view.php` pour que BlitzPHP utilise Blade pour le rendu de vos vues.

<a name="twig"></a>
### Twig

Issu de l'écosytème [Symfony](https://symfony.com/),  [Twig](https://twig.symfony.com/) est un moteur template rapide, sécurisé et flexible

```twig
<ul>
    {% for user in users %}
        <li>Hello, {{ user.name }} </li>
    {% endfor %}
</ul>
```

L'intégration de Twig à BlitzPHP se fait en **une minute**. Vous n'avez qu'à installer le package [twig/twig](https://packagist.org/packages/twig/twig) et modifier votre fichier `app/Config/view.php` pour que BlitzPHP utilise Twig pour le rendu de vos vues.

<a name="smarty"></a>
### Smarty

[Smarty](https://smarty-php.github.io/smarty/stable/) est un moteur de template pour PHP, facilitant la séparation de la présentation (HTML/CSS) de la logique applicative. Il vous permet d'écrire des modèles, en utilisant des variables, des modificateurs, des fonctions et des commentaires.

```smarty
<ul>
    {foreach $users as $user}
        <li>Hello, {$user.name|escape} </li>
    {/foreach}
</ul>
```

L'intégration de Smarty à BlitzPHP se fait en **une minute**. Vous n'avez qu'à installer le package [smarty/smarty](https://packagist.org/packages/smarty/smarty) et modifier votre fichier `app/Config/view.php` pour que BlitzPHP utilise Smarty pour le rendu de vos vues.

<a name="latte"></a>
### Latte

[Latte](https://latte.nette.org/en/) est un moteur de template intuitif et rapide pour ceux qui souhaitent avoir des sites PHP plus sécurisés. Il introduit l’échappement contextuel et se dit être le premier moteur te template véritablement sécurisé et intuitifs pour PHP

```latte
<ul>
    <li n:foreach="$users as $user">
        Hello, {$user->name}
    </li>
</ul>
```

L'intégration de Latte à BlitzPHP se fait en **une minute**. Vous n'avez qu'à installer le package [latte/latte](https://packagist.org/packages/latte/latte) et modifier votre fichier `app/Config/view.php` pour que BlitzPHP utilise Latte pour le rendu de vos vues.

<a name="plates"></a>
### Plates

[Plates](http://platesphp.com/) est un système de template PHP natif rapide, facile à utiliser et facile à étendre. Il s’inspire de l’excellent moteur de template Twig et s’efforce d’apporter des fonctionnalités de langage de template modernes aux modèles PHP natifs. Plates est conçu pour les développeurs qui préfèrent utiliser des modèles PHP natifs plutôt que des langages de modèles compilés, tels que Twig ou Smarty.

```php
<ul>
    <?php foreach($users as $user): ?>
        <li>Hello, <?=$this->e($friend->name)?></li>
    <?php endforeach ?>
</ul>
```

L'intégration de Plates à BlitzPHP se fait en **une minute**. Vous n'avez qu'à installer le package [league/plates](https://packagist.org/packages/league/plates) et modifier votre fichier `app/Config/view.php` pour que BlitzPHP utilise Plates pour le rendu de vos vues.

> **Note**  
> L'installation et la configuration de tous ses différents moteurs de template seront étudiées en détails dans le chapitre réservé aux [vues](/docs/{version}/vues)

<a name="utilisation-de-vuejs-react"></a>
## Utilisation VueJS / React

Bien que BlitzPHP intègre plusieurs moteurs de template puissants pour la création des vues, de nombreux développeurs préfèrent toujours exploiter la puissance d'un framework JavaScript comme Vue ou React. Cela permet aux développeurs de profiter du riche écosystème de packages et d'outils JavaScript disponibles via NPM.

La première approche pour profiter de l'écosystème NPM et BlitzPHP simultanément pourrait être l'adoption des API Rest. Ainsi, BlitzPHP fera des API et un frontend VueJS ou React sera chargé de consommer ces API. 

Le problème de cette approche réside dans le fait que les développeurs doivent gérer deux dépôts de code distincts, et doivent souvent coordonner la maintenance, les versions et les déploiements entre les deux dépôts. Bien que ces problèmes ne soient pas insurmontables, les contributeurs de l'écosystème [Laravel](https://laravel.com) ont pensé que ce n'était pas un moyen productif ou agréable de développer des applications. 

BlitzPHP soutient également ce point de vue et dispose des adapteurs pour profiter des outils tel que [Inertia](https://inertiajs.com) et régler le problème sus-évoqué.

<a name="inertia"></a>
### Inertia

BlitzPHP dispose d'un adapteur pour vous permettre d'utiliser [Inertia](https://inertiajs.com) qui comblera le fossé entre votre application BlitzPHP et votre interface Vue ou React moderne. Ce qui vous permettra de créer des interfaces modernes à part entière à l’aide de Vue ou React tout en tirant parti de toutes les fonctionnalités offertes par BlitzPHP, le tout dans un dépôt de code unique.

Après avoir installé **Inertia** dans votre application, vous allez écrire des routes et des contrôleurs comme d’habitude. Cependant, au lieu de renvoyer une vue à partir de votre contrôleur, vous renverrez une page Inertia :

```php
<?php

namespace App\Controllers;

use App\Entities\UserEntity;

class UserController extends AppController
{
    /**
     * Affiche le profil de l'utilisateur donné.
     */
    public function show(string $id): Response
    {
        return inertia('Users/Profile', [
            'user' => UserEntity::findOrFail($id)
        ]);
    }
}
```

Une page Inertia correspond à un composant Vue ou React, généralement stocké dans le dossier `/resources/js/Pages` de votre application. Les données transmises à la page via la fonction `inertia` seront utilisées pour hydrater les « props » du composant de page:

```html
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/inertia-vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="Profil d'utilisateur" />

    <Layout>
        <template #header>
            <h2 class="font-weight-bold display-3 text-muted">
                Profil
            </h2>
        </template>

        <div class="py-2">
            Bonjour, {{ user.name }}
        </div>
    </Layout>
</template>
```

Inertia vous permet donc de tirer parti de toute la puissance de Vue ou de React lors de la construction de votre frontend, tout en fournissant un pont léger entre votre backend alimenté par BlitzPHP et votre frontend alimenté par JavaScript. 

> **Note**  
> Veuillez lire [la section consacrée à inertia](/docs/{version}/inertia) pour en savoir plus sur son intégration et son utilisation avec BlitzPHP.