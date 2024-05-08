# Aide-mémoire pour la prévention du bourrage d'informations d'identification

## Introduction

Cette aide-mémoire couvre les défenses contre deux types courants d'attaques liées à l'authentification : le credential stuffing et la pulvérisation de mots de passe. Bien qu’il s’agisse d’attaques distinctes, dans de nombreux cas, les défenses qui seraient mises en œuvre pour s’en protéger sont les mêmes, et elles seraient également efficaces pour se protéger contre les attaques par force brute. Un résumé de ces différentes attaques est listé ci-dessous :

| Type d'attaque                           | Description                                                                                          |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Brute Force                              | Test de plusieurs mots de passe à partir d'un dictionnaire ou d'une autre source sur un seul compte. |
| Bourrage d'informations d'identification | Test des paires nom d'utilisateur/mot de passe obtenues lors de la violation d'un autre site.        |
| Pulvérisation de mot de passe            | Tester un seul mot de passe faible sur un grand nombre de comptes différents.                        |

## Authentification multifacteur

[Authentification multifacteur (MFA)](Multifactor_Authentication_Cheat_Sheet.md)est de loin la meilleure défense contre la majorité des attaques liées aux mots de passe, y compris le credential stuffing et la pulvérisation de mots de passe, une analyse de Microsoft suggérant qu'elle aurait arrêté[99,9 % des comptes compromis](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984). En tant que tel, il devrait être mis en œuvre dans la mesure du possible. Historiquement, en fonction du public de l'application, il n'était peut-être pas pratique ou réalisable d'imposer l'utilisation de l'AMF. Cependant, avec les navigateurs et les appareils mobiles modernes prenant désormais en charge les clés d'accès FIDO2 et d'autres formes d'AMF, cela est réalisable dans la plupart des cas d'utilisation.

Afin d'équilibrer sécurité et convivialité, l'authentification multifacteur peut être combinée avec d'autres techniques pour exiger le deuxième facteur uniquement dans des circonstances spécifiques où il y a des raisons de soupçonner que la tentative de connexion n'est peut-être pas légitime, comme une connexion depuis :

-   Un nouveau navigateur/appareil ou adresse IP.
-   Un pays ou un lieu inhabituel.
-   Pays spécifiques considérés comme non fiables ou ne contenant généralement pas d'utilisateurs d'un service.
-   Adresse IP qui apparaît sur des listes de refus connues ou qui est associée à des services d'anonymisation, tels que des services proxy ou VPN.
-   Une adresse IP qui a tenté de se connecter à plusieurs comptes.
-   Une tentative de connexion qui semble être scriptée ou provenir d'un robot plutôt que d'un humain (c'est-à-dire un volume de connexion important provenant d'une seule adresse IP ou d'un seul sous-réseau).

Une organisation peut également choisir d'exiger l'authentification MFA sous la forme d'une authentification « renforcée » pour les scénarios ci-dessus au cours d'une session combinée à une demande d'activité à haut risque telle que :

-   Opérations en devises importantes
-   Modifications de la configuration privilégiée ou administrative

De plus, pour les applications d'entreprise, des plages d'adresses IP fiables connues peuvent être ajoutées à une liste verte afin que l'authentification MFA ne soit pas requise lorsque les utilisateurs se connectent à partir de ces plages.

## Défenses alternatives

Lorsqu’il n’est pas possible de mettre en œuvre la MFA, il existe de nombreuses défenses alternatives qui peuvent être utilisées pour se protéger contre le credential stuffing et la pulvérisation de mots de passe. Pris isolément, aucun d'entre eux n'est aussi efficace que l'AMF, mais des défenses multiples et superposées peuvent fournir un degré de protection raisonnable. Dans de nombreux cas, ces mécanismes protègent également contre les attaques par force brute ou par pulvérisation de mots de passe.

Lorsqu'une application dispose de plusieurs rôles d'utilisateur, il peut être approprié de mettre en œuvre différentes défenses pour différents rôles. Par exemple, il n’est peut-être pas possible d’appliquer l’authentification MFA à tous les utilisateurs, mais il devrait être possible d’exiger que tous les administrateurs l’utilisent.

## Défense en profondeur et mesures

Bien qu’il ne s’agisse pas d’une technique spécifique, il est important de mettre en œuvre des défenses qui prennent en compte l’impact de la défaite ou de l’échec des défenses individuelles. À titre d'exemple, les défenses côté client, telles que les empreintes digitales des appareils ou les défis JavaScript, peuvent être usurpées ou contournées et d'autres couches de défense doivent être mises en œuvre pour en tenir compte.

De plus, chaque défense doit générer des mesures de volume à utiliser comme mécanisme de détection. Idéalement, les mesures incluront à la fois le volume d'attaques détectées et atténuées et permettront de filtrer des champs tels que l'adresse IP. La surveillance et la création de rapports sur ces mesures peuvent identifier les défaillances de la défense ou la présence d'attaques non identifiées, ainsi que l'impact des défenses nouvelles ou améliorées.

Enfin, lorsque l'administration de différentes défenses est effectuée par plusieurs équipes, il convient de veiller à assurer une communication et une coordination lorsque des équipes distinctes effectuent la maintenance, le déploiement ou modifient de toute autre manière des défenses individuelles.

### Mots de passe secondaires, codes PIN et questions de sécurité

En plus d'exiger qu'un utilisateur saisisse son mot de passe lors de son authentification, les utilisateurs peuvent également être invités à fournir des informations de sécurité supplémentaires telles que :

-   Un code PIN
-   Caractères spécifiques d'un mot de passe secondaire ou d'un mot mémorable
-   Réponses à[questions de sécurité](Choosing_and_Using_Security_Questions_Cheat_Sheet.md)

Il faut souligner que cela**ne fait pas**constituent une authentification multifacteur (car les deux facteurs sont identiques - quelque chose que vous savez). Cependant, il peut toujours fournir une couche de protection utile contre le credential stuffing et la pulvérisation de mots de passe là où une MFA appropriée ne peut pas être mise en œuvre.

### CAPTCHA

Exiger d'un utilisateur qu'il résolve un « test de Turing public entièrement automatisé pour distinguer les ordinateurs des humains » (CAPTCHA) ou un casse-tête similaire pour chaque tentative de connexion peut aider à identifier les attaques automatisées/bots et à empêcher les tentatives de connexion automatisées, et peut ralentir le bourrage d'informations d'identification. ou des attaques par pulvérisation de mots de passe. Cependant, les CAPTCHA ne sont pas parfaits et, dans de nombreux cas, il existe des outils ou des services qui peuvent être utilisés pour les briser avec un taux de réussite raisonnablement élevé. La surveillance des taux de résolution de CAPTCHA peut aider à identifier l'impact sur les bons utilisateurs, ainsi que la technologie automatisée de rupture de CAPTCHA, éventuellement indiquée par des taux de résolution anormalement élevés.

Pour améliorer la convivialité, il peut être souhaitable de demander à l'utilisateur de résoudre un CAPTCHA uniquement lorsque la demande de connexion est considérée comme suspecte ou à haut risque, en utilisant les mêmes critères abordés dans la section MFA.

### Atténuation et renseignement sur la propriété intellectuelle

Le blocage des adresses IP peut suffire à stopper les attaques moins sophistiquées, mais ne doit pas être utilisé comme moyen de défense principal en raison de la facilité de contournement. Il est plus efficace d’avoir une réponse graduée aux abus qui s’appuie sur plusieurs mesures défensives en fonction des différents facteurs de l’attaque.

Tout processus ou décision visant à atténuer (y compris le blocage et le CAPTCHA) le trafic de bourrage d'informations d'identification à partir d'une adresse IP doit prendre en compte une multitude de scénarios d'abus et ne pas s'appuyer sur une seule limite de volume prévisible. Des périodes courtes (c'est-à-dire en rafale) et longues doivent être prises en compte, ainsi qu'un volume de requêtes élevé et des cas où une adresse IP, probablement de concert avec_beaucoup_d'autres adresses IP, génère des volumes de trafic faibles mais constants. De plus, les décisions d’atténuation doivent tenir compte de facteurs tels que la classification des adresses IP (par exemple : résidentielle ou hébergement) et la géolocalisation. Ces facteurs peuvent être exploités pour augmenter ou diminuer les seuils d’atténuation afin de réduire l’impact potentiel sur les utilisateurs légitimes ou d’atténuer de manière plus agressive les abus provenant de sources anormales. Les mesures d'atténuation, en particulier le blocage d'une adresse IP, doivent être temporaires et des processus doivent être en place pour supprimer une adresse IP d'un état atténué à mesure que les abus diminuent ou cessent.

De nombreuses boîtes à outils de credential stuffing, telles que[MBA Sentinelle](https://federalnewsnetwork.com/wp-content/uploads/2020/06/Shape-Threat-Research-Automating-Cybercrime-with-SentryMBA.pdf), proposent une utilisation intégrée de réseaux proxy pour distribuer les requêtes sur un grand volume d'adresses IP uniques. Cela peut mettre en échec à la fois les listes de blocage IP et la limitation de débit, car le volume de requêtes IP peut rester relativement faible, même en cas d'attaques à volume élevé. La corrélation du trafic d'authentification avec les informations de proxy et d'adresses IP similaires, ainsi que les plages d'adresses IP des fournisseurs d'hébergement, peuvent aider à identifier les attaques de credential stuffing hautement distribuées et servir de déclencheur d'atténuation. Par exemple, chaque requête provenant d'un fournisseur d'hébergement pourrait être obligée de résoudre le CAPTCHA.

Il existe des sources publiques et commerciales de renseignements et de classification des adresses IP qui peuvent être exploitées comme sources de données à cette fin. De plus, certains fournisseurs d'hébergement publient leur propre espace d'adressage IP, comme[AWS](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html).

Outre le blocage des connexions réseau, envisagez de stocker l’historique d’authentification de l’adresse IP d’un compte. Dans le cas où une adresse IP récente est ajoutée à une liste de blocage ou d'atténuation, il peut être approprié de verrouiller le compte et d'en informer l'utilisateur.

### Empreinte digitale de l'appareil

Outre l’adresse IP, un certain nombre de facteurs différents peuvent être utilisés pour tenter de prendre l’empreinte digitale d’un appareil. Certains d'entre eux peuvent être obtenus passivement par le serveur à partir des entêtes HTTP (notamment l'entête "User-Agent"), notamment :

-   Système d'exploitation et version
-   Navigateur et version
-   Langue

En utilisant JavaScript, il est possible d'accéder à beaucoup plus d'informations, telles que :

-   Résolution d'écran
-   Polices installées
-   Plugins de navigateur installés

A l’aide de ces différents attributs, il est possible de créer une empreinte digitale de l’appareil. Cette empreinte digitale peut ensuite être comparée à n'importe quel navigateur tentant de se connecter au compte, et si elle ne correspond pas, l'utilisateur peut être invité à une authentification supplémentaire. De nombreux utilisateurs utilisent plusieurs appareils ou navigateurs. Il n'est donc pas pratique de simplement bloquer les tentatives qui ne correspondent pas aux empreintes digitales existantes. Cependant, il est courant de définir un processus permettant aux utilisateurs ou aux clients d'afficher l'historique de leur appareil et de gérer les informations mémorisées. dispositifs. Ces attributs peuvent également être utilisés pour détecter une activité anormale, telle qu'un appareil semblant exécuter une ancienne version du système d'exploitation ou du navigateur.

Le[empreinte digitalejs2](https://github.com/Valve/fingerprintjs2)La bibliothèque JavaScript peut être utilisée pour effectuer des empreintes digitales côté client.

Il convient de noter que toutes ces informations étant fournies par le client, elles peuvent potentiellement être usurpées par un attaquant. Dans certains cas, l'usurpation de ces attributs est triviale (comme l'en-tête "User-Agent"), mais dans d'autres cas, il peut être plus difficile de modifier ces attributs.

### Empreinte digitale de connexion

À l’instar de la prise d’empreintes digitales des appareils, il existe de nombreuses techniques de prise d’empreintes digitales disponibles pour les connexions réseau. Quelques exemples incluent[Faim](https://github.com/salesforce/ja3), empreintes HTTP/2 et ordre des en-têtes HTTP. Comme ces techniques se concentrent généralement sur la manière dont une connexion est établie, les empreintes digitales de connexion peuvent fournir des résultats plus précis que d'autres défenses qui s'appuient sur un indicateur, tel qu'une adresse IP, ou sur des données de demande, telles qu'une chaîne d'agent utilisateur.

Les empreintes digitales de connexion peuvent également être utilisées conjointement avec d’autres moyens de défense pour vérifier la véracité d’une demande d’authentification. Par exemple, si l’en-tête de l’agent utilisateur et l’empreinte digitale de l’appareil indiquent un appareil mobile, mais que l’empreinte digitale de connexion indique un script Python, la requête est probablement suspecte.

### Exiger des noms d'utilisateur imprévisibles

Les attaques de credential stuffing reposent non seulement sur la réutilisation de mots de passe entre plusieurs sites, mais également sur la réutilisation de noms d’utilisateur. Un nombre important de sites Web utilisent l'adresse e-mail comme nom d'utilisateur, et comme la plupart des utilisateurs ont une seule adresse e-mail qu'ils utilisent pour tous leurs comptes, cela rend la combinaison d'une adresse e-mail et d'un mot de passe très efficace pour les attaques de credential stuffing.

Exiger des utilisateurs qu'ils créent leur propre nom d'utilisateur lors de leur inscription sur le site Web rend plus difficile pour un attaquant d'obtenir des paires de nom d'utilisateur et de mot de passe valides pour le bourrage d'informations d'identification, car la plupart des listes d'informations d'identification disponibles n'incluent que des adresses e-mail. Fournir à l'utilisateur un nom d'utilisateur généré peut offrir un degré de protection plus élevé (car les utilisateurs sont susceptibles de choisir le même nom d'utilisateur sur la plupart des sites Web), mais cela n'est pas convivial. De plus, il faut veiller à ce que le nom d'utilisateur généré ne soit pas prévisible (par exemple, s'il est basé sur le nom complet de l'utilisateur ou sur des identifiants numériques séquentiels), car cela pourrait faciliter l'énumération des noms d'utilisateur valides pour une attaque par pulvérisation de mot de passe.

### Processus de connexion en plusieurs étapes

La majorité des outils disponibles dans le commerce sont conçus pour un processus de connexion en une seule étape, dans lequel les informations d'identification sont envoyées au serveur et la réponse indique si la tentative de connexion a réussi ou non. En ajoutant des étapes supplémentaires à ce processus, comme exiger que le nom d'utilisateur et le mot de passe soient saisis de manière séquentielle, ou exiger que l'utilisateur obtienne d'abord un code aléatoire.[Jeton CSRF](Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md)Avant de pouvoir se connecter, cela rend l'attaque légèrement plus difficile à réaliser et double le nombre de requêtes que l'attaquant doit effectuer.

Il convient toutefois de garder à l'esprit que les processus de connexion en plusieurs étapes ne facilitent pas[énumération des utilisateurs](Authentication_Cheat_Sheet.md). L’énumération des utilisateurs avant une attaque de credential stuffing peut entraîner une attaque plus difficile à identifier et avec un volume de requêtes inférieur.

### Exiger JavaScript et bloquer les navigateurs sans tête

La plupart des outils utilisés pour ces types d'attaques effectueront des requêtes POST directes au serveur et liront les réponses, mais ne téléchargeront ni n'exécuteront le JavaScript qu'ils contiennent. En obligeant l'attaquant à évaluer JavaScript dans la réponse (par exemple pour générer un jeton valide qui doit être soumis avec la requête), cela oblige l'attaquant soit à utiliser un vrai navigateur avec un framework d'automatisation comme Selenium ou Headless Chrome, soit à implémenter Analyse JavaScript avec un autre outil tel que PhantomJS. De plus, il existe un certain nombre de techniques qui peuvent être utilisées pour identifier[Chrome sans tête](https://antoinevastel.com/bot%20detection/2018/01/17/detect-chrome-headless-v2.html)ou[PhantomJS](https://blog.shapesecurity.com/2015/01/22/detecting-phantomjs-based-visitors/).

Veuillez noter que le blocage des visiteurs dont JavaScript est désactivé réduira l'accessibilité du site Web, en particulier pour les visiteurs qui utilisent des lecteurs d'écran. Dans certaines juridictions, cela peut constituer une violation de la législation sur l’égalité.

### Dégradation

Une défense plus agressive contre le credential stuffing consiste à mettre en œuvre des mesures qui augmentent le temps nécessaire à l’attaque. Cela peut inclure une augmentation progressive de la complexité du code JavaScript qui doit être évalué, l'introduction de longues périodes d'attente avant de répondre aux demandes, le renvoi d'actifs HTML trop volumineux ou le renvoi de messages d'erreur aléatoires.

En raison de leur impact négatif potentiel sur les utilisateurs légitimes, ce type de défense doit être très prudent, même si cela peut être nécessaire pour atténuer les attaques de credential stuffing plus sophistiquées.

### Identifier les mots de passe divulgués

[Exigences de sécurité du mot de passe ASVS v4.0](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x11-V2-Authentication.md#v21-password-security-requirements)la disposition (2.1.7) sur la vérification de la présence de nouveaux mots de passe dans les ensembles de données de mots de passe violés devrait être mise en œuvre.

Il existe des services commerciaux et gratuits qui peuvent être utiles pour valider la présence de mots de passe lors de violations antérieures. Un service gratuit bien connu pour cela est[Mots de passe activés](https://haveibeenpwned.com/Passwords). Vous pouvez héberger vous-même une copie de l'application ou utiliser le[API](https://haveibeenpwned.com/API/v2#PwnedPasswords).

### Informer les utilisateurs des événements de sécurité inhabituels

Lorsqu'une activité suspecte ou inhabituelle est détectée, il peut être approprié d'avertir ou d'avertir l'utilisateur. Cependant, il faut veiller à ce que l'utilisateur ne soit pas submergé par un grand nombre de notifications qui ne sont pas importantes pour lui, sinon il commencera simplement à les ignorer ou à les supprimer. De plus, en raison de la réutilisation fréquente des mots de passe sur plusieurs sites, il convient d'envisager la possibilité que le compte de messagerie de l'utilisateur ait également été compromis.

Par exemple, il ne serait généralement pas approprié d'informer un utilisateur qu'il y a eu une tentative de connexion à son compte avec un mot de passe incorrect. Cependant, s'il y a eu une connexion avec le mot de passe correct, mais qui a ensuite échoué au contrôle MFA ultérieur, l'utilisateur doit être averti afin qu'il puisse modifier son mot de passe. Par la suite, si l'utilisateur demande plusieurs réinitialisations de mot de passe à partir de différents appareils ou adresses IP, il peut être approprié d'empêcher tout accès ultérieur au compte en attendant des processus de vérification supplémentaires de l'utilisateur.

Les détails liés aux connexions actuelles ou récentes doivent également être rendus visibles à l'utilisateur. Par exemple, lorsqu'ils se connectent à l'application, la date, l'heure et le lieu de leur précédente tentative de connexion pourraient leur être affichés. De plus, si l'application prend en charge les sessions simultanées, l'utilisateur doit pouvoir afficher une liste de toutes les sessions actives et mettre fin à toute autre session non légitime.

## Les références

-   [Article sur le bourrage d'informations d'identification de l'OWASP](https://owasp.org/www-community/attacks/Credential_stuffing)
-   [Menaces automatisées OWASP contre les applications Web](https://owasp.org/www-project-automated-threats-to-web-applications/)
-   Projet:[OAT-008 Bourrage d'informations d'identification](https://owasp.org/www-community/attacks/Credential_stuffing), qui est l'une des 20 menaces définies dans le[Manuel des menaces automatisées de l'OWASP](https://owasp.org/www-pdf-archive/Automated-threat-handbook.pdf)ce projet a produit.
