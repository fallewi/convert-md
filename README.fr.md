# Aide-mémoire pour la gestion des secrets

## 1. Introduction

Les secrets sont utilisés partout de nos jours, notamment avec la popularité du mouvement DevOps. Clés d'interface de programmation d'application (API), informations d'identification de base de données, autorisations de gestion des identités et des accès (IAM), clés Secure Shell (SSH), certificats, etc. De nombreuses organisations les ont codées en dur dans le code source en texte brut, éparpillées dans les fichiers de configuration et la gestion de la configuration. outils.

Les organisations ont de plus en plus besoin de centraliser le stockage, l’approvisionnement, l’audit, la rotation et la gestion des secrets pour contrôler l’accès aux secrets et empêcher qu’ils ne soient divulgués et compromettent l’organisation. Souvent, les services partagent les mêmes secrets, ce qui rend difficile l’identification de la source de compromission ou de fuite.

Cette aide-mémoire propose les meilleures pratiques et lignes directrices pour vous aider à mettre en œuvre correctement la gestion des secrets.

## 2 Gestion des secrets généraux

Les sections suivantes abordent les principaux concepts liés à la gestion des secrets.

### 2.1 Haute disponibilité

Il est essentiel de sélectionner une technologie suffisamment robuste pour gérer le trafic de manière fiable :

-   Utilisateurs (par exemple, clés SSH, mots de passe du compte root). Dans un scénario de réponse aux incidents, les utilisateurs s'attendent à recevoir rapidement des informations d'identification, afin de pouvoir récupérer les services qui ont été mis hors ligne. Devoir attendre les informations d'identification pourrait avoir un impact sur la réactivité de l'équipe opérationnelle.
-   Applications (par exemple, informations d'identification de base de données et clés API). Si le service n'est pas performant, cela pourrait dégrader la disponibilité des applications dépendantes ou augmenter les temps de démarrage des applications.

Un tel service pourrait recevoir un volume considérable de demandes au sein d’une grande organisation.

### 2.2 Centraliser et standardiser

Les secrets utilisés par vos équipes DevOps pour vos applications peuvent être consommés différemment des secrets stockés par vos spécialistes marketing ou votre équipe SRE. Vous trouvez souvent des secrets mal entretenus où les besoins des consommateurs ou des producteurs secrets ne correspondent pas. Vous devez donc standardiser et centraliser la solution de gestion des secrets avec soin. La standardisation et la centralisation peuvent signifier que vous utilisez plusieurs solutions de gestion des secrets. Par exemple : vos équipes de développement cloud natives choisissent d'utiliser la solution fournie par le fournisseur de cloud, tandis que votre cloud privé utilise une solution tierce et que tout le monde dispose d'un compte pour un gestionnaire de mots de passe sélectionné.
En veillant à ce que les équipes standardisent l’interaction avec ces différentes solutions, celles-ci restent maintenables et utilisables en cas d’incident.
Même lorsqu'une entreprise centralise la gestion de ses secrets sur une seule solution, vous devrez souvent sécuriser le secret principal de cette solution de gestion des secrets dans une solution de gestion des secrets secondaire. Par exemple, vous pouvez utiliser les installations d'un fournisseur de cloud pour stocker des secrets, mais les informations d'identification racine/gestion de ce fournisseur de cloud doivent être stockées ailleurs.

La normalisation doit inclure la gestion du cycle de vie des secrets, l'authentification, l'autorisation et la comptabilité de la solution de gestion des secrets, ainsi que la gestion du cycle de vie. Notez qu’une organisation doit savoir immédiatement à quoi sert un secret et où le trouver. Plus vous utilisez de solutions de gestion des secrets, plus vous avez besoin de documentation.

### 2.3 Contrôle d'accès

Lorsque les utilisateurs peuvent lire le secret dans un système de gestion des secrets et/ou le mettre à jour, cela signifie que le secret peut désormais fuir via cet utilisateur et le système qu'il a utilisé pour toucher le secret.
Par conséquent, les ingénieurs ne devraient pas avoir accès à tous les secrets du système de gestion des secrets et le principe du moindre privilège devrait être appliqué. Le système de gestion des secrets doit offrir la possibilité de configurer des contrôles d'accès granulaires sur chaque objet et composant pour mettre en œuvre le principe du moindre privilège.

### 2.4 Automatiser la gestion des secrets

La maintenance manuelle n’augmente pas seulement le risque de fuite ; cela introduit le risque d’erreurs humaines tout en préservant le secret. De plus, cela peut devenir du gaspillage.
Par conséquent, il est préférable de limiter ou de supprimer l’interaction humaine avec les secrets réels. Vous pouvez restreindre les interactions humaines de plusieurs manières :

-   **Pipeline de secrets :**Disposer d'un pipeline de secrets qui effectue une grande partie de la gestion des secrets (par exemple, création, rotation, etc.)
-   **Utilisation de secrets dynamiques :**Lorsqu'une application démarre, elle peut demander ses informations d'identification de base de données, qui, une fois générées dynamiquement, recevront de nouvelles informations d'identification pour cette session. Les secrets dynamiques doivent être utilisés dans la mesure du possible pour réduire la surface de réutilisation des informations d'identification. Si les informations d'identification de la base de données de l'application étaient volées, elles expireraient au redémarrage.
-   **Rotation automatisée des secrets statiques :**La rotation des clés est un processus difficile lorsqu'elle est mise en œuvre manuellement et peut conduire à des erreurs. Mieux vaut donc automatiser la rotation des clés ou au moins s'assurer que le processus est suffisamment pris en charge par l'informatique.

La rotation de certaines clés, telles que les clés de chiffrement, peut déclencher un rechiffrement total ou partiel des données. Différentes stratégies de rotation des clés existent :

-   Rotation progressive
-   Présentation de nouvelles clés pour les opérations d'écriture
-   Laisser les anciennes clés pour les opérations de lecture
-   Rotation rapide
-   Rotation programmée
-   et plus...

### 2.5 Gestion des secrets en mémoire

Un niveau de sécurité supplémentaire peut être obtenu en minimisant la fenêtre de temps
où un secret est en mémoire et limiter l'accès à son espace mémoire.

Selon les circonstances particulières de votre demande, cela peut être difficile
à mettre en œuvre de manière à garantir la sécurité de la mémoire. En raison de ce potentiel
complexité de la mise en œuvre, vous êtes d'abord encouragé à développer un modèle de menace afin de clairement
faites également apparaître vos hypothèses implicites sur l'environnement de déploiement de votre application
comme comprendre les capacités de vos adversaires.

Souvent, tenter de protéger les secrets en mémoire sera considéré comme excessif.
car lorsque vous évaluez un modèle de menace, la menace potentielle
des acteurs que vous considérez soit comme n’ayant pas les capacités nécessaires pour mener de telles attaques
ou le coût de la défense dépasse de loin l'impact probable d'un compromis résultant de
exposer les secrets de la mémoire. Il convient également de garder cela à l'esprit lors de l'élaboration d'un
modèle de menace approprié, que si un attaquant a déjà accès à la mémoire de
le processus traitant le secret, à ce moment-là, une faille de sécurité peut déjà avoir été
s'est produit. En outre, il convient de reconnaître qu'avec l'avènement d'attaques comme[Marteau à rames](https://arxiv.org/pdf/2211.07613.pdf), ou[Fusion et Spectre](https://meltdownattack.com/), C'est important
comprendre que le système d’exploitation seul ne suffit pas à protéger votre processus
mémoire de ce type d’attaques. Cela devient particulièrement important lorsque votre
l'application est déployée sur le cloud. La seule approche infaillible pour protéger la mémoire
contre ces attaques et des attaques similaires pour isoler complètement physiquement la mémoire de votre processus de toutes les autres
processus non fiables.

Malgré les difficultés de mise en œuvre, dans des régions très sensibles
environnements, la protection des secrets en mémoire peut
être une couche de sécurité supplémentaire précieuse. Par exemple, dans les scénarios où un
un attaquant avancé peut provoquer le crash d'un système et accéder à un vidage mémoire,
ils pourront peut-être en extraire des secrets. Par conséquent, en protégeant soigneusement
les secrets en mémoire sont recommandés pour les environnements non fiables ou les situations où
une sécurité renforcée est de la plus haute importance.

De plus, dans les langages de niveau inférieur comme C/C++, il est relativement facile de protéger
secrets en mémoire. Il peut donc être intéressant de mettre en œuvre cette pratique même si
le risque qu'un attaquant accède à la mémoire est faible. En revanche, pour
langages de programmation qui s'appuient sur le garbage collection, sécurisant les secrets en mémoire
en général, c'est beaucoup plus difficile.

-   **Structures et classes :**Dans .NET et Java, n'utilisez pas de structures immuables
    comme Strings pour stocker des secrets, car il est impossible de les forcer à
    être ramassés. Utilisez plutôt des types primitifs tels que des tableaux d'octets ou
    tableaux de caractères, où la mémoire peut être directement écrasée. Vous pouvez aussi
    utiliser Java[Chaîne gardée](https://docs.oracle.com/html/E28160_01/org/identityconnectors/common/security/GuardedString.html)ou .NET[ChaîneSécurisée](https://learn.microsoft.com/en-us/dotnet/api/system.security.securestring#string-versus-securestring)qui sont conçus pour résoudre précisément ce problème.

-   **Remise à zéro de la mémoire :**Après qu'un secret a été utilisé, la mémoire qu'il occupe
    doit être remis à zéro pour éviter qu'il ne reste en mémoire là où il pourrait
    potentiellement accessible.
    -   Si vous utilisez GuardedString de Java, appelez le`dispose()`méthode.
    -   Si vous utilisez SecureString de .NET, appelez le`Dispose()`méthode.

-   **Cryptage de la mémoire :**Dans certains cas, il peut être possible d'utiliser du matériel ou
    fonctionnalités du système d'exploitation pour chiffrer tout l'espace mémoire du processus
    gérer le secret. Cela peut fournir une couche de sécurité supplémentaire. Pour
    Par exemple, GuardedString en Java chiffre les valeurs en mémoire et SecureString
    dans .NET le fait sous Windows.

N'oubliez pas que l'objectif est de minimiser la fenêtre de temps pendant laquelle se trouve le secret.
texte en clair en mémoire autant que possible.

Pour des informations plus détaillées, voir[Test de la mémoire pour les données sensibles](https://mas.owasp.org/MASTG/tests/android/MASVS-STORAGE/MASTG-TEST-0011)du projet OWASP MAS.

### 2.6 Audit

L'audit est un élément essentiel de la gestion des secrets en raison de la nature de l'application. Vous devez mettre en œuvre l’audit de manière sécurisée pour résister aux tentatives de falsification ou de suppression des journaux d’audit. Au minimum, vous devez auditer les éléments suivants :

-   Qui a demandé un secret et pour quel système et rôle.
-   Si la demande secrète a été approuvée ou rejetée.
-   Quand le secret a été utilisé et par qui/quoi.
-   Quand le secret est expiré.
-   S'il y a eu des tentatives de réutilisation de secrets expirés.
-   S'il y a eu des erreurs d'authentification ou d'autorisation.
-   Quand le secret a été mis à jour et par qui/quoi.
-   Toutes les actions administratives et activités possibles des utilisateurs sur la pile d'infrastructure de support sous-jacente.

Il est essentiel que tous les audits aient des horodatages corrects. Par conséquent, la solution de gestion des secrets doit disposer de protocoles de synchronisation temporelle appropriés mis en place au niveau de son infrastructure de support. Vous devez surveiller la pile sur laquelle la solution s'exécute pour détecter d'éventuels décalages d'horloge et ajustements manuels de l'heure.

### 2.7 Cycle de vie secret

Les secrets suivent un cycle de vie. Les étapes du cycle de vie sont les suivantes :

-   Création
-   Rotation
-   Révocation
-   Expiration

#### 2.7.1 Création

Les nouveaux secrets doivent être générés de manière sécurisée et suffisamment robustes sur le plan cryptographique pour leur objectif. Les secrets doivent disposer des privilèges minimum qui leur sont attribués pour permettre leur utilisation/rôle requis.

Vous devez transmettre les informations d'identification de manière sécurisée, de sorte qu'idéalement, vous n'envoyiez pas le mot de passe avec le nom d'utilisateur lorsque vous demandez des comptes d'utilisateurs. Au lieu de cela, vous devez envoyer le mot de passe via un canal sécurisé (par exemple, une connexion mutuellement authentifiée) ou un canal secondaire tel qu'une notification push, un SMS ou un e-mail. Se référer au[Aide-mémoire pour l'authentification multifacteur](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet)pour connaître les avantages et les inconvénients de chaque canal.

Les applications peuvent ne pas bénéficier de plusieurs canaux de communication, vous devez donc fournir les informations d'identification en toute sécurité.

Voir[le projet Open CRE sur la recherche de secrets](https://www.opencre.org/cre/223-780)pour des recommandations plus techniques sur la création de secrets.

#### 2.7.2 Rotation

Vous devez régulièrement alterner les secrets afin que les informations d'identification volées ne fonctionnent que pendant une courte période. Une rotation régulière réduira également la tendance des utilisateurs à retomber dans de mauvaises habitudes telles que la réutilisation des informations d'identification.

En fonction de la fonction d'un secret et de ce qu'il protège, la durée de vie peut aller de quelques minutes (pensez aux discussions cryptées de bout en bout avec une confidentialité parfaite) à des années (pensez aux secrets matériels).

Les informations d’identification des utilisateurs sont exclues de la rotation régulière. Ceux-ci ne devraient être alternés que s'il existe des soupçons ou des preuves qu'ils ont été compromis, selon[Recommandations du NIST](https://pages.nist.gov/800-63-FAQ/#q-b05).

#### 2.7.3 Révocation

Lorsque les secrets ne sont plus nécessaires ou potentiellement compromis, vous devez les révoquer en toute sécurité pour restreindre l'accès. Avec les certificats (TLS), cela implique également la révocation du certificat.

#### 2.7.4 Expiration

Dans la mesure du possible, vous devez créer des secrets qui expireront après une période définie. Cette expiration peut être soit une expiration active par le système consommateur de secrets, soit une date d'expiration fixée au niveau du système de gestion des secrets forçant le déclenchement des processus de support, entraînant une rotation des secrets.
Vous devez appliquer des politiques via la solution de gestion des secrets pour garantir que les informations d'identification ne sont disponibles que pendant une durée limitée, adaptée au type d'informations d'identification. Les applications doivent vérifier que le secret est toujours actif avant de lui faire confiance.

### 2.8 Sécurité de la couche transport (TLS) partout

Ne transmettez jamais de secrets en clair. De nos jours, il n’y a aucune excuse étant donné l’adoption omniprésente de TLS.

De plus, vous pouvez utiliser efficacement des solutions de gestion des secrets pour fournir des certificats TLS.

### 2.9 Temps d'arrêt, bris de vitre, sauvegarde et restauration

Considérez la possibilité qu'un service de gestion des secrets devienne indisponible pour diverses raisons, telles qu'une interruption programmée pour la maintenance. Il pourrait être impossible de récupérer les informations d'identification requises pour restaurer le service si vous ne les avez pas acquises au préalable. Ainsi, choisissez soigneusement les fenêtres de maintenance en fonction des métriques antérieures et des journaux d’audit.

Ensuite, les procédures de sauvegarde et de restauration du système doivent être régulièrement testées et auditées pour leur sécurité. Quelques exigences concernant la sauvegarde et la restauration. Veiller à ce que:

-   Une procédure de sauvegarde automatisée est en place et exécutée périodiquement ; basez la fréquence des sauvegardes et des instantanés sur le nombre de secrets et leur cycle de vie.
-   Testez fréquemment les procédures de restauration pour garantir que les sauvegardes sont intactes.
-   Cryptez les sauvegardes et placez-les sur un stockage sécurisé avec des droits d'accès réduits. Surveillez l’emplacement de sauvegarde pour les accès (non autorisés) et les actions administratives.

Enfin, vous devez mettre en œuvre des processus d'urgence (« bris de vitre ») pour restaurer le service si le système devient indisponible pour des raisons autres que la maintenance régulière. Par conséquent, les informations d’identification en cas d’urgence doivent être régulièrement sauvegardées en toute sécurité dans un système de gestion des secrets secondaire et testées régulièrement pour vérifier leur fonctionnement.

### 2.10 Politiques

Appliquez systématiquement des politiques définissant les exigences minimales de complexité des mots de passe et des algorithmes de chiffrement approuvés à l’échelle de l’organisation. L’utilisation d’une solution centralisée de gestion des secrets peut aider les entreprises à mettre en œuvre ces politiques.

Ensuite, disposer d’une politique de gestion des secrets à l’échelle de l’organisation peut aider à appliquer les meilleures pratiques définies dans cette aide-mémoire.

### 2.11 Métadonnées : préparez-vous à déplacer le secret

Une solution de gestion des secrets doit offrir la possibilité de stocker au moins les métadonnées suivantes concernant un secret :

-   Quand il a été créé/consommé/archivé/roté/supprimé
-   Qui l'a créé/consommé/archivé/roté/supprimé (par exemple, à la fois le producteur réel et l'ingénieur utilisant la méthode de production)
-   Qu'est-ce qui l'a créé/consommé/archivé/fait pivoter/supprimé
-   Qui contacter en cas de problème avec le secret ou si vous avez des questions à ce sujet
-   Pour quelle raison le secret est-il utilisé (par exemple, les consommateurs prévus désignés et le but du secret)
-   De quel type de secret il s'agit (par exemple, clé AES, clé HMAC, clé privée RSA)
-   Lorsque vous devez le faire pivoter, si cela est fait manuellement

Remarque : si vous ne stockez pas de métadonnées sur le secret et ne vous préparez pas à le déplacer, vous augmenterez la probabilité de dépendance envers un fournisseur.

## 3 Intégration continue (CI) et déploiement continu (CD)

La création, le test et le déploiement de modifications nécessitent généralement l'accès à de nombreux systèmes. Les outils d'intégration continue (CI) et de déploiement continu (CD) stockent généralement des secrets pour fournir la configuration de l'application ou pendant le déploiement. Alternativement, ils interagissent fortement avec le système de gestion des secrets. Diverses bonnes pratiques peuvent aider à faciliter la gestion des secrets dans CI/CD ; nous en traiterons quelques-uns dans cette section.

### 3.1 Renforcer votre pipeline CI/CD

Les outils CI/CD consomment régulièrement des informations d'identification (à privilèges élevés). Assurez-vous que le pipeline ne peut pas être facilement piraté ou utilisé à mauvais escient par les employés. Voici quelques lignes directrices qui peuvent vous aider :

-   Traitez vos outils CI/CD comme un environnement de production : renforcez-les, corrigez-les et renforcez l'infrastructure et les services sous-jacents.
-   Avoir une surveillance des événements de sécurité en place.
-   Implémentez l’accès au moindre privilège : les développeurs n’ont pas besoin de pouvoir administrer les projets. Au lieu de cela, ils doivent uniquement être capables d'exécuter les fonctions requises, telles que la configuration des pipelines, leur exécution et l'utilisation du code. Les tâches administratives peuvent être rapidement effectuées à l'aide de la configuration en tant que code dans un référentiel distinct utilisé par le système CI/CD pour mettre à jour sa configuration. Il n'est pas nécessaire de disposer de rôles privilégiés susceptibles d'avoir accès aux secrets.
-   Assurez-vous que la sortie du pipeline ne divulgue pas de secrets et que vous ne pouvez pas écouter les pipelines de production avec des outils de débogage.
-   Assurez-vous que vous ne pouvez pas exécuter d'exécutables ni de travailleurs pour un système CI/CD.
-   Avoir une authentification, une autorisation et une comptabilité appropriées en place.
-   Assurez-vous que seul un processus approuvé peut créer des pipelines, y compris des étapes MR/PR pour garantir qu'un pipeline créé est examiné en termes de sécurité.

### 3.2 Où doit se trouver un secret ?

Il existe différents endroits où vous pouvez stocker un secret pour exécuter des actions CI/CD :

-   Dans le cadre de vos outils CI/CD : vous pouvez stocker un secret dans[GitLab](https://docs.gitlab.com/charts/installation/secrets.html)/[GitHub](https://docs.github.com/en/actions/security-guides/encrypted-secrets)/[jenkins](https://www.jenkins.io/doc/developer/security/secrets/). Ce n’est pas la même chose que de le valider dans le code.
-   Dans le cadre de votre système de gestion des secrets : vous pouvez stocker un secret dans un système de gestion des secrets, tel que des installations fournies par un fournisseur de cloud ([Gestionnaire de secrets AWS](https://aws.amazon.com/secrets-manager/),[Coffre de clés Azure](https://azure.microsoft.com/nl-nl/services/key-vault/),[Gestionnaire de secrets Google](https://cloud.google.com/secret-manager)), ou d'autres installations tierces ([Coffre Hashicorp](https://www.vaultproject.io/),[Conjurer](https://www.conjur.org/),[Gardien](https://www.keepersecurity.com/),[Confident](https://lyft.github.io/confidant/)). Dans ce cas, les outils de pipeline CI/CD nécessitent des informations d'identification pour se connecter à ces systèmes de gestion des secrets afin d'avoir des secrets en place. Voir[Fournisseurs de cloud](#4-cloud-providers)pour plus de détails sur l'utilisation du système de gestion des secrets d'un fournisseur de cloud.

Une autre alternative ici consiste à utiliser le pipeline CI/CD pour exploiter le cryptage en tant que service des systèmes de gestion des secrets pour effectuer le cryptage d'un secret. Les outils CI/CD peuvent ensuite valider le secret chiffré dans git, qui peut être récupéré par le service consommateur lors du déploiement et déchiffré à nouveau. Voir la section 3.6 pour plus de détails.

Remarque : tous les secrets ne doivent pas nécessairement se trouver dans le pipeline CI/CD pour accéder au déploiement réel. Assurez-vous plutôt que les services déployés prennent en charge une partie de la gestion de leurs secrets lors de leur propre cycle de vie (par exemple, déploiement, exécution et destruction).

#### 3.2.1 Dans le cadre de votre outil CI/CD

Lorsque les secrets font partie de vos outils CI/CD, cela signifie que ces secrets sont exposés à vos tâches CI/CD. Les outils CI/CD peuvent comprendre, par ex. Secrets GitHub, secrets du référentiel GitLab, groupes ENV Vars/Var dans Microsoft Azure DevOps, secrets Kubernetes, etc.
Ces secrets sont souvent configurables/visibles par des personnes autorisées à le faire (par exemple un responsable dans GitHub, un propriétaire de projet dans GitLab, un administrateur dans Jenkins, etc.), ce qui, ensemble, s'aligne sur les meilleures pratiques suivantes :

-   Pas de « grand secret » : assurez-vous que les secrets de vos outils CI/CD qui ne durent pas à long terme, n'ont pas un large rayon d'action et n'ont pas une valeur élevée. Limitez également les secrets partagés (par exemple, ne jamais avoir un seul mot de passe pour tous les utilisateurs administratifs).
-   En l'état/à être : ayez un aperçu clair des utilisateurs qui peuvent consulter ou modifier les secrets. Souvent, les responsables d'un projet GitLab/GitHub peuvent voir ou extraire ses secrets.
-   Réduisez le nombre de personnes pouvant effectuer des tâches administratives sur le projet afin de limiter l’exposition.
-   Journal et alerte : rassemblez tous les journaux des outils CI/CD et mettez en place des règles pour détecter l'extraction de secrets ou les utilisations abusives, que ce soit en y accédant via une interface Web ou en les vidant tout en les codant en double base64 ou en les chiffrant avec OpenSSL.
-   Rotation : faites régulièrement pivoter les secrets.
-   Le fork ne doit pas fuir : vérifiez qu'un fork du référentiel ou une copie de la définition de travail ne copie pas le secret.
-   Documenter : assurez-vous de documenter les secrets que vous stockez dans le cadre de vos outils CI/CD et pourquoi, afin de pouvoir les migrer facilement en cas de besoin.

#### 3.2.2 Le stocker dans un système de gestion des secrets

Naturellement, vous pouvez stocker les secrets dans une solution de gestion des secrets désignée. Par exemple, vous pouvez utiliser une solution proposée par votre fournisseur d'infrastructure (cloud), telle que[Gestionnaire de secrets AWS](https://aws.amazon.com/secrets-manager/),[Gestionnaire de secrets Google](https://cloud.google.com/secret-manager), ou[Azure KeyVault](https://azure.microsoft.com/nl-nl/services/key-vault/). Vous pouvez trouver plus d'informations à ce sujet dans[Section 4](#4-cloud-providers)de cette aide-mémoire. Une autre option est un système de gestion des secrets dédié, tel que[Coffre Hashicorp](https://www.vaultproject.io/),[Gardien](https://www.keepersecurity.com/),[Confident](https://lyft.github.io/confidant/),[Conjurer](https://www.conjur.org/).
Voici quelques choses à faire et à ne pas faire pour l'interaction CI/CD avec ces systèmes. Assurez-vous que les éléments suivants sont pris en compte :

-   Rotation/Temporalité : les informations d'identification utilisées par les outils CI/CD pour s'authentifier auprès du système de gestion des secrets sont fréquemment alternées et expirent une fois la tâche terminée.
-   Portée de l'autorisation : les informations d'identification de portée utilisées par l'outil CI/CD (par exemple, rôles, utilisateurs, etc.), autorisent uniquement les secrets et services du système de gestion des secrets requis pour que l'outil CI/CD exécute son travail.
-   Attribution de l'appelant : les identifiants utilisés par les outils CI/CD détiennent toujours l'attribution de celui qui appelle la solution de gestion des secrets. Assurez-vous de pouvoir attribuer tous les appels effectués par les outils CI/CD à une personne ou à un service qui a demandé les actions des outils CI/CD. Si cela n'est pas possible via la configuration par défaut du gestionnaire de secrets, assurez-vous d'avoir une configuration de corrélation en termes de paramètres de requête.
-   Tout ce qui précède : suivez toujours les choses à faire et à ne pas faire répertoriées dans la section 3.2.1 : journaliser et alerter, prendre soin du forking, etc.
-   Sauvegarde : sauvegardez les secrets des opérations critiques pour le produit dans un stockage séparé (par exemple, un stockage froid), en particulier les clés de chiffrement.

#### 3.2.3 Pas du tout touché par CI/CD

Les secrets ne doivent pas nécessairement être transmis à un consommateur du secret par un pipeline CI/CD. C'est encore mieux lorsque le consommateur du secret récupère le secret. Dans ce cas, le pipeline CI/CD doit toujours demander au système d'orchestration (par ex.[Kubernetes](https://kubernetes.io/)) qu'il doit planifier un service spécifique avec un compte de service donné avec lequel le consommateur peut ensuite récupérer le secret requis. L'outil CI/CD dispose alors toujours des informations d'identification pour la plateforme d'orchestration mais n'a plus accès aux secrets eux-mêmes. Les choses à faire et à ne pas faire concernant ces types d'informations d'identification sont similaires à celles décrites dans la section 3.2.2.

### 3.3 Authentification et autorisation des outils CI/CD

Les outils CI/CD doivent avoir des comptes de service désignés, qui ne peuvent fonctionner que dans le cadre des secrets requis ou de l'orchestration des consommateurs d'un secret. De plus, une exécution de pipeline CI/CD doit être facilement attribuable à celui qui a défini la tâche ou l'a déclenchée pour détecter qui a tenté d'exfiltrer des secrets ou de les manipuler. Lorsque vous utilisez l'authentification basée sur un certificat, l'appelant de l'identité du pipeline doit faire partie du certificat. Si vous utilisez un jeton pour vous authentifier auprès des systèmes mentionnés, assurez-vous de définir le principal demandant ces actions (par exemple l'utilisateur ou le créateur du travail).

Vérifiez périodiquement si c'est (encore) le cas pour votre système afin de pouvoir effectuer efficacement la journalisation, l'attribution et les alertes de sécurité sur les actions suspectes.

### 3.4 Journalisation et comptabilité

Les attaquants peuvent utiliser les outils CI/CD pour extraire des secrets. Ils pourraient par exemple utiliser des interfaces administratives ou la création de jobs qui exfiltrent le secret grâce au chiffrement ou au double encodage base64. Par conséquent, vous devez enregistrer chaque action dans un outil CI/CD. Vous devez définir des règles d'alerte de sécurité à chaque manipulation non standard de l'outil de pipeline et de son interface d'administration pour surveiller l'utilisation des secrets.
Les journaux doivent être interrogeables pendant au moins 90 jours et stockés pendant une période plus longue dans un entrepôt frigorifique. Il faudra peut-être du temps aux équipes de sécurité pour comprendre comment les attaquants peuvent exfiltrer ou manipuler un secret à l’aide des outils CI/CD.

### 3.5 Rotation vs création dynamique

Vous pouvez tirer parti des outils CI/CD pour effectuer une rotation des secrets ou demander à d'autres composants d'effectuer la rotation du secret. Par exemple, l'outil CI/CD peut demander à un système de gestion des secrets ou à une autre application de faire tourner le secret. Alternativement, l'outil CI/CD ou un autre composant pourrait configurer un secret dynamique : un secret qu'un consommateur doit utiliser aussi longtemps qu'il vit. Le secret est invalidé lorsque le consommateur ne vit plus. Cette procédure réduit les fuites possibles d’un secret et permet une détection facile d’une utilisation abusive. Si un attaquant utilise un secret provenant d'un endroit autre que l'adresse IP du consommateur, vous pouvez facilement le détecter.

### 3.6 Secrets créés par le pipeline

Vous pouvez utiliser des outils de pipeline pour générer des secrets et soit les proposer directement au service déployé par l'outil, soit fournir le secret à une solution de gestion des secrets. Alternativement, le secret peut être stocké crypté dans git afin que le secret et ses métadonnées soient aussi proches que possible du lieu de travail quotidien du développeur. Un secret stocké sur Git nécessite que les développeurs ne puissent pas déchiffrer les secrets eux-mêmes et que chaque consommateur d'un secret dispose de sa variante chiffrée du secret. Par exemple : le secret doit alors être différent selon l'environnement DTAP et être chiffré avec une autre clé. Pour chaque environnement, seul le consommateur désigné dans cet environnement doit être en mesure de déchiffrer le secret spécifique. Un secret ne fuit pas d’un environnement à l’autre et peut toujours être facilement stocké à côté du code.
Les consommateurs d'un secret pouvaient désormais déchiffrer le secret à l'aide d'un side-car, comme décrit à la section 5.2. Au lieu de récupérer les secrets, le consommateur utiliserait le side-car pour décrypter le secret.

Lorsqu'un pipeline crée lui-même un secret, assurez-vous que les scripts ou les binaires impliqués respectent les meilleures pratiques en matière de génération de secrets. Les meilleures pratiques incluent le caractère aléatoire sécurisé, la durée appropriée de création du secret, etc. et le fait que le secret soit créé sur la base de métadonnées bien définies stockées quelque part dans git ou ailleurs.

## 4 fournisseurs de cloud

Pour les fournisseurs de cloud, il y a au moins quatre sujets essentiels à aborder :

-   Solutions de stockage/gestion secrètes désignées. Quel(s) service(s) utilisez-vous ?
-   Chiffrement de l'enveloppe et côté client
-   Gestion des identités et des accès : diminuer le rayon de souffle
-   Quotas d'API ou limites de service

### 4.1 Services à utiliser

Il est préférable d’utiliser une solution de gestion des secrets désignée dans n’importe quel environnement. La plupart des fournisseurs de cloud disposent d'au moins un service proposant la gestion des secrets. Bien entendu, il est également possible d'exécuter une autre solution de gestion des secrets (par exemple HashiCorp Vault ou Conjur) sur les ressources de calcul dans le cloud. Nous examinerons les offres de services des fournisseurs de cloud dans cette section.

Parfois, il est possible de faire pivoter automatiquement votre secret, soit via un service fourni par votre fournisseur de cloud, soit via une fonction (personnalisée). Généralement, vous devriez préférer la solution du fournisseur de cloud car la barrière à l'entrée et le risque de mauvaise configuration sont plus faibles. Si vous utilisez une solution personnalisée, assurez-vous que le rôle de la fonction consistant à effectuer sa rotation ne peut être assumé que par ladite fonction.

#### 4.1.1 AWS

Pour AWS, la solution recommandée est[Gestionnaire de secrets AWS](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html).

Les autorisations sont accordées au niveau secret. Vérifiez[Meilleures pratiques de Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html).

Il est également possible d'utiliser le[Magasin de paramètres Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html), qui est moins cher, mais qui présente quelques inconvénients :

-   vous devrez vous assurer que vous avez spécifié le cryptage vous-même (le gestionnaire de secrets le fait par défaut)
-   il offre moins de capacités de rotation automatique (vous devrez probablement créer une fonction personnalisée)
-   il ne prend pas en charge l'accès entre comptes
-   il ne prend pas en charge la réplication entre régions
-   il y a moins[contrôles du centre de sécurité](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html)disponible

##### 4.1.1.1 Enclaves AWS Nitro

Avec[Enclaves AWS Nitro](https://aws.amazon.com/ec2/nitro/nitro-enclaves/), vous pouvez créer des environnements d'exécution fiables. Ainsi, aucun accès humain n’est possible une fois l’application exécutée. De plus, les enclaves ne sont dotées d’aucun stockage permanent. Par conséquent, les secrets et autres données sensibles stockés sur les enclaves nitro disposent d’une couche de sécurité supplémentaire.

##### 4.1.1.2 AWS CloudHSM

Pour les secrets utilisés dans des applications hautement confidentielles, il peut être nécessaire de mieux contrôler le cryptage et le stockage de ces clés. Offres AWS[CloudHSM](https://aws.amazon.com/cloudhsm/), qui vous permet d'apporter votre propre clé (BYOK) pour les services AWS. Ainsi, vous aurez plus de contrôle sur la création, le cycle de vie et la durabilité des clés. CloudHSM permet la mise à l'échelle et la sauvegarde automatiques de vos données. Le fournisseur de services cloud, Amazon, n’aura aucun accès aux clés stockées dans Azure Dedicated HSM.

#### 4.1.2 GCP

Pour GCP, le service recommandé est[Gestionnaire de secrets](https://cloud.google.com/secret-manager/docs).

Les autorisations sont accordées au niveau secret.

Vérifiez[Meilleures pratiques de Secret Manager](https://cloud.google.com/secret-manager/docs/best-practices).

##### 4.1.2.1 Informatique confidentielle de Google Cloud

[Informatique confidentielle GCP](https://cloud.google.com/confidential-computing)permet le cryptage des données pendant l'exécution. Ainsi, le code et les données des applications sont gardés secrets, cryptés et inaccessibles aux humains ou aux outils.

#### 4.1.3 Azur

Pour Azure, le service recommandé est[Coffre-fort de clés](https://docs.microsoft.com/en-us/azure/key-vault/).

Contrairement aux autres cloud, les autorisations sont accordées au_**Coffre-fort de clés**_niveau. Cela signifie que les secrets pour des charges de travail distinctes et des niveaux de sensibilité distincts doivent se trouver en conséquence dans des Key Vaults séparés.

Vérifiez[Bonnes pratiques de Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/best-practices).

##### 4.1.3.1 Informatique confidentielle Azure

Avec[Informatique confidentielle Azure](https://azure.microsoft.com/en-us/solutions/confidential-compute/#overview), vous pouvez créer des environnements d'exécution fiables. Ainsi, chaque application sera exécutée dans une enclave cryptée qui protège les données et le code consommé par l'application est protégé de bout en bout. De plus, toute application exécutée dans des enclaves n’est accessible par aucun outil ni humain.

##### 4.1.3.2 HSM dédié Azure

Pour les secrets utilisés dans les environnements Azure et nécessitant des considérations de sécurité particulières, Azure propose[HSM dédié Azure](https://azure.microsoft.com/en-us/services/azure-dedicated-hsm/). Cela vous permet de mieux contrôler les secrets qui y sont stockés, y compris un contrôle administratif et cryptographique amélioré. Le fournisseur de services cloud, Microsoft, n’aura aucun accès aux clés stockées dans Azure Dedicated HSM.

#### 4.1.4 Autres cloud, multi-cloud et indépendant du cloud

Si vous utilisez plusieurs fournisseurs de cloud, vous devriez envisager d'utiliser une solution de gestion des secrets indépendante du cloud. Cela vous permettra d'utiliser la même solution de gestion des secrets sur tous vos fournisseurs de cloud (et éventuellement également sur site). Un autre avantage est que cela évite la dépendance vis-à-vis d'un fournisseur de cloud spécifique, car la solution peut être utilisée sur n'importe quel fournisseur de cloud.

Il existe des solutions open source et commerciales disponibles. Certains exemples sont:

-   [CyberArk Conjurer](https://www.conjur.org/)
-   [Coffre HashiCorp](https://www.vaultproject.io/)

### 4.2 Enveloppe et chiffrement côté client

Cette section décrira comment un secret est chiffré et comment vous pouvez gérer les clés de ce chiffrement dans le cloud.

#### 4.2.1 Chiffrement côté client versus chiffrement côté serveur

Le chiffrement des secrets côté serveur garantit que le fournisseur de cloud prend en charge le chiffrement du secret stocké. Le secret est ensuite protégé contre toute compromission au repos. Le chiffrement au repos ne nécessite souvent pas de travail supplémentaire autre que la sélection de la clé avec laquelle le chiffrer (voir section 4.2.2). Cependant : lorsque vous soumettez le secret à un autre service, il ne sera plus crypté. Il est déchiffré avant d'être partagé avec le service prévu ou l'utilisateur humain.

Le chiffrement des secrets côté client garantit que le secret reste chiffré jusqu'à ce que vous le déchiffriez activement. Cela signifie qu'il n'est déchiffré que lorsqu'il arrive chez le consommateur. Vous devez disposer d’un système de cryptographie approprié pour répondre à cela. Pensez à des mécanismes tels que PGP utilisant une configuration sécurisée et à d'autres systèmes plus évolutifs et relativement faciles à utiliser. Le chiffrement côté client peut fournir un chiffrement de bout en bout du secret : du producteur au consommateur.

#### 4.2.2 Apportez votre propre clé par rapport à la clé du fournisseur de cloud

Lorsque vous chiffrez un secret au repos, la question est : quelle clé souhaitez-vous utiliser ? Moins vous avez confiance dans le fournisseur de cloud, plus vous souhaiterez vous gérer vous-même.

Souvent, vous pouvez soit chiffrer un secret avec une clé gérée par le service de gestion des secrets, soit utiliser une solution de gestion de clés du fournisseur de cloud pour chiffrer le secret. La clé proposée via la solution de gestion des clés du fournisseur de cloud peut être gérée soit par le fournisseur de cloud, soit par vous-même. Les normes de l'industrie appellent ce dernier « apportez votre propre clé » (BYOK). Vous pouvez soit importer ou générer directement cette clé au niveau de la solution de gestion des clés, soit à l'aide du cloud HSM pris en charge par le fournisseur de cloud.
Vous pouvez ensuite utiliser soit votre clé, soit la clé principale client du fournisseur pour chiffrer la clé de données de la solution de gestion des secrets. La clé de données, à son tour, chiffre le secret. En gérant la CMK, vous contrôlez la clé de données au niveau de la solution de gestion des secrets.

Bien que l'importation de votre propre matériel de clé puisse généralement être effectuée auprès de tous les fournisseurs ([AWS](https://docs.aws.amazon.com/kms/latest/developerguide/importing-keys.html),[Azur](https://docs.microsoft.com/en-us/azure/key-vault/keys/byok-specification),[GCP](https://cloud.google.com/kms/docs/key-import)), à moins que vous ne sachiez ce que vous faites et que votre modèle de menace et votre politique l'exigent, cette solution n'est pas recommandée en raison de sa complexité et de sa difficulté d'utilisation.

### 4.3 Gestion des identités et des accès (IAM)

IAM s'applique à la fois aux configurations sur site et dans le cloud : pour gérer efficacement les secrets, vous devez configurer des politiques et des rôles d'accès appropriés. La mise en place de cela va au-delà des politiques concernant les secrets ; cela devrait inclure le renforcement de la configuration IAM complète, car cela pourrait autrement permettre des attaques par élévation de privilèges. Assurez-vous de ne jamais autoriser les privilèges ouverts de « passer le rôle » ou les privilèges de création IAM sans restriction, car ceux-ci peuvent utiliser ou créer des informations d'identification ayant accès aux secrets. Ensuite, assurez-vous de contrôler étroitement ce qui peut usurper l'identité d'un compte de service : les rôles de vos machines sont-ils accessibles par un attaquant exploitant votre serveur ? Les rôles de service des outils de pipeline de données peuvent-ils accéder facilement aux secrets ? Assurez-vous d'inclure IAM pour chaque composant cloud dans votre modèle de menace (par exemple, demandez-vous : comment pouvez-vous procéder à une élévation de privilèges avec ce composant ?). Voir[cette entrée de blog](https://xebia.com/ten-pitfalls-you-should-look-out-for-in-aws-iam/)pour plusieurs choses à faire et à ne pas faire avec des exemples.

Tirer efficacement parti de la temporalité des principes IAM : par ex. assurez-vous que seuls les rôles et comptes de service spécifiques qui le nécessitent peuvent accéder aux secrets. Surveillez ces comptes afin de savoir qui ou quoi les a utilisés pour accéder aux secrets.

Ensuite, assurez-vous de limiter l’accès à vos secrets : personne ne devrait pas être simplement autorisé à accéder à tous les secrets. Dans GCP et AWS, vous pouvez créer des politiques d'accès précises pour garantir qu'un mandataire ne peut pas accéder à tous les secrets en même temps. Dans Azure, avoir accès au coffre de clés signifie avoir accès à tous les secrets de ce coffre de clés. Il est donc essentiel de disposer de coffres de clés distincts lorsque vous travaillez sur Azure afin de séparer les accès.

### 4.4 Limites de l'API

Les services cloud peuvent généralement fournir un nombre limité d'appels API sur une période donnée. Vous pourriez potentiellement (D)DoS vous-même lorsque vous rencontrez ces limites. La plupart de ces limites s'appliquent par compte, projet ou abonnement, alors répartissez les charges de travail pour limiter votre rayon d'explosion en conséquence. De plus, certains services peuvent prendre en charge la mise en cache des clés de données, empêchant ainsi la charge sur l'API du service de gestion des clés (voir par exemple[Mise en cache des clés de données AWS](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-key-caching.html)). Certains services peuvent exploiter la mise en cache intégrée des clés de données.[S3 en est un exemple](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-key.html).

## 5 conteneurs et orchestrateurs

Vous pouvez enrichir les conteneurs avec des secrets de plusieurs manières : au moment de la construction (non recommandé) et pendant l'orchestration/le déploiement.

### 5.1 Injection de secrets (fichier, en mémoire)

Il existe trois façons d'obtenir les secrets d'une application dans un conteneur Docker.

-   Volumes montés (fichier) : avec cette méthode, nous gardons nos secrets dans un fichier de configuration/secret particulier et montons ce fichier sur notre instance en tant que volume monté. Assurez-vous que ces montages sont montés par l'orchestrateur et jamais intégrés, car cela entraînerait une fuite du secret avec la définition du conteneur. Assurez-vous plutôt que l’orchestrateur monte dans le volume lorsque cela est nécessaire.
-   Récupérer depuis le magasin de secrets (en mémoire) : une application/un conteneur side-car récupère les secrets dont il a besoin directement à partir d'un service de gestion de secrets sans gérer la configuration du docker. Cette solution vous permet d'utiliser des secrets construits dynamiquement sans vous soucier de la visibilité des secrets à partir du système de fichiers ou de la vérification des variables d'environnement du conteneur Docker.
-   Variables d'environnement : nous pouvons fournir des secrets directement dans le cadre de la configuration du conteneur Docker. Remarque : les secrets eux-mêmes ne doivent jamais être codés en dur à l'aide des commandes docker ENV ou docker ARG, car celles-ci peuvent facilement fuir avec les définitions du conteneur. Découvrez les défis Docker sur[Mauvais secrets](https://github.com/OWASP/wrongsecrets)aussi. Au lieu de cela, laissez un orchestrateur écraser la variable d'environnement avec le secret réel et assurez-vous qu'il n'est pas codé en dur. De plus, les variables d'environnement sont généralement accessibles à tous les processus et peuvent être incluses dans les journaux ou les dumps système. L'utilisation de variables d'environnement n'est donc pas recommandée sauf si les autres méthodes ne sont pas possibles.

### 5.2 Conteneurs side-car de courte durée de vie

Pour injecter des secrets, vous pouvez créer des conteneurs side-car de courte durée qui récupèrent les secrets d'un point de terminaison distant, puis les stockent sur un volume partagé monté sur le conteneur d'origine. Le conteneur d'origine peut désormais utiliser les secrets du volume monté. L’avantage d’utiliser cette approche est que nous n’avons pas besoin d’intégrer d’outil ou de code tiers pour obtenir des secrets. Une fois que le side-car a récupéré les secrets, il se termine. Des exemples de ceci incluent[Injecteur side-car Vault Agent](https://developer.hashicorp.com/vault/docs/platform/k8s/injector)et[Fournisseur de secrets de conjuration](https://github.com/cyberark/secrets-provider-for-k8s). En montant des secrets sur un volume partagé avec le pod, les conteneurs du pod peuvent consommer des secrets sans connaître le gestionnaire de secrets.

### 5.3 Accès interne ou externe

Vous ne devez exposer les secrets qu'aux mécanismes de communication entre le conteneur et la représentation de déploiement (par exemple, un pod Kubernetes). N'exposez jamais de secrets via des mécanismes d'accès externes partagés entre les déploiements ou les orchestrateurs (par exemple, un volume partagé).

Lorsque l'orchestrateur stocke des secrets (par exemple, les secrets Kubernetes), assurez-vous que le backend de stockage de l'orchestrateur est chiffré et que vous gérez bien les clés. Voir le[Aide-mémoire de sécurité Kubernetes](Kubernetes_Security_Cheat_Sheet.md)pour plus d'informations.

## 6 Guide de mise en œuvre

Dans cette section, nous discuterons de la mise en œuvre. Notez qu'il est toujours préférable de se référer à la documentation officielle du système de gestion des secrets de votre choix pour la mise en œuvre réelle, car elle sera plus à jour que tout document secondaire tel que cet aide-mémoire.

### 6.1 Politiques clés de gestion du matériel

La gestion des matériaux clés est abordée dans le[Aide-mémoire pour la gestion des clés](Key_Management_Cheat_Sheet.md)

### 6.2 Cas d'utilisation dynamiques et statiques

Nous voyons, entre autres, les cas d'utilisation suivants pour les secrets dynamiques :

-   secrets de courte durée (par exemple, informations d'identification ou clés API) pour un service secondaire qui exprime l'intention de connecter le service principal (par exemple, consommateur) au service.
-   contrôles d'intégrité et de chiffrement de courte durée pour protéger et sécuriser les processus de communication en mémoire et d'exécution. Pensez aux clés de chiffrement qui ne doivent fonctionner que pendant une seule session ou une seule durée de vie de déploiement.
-   des informations d'identification de courte durée pour créer une pile lors du déploiement d'un service permettant d'interagir avec les déployeurs et l'infrastructure de support.

Notez que ces secrets dynamiques doivent souvent être créés avec le service auquel nous devons nous connecter. Pour créer ces types de secrets dynamiques, nous avons généralement besoin de secrets statiques à long terme pour créer les secrets dynamiques eux-mêmes. Autres cas d'utilisation statiques :

-   les éléments de clé qui doivent durer plus longtemps qu'un seul déploiement en raison de la nature de leur utilisation dans l'interaction avec d'autres instances du même service (par exemple, clés de chiffrement de stockage, clés TLS PKI)
-   des éléments clés ou des informations d'identification pour se connecter à des services qui ne prennent pas en charge la création de rôles ou d'informations d'identification temporels.

### 6.3 S'assurer que les limitations sont en place

Les secrets ne devraient jamais pouvoir être récupérés par tout le monde et par tout. Assurez-vous toujours de mettre des garde-corps en place :

-   Avez-vous la possibilité de créer des politiques d’accès ? Assurez-vous que des politiques sont en place pour limiter le nombre d'entités pouvant lire ou écrire le secret. En même temps, rédigez les politiques de manière à pouvoir les étendre facilement et à ce qu’elles ne soient pas trop compliquées à comprendre.
-   N’existe-t-il aucun moyen de réduire l’accès à certains secrets au sein d’une solution de gestion des secrets ? Envisagez de séparer les secrets de production et de développement en disposant de solutions de gestion des secrets distinctes. Ensuite, réduisez l’accès à la solution de gestion des secrets de production.

### 6.4 La surveillance des événements de sécurité est essentielle

Surveillez en permanence qui/quoi, à partir de quelle adresse IP et quelle méthodologie accède au secret. Il existe différents modèles auxquels vous devez prêter attention, tels que, sans toutefois s'y limiter :

-   Surveiller qui accède au secret au niveau du système de gestion des secrets : est-ce un comportement normal ? Si les informations d'identification CI/CD sont utilisées pour accéder à la solution de gestion des secrets à partir d'une adresse IP différente de celle où le système CI/CD est exécuté, fournissez une alerte de sécurité et supposez que le secret est compromis.
-   Surveillez le service nécessitant le secret (si possible), par exemple, si l'utilisateur du secret provient d'une adresse IP attendue, avec un agent utilisateur attendu. Sinon, alertez et supposez que le secret est compromis.

### 6.5 Utilisabilité

Assurez-vous que votre solution de gestion des secrets est facile à utiliser, car vous ne voulez pas que les gens la contournent ou l'utilisent de manière inefficace en raison de sa complexité. Cette convivialité nécessite :

-   Intégration facile de nouveaux secrets et suppression des secrets invalidés.
-   Intégration facile avec le logiciel existant : il doit être facile d'intégrer des applications en tant que consommatrices du système de gestion des secrets. Par exemple, un SDK ou un simple conteneur side-car doit être disponible pour communiquer avec le système de gestion des secrets afin que les logiciels existants soient découplés et ne nécessitent pas de modifications importantes. Vous pouvez en trouver des exemples dans les SDK AWS, Google et Azure. Ces SDK permettent à une application d'interagir avec les solutions de gestion des secrets respectives. Vous pouvez trouver des exemples similaires dans les intégrations du logiciel HashiCorp Vault et dans le[Injecteur side-car Vault Agent](https://developer.hashicorp.com/vault/docs/platform/k8s/injector), ainsi que les intégrations Conjur et[Fournisseur de secrets de conjuration](https://github.com/cyberark/secrets-provider-for-k8s).
-   Une compréhension claire de l’organisation de la gestion des secrets et de ses processus est essentielle.

## 7 Chiffrement

La gestion des secrets va de pair avec le chiffrement. Après tout, les secrets doivent être stockés cryptés quelque part pour protéger leur confidentialité et leur intégrité.

### 7.1 Types de cryptage à utiliser

Vous pouvez utiliser différents types de cryptage pour sécuriser un secret à condition qu'ils offrent une sécurité suffisante, notamment une résistance adéquate contre les attaques basées sur l'informatique quantique. Étant donné qu'il s'agit d'un domaine en évolution, il est préférable de consulter des sources telles que[keylength.com](https://www.keylength.com/en/4/), qui énumère les recommandations à jour sur l'utilisation des types de cryptage et des longueurs de clé pour les normes existantes, ainsi que celles de la NSA.[Suite d'algorithmes de sécurité nationale commerciale 2.0](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF)qui énumère les algorithmes résistants aux quantiques.

Veuillez noter que dans tous les cas, nous devons sélectionner de préférence un algorithme qui assure à la fois le cryptage et la confidentialité, comme l'AES-256 utilisant GCM.[(Gallois Counter Mode)](https://en.wikipedia.org/wiki/Galois/Counter_Mode), ou un mélange de ChaCha20 et Poly1305 selon les meilleures pratiques du domaine.

### 7.2 Chiffrement convergent

[Chiffrement convergent](https://en.wikipedia.org/wiki/Convergent_encryption)garantit qu'un texte en clair donné et sa clé aboutissent au même texte chiffré. Cela peut aider à détecter une éventuelle réutilisation de secrets, aboutissant au même texte chiffré.
Le défi de l’activation du chiffrement convergent est qu’il permet aux attaquants d’utiliser le système pour générer un ensemble de chaînes cryptographiques qui pourraient se retrouver dans le même secret, permettant ainsi à l’attaquant d’en extraire le secret en texte brut. Compte tenu de l'algorithme et de la clé, vous pouvez atténuer ce risque si le système de chiffrement convergent que vous utilisez présente suffisamment de problèmes de ressources lors du chiffrement. Un autre facteur qui peut aider à réduire le risque est de s'assurer qu'un secret est d'une longueur adéquate, ce qui entrave encore davantage le temps d'itération possible requis.

### 7.3 Où stocker les clés de cryptage ?

Vous ne devez pas stocker les clés à côté des secrets qu'elles chiffrent, sauf si ces clés sont elles-mêmes chiffrées (voir chiffrement par enveloppe). Commencez par consulter le[Aide-mémoire pour la gestion des clés](Key_Management_Cheat_Sheet.md)sur où et comment stocker le cryptage et les éventuelles clés HMAC.

### 7.4 Chiffrement en tant que service (EaaS)

EaaS est un modèle dans lequel les utilisateurs s'abonnent à un service de chiffrement basé sur le cloud sans avoir à installer le chiffrement sur leurs propres systèmes. En utilisant EaaS, vous pouvez bénéficier des avantages suivants :

-   Chiffrement au repos
-   Chiffrement en transit (TLS)
-   La gestion des clés et les implémentations cryptographiques sont prises en charge par Encryption Service, et non par les développeurs.
-   Le fournisseur pourrait ajouter plus de services pour interagir avec les données sensibles

## 8 Détection

Il existe de nombreuses approches pour détecter les secrets et certains projets open source très utiles pour y parvenir. Le[Yelp Détecter les secrets](https://github.com/Yelp/detect-secrets)le projet est mature et a une correspondance de signature pour environ 20 secrets. Pour plus d'informations sur d'autres outils pour vous aider dans l'espace de détection, consultez le[Détection des secrets](https://github.com/topics/secrets-detection)sujet sur GitHub.

### 8.1 Approches générales de détection

Les principes Shift-left et DevSecOps s'appliquent également à la détection des secrets. Ces approches générales ci-dessous visent à considérer les secrets plus tôt et à faire évoluer la pratique au fil du temps.

-   Créez des secrets de test standard et utilisez-les universellement dans toute l’organisation. Cela permet de réduire les faux positifs en n’ayant besoin de suivre qu’un seul secret de test pour chaque type de secret.
-   Envisagez d'activer la détection des secrets au niveau du développeur pour éviter de vérifier les secrets dans le code avant la validation/PR, soit dans l'EDI, dans le cadre d'un développement piloté par les tests, soit via un hook de pré-commit.
-   Intégrez la détection des secrets au modèle de menace. Considérez les secrets comme faisant partie de la surface d’attaque lors des exercices de modélisation des menaces.
-   Évaluez souvent les utilitaires de détection et les signatures associées pour vous assurer qu’ils répondent aux attentes.
-   Envisagez de disposer de plusieurs utilitaires de détection et de corréler/dédupliquer les résultats pour identifier les zones potentielles de faiblesse de la détection.
-   Explorez un équilibre entre entropie et facilité de détection. Les secrets aux formats cohérents sont plus faciles à détecter avec des taux de faux positifs plus faibles, mais vous ne voulez pas non plus manquer un mot de passe créé par l'homme simplement parce qu'il ne correspond pas à vos règles de détection.

### 8.2 Types de secrets à détecter

Il existe de nombreux types de secrets et vous devez envisager des signatures pour chacun afin de garantir une détection précise pour tous. Parmi les types les plus courants figurent :

-   Secrets de haute disponibilité (Tokens difficiles à faire tourner)
-   Fichiers de configuration des applications
-   Chaînes de connexion
-   Clés API
-   Informations d'identification
-   Mots de passe
-   Clés 2FA
-   Clés privées (par exemple, clés SSH)
-   Jetons de session
-   Types de secrets spécifiques à la plate-forme (par exemple, Amazon Web Services, Google Cloud)

Pour en savoir plus sur les secrets et vous entraîner à les extraire, consultez le[Mauvais secrets](https://owasp.org/www-project-wrongsecrets/)projet.

### 8.3 Cycle de vie de la détection

Les secrets sont comme n’importe quel autre jeton d’autorisation. Ils devraient:

-   Exister seulement aussi longtemps que nécessaire (roter souvent)
-   Avoir une méthode de rotation automatique
-   Soyez visible uniquement par ceux qui en ont besoin (moindre privilège)
-   Être révocable (y compris la journalisation des tentatives d'utilisation d'un secret révoqué)
-   Ne jamais être connecté (doit mettre en œuvre une approche de cryptage ou de masquage pour éviter de consigner des secrets en texte clair)

Créez des règles de détection pour chacune des étapes du cycle de vie du secret.

### 8.4 Documentation sur la façon de détecter les secrets

Créez une documentation et mettez-la à jour régulièrement pour informer la communauté des développeurs sur les procédures et les systèmes disponibles dans votre organisation et les types de gestion des secrets que vous attendez, comment tester les secrets et que faire en cas de secrets détectés.

Les documents doivent :

-   Exister et être mis à jour souvent, notamment en réponse à un incident

-   Incluez les informations suivantes :
    -   Qui a accès au secret
    -   Comment il tourne
    -   Toute dépendance en amont ou en aval qui pourrait potentiellement être rompue lors de la rotation secrète
    -   Qui est le point de contact lors d’un incident
    -   Impact de l’exposition sur la sécurité

-   Identifiez quand les secrets peuvent être traités différemment en fonction du risque de menace, de la classification des données, etc.

## 9 Réponse aux incidents

Une réponse rapide en cas de révélation d’un secret est peut-être l’une des considérations les plus critiques en matière de gestion des secrets.

### 9.1 Documents

La réponse aux incidents en cas de révélation d'un secret doit garantir que toutes les personnes impliquées dans la chaîne de contrôle sont conscientes et comprennent comment réagir. Cela inclut les créateurs d'applications (chaque membre d'une équipe de développement), la sécurité des informations et le leadership technologique.

La documentation doit inclure :

-   Comment tester les secrets et leur traitement, en particulier lors des revues de continuité d'activité.
-   Qui alerter lorsqu’un secret est détecté.
-   Les mesures à prendre pour le confinement
-   Informations à enregistrer pendant l'événement

### 9.2 Correction

L’objectif principal de la réponse aux incidents est une réponse rapide et un confinement.

Le confinement doit suivre ces procédures :

1.  Révocation : les clés qui ont été exposées doivent être immédiatement révoquées. Le secret doit pouvoir être rapidement annulé et des systèmes doivent être en place pour identifier le statut de révocation.
2.  Rotation : un nouveau secret doit pouvoir être créé et mis en œuvre rapidement, de préférence via un processus automatisé pour garantir la répétabilité, un faible taux d'erreur de mise en œuvre et le moindre privilège (non directement lisible par l'homme).
3.  Suppression : les secrets révoqués/permutés doivent être immédiatement supprimés du système exposé, y compris les secrets découverts dans le code ou les journaux. Les secrets dans le code pourraient avoir un historique de validation pour l'exposition écrasé avant l'introduction du secret, cependant, cela peut introduire d'autres problèmes car il réécrit l'historique de git et rompra tout autre lien vers une validation donnée. Si vous décidez de le faire, soyez conscient des conséquences et planifiez en conséquence. Les secrets dans les journaux doivent disposer d'un processus permettant de supprimer le secret tout en préservant l'intégrité des journaux.
4.  Journalisation : les équipes de réponse aux incidents doivent avoir accès aux informations sur le cycle de vie d'un secret pour faciliter le confinement et la remédiation, notamment :
    -   Qui y avait accès ?
    -   Quand l’ont-ils utilisé ?
    -   Quand a-t-il été tourné précédemment ?

### 9.3 Journalisation

Des considérations supplémentaires concernant la journalisation de l'utilisation des secrets doivent inclure :

-   La journalisation de la réponse aux incidents doit être effectuée dans un emplacement unique accessible aux équipes de réponse aux incidents (IR).
-   Assurer la fidélité des informations de journalisation lors des exercices de l'équipe violette tels que :
    -   Qu'est-ce qui aurait dû être enregistré ?
    -   Qu’est-ce qui a été réellement enregistré ?
    -   Avons-nous mis en place des alertes adéquates pour garantir cela ?

Envisagez d'utiliser un format de journalisation et un vocabulaire standardisés tels que[Aide-mémoire sur le vocabulaire de la journalisation](Logging_Vocabulary_Cheat_Sheet.md)pour garantir que toutes les informations nécessaires sont enregistrées.

## 10 aide-mémoire connexes et lectures complémentaires

-   [Aide-mémoire pour la gestion des clés](Key_Management_Cheat_Sheet.md)
-   [Aide-mémoire sur la journalisation](Logging_Cheat_Sheet.md)
-   [Aide-mémoire pour le stockage des mots de passe](Password_Storage_Cheat_Sheet.md)
-   [Aide-mémoire sur le stockage cryptographique](Cryptographic_Storage_Cheat_Sheet.md)
-   [Projet OWASP WrongSecrets](https://github.com/OWASP/wrongsecrets/)
-   [Blog : 10 conseils sur la gestion des secrets](https://xebia.com/blog/secure-deployment-10-pointers-on-secrets-management/)
-   [Blog : De la construction à l'exécution : conseils sur le déploiement sécurisé](https://xebia.com/from-build-to-run-pointers-on-secure-deployment/)
-   [Liste Github sur les outils de détection de secrets](https://github.com/topics/secrets-detection)
-   [Références OpenCRE aux secrets](https://www.opencre.org/search/secret)
-   [Recommandation NIST SP 800-57 pour la gestion des clés](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
