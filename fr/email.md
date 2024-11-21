---
title: Envoi d'email
---

<a name="introduction"></a>
## Introduction

L'envoi d'emails ne doit pas être compliqué. BlitzPHP fournit une API d'envoi d'emails simple et propre, alimentée par soit par <a href="https://github.com/PHPMailer/PHPMailer" target="_blank">PHPMailer</a>, soit par <a href="https://symfony.com/doc/6.2/mailer.html" target="_blank">Symfony Mailer</a>. BlitzPHP fourni des pilotes pour l'envoi d'emails vous permettant de commencer rapidement à envoyer des emails via un service local ou basé sur le cloud de votre choix.
    
<a name="configuration"></a>
## Configuration

Le service d'email de BlitzPHP peut être configuré via le fichier de configuration `app/Config/mail.php` de votre application. Peu importe le gestionnaire que vous utiliserez, la configuration est la même et BlitzPHP se chargera de faire les transformations nécessaires.

Votre fichier de configuration retourne un tableau avec toutes les clés de configuration nécessaire au fonctionnement du service d'envoie de mail.  
Voici une liste de toutes les configurations qui peuvent être définies lors de l'envoi d'email.

| Configuration  | Valeur par défaut | Options                                                                                                                                  | Description                                                                                                                                                                                                                                                           |
|----------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `from.address` |                   |                                                                                                                                          | Email de l'expéditeur par défaut. Il sera automatiquement défini dans l'en-tête "from" du mail si aucune autre adresse n'est explicitement renseignée lors de l'envoi.                                                                                                |
| `from.name`    |                   |                                                                                                                                          | Nom de l'expéditeur par défaut.                                                                                                                                                                                                                                       |
| `view_dir`     | **emails**        |                                                                                                                                          | Dossier de base dans lequel sera pris les vues des emails. Il doit être un sous dossier de `app/Views`                                                                                                                                                                |
| `template`     |                   |                                                                                                                                          | Layout dont heritera toutes les vues (html) d'emails                                                                                                                                                                                                                  |
| `handler`      | **phpmailer**     | `phpmailer`, `symfony`, `class-string<\BlitzPHP\Mail\Adapters\AbstractAdapter>`                                                        | Gestionnaire à utiliser pour les envoies de mail                                                                                                                                                                                                                      |
| `protocol`     | **smtp**          | * `smtp`, `sendmail`, `mail`<br>* `qmail`: PHPMailer uniquement<br>* `mailgun`, `sendgrid`, `postmark`, `ses`: Symfony Mailer uniquement | Le protocole utilisé pour envoyer les emails                                                                                                                                                                                                                          |
| `host`         |                   |                                                                                                                                          | Adresse du serveur d'envoi de mail                                                                                                                                                                                                                                    |
| `port`         | **25**            |                                                                                                                                          | Port d'écoute du serveur                                                                                                                                                                                                                                              |
| `timeout`      | **5**             |                                                                                                                                          | Délai SMTP (en secondes).                                                                                                                                                                                                                                             |
| `username`     |                   |                                                                                                                                          | Utilisateur SMTP ou clé d'API (Sendgrid par exemple)                                                                                                                                                                                                                  |
| `password`     |                   |                                                                                                                                          | Mot de passe                                                                                                                                                                                                                                                          |
| `encryption`   | **none**          | `none`, `tls`, `ssl`                                                                                                                     | Chiffrement SMTP. La valeur `ssl` permet de créer un canal sécurisé vers le serveur à l'aide de SSL, et la valeur `tls` permet d'envoyer une commande STARTTLS au serveur. Les connexions sur le port 465 doivent avoir pour valeur `none`. |
| `mailType`     | **html**          | `html`, `text`                                                                                                                           | Type de mail. Si vous envoyez un mail en HTML, vous devez l'envoyer comme une page web complète. Assurez-vous que vous n'avez pas de liens relatifs ou de chemins d'accès relatifs aux images, sinon ils ne fonctionneront pas.                                       |
| `charset`      | **UTF-8**         |                                                                                                                                          | Jeu de caractères (utf-8, iso-8859-1, etc.)                                                                                                                                                                                                                           |
| `priority`     | **3**             | 1, 2, 3, 4, 5                                                                                                                            | Priorité de l'email. `1` = la plus élevée. `5` = le plus bas. `3` = normal.                                                                                                                                                                                           |

<a name="utilisation-avec-phpmailer"></a>
### Utilisation avec PHPMailer

De part sa simplicité, <a href="https://github.com/PHPMailer/PHPMailer" target="_blank">PHPMailer</a> est le gestionnaire d'envoi d'email par défaut de BlitzPHP. Pour commencer à l'utiliser; il vous suffit simplement de l'installer via Composer :

```shell
composer require phpmailer/phpmailer
```

Ceci étant fait, il ne vous restera plus qu'à définir vos accès SMTP dans le fichier de configuration `app/Config/mail.php` et BlitzPHP fera toutes les configurations nécessaire pour vous.

<a name="utilisation-avec-symfony-mailer"></a>
### Utilisation avec Symfony Mailer

<a href="https://symfony.com/doc/6.2/mailer.html" target="_blank">Symfony Mailer</a> est l'un des meilleurs outils d'envoi de mail. Il fourni plusieurs protocoles d'envoi de mail par SMTP et via les services Cloud tels que Amazone SES.

Pour commencer, installer le package via la commande suivante:

```shell
composer require symfony/mailer
```

La seconde étape consiste à changer le gestionnaire d'email à utiliser. Par défaut, il est cablé sur `phpmailer`, modifié-le en `symfony` tout simplement.

```php
// app/Config/mail.php

return [
    // ---

    'handler' => 'symfony',

    // ---
];
```

A ce niveau, vous pouvez déjà utiliser Symfony Mailer pour envoyer des mails via le protocole SMTP classique ou sendmail. Si c'est votre intension, vous pouvez directement allez à la grande section suivante. Si par contre vous utiliser un service Cloud, nous vous conseillons de poursuivre votre lecture pour apprendre à bien configurer les différents services pris en charge.

<a name="pilote-mailgun"></a>
#### Pilote Mailgun

Pour utiliser le pilote Mailgun, installez le transport Mailgun Mailer de Symfony via Composer :

```shell
composer require symfony/mailgun-mailer
```

Ensuite, définissez le protocole utilisé à `mailgun`:

```php
return [
    // ...
    'protocol' => 'mailgun',
    // ...
];
```

Vous pouvez également définir des paramètres propres à mailgun tels que la <a href="https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions" target="_blank">région</a> que vous souhaitez utiliser: 

```php
return [
    // ...
    'mailgun' => [
        'region' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net')
    ],
    // ...
];
```

<a name="pilote-sendgrid"></a>
#### Pilote SendGrid

Pour utiliser le pilote SendGrid, installez le transport SendGrid Mailer de Symfony via Composer :

```shell
composer require symfony/sendgrid-mailer
```

Ensuite, définissez le protocole utilisé à `sendgrid`:

```php
return [
    // ...
    'protocol' => 'sendgrid',
    // ...
];
```

Vous devez mapper le paramètre `username` à votre clé d'API SendGrid : 

```php
return [
    // ...
    'username' => env('SENDGRID_API_KEY'),
    // ...
];
```

<a name="pilote-postmark"></a>
#### Pilote Postmark

Pour utiliser le pilote Postmark, installez le transport Postmark Mailer de Symfony via Composer :

```shell
composer require symfony/postmark-mailer
```

Ensuite, définissez le protocole utilisé à `postmark`:

```php
return [
    // ...
    'protocol' => 'postmark',
    // ...
];
```

Vous devez mapper le paramètre `username` à votre token Postmark : 

```php
return [
    // ...
    'username' => env('POSTMARK_TOKEN'),
    // ...
];
```

<a name="pilote-mandrill"></a>
#### Pilote Mandrill

Pour utiliser le pilote Mandrill, installez le transport Mandrill Mailer de Symfony via Composer :

```shell
composer require symfony/mailchimp-mailer
```

Ensuite, définissez le protocole utilisé à `mandrill`:

```php
return [
    // ...
    'protocol' => 'mandrill',
    // ...
];
```

<a name="pilote-amazon-ses"></a>
#### Pilote Amazon SES

Pour utiliser le pilote Amazon SES, vous devez d'abord installer le package Symfony Amazon SES via le gestionnaire de paquets Composer :

```shell
composer require symfony/amazon-mailer
```

Ensuite, définissez le protocole utilisé à `ses`:

```php
return [
    // ...
    'protocol' => 'ses',
    // ...
];
```

Vous pouvez également définir des paramètres propres à Amazon SES tels que la région que vous souhaitez utiliser ou encore les <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html" target="_blank">informations d'identification temporaires</a> AWS : 

```php
return [
    // ...
    'ses' => [
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'session_token' => env('AWS_SESSION_TOKEN'),
    ],
    // ...
];
```

<a name="envoyer-des-mails"></a>
## Envoyer des mails

BlitzPHP fourni le service `mail` pour vous permettre d'envoyer des mails en toute simplicité. Voici un exemple de base montrant comment vous pourriez envoyer un mail :

```php
$mail = service('mail');

$mail->to('someone@example.com')
    ->cc('another@another-example.com')
    ->bcc('them@their-example.com')
    ->replyTo('customer@example.com')
    ->subject('Test de mail')
    ->text("Test de l'envoi des mails.");

$mail->send();
```

> **Note**  
> Vous remarquez que nous n'avons pas défini l'expéditeur du mail dans cet exemple, en effet BlitzPHP utilise par défaut la clé `from` de votre [configuration des mails](#configuration). Néanmoins, vous pouvez changer l'expéditeur à la volée via la méthode `from()`.  
> ```php  
> $mail->from('your@example.com', 'Your Name');
> ```

<a name="configuration-a-la-volee"></a>
### Configuration à la volée

Toutes vos configurations peuvent être modifiées à la volée lors de l'envoie d'un mail. Cela est notamment utile si vous voulez par exemple modifier le gestionnaire d'envoie dans une situation donnée. Pour cela vous avez la méthode `merge`

```php
$mail = service('mail')->merge([
    'charset'  => 'iso-8859-1',
    'priority' => 1,
    'protocol' => 'sendmail',
]);
```

Une nouvelle instance du service d'envoi de mail vous créée avec les configurations souhaitées.

<a name="message-brute-vs-message-html"></a>
### Message brute vs message HTML

Dans l'exemple ci-dessus, nous avons utiliser la méthode `text()` pour définir le contenu de notre mail. Cette approche fonctionne très bien pour des mails simple.   
Cependant, les mails sont souvent bien plus que de simples messages avec du texte. BlitzPHP met alors à votre disposition la méthode `html()` vous permettant d'envoyer du HTML en tant que mail.

```php
$mail = service('mail');

$content = file_get_contents('email-content.html');

$mail->to('someone@example.com')
    ->subject('Test de mail')
    ->html($content);

$mail->send();
```

> **Note**  
> En plus des méthodes `text()` et `html()`, vous disposez également de la méthode `message()`. Cette méthode définira le contenu de votre mail comme étant du texte simple ou du HTML en fonction de la valeur de votre configuration `mailType`.

<a name="utilisation-des-vues"></a>
#### Utilisation des vues

Une fois que vous avez pris conscience du fait que vous devez envoyer vos mails au format html, la plus grande difficulté devient alors la génération de ce HTML. Heureusement, BlitzPHP propose d’envoyer des mails ayant une structure complexe en utilisant [sa couche de rendu](/docs/{version}/vues); vous devez alors utiliser la méthode `view` pour indiquer la vue que vous souhaiter envoyer.

Les vues pour les mails se trouvent par défaut dans le dossier spécial `app/Views/emails`. 

```php
$mail = service('mail');

$data = $db->query('SELECT * FROM students')->all();

$mail->to('director@school.com')
    ->subject('Liste des eleves')
    ->view('students', ['students' => $data]);

$mail->send();
```

> **Note**  
> La vue des mails est une vue classique comme toute autre vue. De ce fait, vous avez accès à toutes les fonctionnalités des [vues BlitzPHP](/docs/{version}/vues).

> **Note**  
> Vous pouvez également utiliser des vues qui ne sont pas dans votre dossier des vues des mails (défini par le paramètre `view_dir` de la configuration `/app/Config/mail.php`). Dans ce cas, vous devez utiliser la syntaxe des[ vues avec namespace](/docs/{version}/vues#vues-avec-namespace).  
>   
> ```php  
> $mail->view('\App\Package\Email\students', ['students' => $data]);  
> ```

<a name="envoyer-des-pieces-jointes"></a>
### Envoyer des pièces jointes

Pour ajouter des pièces jointes à un mail, vous devez utiliser la méthode `attach()` du service d'envoi de mail.   
Tout d'abord, vous pouvez ajouter une pièce jointe en fournissant un chemin d'accès au fichier à joindre :

```php
$mail = service('mail');

$mail->attach('/path/to/file');
```

Lorsque vous joignez des fichiers à un message, vous pouvez également spécifier le nom d'affichage et/ou le type MIME de la pièce jointe à l'aide des 2ème et 3ème paramètres de la méthode `attach()` respectivement :

```php
$mail->attach('/path/to/file', 'name.pdf', 'application/pdf');
```

<a name="joindre-de-donnees-brutes"></a>
#### Joindre de données brutes

La méthode d'attachement `attachBinary()` peut être utilisée pour attacher une chaîne brute d'octets en tant que pièce jointe. Par exemple, vous pouvez utiliser cette méthode si vous avez généré un PDF en mémoire et que vous souhaitez le joindre au mail sans l'écrire sur le disque. La méthode `attachBinary()` accepte une la chaîne binaire à joindre ainsi que le nom à l'attribuer :

```php
$mail = service('mail');

$pdfContent = $html2pdf->output('invoice.pdf', 'S');

$mail->to($user->email)->attachBinary($pdfContent, 'invoice.pdf')->send();
```

Vous pouvez aussi spécifier le type MIME de la pièce jointe à l'aide du 3ème paramètre de la méthode `attachBinary()`.

<a name="pieces-jointes-en-ligne"></a>
#### Pièces jointes en ligne

Si vous souhaitez afficher des images dans votre mail, vous devez les intégrer au lieu de les ajouter en tant que pièces jointes. Pour cela, utilisez la méthode `embedded()` pour ajouter une image à partir d'un fichier :

```php
$mail = service('mail');

$mail->embedded('/path/to/file', 'cid_key');
```

> **Note**  
> Le deuxième argument de la méthode `embedded()` est le nom de l'image ("Content-ID" dans la norme MIME). Sa valeur est une chaîne arbitraire qui doit être unique dans chaque message électronique et qui est utilisée ultérieurement pour référencer les images dans le contenu HTML :

Une fois le fichier "embarqué", vous pouvez le référencer dans votre contenu HTML comme suit:

```html
<img src="cid:cid_key"> 
```

Tout comme la méthode `attach()`, `embedded()` possède également son équivalent pour insérer des données brutes en tant que pièce jointe en ligne :

```php
$mail->embeddedBinary('/path/to/file', 'cid_key', 'logo.png');
```
   
<a name="definir-des-entetes"></a>
### Définir des entêtes

Il est parfois nécessaire de joindre des en-têtes supplémentaires au message sortant. Par exemple, vous pouvez avoir besoin de définir un Message-Id personnalisé ou d'autres en-têtes de texte arbitraires.

Pour ce faire, vous devez utiliser a méthode `header()` :

```php
$mail->header('X-Custom-Header', 'Custom Value')
     ->header('X-Custom-Header-2', 'Custom Value 2');
     
// ou utiliser un tableau associatif


$mail->header([
    'X-Custom-Header' => 'Custom Value',
    'X-Custom-Header-2' => 'Custom Value 2'
]);
```

Vous devriez vous renseigner sur la classe MessageFormatter et sur le formatage ICU sous-jacent afin d'avoir une meilleure idée de ses capacités, comme le remplacement conditionnel, la pluralisation, etc. Les deux liens mentionnés plus haut vous donneront une excellente idée des options disponibles.

<a name="creer-des-mails-reutilisables"></a>
## Créer des mails réutilisables

Jusqu’à présent, nous avons vu comment utiliser le service `mail` pour créer et envoyer des mails. Mais la principale fonctionnalité d’un mailer est de vous permettre de créer des mails réutilisables n’importe où dans votre application. Ils peuvent aussi servir à contenir différentes configurations d’emails en un seul et même endroit, ce qui vous aide à garder votre code DRY et à déplacer la configuration des emails en dehors des autres parties de votre application.

<a name="generer-des-mailings"></a>
### Générer des mailings

Vous pouvez organiser votre application de telle sorte que chaque type d'email envoyé soit représenté par une classe `Mailable`. Ces classes sont stockées dans le répertoire `app/Mail`. Ne vous inquiétez pas si vous ne voyez pas ce répertoire dans votre application, car il sera généré pour vous lorsque vous créerez votre première classe de mail à l'aide de la commande Klinge `make:mail` :

```shell
php klinge make:mail Welcome
```

<a name="utilisation-des-mailings"></a>
### Utilisation des mailings

Une fois que vous avez généré une classe "Mailable", ouvrez-la afin d'en explorer le contenu. La configuration de la classe disponible s'effectue au moyen de plusieurs méthodes, notamment les méthodes `subject()`, `content()` et `attach()`. La méthode `content()` définit la vue qui sera utilisée pour générer le contenu du message.

<a name="configuration-de-l-expediteur"></a>
#### Configuration de l'expéditeur

Commençons par configurer l'expéditeur du mail. En d'autres termes, il s'agit de savoir de qui l'e-mail va provenir. Par défaut, la valeur `from` de votre fichier de configuration sera utilisé. Mais vous pouvez le modifier en implémentant la méthode `from` dans votre classe de mailing :

```php
/**
 * @return string[]
 *
 * @example
 *  ['johndoe@mail.com', 'John Doe']
 *  ['johndoe@mail.com']
 */
public function from(): array;
```

Si vous le souhaitez, vous pouvez également spécifier une adresse de réponse (replyTo) :

```php
/**
 * @return array<string, string>|string[]
 *
 * @example
 *  [
 *      'johndoe@mail.com' => 'john doe',
 *      'janedoe@mail.com',
 *  ]
 */
public function replyTo(): array;
```

Le sujet du message se définit via la méthode `subject()`:

```php
public function subject(): string;
```

<a name="configuration-du-contenu"></a>
#### Configuration du contenu

Dans la méthode `content()` d'une classe de mailing, vous pouvez définir la vue à utiliser pour le rendu du contenu du mail. Étant donné que [chaque mail utilise généralement une vue pour rendre son contenu](#utilisation-des-vues), vous disposez de toute la puissance et de la commodité de [votre moteur de template](/docs/{version}/vues#utilisation-d-un-moteur-de-template) lors de la construction du code HTML de votre mail :

```php
public function content(): array
{
    return [
        'view' => 'mail_view',
    ];
}
```

> **Note**  
> Gardez à l'esprit que les vues des mails doivent être dans le dossier `app/Views/emails`. Si vous souhaitez utiliser un autre dossier, vous devez au préalable modifier votre [configuration "view_dir"](#configuration). Les [vues avec namespace](/docs/{version}/vues#vues-avec-namespace) sont également admis.

<a name="donnees-de-vue"></a>
#### Données de vue

En règle générale, vous voudrez transmettre à votre vue des données que vous pourrez utiliser lors du rendu de l'HTML du mail. Il existe deux façons de mettre des données à la disposition de la vue. Tout d'abord, toute propriété publique définie dans votre classe "Mailable" sera automatiquement mise à la disposition de la vue. Ainsi, par exemple, vous pouvez passer des données dans le constructeur de votre classe disponible et définir ces données dans les propriétés publiques de la classe :

```php
<?php
 
namespace App\Mail;
 
use App\Entities\User;
use BlitzPHP\Mail\Mailable;

class Welcome extends Mailable
{
    public function __construct(public User $user) 
    {
        
    }
 
    public function content(): array
    {
        return [
            'view' => 'welcome',
        ];
    }
}
```

Une fois que les données ont été définies comme propriétés publiques, elles sont automatiquement disponibles dans votre vue et vous pouvez donc y accéder comme vous le feriez pour n'importe quelle autre donnée dans vos vues :

```php
<h2>
    Salut <?= $user->username ?>
</h2>
```

Si vous souhaitez personnaliser le format des données de votre mail avant qu'il ne soit envoyé à la vue, vous pouvez transmettre manuellement vos données via la méthode `with()`. En règle générale, vous continuez à transmettre des données via le constructeur de la classe "Mailable"; cependant, vous devez définir ces données comme des propriétés protégées ou privées afin qu'elles ne soient pas automatiquement mises à la disposition de la vue. Ensuite, lorsque vous implémentez la méthode `with()` qui retournera un tableau de données que vous souhaitez mettre à la disposition de votre vue :

```php
<?php
 
namespace App\Mail;
 
use App\Entities\User;
use BlitzPHP\Mail\Mailable;

class Welcome extends Mailable
{
    public function __construct(private User $user) 
    {
        
    }
 
    public function content(): array
    {
        return [
            'view' => 'welcome',
        ];
    }
    
    public function with(): array
    {
        return [
            'username' => $this->user->username,
        ];
    }
}
```

Une fois que les données ont été renvoyées par la méthode `with()`, elles sont automatiquement disponibles dans votre vue, de sorte que vous pouvez y accéder comme vous le feriez pour n'importe quelle autre donnée :

```php
<h2>
    Salut <?= $username ?>
</h2>
```

<a name="envoyer-un-mailing"></a>
### Envoyer un mailing

Une fois que vous avez configurer votre classe "Mailable", vous pouvez l'envoyer à tout moment en utilisant la méthode `envoi()` du service de mail :

```php
service('mail')->envoi(new \App\Mail\Welcome($user));
```

Cet exemple suppose que le destinataire du message est défini directement dans la classe de mailing. Cette approche peut montre quelque limite, il est donc préférable de définir les destinataires du mail hors de la classe de mailing. Ainsi, nous aurions :

```php
service('mail')->to($user->email)->envoi(new \App\Mail\Welcome($user));
``` 

<a name="passer-en-boucle-sur-les-destinataires"></a>
#### Passer en boucle sur les destinataires

Il peut arriver que vous deviez envoyer un message à une liste de destinataires en itérant sur un tableau d'adresses électroniques. Cependant, comme la méthode `to()` ajoute les adresses électroniques à la liste des destinataires du "Mailable", chaque itération dans la boucle enverra un autre courriel à tous les destinataires précédents. Par conséquent, vous devez toujours recréer l'instance de "Mailable" pour chaque destinataire :

```php
$mail = service('mail');

foreach (['dimtrovich@example.com', 'johndoe@example.com'] as $recipient) {
    $mail->to($recipient)->send(new \App\Mail\Order($order));
}
```

<a name="gestionnaire-de-mail-personnalise"></a>
## Gestionnaire de mail personnalisé

BlitzPHP utilise PHPMailer et Symfony Mailer pour l'envoi des mails ; cependant, vous pouvez souhaiter écrire vos propres gestionnaires pour envoyer des mails via d'autres services qui ne sont pas pris en charge d'emblée.   
Pour commencer, définissez une classe qui étend la classe `BlitzPHP\Mail\Adapters\AbstractAdapter`. Ensuite, implémentez les méthodes suivantes :

```php
abstract public function setPort(int $port): static;

abstract public function setHost(string $host): static;

abstract public function setUsername(string $username): static;

abstract public function setPassword(string $password): static;

abstract public function setDebug(int $debug = 1): static;

abstract public function setProtocol(string $protocol): static;

abstract public function setTimeout(int $timeout): static;

abstract public function setCharset(string $charset): static;

abstract public function setPriority(int $priority): static;

abstract public function setEncryption(?string $encryption): static;
```

Vous devez également implémenter toutes les méthodes de l'interface `BlitzPHP\Contracts\Mail\MailerInterface` définies ci dessous :

```php
public function alt(string $content): static;

public function attach(array|string $path, string $name = '', string $type = '', string $encoding = self::ENCODING_BASE64, string $disposition = 'attachment'): static;

public function attachBinary($binary, string $name, string $type = '', string $encoding = self::ENCODING_BASE64, string $disposition = 'attachment'): static;

public function bcc(array|string $address, bool|string $name = '', bool $set = false): static;

public function cc(array|string $address, bool|string $name = '', bool $set = false): static;

public function dkim(string $pk, string $passphrase = '', string $selector = '', string $domain = ''): static;

public function embedded(string $path, string $cid, string $name = '', string $type = '', string $encoding = self::ENCODING_BASE64, string $disposition = 'inline'): static;

public function embeddedBinary($binary, string $cid, string $name = '', string $type = '', string $encoding = self::ENCODING_BASE64, string $disposition = 'inline'): static;

public function from(string $address, string $name = ''): static;

public function header(array|string $name, ?string $value = null): static;

public function html(string $content): static;

public function init(array $config): static;

public function lastId(): string;

public function message(string $message): static;

public function replyTo(array|string $address, bool|string $name = '', bool $set = false): static;

public function send(): bool;

public function sign(string $cert_filename, string $key_filename, string $key_pass, string $extracerts_filename = ''): static;

public function subject(string $subject): static;

public function text(string $content): static;

public function to(array|string $address, bool|string $name = '', bool $set = false): static;
```

Une fois que vous avez défini votre gestionnaire personnalisé, vous pouvez l'utiliser en changeant la valeur du paramètre `handler` dans votre fichier de configuration ou à la volée via la méthode `merge()` :

```diff-php
// app/Config/mail.php

return [
    // ---

-   'handler' => 'phpmailer',
+   'handler' => \App\Handlers\Mail\CustomMailHandler::class,

    // ---
];
```
