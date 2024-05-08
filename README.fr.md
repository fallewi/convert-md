# Aide-mémoire de sécurité Kubernetes

## Aperçu

Cette aide-mémoire fournit un point de départ pour sécuriser un cluster Kubernetes. Il est réparti dans les catégories suivantes :

-   Recevoir des alertes pour les mises à jour de Kubernetes
-   INTRODUCTION : Qu'est-ce que Kubernetes ?
-   Sécuriser les hôtes Kubernetes
-   Sécuriser les composants Kubernetes
-   Utiliser le tableau de bord Kubernetes
-   Meilleures pratiques de sécurité Kubernetes : phase de construction
-   Meilleures pratiques de sécurité Kubernetes : phase de déploiement
-   Meilleures pratiques de sécurité Kubernetes : phase d'exécution

Pour plus d'informations sur Kubernetes, reportez-vous à l'annexe.

## Recevez des alertes pour les mises à jour de sécurité et les vulnérabilités de rapport

Rejoignez le groupe kubernetes-announce (<https://kubernetes.io/docs/reference/issues-security/security/>) pour les e-mails concernant les annonces de sécurité. Consultez la page de rapports de sécurité (<https://kubernetes.io/docs/reference/issues-security/security>) pour en savoir plus sur la manière de signaler les vulnérabilités.

## INTRODUCTION : Qu'est-ce que Kubernetes ?

Kubernetes est un moteur d'orchestration de conteneurs open source permettant d'automatiser le déploiement, la mise à l'échelle et la gestion des applications conteneurisées. Le projet open source est hébergé par la Cloud Native Computing Foundation (CNCF).

Lorsque vous déployez Kubernetes, vous obtenez un cluster. Un cluster Kubernetes se compose d'un ensemble de machines de travail, appelées nœuds, qui exécutent des applications conteneurisées. Le plan de contrôle gère les nœuds de travail et les pods du cluster.

### Composants du plan de contrôle

Les composants du plan de contrôle prennent des décisions globales concernant le cluster, ainsi que détectent et répondent aux événements du cluster. Il se compose de composants tels que kube-apiserver, etcd, kube-scheduler, kube-controller-manager et cloud-controller-manager.

Composant : kube-apiserver
Description : expose l'API Kubernetes. Le serveur API est le frontal du plan de contrôle Kubernetes.

Composant : etcd
Description : un magasin de valeurs-clés cohérent et hautement disponible utilisé comme magasin de sauvegarde de Kubernetes pour toutes les données du cluster.

Composant : kube-scheduler
Description : surveille les pods nouvellement créés sans nœud attribué et sélectionne un nœud sur lequel ils pourront s'exécuter.

Composant : kube-controller-manager
Description : exécute les processus du contrôleur. Logiquement, chaque contrôleur est un processus distinct, mais pour réduire la complexité, ils sont tous compilés en un seul binaire et exécutés dans un seul processus.

Composant : gestionnaire de contrôleur cloud
Description : Le gestionnaire de contrôleur cloud vous permet de lier votre cluster à l'API de votre fournisseur cloud et de séparer les composants qui interagissent avec cette plate-forme cloud des composants qui interagissent simplement avec votre cluster.

### Composants de nœud

Les composants de nœud s'exécutent sur chaque nœud, maintenant les pods en cours d'exécution et fournissant l'environnement d'exécution Kubernetes. Il se compose de composants tels que kubelet, kube-proxy et le runtime du conteneur.

Composant : Kubelet
Description : un agent qui s'exécute sur chaque nœud du cluster. Il garantit que les conteneurs s'exécutent dans un Pod.

Composant : kube-proxy
Description : un proxy réseau qui s'exécute sur chaque nœud de votre cluster, implémentant une partie du concept de service Kubernetes.

Conteneur : environnement d'exécution
Description : le runtime du conteneur est le logiciel responsable de l'exécution des conteneurs |

## SECTION 1 : Sécuriser les hôtes Kubernetes

Kubernetes peut être déployé de différentes manières : sur système nu, sur site et dans le cloud public (une version Kubernetes personnalisée sur des machines virtuelles OU un service géré). Kubernetes étant conçu pour être hautement portable, les clients peuvent facilement migrer leurs charges de travail et basculer entre plusieurs installations.

Étant donné que Kubernetes peut être conçu pour s'adapter à une grande variété de scénarios, cette flexibilité constitue une faiblesse lorsqu'il s'agit de sécuriser les clusters Kubernetes. Les ingénieurs responsables du déploiement de la plateforme Kubernetes doivent connaître tous les vecteurs d'attaque potentiels et les vulnérabilités de leurs clusters.

Pour renforcer les hôtes sous-jacents des clusters Kubernetes, nous vous recommandons d'installer la dernière version des systèmes d'exploitation, de renforcer les systèmes d'exploitation, de mettre en œuvre les systèmes de gestion des correctifs et de gestion de configuration nécessaires, de mettre en œuvre les règles de pare-feu essentielles et de prendre des mesures de sécurité spécifiques basées sur le centre de données.

### Mise à jour de Kubernetes

Puisque personne ne peut suivre tous les vecteurs d’attaque potentiels de votre cluster Kubernetes, la première et la meilleure défense consiste à toujours exécuter la dernière version stable de Kubernetes.

Si des vulnérabilités sont détectées dans les conteneurs en cours d'exécution, il est recommandé de toujours mettre à jour l'image source et de redéployer les conteneurs.**Essayez d'éviter les mises à jour directes des conteneurs en cours d'exécution, car cela peut rompre la relation image-conteneur.**

    Example: apt-update

**La mise à niveau des conteneurs est extrêmement simple grâce à la fonctionnalité de mises à jour progressives de Kubernetes : cela permet de mettre à jour progressivement une application en cours d'exécution en mettant à niveau ses images vers la dernière version.**

#### Calendrier de sortie pour Kubernetes

Le projet Kubernetes gère les branches de version pour les trois versions mineures les plus récentes et rétroporte les correctifs applicables, y compris les correctifs de sécurité, vers ces trois branches de version, en fonction de la gravité et de la faisabilité. Les versions de correctifs sont supprimées de ces branches à une cadence régulière, ainsi que des versions urgentes supplémentaires, si nécessaire. Il est donc toujours recommandé de mettre à niveau le cluster Kubernetes vers la dernière version stable disponible. Il est recommandé de se référer à la politique de version asymétrique pour plus de détails.<https://kubernetes.io/docs/setup/release/version-skew-policy/>.

Il existe plusieurs techniques, telles que les mises à jour progressives et les migrations de pools de nœuds, qui vous permettent d'effectuer une mise à jour avec un minimum d'interruptions et de temps d'arrêt.

\--

## SECTION 2 : Sécurisation des composants Kubernetes

Cette section explique comment sécuriser les composants Kubernetes. Il couvre les sujets suivants :

-   Sécuriser le tableau de bord Kubernetes
-   Restreindre l'accès à etcd (Important)
-   Contrôler l'accès réseau aux ports sensibles
-   Contrôler l'accès à l'API Kubernetes
-   Implémentation du contrôle d'accès basé sur les rôles dans Kubernetes
-   Limiter l'accès aux Kubelets

\--

### Sécuriser le tableau de bord Kubernetes

Le tableau de bord Kubernetes est une application Web permettant de gérer votre cluster. Il ne fait pas partie du cluster Kubernetes lui-même, il doit être installé par les propriétaires du cluster. Il existe donc de nombreux tutoriels expliquant comment procéder. Malheureusement, la plupart d’entre eux créent un compte de service doté de privilèges très élevés. Cela a provoqué le piratage de Tesla et de quelques autres via un tableau de bord K8 si mal configuré. (Référence : les ressources cloud de Tesla sont piratées pour exécuter des logiciels malveillants d'extraction de crypto-monnaie -<https://arstechnica.com/information-technology/2018/02/tesla-cloud-resources-are-hacked-to-run-cryptocurrency-mining-malware/>)

Pour éviter les attaques via le tableau de bord, vous devez suivre quelques conseils :

-   N'exposez pas le tableau de bord sans authentification supplémentaire au public. Il n'est pas nécessaire d'accéder à un outil aussi puissant depuis l'extérieur de votre réseau local.
-   Activez le contrôle d'accès basé sur les rôles (voir ci-dessous), afin de pouvoir limiter le compte de service utilisé par le tableau de bord.
-   N'accordez pas de privilèges élevés au compte de service du tableau de bord
-   Accordez des autorisations par utilisateur, afin que chaque utilisateur ne puisse voir que ce qu'il est censé voir
-   Si vous utilisez des politiques réseau, vous pouvez bloquer les requêtes vers le tableau de bord même à partir de pods internes (cela n'affectera pas le tunnel proxy via le proxy kubectl)
-   Avant la version 1.8, le tableau de bord disposait d'un compte de service avec tous les privilèges, vérifiez donc qu'il n'y a plus de liaison de rôle pour l'administrateur du cluster.
-   Déployez le tableau de bord avec un proxy inverse d'authentification, avec l'authentification multifacteur activée. Cela peut être fait avec l'OIDC intégré`id_tokens`ou en utilisant l'usurpation d'identité Kubernetes. Cela vous permet d'utiliser le tableau de bord avec les informations d'identification de l'utilisateur au lieu d'utiliser un compte privilégié.`ServiceAccount`. Cette méthode peut être utilisée sur des clusters cloud sur site et gérés.

\--

### Restreindre l'accès à etcd (IMPORTANT)

etcd est un composant Kubernetes critique qui stocke des informations sur les états et les secrets, et il doit être protégé différemment du reste de votre cluster. L'accès en écriture à etcd du serveur API équivaut à obtenir la racine sur l'ensemble du cluster, et même l'accès en lecture peut être utilisé pour élever les privilèges assez facilement.

Le planificateur Kubernetes recherchera dans etcd les définitions de pods qui n'ont pas de nœud. Il envoie ensuite les pods trouvés à un kubelet disponible pour la planification. La validation des pods soumis est effectuée par le serveur API avant de les écrire sur etcd, de sorte que les utilisateurs malveillants écrivant directement sur etcd peuvent contourner de nombreux mécanismes de sécurité - par ex. Politiques de sécurité des pods.

Les administrateurs doivent toujours utiliser des informations d'identification solides entre les serveurs API et leur serveur etcd, telles que l'authentification mutuelle via des certificats clients TLS, et il est souvent recommandé d'isoler les serveurs etcd derrière un pare-feu auquel seuls les serveurs API peuvent accéder.

#### Limitation de l'accès à l'instance etcd principale

Autoriser d'autres composants du cluster à accéder à l'instance etcd principale avec un accès en lecture ou en écriture à l'espace de clés complet équivaut à accorder un accès administrateur du cluster. Il est fortement recommandé d'utiliser des instances etcd distinctes pour d'autres composants ou d'utiliser des ACL etcd pour restreindre l'accès en lecture et en écriture à un sous-ensemble de l'espace de clés.

\--

### Contrôler l'accès au réseau aux ports sensibles

Il est fortement recommandé de configurer l'authentification et l'autorisation sur le cluster et les nœuds du cluster. Étant donné que les clusters Kubernetes écoutent généralement sur une série de ports bien définis et distinctifs, il est plus facile pour les attaquants d'identifier les clusters et de les attaquer.

Un aperçu des ports par défaut utilisés dans Kubernetes est fourni ci-dessous. Assurez-vous que votre réseau bloque l'accès aux ports et vous devriez sérieusement envisager de limiter l'accès au serveur API Kubernetes aux réseaux de confiance.

**Nœud(s) du plan de contrôle :**

| Protocole | Plage de ports | But                             |
| --------- | -------------- | ------------------------------- |
| TCP       | 6443-          | Serveur API Kubernetes          |
| TCP       | 2379-2380      | API client du serveur etcd      |
| TCP       | 10250          | API Kubelet                     |
| TCP       | 10251          | au planificateur                |
| TCP       | 10252          | gestionnaire-de-contrôleur-kube |
| TCP       | 10255          | API Kubelet en lecture seule    |

**Nœuds de travail :**

| Protocole | Plage de ports | But                          |
| --------- | -------------- | ---------------------------- |
| TCP       | 10250          | API Kubelet                  |
| TCP       | 10255          | API Kubelet en lecture seule |
| TCP       | 30000-32767    | Services NodePort            |

\--

### Contrôler l'accès à l'API Kubernetes

La première ligne de défense de Kubernetes contre les attaquants consiste à limiter et à sécuriser l'accès aux requêtes API, car ces requêtes sont utilisées pour contrôler la plateforme Kubernetes. Pour plus d'informations, reportez-vous à la documentation sur<https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/>.

Cette partie contient les sujets suivants :

-   Comment Kubernetes gère l'autorisation API
-   Authentification API externe pour Kubernetes (recommandé)
-   Authentification API intégrée Kubernetes (non recommandée)
-   Implémentation d'un accès basé sur les rôles dans Kubernetes
-   Limiter l'accès aux Kubelets

\--

#### Comment Kubernetes gère l'autorisation API

Dans Kubernetes, vous devez être authentifié (connecté) avant que votre demande puisse être autorisée (autorisation d'accès accordée), et Kubernetes attend des attributs communs aux requêtes de l'API REST. Cela signifie que les systèmes de contrôle d'accès existants à l'échelle de l'organisation ou du fournisseur de cloud, qui peuvent gérer d'autres API, fonctionnent avec l'autorisation Kubernetes.

Lorsque Kubernetes autorise les requêtes API à l'aide du serveur API, les autorisations sont refusées par défaut. Il évalue tous les attributs de la demande par rapport à toutes les politiques et autorise ou refuse la demande. Toutes les parties d'une requête API doivent être autorisées par une politique pour pouvoir continuer.

\--

#### Authentification API externe pour Kubernetes (RECOMMANDÉ)

En raison de la faiblesse des mécanismes internes de Kubernetes pour authentifier les API, nous recommandons fortement aux clusters de plus grande taille ou de production d'utiliser l'une des méthodes d'authentification des API externes.

-   [Connexion OpenID](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens)(OIDC) vous permet d'externaliser l'authentification, d'utiliser des jetons de courte durée et d'exploiter des groupes centralisés pour l'autorisation.
-   Les distributions Kubernetes gérées telles que GKE, EKS et AKS prennent en charge l'authentification à l'aide des informations d'identification de leurs fournisseurs IAM respectifs.
-   [Usurpation d’identité Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)peut être utilisé avec des clusters cloud gérés et des clusters sur site pour externaliser l'authentification sans avoir à accéder aux paramètres de configuration du serveur API.

En plus de choisir le système d'authentification approprié, l'accès aux API doit être considéré comme privilégié et utiliser l'authentification multifacteur (MFA) pour tous les accès utilisateur.

Pour plus d'informations, consultez la documentation de référence sur l'authentification Kubernetes à l'adresse<https://kubernetes.io/docs/reference/access-authn-authz/authentication>.

\--

#### Options pour l'authentification API intégrée Kubernetes (NON RECOMMANDÉ)

Kubernetes fournit un certain nombre de mécanismes internes pour l'authentification du serveur API, mais ceux-ci ne conviennent généralement qu'aux non-production ou aux petits clusters. Nous discuterons brièvement de chaque mécanisme interne et expliquerons pourquoi vous ne devriez pas les utiliser.

-   [Fichier de jeton statique](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file): L'authentification utilise des jetons en texte clair stockés dans un fichier CSV sur le(s) nœud(s) du serveur API. AVERTISSEMENT : vous ne pouvez pas modifier les informations d'identification dans ce fichier tant que le serveur API n'est pas redémarré.

-   [Certificats clients X509](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#x509-client-certs)sont disponibles mais ne conviennent pas à une utilisation en production, car Kubernetes ne le fait pas.[ne prend pas en charge la révocation de certificat](https://github.com/kubernetes/kubernetes/issues/18982). Par conséquent, ces informations d'identification utilisateur ne peuvent pas être modifiées ou révoquées sans faire tourner la clé de l'autorité de certification racine et réémettre tous les certificats de cluster.

-   [Jetons de comptes de service](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)sont également disponibles pour l'authentification. Leur utilisation principale est de permettre aux charges de travail exécutées dans le cluster de s'authentifier auprès du serveur API, mais ils peuvent également être utilisés pour l'authentification des utilisateurs.

\--

### Implémentation du contrôle d'accès basé sur les rôles dans Kubernetes

Le contrôle d'accès basé sur les rôles (RBAC) est une méthode permettant de réguler l'accès aux ressources informatiques ou réseau en fonction des rôles des utilisateurs individuels au sein de votre organisation. Heureusement, Kubernetes est livré avec un composant intégré de contrôle d'accès basé sur les rôles (RBAC) avec des rôles par défaut qui vous permettent de définir les responsabilités des utilisateurs en fonction des actions qu'un client pourrait souhaiter effectuer. Vous devez utiliser les autorisateurs Node et RBAC ensemble en combinaison avec le plugin d'admission NodeRestriction.

Le composant RBAC associe un utilisateur ou un groupe entrant à un ensemble d'autorisations liées aux rôles. Ces autorisations combinent des verbes (obtenir, créer, supprimer) avec des ressources (pods, services, nœuds) et peuvent être limitées à un espace de noms ou à un cluster. L'autorisation RBAC utilise le groupe d'API rbac.authorization.k8s.io pour piloter les décisions d'autorisation, vous permettant de configurer dynamiquement des stratégies via l'API Kubernetes.

Pour activer RBAC, démarrez le serveur API avec l'indicateur --authorization-mode défini sur une liste séparée par des virgules qui inclut RBAC ; Par exemple:

```bash
kube-apiserver --authorization-mode=Example,RBAC --other-options --more-options
```

Pour des exemples détaillés d'utilisation de RBAC, reportez-vous à la documentation Kubernetes à l'adresse<https://kubernetes.io/docs/reference/access-authn-authz/rbac>

\--

### Limiter l'accès aux Kubelets

Les Kubelets exposent des points de terminaison HTTPS qui accordent un contrôle puissant sur le nœud et les conteneurs. Par défaut, les Kubelets autorisent un accès non authentifié à cette API. Les clusters de production doivent activer l'authentification et l'autorisation Kubelet.

Pour plus d'informations, reportez-vous à la documentation d'authentification/autorisation Kubelet à l'adresse<https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/>

\--

## SECTION 3 : Meilleures pratiques de sécurité Kubernetes : phase de construction

Pendant la phase de construction, vous devez sécuriser vos images de conteneur Kubernetes en créant des images sécurisées et en analysant ces images pour détecter toute vulnérabilité connue.

\--

### Qu'est-ce qu'une image de conteneur ?

Une image de conteneur (CI) est un package logiciel immuable, léger, autonome et exécutable qui comprend tout ce dont vous avez besoin pour exécuter une application : code, environnement d'exécution, outils système, bibliothèques système et paramètres.[<https://www.docker.com/resources/what-container>]. Chaque image partage le noyau du système d'exploitation présent dans la machine hôte.

Vos CI doivent être construits sur une image de base approuvée et sécurisée. Cette image de base doit être numérisée et surveillée à intervalles réguliers pour garantir que tous les CI reposent sur une image sécurisée et authentique. Mettez en œuvre des politiques de gouvernance solides qui déterminent la manière dont les images sont créées et stockées dans des registres d’images fiables.

\--

#### Assurez-vous que les CI sont à jour

Assurez-vous que vos images (et tous les outils tiers que vous incluez) sont à jour et utilisent les dernières versions de leurs composants.

\--

### Utilisez uniquement des images autorisées dans votre environnement

Télécharger et exécuter des CI à partir de sources inconnues est très dangereux. Assurez-vous que seules les images adhérant à la politique de l’organisation sont autorisées à s’exécuter, sinon l’organisation s’expose au risque d’exécuter des conteneurs vulnérables, voire malveillants.

\--

### Utilisez un pipeline CI pour contrôler et identifier les vulnérabilités

Le registre de conteneurs Kubernetes sert de référentiel central de toutes les images de conteneurs du système. Selon vos besoins, vous pouvez utiliser un référentiel public ou disposer d'un référentiel privé comme registre de conteneurs. Nous vous recommandons de stocker vos images approuvées dans un registre privé et de transférer uniquement les images approuvées vers ces registres, ce qui réduit automatiquement le nombre d'images potentielles entrant dans votre pipeline à une fraction des centaines de milliers d'images accessibles au public.

En outre, nous vous recommandons fortement d'ajouter un pipeline CI qui intègre une évaluation de la sécurité (comme l'analyse des vulnérabilités) dans le processus de génération. Ce pipeline doit vérifier tout le code approuvé pour la production et utilisé pour créer les images. Une fois qu'une image est créée, elle doit être analysée pour détecter les failles de sécurité. Ce n'est que si aucun problème n'est détecté que l'image sera transférée vers un registre privé puis déployée en production. Si le mécanisme d'évaluation de la sécurité échoue dans un code, cela devrait créer une défaillance dans le pipeline, ce qui vous aidera à trouver les images présentant des problèmes de sécurité et à les empêcher d'entrer dans le registre d'images.

De nombreux référentiels de code source offrent des capacités d'analyse (par ex.[GitHub](https://docs.github.com/en/code-security/supply-chain-security),[GitLab](https://docs.gitlab.com/ee/user/application_security/container_scanning/index.html)), et de nombreux outils CI offrent une intégration avec des scanners de vulnérabilités open source tels que[Voyages](https://github.com/aquasecurity/trivy)ou[Grype](https://github.com/anchore/grype).

Les projets développent des plugins d'autorisation d'image pour Kubernetes qui empêchent l'expédition d'images non autorisées. Pour plus d'informations, reportez-vous au PR<https://github.com/kubernetes/kubernetes/pull/27129>.

\--

### Réduire les fonctionnalités dans tous les CI

À titre de bonne pratique, Google et d’autres géants de la technologie limitent strictement le code dans leur conteneur d’exécution depuis des années. Cette approche améliore le rapport signal/bruit des scanners (par exemple CVE) et réduit la charge d'établissement de la provenance à ce dont vous avez juste besoin.

Pensez à utiliser des CI minimaux tels que des images sans distribution (voir ci-dessous). Si cela n'est pas possible, n'incluez pas les gestionnaires de packages de système d'exploitation ou les shells dans les CI, car ils peuvent présenter des vulnérabilités inconnues. Si vous devez absolument inclure des packages de système d'exploitation, supprimez le gestionnaire de packages à une étape ultérieure du processus de génération.

\--

#### Utilisez des images sans distribution ou vides lorsque cela est possible

Les images sans distribution réduisent considérablement la surface d'attaque car elles n'incluent pas de shells et contiennent moins de packages que les autres images. Pour plus d'informations sur les images sans distribution, reportez-vous à<https://github.com/GoogleContainerTools/distroless>.

Une image vide, idéale pour les langages compilés statiquement comme Go, car l'image est vide - la surface d'attaque est vraiment minime - seulement votre code !

Pour plus d'informations, reportez-vous à<https://hub.docker.com/_/scratch>

* * *

## SECTION 4 : Meilleures pratiques de sécurité Kubernetes : phase de déploiement

Une fois une infrastructure Kubernetes en place, vous devez la configurer de manière sécurisée avant de déployer des charges de travail. Et lorsque vous configurez votre infrastructure, assurez-vous d'avoir une visibilité sur les CI déployés et sur la manière dont ils sont déployés, sinon vous ne serez pas en mesure d'identifier et de répondre aux violations des politiques de sécurité. Avant le déploiement, votre système doit connaître et être en mesure de vous indiquer :

-   **Ce qui est déployé**- y compris des informations sur l'image utilisée, telles que les composants ou les vulnérabilités, et les pods qui seront déployés.
-   **Où il va être déployé**- quels clusters, espaces de noms et nœuds.
-   **Comment il est déployé**- s'il s'exécute de manière privilégiée, avec quels autres déploiements il peut communiquer, le contexte de sécurité du pod appliqué, le cas échéant.
-   **À quoi il peut accéder**- y compris les secrets, les volumes et d'autres composants d'infrastructure tels que l'API de l'hôte ou de l'orchestrateur.
-   **Est-ce conforme ?**- s'il est conforme à vos politiques et exigences de sécurité.

\--

### Code qui utilise des espaces de noms pour isoler les ressources Kubernetes

Les espaces de noms vous donnent la possibilité de créer des partitions logiques, d'imposer la séparation de vos ressources et de limiter la portée des autorisations des utilisateurs.

\--

#### Définition de l'espace de noms pour une requête

Pour définir l'espace de noms d'une requête en cours, utilisez l'indicateur --namespace. Reportez-vous aux exemples suivants :

```bash
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

\--

#### Définition de la préférence d'espace de noms

Vous pouvez enregistrer définitivement l'espace de noms pour toutes les commandes kubectl suivantes dans ce contexte avec :

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```

Validez-le ensuite avec la commande suivante :

```bash
kubectl config view --minify | grep namespace:
```

En savoir plus sur les espaces de noms sur<https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces>

\--

### Utilisez ImagePolicyWebhook pour gérer la provenance des images

Nous vous recommandons fortement d'utiliser le contrôleur d'admission ImagePolicyWebhook pour empêcher l'utilisation d'images non approuvées, rejeter les pods qui utilisent des images non approuvées et refuser les CI qui répondent aux critères suivants :

-   Images qui n'ont pas été numérisées récemment
-   Images qui utilisent une image de base qui n'est pas explicitement autorisée
-   Images provenant de registres non sécurisés

En savoir plus sur le webhook sur<https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook>

\--

### Mettre en œuvre une analyse continue des vulnérabilités de sécurité

Étant donné que de nouvelles vulnérabilités sont constamment découvertes, vous ne savez pas toujours si vos conteneurs peuvent contenir des vulnérabilités récemment divulguées (CVE) ou des packages obsolètes. Pour maintenir une sécurité renforcée, effectuez régulièrement une analyse de production des conteneurs propriétaires (applications que vous avez créées et analysées précédemment) ainsi que des conteneurs tiers (provenant de référentiels et de fournisseurs fiables).

Des projets Open Source tels que[ThreatMapper](https://github.com/deepfence/ThreatMapper)peut aider à identifier et à hiérarchiser les vulnérabilités.

\--

### Évaluer en continu les privilèges utilisés par les conteneurs

Nous recommandons fortement que tous vos conteneurs adhèrent au principe du moindre privilège, car votre risque de sécurité est fortement influencé par les capacités, les liaisons de rôle et les privilèges accordés aux conteneurs. Chaque conteneur ne doit disposer que des privilèges et capacités minimaux qui lui permettent de remplir la fonction prévue.

**Utilisez les stratégies de sécurité des pods pour contrôler les attributs liés à la sécurité des pods, qui incluent les niveaux de privilèges des conteneurs.**

Toutes les politiques de sécurité doivent inclure les conditions suivantes :

-   Les processus d'application ne s'exécutent pas en tant que root.
-   L’élévation des privilèges n’est pas autorisée.
-   Le système de fichiers racine est en lecture seule.
-   Le montage du système de fichiers /proc par défaut (masqué) est utilisé.
-   Le réseau hôte ou l'espace de processus ne doivent PAS être utilisés - en utilisant`hostNetwork: true`entraînera l'ignorance des NetworkPolicies puisque le Pod utilisera son réseau hôte.
-   Les fonctionnalités Linux inutilisées et inutiles sont éliminées.
-   Utilisez les options SELinux pour des contrôles de processus plus précis.
-   Attribuez à chaque application son propre compte de service Kubernetes.
-   Si un conteneur n'a pas besoin d'accéder à l'API Kubernetes, ne le laissez pas monter les informations d'identification du compte de service.

Pour plus d'informations sur les politiques de sécurité des pods, consultez la documentation sur<https://kubernetes.io/docs/concepts/policy/pod-security-policy/>.

\--

### Appliquez un contexte de sécurité à vos pods et conteneurs

Le contexte de sécurité est une propriété définie dans le yaml de déploiement et contrôle les paramètres de sécurité pour tous les pods/conteneurs/volumes, et il doit être appliqué dans toute votre infrastructure. Lorsque la propriété de contexte de sécurité est correctement implémentée partout, elle peut éliminer des classes entières d’attaques qui reposent sur un accès privilégié. Par exemple, toute attaque qui dépend de l'installation d'un logiciel ou de l'écriture sur le système de fichiers sera stoppée si vous spécifiez des systèmes de fichiers racine en lecture seule dans le contexte de sécurité.

Lorsque vous configurez le contexte de sécurité de vos pods, accordez uniquement les privilèges nécessaires au fonctionnement des ressources dans vos conteneurs et volumes. Certains des paramètres importants de la propriété de contexte de sécurité sont :

Paramètres de contexte de sécurité :

1.  SecurityContext->runAsNonRoot
    Description : indique que les conteneurs doivent être exécutés en tant qu'utilisateur non root.

2.  Paramètre de contexte de sécurité : SecurityContext->Capacités
    Description : contrôle les fonctionnalités Linux attribuées au conteneur.

3.  Paramètre de contexte de sécurité : SecurityContext->readOnlyRootFilesystem
    Description : contrôle si un conteneur pourra écrire dans le système de fichiers racine.

4.  Paramètre de contexte de sécurité : PodSecurityContext->runAsNonRoot
    Description : empêche l'exécution d'un conteneur avec un utilisateur « root » dans le cadre du pod |

#### Exemple de contexte de sécurité : une définition de pod qui inclut des paramètres de contexte de sécurité

```yaml
apiVersion: v1

kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  # specification of the pod’s containers
  # ...
  # ...
  # Security Context
  securityContext:
    readOnlyRootFilesystem: true
    runAsNonRoot: true
```

Pour plus d'informations sur le contexte de sécurité des pods, reportez-vous à la documentation sur<https://kubernetes.io/docs/tasks/configure-pod-container/security-context>

### Fournir une sécurité supplémentaire avec un maillage de services

Un maillage de services est une couche d'infrastructure capable de gérer les communications entre les services dans les applications de manière rapide, sécurisée et fiable, ce qui peut contribuer à réduire la complexité de la gestion des microservices et des déploiements. Ils offrent un moyen uniforme de sécuriser, de connecter et de surveiller les microservices. et un maillage de services est idéal pour résoudre les défis et problèmes opérationnels lors de l’exécution de ces conteneurs et microservices.

#### Avantages d'un maillage de services

Un maillage de services offre les avantages suivants :

1.  Observabilité

Il génère des métriques de traçage et de télémétrie, qui facilitent la compréhension de votre système et provoquent rapidement tout problème.

2.  Fonctions de sécurité spécialisées

Il fournit des fonctionnalités de sécurité qui identifient rapidement tout trafic compromettant qui entre dans votre cluster et peut sécuriser les services au sein de votre réseau s'ils sont correctement mis en œuvre. Il peut également vous aider à gérer la sécurité via mTLS, le contrôle des entrées et des sorties, et bien plus encore.

3.  Possibilité de sécuriser les microservices avec mTLS

Étant donné que la sécurisation des microservices est difficile, il existe de nombreux outils qui abordent la sécurité des microservices. Cependant, le maillage de services constitue la solution la plus élégante pour gérer le chiffrement du trafic filaire au sein du réseau.

Il assure une défense grâce au cryptage TLS mutuel (mTLS) du trafic entre vos services, et le maillage peut automatiquement crypter et déchiffrer les requêtes et les réponses, ce qui élimine ce fardeau pour les développeurs d'applications. Le maillage peut également améliorer les performances en donnant la priorité à la réutilisation des connexions existantes et persistantes, ce qui réduit le besoin de création de nouvelles, coûteuses en termes de calcul. Avec le maillage de services, vous pouvez sécuriser le trafic sur le réseau et également effectuer une authentification et des autorisations fortes basées sur l'identité pour chaque microservice.

Nous constatons qu'un maillage de services a beaucoup de valeur pour les entreprises, car un maillage vous permet de voir si mTLS est activé et fonctionne entre chacun de vos services. De plus, vous pouvez recevoir des alertes immédiates si l'état de sécurité change.

4.  Contrôle d'entrée et de sortie

Il vous permet de surveiller et de traiter le trafic compromettant lorsqu'il traverse le maillage. Par exemple, si Istio s'intègre à Kubernetes en tant que contrôleur d'entrée, il peut prendre en charge l'équilibrage de charge pour l'entrée. Cela permet aux défenseurs d'ajouter un niveau de sécurité au périmètre avec des règles d'entrée, tandis que le contrôle de sortie vous permet de voir et de gérer les services externes et de contrôler la manière dont vos services interagissent avec le trafic.

5.  Contrôle opérationnel

Il peut aider les équipes de sécurité et de plate-forme à définir les macro-contrôles appropriés pour appliquer les contrôles d'accès, tout en permettant aux développeurs d'effectuer les personnalisations dont ils ont besoin pour évoluer rapidement au sein de ces garde-fous.

6.  Capacité à gérer le RBAC

Un maillage de services peut aider les défenseurs à mettre en œuvre un système de contrôle d'accès basé sur les rôles (RBAC) solide, qui constitue sans doute l'une des exigences les plus critiques des grandes organisations d'ingénierie. Même un système sécurisé peut être facilement contourné par des utilisateurs ou des employés trop privilégiés, et un système RBAC peut :

-   Restreindre les utilisateurs privilégiés au minimum de privilèges nécessaires pour exercer leurs responsabilités professionnelles
-   Assurez-vous que l'accès aux systèmes est défini sur « tout refuser » par défaut
-   Aidez les développeurs à s'assurer qu'une documentation appropriée détaillant les rôles et les responsabilités est en place, ce qui constitue l'un des problèmes de sécurité les plus critiques de l'entreprise.

#### Inconvénients du maillage de sécurité

Bien qu’un maillage de services présente de nombreux avantages, il présente également un ensemble unique de défis, dont quelques-uns sont répertoriés ci-dessous :

-   Ajoute une nouvelle couche de complexité

Lorsque des proxys, side-cars et autres composants sont introduits dans un environnement déjà sophistiqué, cela augmente considérablement la complexité du développement et des opérations.

-   Une expertise supplémentaire est requise

Si un maillage comme Istio est ajouté à un orchestrateur tel que Kubernetes, les opérateurs doivent devenir experts dans les deux technologies.

-   Les infrastructures peuvent être ralenties

Un maillage de services étant une technologie invasive et complexe, il peut ralentir considérablement une architecture.

-   Nécessite l'adoption d'une autre plate-forme

Les maillages de services étant invasifs, ils obligent les développeurs et les opérateurs à s’adapter à une plateforme très opiniâtre et à se conformer à ses règles.

### Mise en œuvre d’une gestion centralisée des politiques

Il existe de nombreux projets capables de fournir une gestion centralisée des politiques pour un cluster Kubernetes, notamment le[Agent de stratégie ouverte](https://www.openpolicyagent.org/)(OPA),[Kyverno](https://kyverno.io/), ou la[Validation de la politique d'admission](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)(une fonctionnalité intégrée, mais bêta (c'est-à-dire désactivée par défaut) à partir de la version 1.28). Afin de fournir un exemple plus approfondi, nous nous concentrerons sur OPA dans cet aide-mémoire.

OPA a été lancée en 2016 pour unifier l'application des politiques sur différentes technologies et systèmes, et elle peut être utilisée pour appliquer des politiques sur une plate-forme comme Kubernetes. Actuellement, l'OPA fait partie du CNCF en tant que projet incubateur. Il peut créer une méthode unifiée pour appliquer la politique de sécurité dans la pile. Bien que les développeurs puissent imposer un contrôle précis sur le cluster avec les politiques de sécurité RBAC et Pod, ces technologies s'appliquent uniquement au cluster mais pas en dehors du cluster.

Étant donné qu’OPA est un outil d’application de politiques à usage général, indépendant du domaine et qui n’est basé sur aucun autre projet, les requêtes et décisions de politiques ne suivent pas de format spécifique. Ainsi, il peut être intégré aux API, au démon Linux SSH, à un magasin d'objets comme Ceph, et vous pouvez utiliser n'importe quelle donnée JSON valide comme attributs de requête tant qu'elle fournit les données requises. OPA vous permet de choisir ce qui est en entrée et ce qui est en sortie. Par exemple, vous pouvez choisir qu'OPA renvoie un objet JSON Vrai ou Faux, un nombre, une chaîne ou même un objet de données complexe.

#### Cas d'utilisation les plus courants de l'OPA

##### OPA pour demande d'autorisation

OPA peut fournir aux développeurs une technologie d'autorisation déjà développée afin que l'équipe n'ait pas à en développer une à partir de zéro. Il utilise un langage de politique déclaratif spécialement conçu pour écrire et appliquer des règles telles que « Alice peut écrire dans ce référentiel » ou « Bob peut mettre à jour ce compte ». Cette technologie fournit une riche suite d'outils qui peuvent permettre aux développeurs d'intégrer des politiques dans leurs applications et permettre aux utilisateurs finaux de créer également des politiques pour leurs locataires.

Si vous disposez déjà d’une solution d’autorisation d’application développée en interne, vous ne souhaiterez peut-être pas passer à OPA. Mais si vous souhaitez améliorer l’efficacité des développeurs en passant à une solution qui évolue avec les microservices et vous permet de décomposer les applications monolithiques, vous aurez besoin d’un système d’autorisation distribué et OPA (ou l’un des concurrents associés) pourrait être la réponse.

##### OPA pour le contrôle d'admission Kubernetes

Étant donné que Kubernetes donne aux développeurs un contrôle considérable sur les silos traditionnels de « calcul, réseau et stockage », ils peuvent l'utiliser pour configurer leur réseau exactement comme ils le souhaitent et configurer le stockage exactement comme ils le souhaitent. Mais cela signifie que les administrateurs et les équipes de sécurité doivent s’assurer que les développeurs ne se tirent pas une balle dans le pied (ou ne tirent pas une balle dans le pied de leurs voisins).

OPA peut résoudre ces problèmes de sécurité en permettant à la sécurité d'élaborer des politiques qui exigent que toutes les images de conteneurs proviennent de sources fiables, empêchent les développeurs d'exécuter des logiciels en tant que root, s'assurent que le stockage est toujours marqué avec le bit de chiffrement et que le stockage n'est pas supprimé simplement parce qu'un Le pod est redémarré, ce qui limite l'accès à Internet, etc.

Cela peut également permettre aux administrateurs de s’assurer que les changements de politique ne causent pas par inadvertance plus de dégâts que de bien. OPA s'intègre directement au serveur API Kubernetes et dispose de l'autorité complète pour rejeter toute ressource qui, selon la politique d'admission, n'appartient pas à un cluster, qu'elle soit liée au calcul, au réseau, au stockage, etc. peut être exécuté hors bande pour surveiller les résultats et les politiques de l'OPA peuvent être exposées tôt dans le cycle de vie du développement (par exemple, le pipeline CICD ou même sur les ordinateurs portables des développeurs) si les développeurs ont besoin de commentaires précoces.

##### OPA pour l'autorisation du maillage de services

Et enfin, OPA peut réguler l’utilisation des architectures de service mesh. Souvent, les administrateurs s'assurent que les réglementations de conformité sont respectées en intégrant des politiques dans le maillage de services, même lorsque la modification du code source est impliquée. Même si vous n'intégrez pas OPA pour implémenter la logique d'autorisation des applications (le principal cas d'utilisation évoqué ci-dessus), vous pouvez contrôler les microservices des API en insérant des politiques d'autorisation dans le maillage de services. Mais si la sécurité vous motive, vous pouvez mettre en œuvre des politiques dans le maillage de services pour limiter les mouvements latéraux au sein d’une architecture de microservices.

### Limiter l'utilisation des ressources sur un cluster

Il est important de définir des quotas de ressources pour les conteneurs dans Kubernetes, car toutes les ressources d'un cluster Kubernetes sont créées par défaut avec des limites de CPU et des demandes/limites de mémoire illimitées. Si vous exécutez des conteneurs sans ressources, votre système sera exposé à un risque de déni de service (DoS) ou de scénarios de « voisin bruyant ». Heureusement, OPA peut utiliser des quotas de ressources sur un espace de noms, ce qui limitera le nombre ou la capacité des ressources accordées à cet espace de noms et restreindra cet espace de noms en définissant sa capacité de processeur, sa mémoire ou son espace disque persistant.

De plus, l'OPA peut limiter le nombre de pods, de services ou de volumes existants dans chaque espace de noms, ainsi que la taille maximale ou minimale de certaines des ressources ci-dessus. Les quotas de ressources fournissent des limites par défaut lorsqu'aucune n'est spécifiée et empêchent les utilisateurs de demander des valeurs déraisonnablement élevées ou faibles pour des ressources couramment réservées comme la mémoire.

Vous trouverez ci-dessous un exemple de définition d'un quota de ressources d'espace de noms dans le yaml approprié. Il limite le nombre de pods dans l'espace de noms à 4, limite leurs demandes de CPU entre 1 et 2 et les demandes de mémoire entre 1 Go et 2 Go.

`compute-resources.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Attribuez un quota de ressources à l'espace de noms :

```bash
kubectl create -f ./compute-resources.yaml --namespace=myspace
```

Pour plus d'informations sur la configuration des quotas de ressources, reportez-vous à la documentation Kubernetes à l'adresse<https://kubernetes.io/docs/concepts/policy/resource-quotas/>.

### Utiliser les stratégies réseau Kubernetes pour contrôler le trafic entre les pods et les clusters

Si votre cluster exécute différentes applications, une application compromise pourrait attaquer d'autres applications voisines. Ce scénario peut se produire car Kubernetes autorise chaque pod à contacter tous les autres pods par défaut. Si l'entrée depuis un point de terminaison de réseau externe est autorisée, le pod pourra envoyer son trafic vers un point de terminaison en dehors du cluster.

Il est fortement recommandé aux développeurs de mettre en œuvre la segmentation du réseau, car il s'agit d'un contrôle de sécurité clé qui garantit que les conteneurs ne peuvent communiquer qu'avec d'autres conteneurs approuvés et empêche les attaquants de poursuivre un mouvement latéral entre les conteneurs. Cependant, l'application de la segmentation du réseau dans le cloud est difficile en raison de la nature « dynamique » des identités de réseau de conteneurs (IP).

Alors que les utilisateurs de Google Cloud Platform peuvent bénéficier de règles de pare-feu automatiques, qui empêchent la communication entre clusters, d'autres utilisateurs peuvent appliquer des implémentations similaires en déployant sur site à l'aide de pare-feu réseau ou de solutions SDN. En outre, le Kubernetes Network SIG travaille sur des méthodes qui amélioreront considérablement les politiques de communication de pod à pod. Une nouvelle API de politique réseau devrait répondre à la nécessité de créer des règles de pare-feu autour des pods, limitant l'accès au réseau qu'un conteneur peut avoir.

Voici un exemple de stratégie réseau qui contrôle le réseau pour les pods « backend », qui autorise uniquement l'accès au réseau entrant à partir des pods « frontend » :

```json
POST /apis/net.alpha.kubernetes.io/v1alpha1/namespaces/tenant-a/networkpolicys
{
  "kind": "NetworkPolicy",
  "metadata": {
    "name": "pol1"
  },
  "spec": {
    "allowIncoming": {
      "from": [{
        "pods": { "segment": "frontend" }
      }],
      "toPorts": [{
        "port": 80,
        "protocol": "TCP"
      }]
    },
    "podSelector": {
      "segment": "backend"
    }
  }
}
```

Pour plus d'informations sur la configuration des stratégies réseau, reportez-vous à la documentation Kubernetes à l'adresse<https://kubernetes.io/docs/concepts/services-networking/network-policies>.

### Sécuriser les données

#### Gardez les secrets comme des secrets

Il est important de savoir comment les données sensibles telles que les informations d'identification et les clés sont stockées et accessibles dans votre infrastructure. Kubernetes les conserve dans un « secret », qui est un petit objet contenant des données sensibles, comme un mot de passe ou un jeton.

Il est préférable que les secrets soient montés dans des volumes en lecture seule dans vos conteneurs, plutôt que de les exposer en tant que variables d'environnement. De plus, les secrets doivent être séparés d'une image ou d'un pod, sinon toute personne ayant accès à l'image aurait également accès au secret, même si un pod ne peut pas accéder aux secrets d'un autre pod. Les applications complexes qui gèrent plusieurs processus et ont un accès public sont particulièrement vulnérables à cet égard.

#### Chiffrer les secrets au repos

Chiffrez toujours vos sauvegardes à l'aide d'une solution de sauvegarde et de chiffrement bien étudiée et envisagez d'utiliser le chiffrement complet du disque lorsque cela est possible, car la base de données etcd contient toutes les informations accessibles via l'API Kubernetes. L'accès à cette base de données pourrait fournir à un attaquant une visibilité significative sur l'état de votre cluster.

Kubernetes prend en charge le chiffrement au repos, une fonctionnalité introduite dans la version 1.7 et la version bêta v1 depuis la version 1.13, qui chiffrera les ressources secrètes dans etcd et empêchera les parties ayant accès à vos sauvegardes etcd de visualiser le contenu de ces secrets. Bien que cette fonctionnalité soit actuellement en version bêta, elle offre un niveau de défense supplémentaire lorsque les sauvegardes ne sont pas chiffrées ou qu'un attaquant obtient un accès en lecture à etcd.

#### Alternatives aux ressources secrètes de Kubernetes

Étant donné qu'un gestionnaire de secrets externe peut stocker et gérer vos secrets plutôt que de les stocker dans Kubernetes Secrets, vous souhaiterez peut-être envisager cette alternative de sécurité. Un gestionnaire offre un certain nombre d'avantages par rapport à l'utilisation des secrets Kubernetes, notamment la possibilité de gérer les secrets sur plusieurs clusters (ou cloud) et la possibilité de contrôler et de faire pivoter les secrets de manière centralisée.

Pour plus d'informations sur les secrets et leurs alternatives, reportez-vous à la documentation sur<https://kubernetes.io/docs/concepts/configuration/secret/>.

Voir aussi le[Gestion des secrets](Secrets_Management_Cheat_Sheet.md)aide-mémoire pour plus de détails et les meilleures pratiques sur la gestion des secrets.

#### Trouver des secrets révélés

Nous vous recommandons fortement d'examiner les éléments secrets présents sur le conteneur par rapport au principe du « moindre privilège » et d'évaluer le risque posé par une compromission.

N'oubliez pas que les outils open source tels que[Scanner secret](https://github.com/deepfence/SecretScanner)et[ThreatMapper](https://github.com/deepfence/ThreatMapper)peut analyser les systèmes de fichiers de conteneurs à la recherche de ressources sensibles, telles que les jetons API, les mots de passe et les clés. De telles ressources seraient accessibles à tout utilisateur ayant accès au système de fichiers du conteneur non chiffré, que ce soit pendant la construction, au repos dans un registre ou une sauvegarde, ou en cours d'exécution.

* * *

## SECTION 5 : Meilleures pratiques de sécurité Kubernetes : phase d'exécution

Lorsque l'infrastructure Kubernetes entre dans la phase d'exécution, les applications conteneurisées sont exposées à une multitude de nouveaux défis de sécurité. Vous devez gagner en visibilité sur votre environnement d'exécution afin de pouvoir détecter et répondre aux menaces dès qu'elles surviennent.

Si vous sécurisez de manière proactive vos conteneurs et vos déploiements Kubernetes lors des phases de création et de déploiement, vous pouvez réduire considérablement la probabilité d'incidents de sécurité au moment de l'exécution et les efforts ultérieurs nécessaires pour y répondre.

Tout d’abord, surveillez les activités des conteneurs les plus pertinentes pour la sécurité, notamment :

-   Activité de processus
-   Communications réseau entre services conteneurisés
-   Communications réseau entre les services conteneurisés et les clients et serveurs externes

Détecter les anomalies en observant le comportement des conteneurs est généralement plus facile dans les conteneurs que dans les machines virtuelles en raison de la nature déclarative des conteneurs et de Kubernetes. Ces attributs permettent une introspection plus facile sur ce que vous avez déployé et son activité attendue.

### Utilisez l'admission de sécurité des pods pour empêcher le déploiement de conteneurs/pods à risque

Le recommandé précédemment[Politique de sécurité des pods](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)est obsolète et remplacé par[Admission à la sécurité des pods](https://kubernetes.io/docs/concepts/security/pod-security-admission/), une nouvelle fonctionnalité qui vous permet d'appliquer des politiques de sécurité sur les pods d'un cluster Kubernetes.

Il est recommandé d'utiliser le`baseline`niveau comme exigence de sécurité minimale pour tous les pods afin de garantir un niveau de sécurité standard dans l'ensemble du cluster. Toutefois, les clusters devraient s'efforcer d'appliquer les`restricted`niveau qui suit les meilleures pratiques de durcissement des gousses.

Pour plus d'informations sur la configuration de l'admission de sécurité des pods, reportez-vous à la documentation sur<https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/>.

### Sécurité d'exécution des conteneurs

Si les conteneurs sont renforcés au moment de l'exécution, les équipes de sécurité ont la capacité de détecter et de répondre aux menaces et aux anomalies pendant que les conteneurs ou les charges de travail sont en cours d'exécution. Généralement, cela se fait en interceptant les appels système de bas niveau et en recherchant les événements pouvant indiquer une compromission. Voici quelques exemples d'événements qui devraient déclencher une alerte :

-   Un shell est exécuté dans un conteneur
-   Un conteneur monte un chemin sensible depuis l'hôte tel que /proc
-   Un fichier sensible est lu de manière inattendue dans un conteneur en cours d'exécution tel que /etc/shadow
-   Une connexion réseau sortante est établie

Les outils open source tels que Falco de Sysdig peuvent aider les opérateurs à être opérationnels avec la sécurité d'exécution des conteneurs en fournissant aux défenseurs un grand nombre de détections prêtes à l'emploi ainsi que la possibilité de créer des règles personnalisées.

### Bac à sable de conteneurs

Lorsque les environnements d'exécution de conteneur sont autorisés à effectuer des appels directs au noyau hôte, celui-ci interagit souvent avec le matériel et les périphériques pour répondre à la requête. Bien que les groupes de contrôle et les espaces de noms confèrent aux conteneurs un certain degré d'isolement, le noyau présente toujours une grande surface d'attaque. Lorsque les défenseurs doivent gérer des clusters multi-locataires et très peu fiables, ils ajoutent souvent une couche supplémentaire de sandboxing pour garantir qu'il n'y a pas de fuite de conteneur ni d'exploit du noyau. Ci-dessous, nous explorerons quelques technologies OSS qui permettent d'isoler davantage les conteneurs en cours d'exécution du noyau hôte :

-   Conteneurs Kata : Kata Containers est un projet OSS qui utilise des machines virtuelles simplifiées pour minimiser l'empreinte des ressources et maximiser les performances afin d'isoler davantage les conteneurs.
-   gVisor : gVisor est un noyau plus léger qu'une VM (même simplifié). Il s'agit de son propre noyau indépendant écrit en Go et situé au milieu d'un conteneur et du noyau hôte. Il s'agit d'un bac à sable puissant : gVisor prend en charge environ 70 % des appels système Linux à partir du conteneur, mais utilise UNIQUEMENT environ 20 appels système vers le noyau hôte.
-   Firecracker : Il s’agit d’une VM ultra légère qui s’exécute dans l’espace utilisateur. Puisqu'il est verrouillé par les politiques seccomp, cgroup et d'espace de noms, les appels système sont très limités. Firecracker est conçu dans un souci de sécurité, mais il peut ne pas prendre en charge tous les déploiements Kubernetes ou d'exécution de conteneurs.

### Empêcher les conteneurs de charger des modules de noyau indésirables

Étant donné que le noyau Linux charge automatiquement les modules du noyau à partir du disque si nécessaire dans certaines circonstances, par exemple lorsqu'un élément matériel est connecté ou qu'un système de fichiers est monté, cela peut constituer une surface d'attaque importante. Ce qui est particulièrement pertinent pour Kubernetes, même les processus non privilégiés peuvent provoquer le chargement de certains modules du noyau liés au protocole réseau, simplement en créant un socket du type approprié. Cette situation peut permettre aux attaquants d'exploiter une faille de sécurité dans les modules du noyau que l'administrateur pensait ne pas être utilisés.

Pour empêcher le chargement automatique de modules spécifiques, vous pouvez les désinstaller du nœud ou ajouter des règles pour les bloquer. Sur la plupart des distributions Linux, vous pouvez le faire en créant un fichier tel que`/etc/modprobe.d/kubernetes-blacklist.conf`avec des contenus comme :

```conf
# DCCP is unlikely to be needed, has had multiple serious
# vulnerabilities, and is not well-maintained.
blacklist dccp

# SCTP is not used in most Kubernetes clusters, and has also had
# vulnerabilities in the past.
blacklist sctp
```

Pour bloquer le chargement des modules de manière plus générique, vous pouvez utiliser un module de sécurité Linux (tel que SELinux) pour refuser complètement l'autorisation module_request aux conteneurs, empêchant ainsi le noyau de charger des modules pour les conteneurs en aucune circonstance. (Les pods pourraient toujours utiliser des modules chargés manuellement ou des modules chargés par le noyau pour le compte d'un processus plus privilégié).

### Comparez et analysez différentes activités d'exécution dans les pods des mêmes déploiements

Lorsque des applications conteneurisées sont répliquées pour des raisons de haute disponibilité, de tolérance aux pannes ou d'évolutivité, ces réplicas doivent se comporter presque de la même manière. Si une réplique présente des écarts significatifs par rapport aux autres, les défenseurs souhaiteraient une enquête plus approfondie. Votre outil de sécurité Kubernetes doit être intégré à d'autres systèmes externes (e-mail, PagerDuty, Slack, Google Cloud Security Command Center, SIEM).[gestion des informations de sécurité et des événements], etc.) et exploitez les étiquettes ou annotations de déploiement pour alerter l'équipe responsable d'une application donnée lorsqu'une menace potentielle est détectée. Si vous choisissez de faire appel à un fournisseur commercial de sécurité Kubernetes, celui-ci doit prendre en charge un large éventail d'intégrations avec des outils externes.

### Surveillez le trafic réseau pour limiter les communications inutiles ou non sécurisées

Les applications conteneurisées utilisent généralement largement la mise en réseau en cluster. L'observation du trafic réseau actif est donc un bon moyen de comprendre comment les applications interagissent les unes avec les autres et d'identifier les communications inattendues. Vous devez observer votre trafic réseau actif et comparer ce trafic à ce qui est autorisé en fonction de vos politiques réseau Kubernetes.

Dans le même temps, comparer le trafic actif avec ce qui est autorisé vous donne des informations précieuses sur ce qui ne se produit pas mais qui est autorisé. Grâce à ces informations, vous pouvez renforcer davantage vos politiques réseau autorisées afin de supprimer les connexions superflues et de réduire votre surface d'attaque globale.

Des projets open source comme<https://github.com/kinvolk/inspektor-gadget>ou<https://github.com/deepfence/PacketStreamer>peut y contribuer, et les solutions de sécurité commerciales fournissent différents degrés d’analyse du trafic du réseau de conteneurs.

### En cas de violation, mettez à zéro les pods suspects

Contenez une violation réussie en utilisant les contrôles natifs de Kubernetes pour mettre à zéro les pods suspects ou tuer puis redémarrer les instances des applications violées.

### Effectuez une rotation fréquente des informations d'identification de l'infrastructure

Plus la durée de vie d’un secret ou d’un identifiant est courte, plus il est difficile pour un attaquant d’utiliser cet identifiant. Définissez des durées de vie courtes pour les certificats et automatisez leur rotation. Utilisez un fournisseur d’authentification capable de contrôler la durée de disponibilité des jetons émis et utilisez des durées de vie courtes lorsque cela est possible. Si vous utilisez des jetons de compte de service dans des intégrations externes, prévoyez de faire pivoter ces jetons fréquemment. Par exemple, une fois la phase d'amorçage terminée, un jeton d'amorçage utilisé pour la configuration des nœuds doit être révoqué ou son autorisation supprimée.

### Enregistrement

Kubernetes fournit une journalisation basée sur le cluster, qui vous permet de consigner l'activité des conteneurs dans un hub de journaux central. Lorsqu'un cluster est créé, la sortie standard et la sortie d'erreur standard de chaque conteneur peuvent être ingérées à l'aide d'un agent Fluentd exécuté sur chaque nœud (dans Google Stackdriver Logging ou dans Elasticsearch) et visualisées avec Kibana.

#### Activer la journalisation d'audit

L'enregistreur d'audit est une fonctionnalité bêta qui enregistre les actions entreprises par l'API pour une analyse ultérieure en cas de compromission. Il est recommandé d'activer la journalisation d'audit et d'archiver le fichier d'audit sur un serveur sécurisé.

Assurez-vous que les journaux surveillent les appels d'API anormaux ou indésirables, en particulier tout échec d'autorisation (ces entrées de journal auront un message d'état « Interdit »). Les échecs d’autorisation peuvent signifier qu’un attaquant tente d’abuser des informations d’identification volées.

Les fournisseurs Kubernetes gérés, dont GKE, donnent accès à ces données dans leur console cloud et peuvent vous permettre de configurer des alertes en cas d'échec d'autorisation.

##### Journaux d'audit

Les journaux d'audit peuvent être utiles pour la conformité, car ils doivent vous aider à répondre aux questions de savoir ce qui s'est passé, qui a fait quoi et quand. Kubernetes fournit un audit flexible des requêtes kube-apiserver en fonction des politiques. Ceux-ci vous aident à suivre toutes les activités par ordre chronologique.

Voici un exemple de journal d'audit :

```json
{
  "kind":"Event",
  "apiVersion":"audit.k8s.io/v1beta1",
  "metadata":{ "creationTimestamp":"2019-08-22T12:00:00Z" },
  "level":"Metadata",
  "timestamp":"2019-08-22T12:00:00Z",
  "auditID":"23bc44ds-2452-242g-fsf2-4242fe3ggfes",
  "stage":"RequestReceived",
  "requestURI":"/api/v1/namespaces/default/persistentvolumeclaims",
  "verb":"list",
  "user": {
    "username":"user@example.org",
    "groups":[ "system:authenticated" ]
  },
  "sourceIPs":[ "172.12.56.1" ],
  "objectRef": {
    "resource":"persistentvolumeclaims",
    "namespace":"default",
    "apiVersion":"v1"
  },
  "requestReceivedTimestamp":"2019-08-22T12:00:00Z",
  "stageTimestamp":"2019-08-22T12:00:00Z"
}
```

#### Définir des politiques d'audit

La politique d'audit définit des règles qui définissent quels événements doivent être enregistrés et quelles données sont stockées lorsqu'un événement inclut. La structure de l'objet de stratégie d'audit est définie dans le groupe d'API audit.k8s.io. Lorsqu'un événement est traité, il est comparé à la liste des règles dans l'ordre. La première règle de correspondance définit le « niveau d'audit » de l'événement.

Les niveaux d'audit connus sont :

-   Aucun – n'enregistre pas les événements qui correspondent à cette règle
-   Métadonnées : métadonnées de la demande de journal (utilisateur demandeur, horodatage, ressource, verbe, etc.), mais pas le corps de la demande ou de la réponse.
-   Requête : consigne les métadonnées de l'événement et le corps de la demande, mais pas le corps de la réponse. Cela ne s'applique pas aux demandes sans ressources
-   RequestResponse : consigne les métadonnées des événements, les corps de requête et de réponse. Cela ne s'applique pas aux demandes sans ressources

Vous pouvez transmettre un fichier avec la stratégie à kube-apiserver à l'aide de l'indicateur --audit-policy-file. Si l'indicateur est omis, aucun événement n'est enregistré. Notez que le champ des règles doit être fourni dans le fichier de stratégie d'audit. Une politique sans (0) règle est considérée comme illégale.

#### Comprendre la journalisation

L’un des principaux défis de la journalisation Kubernetes est de comprendre quels journaux sont générés et comment les utiliser. Commençons par examiner l’image globale de l’architecture de journalisation de Kubernetes.

##### Journalisation des conteneurs

La première couche de journaux pouvant être collectée à partir d'un cluster Kubernetes est celle générée par vos applications conteneurisées. La méthode la plus simple pour journaliser les conteneurs consiste à écrire dans les flux de sortie standard (stdout) et d'erreur standard (stderr).

Le manifeste est le suivant.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
    - name: example
      image: busybox
      args: [/bin/sh, -c, 'while true; do echo $(date); sleep 1; done']
```

Pour appliquer le manifeste, exécutez :

```bash
kubectl apply -f example.yaml
```

Pour consulter les journaux de ce conteneur, exécutez :

```bash
kubectl log <container-name> command.
```

Pour les journaux de conteneur persistants, l’approche courante consiste à écrire les journaux dans un fichier journal, puis à utiliser un conteneur side-car. Comme indiqué ci-dessous dans la configuration du pod ci-dessus, un conteneur side-car s'exécutera dans le même pod avec le conteneur d'application, montant le même volume et traitant les journaux séparément.

Un exemple de manifeste de pod est présenté ci-dessous :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: example
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      while true;
      do
        echo "$(date)\n" >> /var/log/example.log;
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -f /var/log/example.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

##### Journalisation des nœuds

Lorsqu'un conteneur exécuté sur Kubernetes écrit ses journaux dans des flux stdout ou stderr, le moteur de conteneur les diffuse vers le pilote de journalisation défini par la configuration Kubernetes.

Dans la plupart des cas, ces journaux se retrouveront dans le répertoire /var/log/containers sur votre hôte. Docker prend en charge plusieurs pilotes de journalisation, mais malheureusement, la configuration des pilotes n'est pas prise en charge via l'API Kubernetes.

Une fois qu'un conteneur est terminé ou redémarré, kubelet stocke les journaux sur le nœud. Pour éviter que ces fichiers ne consomment tout le stockage de l'hôte, le nœud Kubernetes implémente un mécanisme de rotation des journaux. Lorsqu'un conteneur est expulsé du nœud, tous les conteneurs avec les fichiers journaux correspondants sont expulsés.

En fonction du système d'exploitation et des services supplémentaires que vous exécutez sur votre machine hôte, vous devrez peut-être consulter des journaux supplémentaires.

Par exemple, les journaux systemd peuvent être récupérés à l'aide de la commande suivante :

```bash
journalctl -u
```

##### Journalisation du cluster

Dans le cluster Kubernetes lui-même, il existe une longue liste de composants de cluster qui peuvent être enregistrés ainsi que des types de données supplémentaires qui peuvent être utilisés (événements, journaux d'audit). Ensemble, ces différents types de données peuvent vous donner une visibilité sur les performances de Kubernetes en tant que système.

Certains de ces composants s'exécutent dans un conteneur et d'autres s'exécutent au niveau du système d'exploitation (dans la plupart des cas, un service systemd). Les services systemd écrivent dans journald et les composants exécutés dans des conteneurs écrivent les journaux dans le répertoire /var/log, sauf si le moteur de conteneur a été configuré pour diffuser les journaux différemment.

#### Événements

Les événements Kubernetes peuvent indiquer des modifications et des erreurs dans l'état des ressources Kubernetes, telles qu'un dépassement de quota de ressources ou des pods en attente, ainsi que des messages d'information.

La commande suivante renvoie tous les événements dans un espace de noms spécifique :

```bash
kubectl get events -n <namespace>

NAMESPACE LAST SEEN TYPE   REASON OBJECT MESSAGE
kube-system  8m22s  Normal   Scheduled            pod/metrics-server-66dbbb67db-lh865                                       Successfully assigned kube-system/metrics-server-66dbbb67db-lh865 to aks-agentpool-42213468-1
kube-system     8m14s               Normal    Pulling                   pod/metrics-server-66dbbb67db-lh865                                       Pulling image "aksrepos.azurecr.io/mirror/metrics-server-amd64:v0.2.1"
kube-system     7m58s               Normal    Pulled                    pod/metrics-server-66dbbb67db-lh865                                       Successfully pulled image "aksrepos.azurecr.io/mirror/metrics-server-amd64:v0.2.1"
kube-system     7m57s               Normal     Created                   pod/metrics-server-66dbbb67db-lh865                                       Created container metrics-server
kube-system     7m57s               Normal    Started                   pod/metrics-server-66dbbb67db-lh865                                       Started container metrics-server
kube-system     8m23s               Normal    SuccessfulCreate          replicaset/metrics-server-66dbbb67db             Created pod: metrics-server-66dbbb67db-lh865
```

La commande suivante affichera les derniers événements pour cette ressource Kubernetes spécifique :

```bash
kubectl describe pod <pod-name>

Events:
  Type    Reason     Age   From                               Message
  ----    ------     ----  ----                               -------
  Normal  Scheduled  14m   default-scheduler                  Successfully assigned kube-system/coredns-7b54b5b97c-dpll7 to aks-agentpool-42213468-1
  Normal  Pulled     13m   kubelet, aks-agentpool-42213468-1  Container image "aksrepos.azurecr.io/mirror/coredns:1.3.1" already present on machine
  Normal  Created    13m   kubelet, aks-agentpool-42213468-1  Created container coredns
  Normal  Started    13m   kubelet, aks-agentpool-42213468-1  Started container coredns
```

## SECTION 5 : Réflexions finales

### Intégrez la sécurité dans le cycle de vie des conteneurs le plus tôt possible

Vous devez intégrer la sécurité plus tôt dans le cycle de vie des conteneurs et garantir l'alignement et les objectifs partagés entre les équipes de sécurité et DevOps. La sécurité peut (et doit) être un catalyseur qui permet à vos développeurs et à vos équipes DevOps de créer et de déployer en toute confiance des applications prêtes pour la production en termes d'évolutivité, de stabilité et de sécurité.

### Utilisez les contrôles de sécurité natifs de Kubernetes pour réduire les risques opérationnels

Tirez parti des contrôles natifs intégrés à Kubernetes chaque fois qu'ils sont disponibles afin d'appliquer des politiques de sécurité afin que vos contrôles de sécurité n'entrent pas en conflit avec l'orchestrateur. Au lieu d'utiliser un proxy ou un shim tiers pour appliquer la segmentation du réseau, vous pouvez utiliser les stratégies réseau Kubernetes pour garantir une communication réseau sécurisée.

### Tirer parti du contexte fourni par Kubernetes pour prioriser les efforts de remédiation

Notez que le tri manuel des incidents de sécurité et des violations de stratégie prend du temps dans les environnements Kubernetes tentaculaires.

Par exemple, un déploiement contenant une vulnérabilité avec un score de gravité de 7 ou plus doit être déplacé vers le haut en priorité de correction si ce déploiement contient des conteneurs privilégiés et est ouvert à Internet, mais déplacé vers le bas s'il se trouve dans un environnement de test et prend en charge une application non critique. .

* * *

![Kubernetes Architecture](../assets/Kubernetes_Architecture.png)

## Les références

Documentation du plan de contrôle -<https://kubernetes.io>

1.  Meilleures pratiques de sécurité Kubernetes que tout le monde doit suivre -<https://www.cncf.io/blog/2019/01/14/9-kubernetes-security-best-practices-everyone-must-follow>
2.  Sécuriser un cluster -<https://kubernetes.io/blog/2016/08/security-best-practices-kubernetes-deployment>
3.  Meilleures pratiques de sécurité pour le déploiement de Kubernetes -<https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster>
4.  Meilleures pratiques de sécurité Kubernetes -<https://phoenixnap.com/kb/kubernetes-security-best-practices>
5.  Kubernetes Security 101 : risques et 29 bonnes pratiques -<https://www.stackrox.com/post/2020/05/kubernetes-security-101>
6.  15 bonnes pratiques de sécurité Kubernetes pour sécuriser votre cluster -<https://www.mobilise.cloud/15-kubernetes-security-best-practice-to-secure-your-cluster>
7.  Le guide ultime de la sécurité Kubernetes -<https://neuvector.com/container-security/kubernetes-security-guide>
8.  Guide du hacker sur la sécurité de Kubernetes -<https://techbeacon.com/enterprise-it/hackers-guide-kubernetes-security>
9.  11 façons (de ne pas) se faire pirater -<https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked>
10. 12 bonnes pratiques de configuration Kubernetes -<https://www.stackrox.com/post/2019/09/12-kubernetes-configuration-best-practices/#6-securely-configure-the-kubernetes-api-server>
11. Un guide pratique de la journalisation Kubernetes -<https://logz.io/blog/a-practical-guide-to-kubernetes-logging>
12. Interface utilisateur Web Kubernetes (tableau de bord) -<https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard>
13. Les ressources cloud de Tesla sont piratées pour exécuter des logiciels malveillants d'extraction de crypto-monnaie -<https://arstechnica.com/information-technology/2018/02/tesla-cloud-resources-are-hacked-to-run-cryptocurrency-mining-malware>
14. AGENT DE POLITIQUE OUVERTE : AUTORISATION CLOUD NATIVE -<https://blog.styra.com/blog/open-policy-agent-authorization-for-the-cloud>
15. Présentation de Policy As Code : l'Open Policy Agent (OPA) -<https://www.magalix.com/blog/introducing-policy-as-code-the-open-policy-agent-opa>
16. Ce que fournit le maillage de services -<https://aspenmesh.io/wp-content/uploads/2019/10/AspenMesh_CompleteGuide.pdf>
17. Trois avantages techniques des maillages de services et leurs limites opérationnelles, partie 1 -<https://glasnostic.com/blog/service-mesh-istio-limits-and-benefits-part-1>
18. Agent de politique ouverte : qu'est-ce que l'OPA et comment il fonctionne (exemples) -<https://spacelift.io/blog/what-is-open-policy-agent-and-how-it-works>
19. Envoyer des métriques Kubernetes à Kibana et Elasticsearch -<https://logit.io/sources/configure/kubernetes/>
20. Liste de contrôle de sécurité Kubernetes -<https://kubernetes.io/docs/concepts/security/security-checklist/>
