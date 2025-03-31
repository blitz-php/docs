---
title: Middlewares de sécurité
---

<a name="introduction"></a>
## Introduction

Comme vous le savez déjà certainement, la sécurité est l'un des sujet primordiaux chez BlitzPHP. A cette effet, BlitzPHP met à votre disposition un batterie d'outils pour vous protéger au maximum des attaques malveillantes.

Nous vous proposons dans cette section, d'explorer [les middlewares](/docs/{version}/middleware) de sécurité qui vous sont offert pour vous protéger au mieux de nombreuses attaques.

<a name="gestion-de-cookies-chiffres"></a>
## Gestion de Cookies Chiffrés

Si votre application utilise des cookies qui contiennent des données que vous avez besoin de masquer pour vous protéger contre les modifications utilisateurs, vous pouvez utiliser le middleware de gestion des cookies chiffrés de BlitzPHP pour chiffrer et déchiffrer les données des cookies. Les données des cookies sont chiffrées via OpenSSL, en AES.

Pour l'activer, il vous suffit d'ajouter le middleware `App\Middlewares\App\Middlewares\EncryptCookies` dans le tableau `globals` de votre fichier de configuration de middleware.

```php
// app/Config/middlewares.php

return [
    // ...
    /**
     * @var array<class-string|Closure|string>
     */
    'globals' => [
        // ...
        App\Middlewares\App\Middlewares\EncryptCookies::class,
        // ...
    ],
    // ...
];
```

Ceci étant fait, TOUS les cookies émis par votre application seront [chiffrés](/docs/{version}/chiffrement) avant d'être envoyés au navigateur.  
Si vous voulez que certains cookie ne soient pas chiffrés, vous devez ajouter leurs noms à la propriété `$except` de ce middleware.

```php
// app/Middlewares/EncryptCookies;

<?php

namespace App\Middlewares;

use BlitzPHP\Middlewares\EncryptCookies as Middleware;

class EncryptCookies extends Middleware
{
    /**
     * @var list<string>
     */
    protected array $except = [
       'visible-cookie', 
    ];
}
```

<a name="content-security-policy"></a>
## Content Security Policy (CSP)

L'une des meilleures protections dont vous disposez contre les attaques XSS est de mettre en place un Politique de sécurité (CSP) sur le site. Pour cela, vous devez spécifier et autoriser chaque source de contenu incluse dans le code HTML de votre site, y compris les images, feuilles de style, fichiers JavaScript, etc. Le navigateur rejettera le contenu de sources qui ne sont pas explicitement approuvées. Cette autorisation est définie dans l’en-tête `Content-Security-Policy` de la réponse et propose diverses options de configurations.

Le middleware `Csp` rend les choses plus simples pour ajouter des en-têtes `Content-Security-Policy` dans votre application. Avant de l’utiliser, vous devez installer la dépendance `paragonie/csp-builder`:

```bash
composer require paragonie/csp-builder
```

Pour utiliser ce middleware, vous devez le configurer via la clé `build` de votre fichier `app/Config/middlewares.php`. Cette configuration peut se faire en utilisant un tableau, ou en lui passant un objet `CSPBuilder` déjà construit:

```php
use BlitzPHP\Http\MiddlewareQueue;
use BlitzPHP\Middlewares\Csp;

return [
    // ...
    'build' => function(MiddlewareQueue $queue) {
        $csp = new Csp([
            'script-src' => [
                'allow' => [
                    'https://www.google-analytics.com',
                ],
                'self' => true,
                'unsafe-inline' => false,
                'unsafe-eval' => false,
            ],
        ]);

        $queue->add($csp);
    },
    // ...
];
```

Si vous voulez utiliser une configuration CSP plus stricte, vous pouvez activer des règles CSP basées sur le nonce avec les options `script_nonce` et `style_nonce`. Lorsqu'elles sont activées, ces options vont modifier votre politique CSP et définir les attributs `cspScriptNonce` et `cspStyleNonce` dans la requête. Ces attributs sont appliqués à l’attribut `nonce` de tous les éléments scripts et liens CSS. Cela simplifie l’adoption de stratégies utilisant un <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src" target="_blank">nonce-base64</a> et `strict-dynamic` pour un surcroît de sécurité et une maintenance plus facile:

```php
$policy = [
    // Doivent exister, même vides, pour définir le nonce pour script-src
    'script-src' => [],
    'style-src' => [],
];
// Active l'ajout automatique du nonce aux tags script & liens CSS.
$csp = new Csp($policy, [
    'script_nonce' => true,
    'style_nonce' => true,
]);
$queue->add($csp);
```

<a name="headers-de-securite"></a>
## Headers de sécurité

Le middleware `BlitzPHP\Middlewares\SecureHeaders` vous permet d’ajouter à votre application des headers liés à la sécurité. Une fois configuré, le middleware peut ajouter les headers suivants aux réponses:

- `X-Content-Type-Options`
- `X-Download-Options`
- `X-Frame-Options`
- `X-Permitted-Cross-Domain-Policies`
- `Referrer-Policy`

L'activation de ce middleware se fait simplement en ajoutant `secureheaders` au tableau `globals` du fichier de configuration des middlewares :

```php
// app/Config/middlewares.php

return [
    // ...
    /**
     * @var array<class-string|Closure|string>
     */
    'globals' => [
        // ...
        'secureheaders',
        // ...
    ],
    // ...
];
```

Pour personnaliser les en-têtes, étendez la classe `BlitzPHP\Middlewares\SecureHeaders` et remplacez la propriété `$headers`. Modifiez ensuite la clé `aliases` dans le fichier `app/Config/middlewares.php` :

```php
// app/Config/middlewares.php

return [
    // ...
    /**
     * @var array<class-string>
     */
    'aliases' => [
        // ...
        'secureheaders' => \App\Middlewares\SecureHeaders::class
        // ...
    ],
    // ...
];
```

Si vous souhaitez en savoir plus sur les en-têtes sécurisés, consultez <a href="https://owasp.org/www-project-secure-headers/" target="_blank">le projet OWASP Secure Headers</a>.

<a name="forcer-le-https"></a>
## Forcer le https

Si vous voulez que votre application soit accessible uniquement par des connexions HTTPS, le middleware `BlitzPHP\Middlewares\ForceHTTPS` est mis à votre disposition pour le faire.

Si vous attribuer la valeur "`true`" à la clé `force_global_secure_requests` du fichier `app/Config/app.php`, toutes les requêtes adressées à cette application seront obligatoirement effectuées via une connexion sécurisée (HTTPS). Si la requête entrante n'est pas sécurisée, l'utilisateur sera redirigé vers une version sécurisée de la page et l'en-tête HTTP Strict Transport Security (HSTS) sera défini.