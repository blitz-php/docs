---
title: Guide de sécurité
---

<a name="introduction"></a>
## Introduction

Nous prenons la sécurité au sérieux. BlitzPHP incorpore un certain nombre de fonctionnalités et de techniques pour renforcer les bonnes pratiques de sécurité, ou pour vous permettre de le faire facilement.

Nous respectons l'<a href="https://owasp.org/" target="_blank">Open Web Application Security Project (OWASP)</a> et suivons ses recommandations autant que possible.

Ce qui suit provient du <a href="https://owasp.org/www-project-top-ten/" target="_blank">Top 10 de l'OWASP</a> et du <a href="https://owasp.org/API-Security/editions/2023/en/0x11-t10/" target="_blank">Top 10 de l'OWASP API Security</a>, qui identifient les principales vulnérabilités des applications web et des apis. Pour chacune d'entre elles, nous fournissons une brève description, les recommandations de l'OWASP, puis les dispositions de BlitzPHP pour résoudre le problème.

<a name="owasp-top-10-2021"></a>
## OWASP Top 10 2021

<a name="a01-2021-controle-d-acces-brise"></a>
### A01:2021 Contrôle d'accès brisé

Le contrôle d'accès permet d'appliquer la politique de manière à ce que les utilisateurs ne puissent pas agir en dehors des autorisations qui leur ont été accordées. Les défaillances conduisent généralement à la divulgation d'informations non autorisées, à la modification ou à la destruction de toutes les données ou à l'exécution d'une fonctionnalité en dehors des limites de l'utilisateur.

Les vulnérabilités les plus courantes en matière de contrôle d'accès sont les suivantes:    

* Violation du principe du moindre privilège ou du refus par défaut, où l'accès ne devrait être accordé que pour des capacités, des rôles ou des utilisateurs particuliers, mais est disponible pour tout le monde.
* Contournement des contrôles d'accès par la modification de l'URL (altération des paramètres ou navigation forcée), de l'état interne de l'application ou de la page HTML, ou par l'utilisation d'un outil d'attaque modifiant les requêtes d'API.
* Permettre la visualisation ou la modification du compte de quelqu'un d'autre, en fournissant son identifiant unique (références directes d'objets non sécurisées).
* Accéder à l'API avec des contrôles d'accès manquants pour POST, PUT et DELETE.
* Élévation des privilèges. Agir en tant qu'utilisateur sans être connecté ou agir en tant qu'administrateur lorsque l'on est connecté en tant qu'utilisateur.
* Manipulation de métadonnées, comme la relecture ou l'altération d'un jeton de contrôle d'accès JSON Web Token (JWT), d'un cookie ou d'un champ caché manipulé pour élever les privilèges ou abuser de l'invalidation du JWT.
* Une mauvaise configuration de CORS permet l'accès à l'API à partir d'origines non autorisées/non fiables.
* Forcer la navigation vers des pages authentifiées en tant qu'utilisateur non authentifié ou vers des pages privilégiées en tant qu'utilisateur standard.

<a name="a01-recommandations-de-l-owasp"></a>
#### Recommandations de l'OWASP

Le contrôle d'accès n'est efficace que dans un code serveur de confiance ou une API sans serveur, où l'attaquant ne peut pas modifier le contrôle d'accès ou les métadonnées.

* Sauf pour les ressources publiques, refuser par défaut.
* Mettre en œuvre des mécanismes de contrôle d'accès une seule fois et les réutiliser dans l'ensemble de l'application, notamment en réduisant au minimum l'utilisation de CORS (Cross-Origin Resource Sharing).
* Les modèles de contrôle d'accès devraient imposer la propriété des enregistrements plutôt que d'accepter que l'utilisateur puisse créer, lire, mettre à jour ou supprimer n'importe quel enregistrement.
* Les modèles de domaine doivent respecter les exigences uniques en matière de limites d'activité de l'application.
* Désactiver la liste des dossiers du serveur web et s'assurer que les métadonnées des fichiers (par exemple, .git) et les fichiers de sauvegarde ne sont pas présents dans les racines web.
* Consigner les échecs du contrôle d'accès et alerter les administrateurs le cas échéant (par exemple, en cas d'échecs répétés).
* Limiter le taux d'accès à l'API et au contrôleur afin de minimiser les dommages causés par les outils d'attaque automatisés.
* Les identifiants de session avec état doivent être invalidés sur le serveur après la déconnexion. Les jetons JWT sans état devraient plutôt être de courte durée afin de minimiser la fenêtre d'opportunité pour un attaquant. Pour les JWT à durée de vie plus longue, il est fortement recommandé de suivre les normes OAuth pour révoquer l'accès.

<a name="a01-dispositions-de-blitzphp"></a>
#### Dispositions de BlitzPHP

* [Le dossier `public`](/docs/{version}/structure#le-dossier-public) est séparé du dossier de l'application
* Présence d'une bibliothèque de [validation](/docs/{version}/validation)
* Présence d'un middleware de [protection CSRF](/docs/{version}/csrf)
* Présence d'une bibliothèque de [session](/docs/{version}/session)
* Présence d'un Throttler pour la limite de taux
* Présence d'un middleware [Cross-Origin Resource Sharing (CORS)](/docs/{version}/cors)
* Présence de la fonction `logger()` pour la journalisation
* Mise à disposition d'un package officiel [d'authentification et d'autorisation](/docs/{version}/schild)

<a name="a02-2021-defaillances-cryptographiques"></a>
### A02:2021 Défaillances cryptographiques

La première chose à faire est de déterminer les besoins de protection des données en transit et au repos. Par exemple, les mots de passe, les numéros de carte de crédit, les dossiers médicaux, les informations personnelles et les secrets d'affaires nécessitent une protection supplémentaire, principalement si ces données relèvent des lois sur la protection de la vie privée, par exemple le règlement général sur la protection des données (RGPD) de l'UE, ou des réglementations, par exemple la protection des données financières telles que la norme de sécurité des données PCI (PCI DSS). Pour toutes ces données :

* Des données sont-elles transmises en texte clair ? Cela concerne les protocoles tels que HTTP, SMTP, FTP et les mises à jour TLS telles que STARTTLS. Le trafic internet externe est dangereux. Vérifiez tout le trafic interne, par exemple entre les équilibreurs de charge (load balancers), les serveurs web ou les systèmes dorsaux.
* Des algorithmes ou protocoles cryptographiques anciens ou faibles sont-ils utilisés par défaut ou dans un code plus ancien ?
* Des clés cryptographiques par défaut sont-elles utilisées, des clés cryptographiques faibles sont-elles générées ou réutilisées, ou une gestion ou une rotation appropriée des clés fait-elle défaut ? Les clés de chiffrement sont-elles vérifiées dans les référentiels de code source ?
* Le chiffrement n'est-il pas appliqué, par exemple, des directives de sécurité ou des en-têtes HTTP (navigateur) manquent-ils ?
* Le certificat de serveur reçu et la chaîne de confiance sont-ils correctement validés ?
* Les vecteurs d'initialisation sont-ils ignorés, réutilisés ou non générés de manière suffisamment sûre pour le mode de fonctionnement cryptographique ? Un mode de fonctionnement non sécurisé, tel que l'ECB, est-il utilisé ? Le chiffrement est-il utilisé alors qu'un chiffrement authentifié serait plus approprié ?
* Des mots de passe sont-ils utilisés comme clés cryptographiques en l'absence d'une fonction de dérivation de clé basée sur un mot de passe ?
* Utilise-t-on à des fins cryptographiques des données aléatoires qui n'ont pas été conçues pour répondre à des exigences cryptographiques ? Même si la fonction correcte est choisie, doit-elle être ensemencée par le développeur et, si ce n'est pas le cas, le développeur a-t-il remplacé la fonctionnalité d'ensemencement forte intégrée par une semence qui n'a pas une entropie/imprévisibilité suffisante ?
* Des fonctions de hachage obsolètes telles que MD5 ou SHA1 sont-elles utilisées, ou des fonctions de hachage non cryptographiques sont-elles utilisées lorsque des fonctions de hachage cryptographiques sont nécessaires ?
* Des méthodes de remplissage cryptographique obsolètes telles que PKCS number 1 v1.5 sont-elles utilisées ?
* Les messages d'erreur cryptographiques ou les informations des canaux latéraux sont-ils exploitables, par exemple sous la forme d'attaques par oracle de remplissage ?

<a name="a02-recommandations-de-l-owasp"></a>
#### Recommandations de l'OWASP

Effectuer au minimum les opérations suivantes et consulter les références :

* Classer les données traitées, stockées ou transmises par une application. Identifier les données sensibles en fonction des lois sur la protection de la vie privée, des exigences réglementaires ou des besoins de l'entreprise.
* Ne stockez pas inutilement des données sensibles. Jetez-les dès que possible ou utilisez la tokenisation conforme à la norme PCI DSS ou même la troncature. Les données qui ne sont pas conservées ne peuvent pas être volées.
* Veillez à crypter toutes les données sensibles au repos.
* Veillez à ce que des algorithmes, des protocoles et des clés standard solides et actualisés soient en place ; utilisez une gestion des clés appropriée.
* Chiffrer toutes les données en transit à l'aide de protocoles sécurisés tels que TLS avec des algorithmes de chiffrement FS (forward secrecy), la priorisation du chiffrement par le serveur et des paramètres sécurisés. Appliquer le chiffrement à l'aide de directives telles que HTTP Strict Transport Security (HSTS).
* Désactiver la mise en cache des réponses contenant des données sensibles.
* Appliquer les contrôles de sécurité requis en fonction de la classification des données.
* N'utilisez pas les protocoles traditionnels tels que FTP et SMTP pour le transport de données sensibles.
* Stocker les mots de passe en utilisant des fonctions de hachage adaptatives et salées fortes avec un facteur de travail (facteur de retard), telles que Argon2, scrypt, bcrypt ou PBKDF2.
* Les vecteurs d'initialisation doivent être choisis en fonction du mode de fonctionnement. Pour de nombreux modes, cela signifie l'utilisation d'un CSPRNG (générateur de nombres pseudo-aléatoires cryptographiquement sûr). Pour les modes qui nécessitent un nonce, le vecteur d'initialisation (IV) n'a pas besoin d'un CSPRNG. Dans tous les cas, le vecteur d'initialisation ne doit jamais être utilisé deux fois pour une clé fixe.
* Il faut toujours utiliser un chiffrement authentifié plutôt qu'un simple chiffrement.
* Les clés doivent être générées de manière cryptographique et aléatoire et stockées en mémoire sous forme de tableaux d'octets. Si un mot de passe est utilisé, il doit être converti en clé au moyen d'une fonction de dérivation de clé basée sur un mot de passe approprié.
* Veillez à ce que le caractère aléatoire de la cryptographie soit utilisé le cas échéant et qu'il n'ait pas été semé de manière prévisible ou avec une faible entropie. La plupart des API modernes n'obligent pas le développeur à ensemencer le CSPRNG pour obtenir la sécurité.
* Évitez les fonctions cryptographiques et les schémas de remplissage obsolètes, tels que MD5, SHA1, PKCS number 1 v1.5.
* Vérifiez de manière indépendante l'efficacité de la configuration et des paramètres.

<a name="a02-dispositions-de-blitzphp"></a>
#### Dispositions de BlitzPHP

* Présence d'une configuration pour l'accès sécurisé global (`app/config/app::$force_global_secure_requests`)
* Présence d'un service de [chiffrement de données](/docs/{version}/chiffrement)
* Présence de la fonction `force_https()`
* Présence d'une configuration pour chiffrer les base de données (`encrypt`)
* Mise à disposition d'un package officiel [d'authentification et d'autorisation](/docs/{version}/schild)

<a name="a03-2021-injection"></a>
### A03:2021 Injection

Une application est vulnérable aux attaques lorsque :

* Les données fournies par l'utilisateur ne sont pas validées, filtrées ou assainies par l'application.
* Les requêtes dynamiques ou les appels non paramétrés sans échappement contextuel sont utilisés directement dans l'interpréteur.
* Des données hostiles sont utilisées dans les paramètres de recherche de l'ORM pour extraire des enregistrements sensibles supplémentaires.
* Les données hostiles sont directement utilisées ou concaténées. Le code SQL ou la commande contient la structure et les données malveillantes dans les requêtes dynamiques, les commandes ou les procédures stockées.

Parmi les injections les plus courantes, citons les injections SQL, NoSQL, les commandes OS, les injections ORM (Object Relational Mapping), les injections LDAP et les injections EL (Expression Language) ou OGNL (Object Graph Navigation Library). Le concept est identique pour tous les interprètes. L'examen du code source est la meilleure méthode pour détecter si les applications sont vulnérables aux injections. Les tests automatisés de tous les paramètres, en-têtes, URL, cookies, JSON, SOAP et entrées de données XML sont fortement encouragés. Les organisations peuvent inclure des outils de test de sécurité des applications statiques (SAST), dynamiques (DAST) et interactives (IAST) dans le pipeline CI/CD afin d'identifier les failles d'injection introduites avant le déploiement de la production.

<a name="a03-recommandations-de-l-owasp"></a>
#### Recommandations de l'OWASP

Pour éviter les injections, il faut séparer les données des commandes et des requêtes :

* L'option préférée est d'utiliser une API sûre, qui évite entièrement l'utilisation de l'interpréteur, fournit une interface paramétrée ou migre vers des outils de mappage relationnel d'objets (ORM).
    - Remarque : même lorsqu'elles sont paramétrées, les procédures stockées peuvent toujours introduire une injection SQL si PL/SQL ou T-SQL concatène des requêtes et des données ou exécute des données hostiles avec EXECUTE IMMEDIATE ou exec().
* Utiliser une validation positive des entrées côté serveur. Il ne s'agit pas d'une défense complète, car de nombreuses applications nécessitent des caractères spéciaux, comme les zones de texte ou les API pour les applications mobiles.
* Pour toute requête dynamique résiduelle, échappez les caractères spéciaux en utilisant la syntaxe d'échappement spécifique à cet interpréteur.
    - Remarque : les structures SQL telles que les noms de tables, de colonnes, etc. ne peuvent pas être échappées, et les noms de structures fournis par l'utilisateur sont donc dangereux. Il s'agit d'un problème courant dans les logiciels de rédaction de rapports.
* Utilisez LIMIT et d'autres contrôles SQL dans les requêtes pour empêcher la divulgation massive d'enregistrements en cas d'injection SQL.

<a name="a03-dispositions-de-blitzphp"></a>
#### Dispositions de BlitzPHP

* Sécurité des URI
* Middleware InvalidChars
* Bibliothèque de [validation](/docs/{version}/validation)
* Fonction esc()
* La bibliothèque [HTTP](/docs/{version}/requetes) permet de filtrer les champs d'entrée
* Prise en charge de la politique de sécurité du contenu
* Classe Query Builder
* Méthodes d'échappement de la base de données
* Liaisons de requêtes                                                                                       

