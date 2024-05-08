# Aide-mémoire de sécurité Docker

## Introduction

Docker est la technologie de conteneurisation la plus populaire. Lorsqu'il est utilisé correctement, il peut améliorer la sécurité par rapport à l'exécution d'applications directement sur le système hôte. Cependant, certaines erreurs de configuration peuvent réduire les niveaux de sécurité ou introduire de nouvelles vulnérabilités.

L'objectif de cette aide-mémoire est de fournir une liste simple d'erreurs de sécurité courantes et de bonnes pratiques pour vous aider à sécuriser vos conteneurs Docker.

## Règles

### RÈGLE#0 - Garder l'hôte et le Docker à jour

Pour se protéger contre les vulnérabilités connues d'évasion de conteneur telles que[Navires qui fuient](https://snyk.io/blog/cve-2024-21626-runc-process-cwd-container-breakout/), ce qui permet généralement à l'attaquant d'obtenir un accès root à l'hôte, il est essentiel de maintenir l'hôte et Docker à jour. Cela inclut la mise à jour régulière du noyau hôte ainsi que du Docker Engine.

Cela est dû au fait que les conteneurs partagent le noyau de l'hôte. Si le noyau de l'hôte est vulnérable, les conteneurs le sont également. Par exemple, l'exploit d'élévation de privilèges du noyau[Sale vache](https://github.com/scumjr/dirtycow-vdso)exécuté dans un conteneur bien isolé entraînerait toujours un accès root sur un hôte vulnérable.

### RÈGLE#1 - N'exposez pas le socket du démon Docker (même aux conteneurs)

Prise Docker_/var/run/docker.sock_est le socket UNIX que Docker écoute. Il s'agit du principal point d'entrée de l'API Docker. Le propriétaire de ce socket est root. Donner à quelqu'un l'accès équivaut à donner un accès root sans restriction à votre hôte.

**Ne permettent pas_tcp_Socket du démon Docker.**Si vous exécutez le démon Docker avec`-H tcp://0.0.0.0:XXX`ou similaire, vous exposez un accès direct non crypté et non authentifié au démon Docker, si l'hôte est connecté à Internet, cela signifie que le démon Docker sur votre ordinateur peut être utilisé par n'importe qui depuis l'Internet public.
Si tu es vraiment,**vraiment**devez le faire, vous devez le sécuriser. Vérifiez comment procéder[suivant la documentation officielle de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option).

**Ne pas exposer_/var/run/docker.sock_vers d'autres conteneurs**. Si vous exécutez votre image Docker avec`-v /var/run/docker.sock://var/run/docker.sock`ou similaire, vous devriez le changer. N'oubliez pas que monter le socket en lecture seule n'est pas une solution mais le rend seulement plus difficile à exploiter. L'équivalent dans le fichier Docker Compose ressemble à ceci :

```yaml
    volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
```

### RÈGLE#2 - Définir un utilisateur

Configurer le conteneur pour utiliser un utilisateur non privilégié est le meilleur moyen de prévenir les attaques par élévation de privilèges. Cela peut être accompli de trois manières différentes comme suit :

1.  Pendant l'exécution en utilisant`-u`option de`docker run`commande par exemple :

```bash
docker run -u 4000 alpine
```

2.  Pendant le temps de construction. Ajoutez simplement un utilisateur dans Dockerfile et utilisez-le. Par exemple:

```dockerfile
FROM alpine
RUN groupadd -r myuser && useradd -r -g myuser myuser
#    <HERE DO WHAT YOU HAVE TO DO AS A ROOT USER LIKE INSTALLING PACKAGES ETC.>
USER myuser
```

3.  Activer la prise en charge de l'espace de noms utilisateur (`--userns-remap=default`) dans[Démon Docker](https://docs.docker.com/engine/security/userns-remap/#enable-userns-remap-on-the-daemon)

Plus d'informations sur ce sujet peuvent être trouvées sur[Documentation officielle de Docker](https://docs.docker.com/engine/security/userns-remap/). Pour plus de sécurité, vous pouvez également exécuter en mode sans racine, comme indiqué dans[Règle#11](#rule-11---run-docker-in-rootless-mode).

Dans Kubernetes, cela peut être configuré dans[Contexte de sécurité](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)en utilisant le`runAsUser`champ avec l'ID utilisateur, par exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: example
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 4000 # <-- This is the pod user ID
```

En tant qu'administrateur de cluster Kubernetes, vous pouvez configurer une valeur par défaut renforcée à l'aide de l'outil[`Restricted`niveau](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted)avec intégré[Contrôleur d'admission Pod Security](https://kubernetes.io/docs/concepts/security/pod-security-admission/), si une plus grande personnalisation est souhaitée, envisagez d'utiliser[Webhooks d’admission](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)ou un[alternative à un tiers](https://kubernetes.io/docs/concepts/security/pod-security-standards/#alternatives).

### RÈGLE#3 - Limiter les capacités (accorder uniquement des capacités spécifiques, nécessaires à un conteneur)

[Capacités du noyau Linux](http://man7.org/linux/man-pages/man7/capabilities.7.html)sont un ensemble de privilèges qui peuvent être utilisés par des privilégiés. Docker, par défaut, s'exécute avec seulement un sous-ensemble de fonctionnalités.
Vous pouvez le modifier et supprimer certaines fonctionnalités (en utilisant`--cap-drop`) pour renforcer vos conteneurs Docker, ou ajouter des fonctionnalités (en utilisant`--cap-add`) si besoin.
N'oubliez pas de ne pas exécuter de conteneurs avec le`--privileged`flag - cela ajoutera TOUTES les fonctionnalités du noyau Linux au conteneur.

La configuration la plus sécurisée consiste à supprimer toutes les fonctionnalités`--cap-drop all`puis ajoutez uniquement ceux requis. Par exemple:

```bash
docker run --cap-drop all --cap-add CHOWN alpine
```

**Et n'oubliez pas : n'exécutez pas de conteneurs avec le_--privilégié_drapeau!!!**

Dans Kubernetes, cela peut être configuré dans[Contexte de sécurité](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)en utilisant`capabilities`champ par exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: example
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
          capabilities:
            drop:
              - ALL
            add: ["CHOWN"]
```

En tant qu'administrateur de cluster Kubernetes, vous pouvez configurer une valeur par défaut renforcée à l'aide de l'outil[`Restricted`niveau](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted)avec intégré[Contrôleur d'admission Pod Security](https://kubernetes.io/docs/concepts/security/pod-security-admission/), si une plus grande personnalisation est souhaitée, envisagez d'utiliser[Webhooks d’admission](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)ou un[alternative à un tiers](https://kubernetes.io/docs/concepts/security/pod-security-standards/#alternatives).

### RÈGLE#4 - Empêcher l'élévation des privilèges dans le conteneur

Exécutez toujours vos images Docker avec`--security-opt=no-new-privileges`afin d'éviter toute élévation de privilèges. Cela empêchera le conteneur d'obtenir de nouveaux privilèges via`setuid`ou`setgid`binaires.

Dans Kubernetes, cela peut être configuré dans[Contexte de sécurité](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)en utilisant`allowPrivilegeEscalation`champ par exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: example
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      allowPrivilegeEscalation: false
```

En tant qu'administrateur de cluster Kubernetes, vous pouvez configurer une valeur par défaut renforcée à l'aide de l'outil[`Restricted`niveau](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted)avec intégré[Contrôleur d'admission Pod Security](https://kubernetes.io/docs/concepts/security/pod-security-admission/), si une plus grande personnalisation est souhaitée, envisagez d'utiliser[Webhooks d’admission](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)ou un[alternative à un tiers](https://kubernetes.io/docs/concepts/security/pod-security-standards/#alternatives).

### RÈGLE#5 - Soyez attentif à la connectivité inter-conteneurs

La connectivité inter-conteneurs (icc) est activée par défaut, permettant à tous les conteneurs de communiquer entre eux via le[`docker0`réseau ponté](https://docs.docker.com/network/drivers/bridge/). Au lieu d'utiliser le`--icc=false`avec le démon Docker, qui désactive complètement la communication entre conteneurs, pensez à définir des configurations réseau spécifiques. Ceci peut être réalisé en créant des réseaux Docker personnalisés et en spécifiant quels conteneurs doivent y être attachés. Cette méthode fournit un contrôle plus granulaire sur la communication du conteneur.

Pour obtenir des conseils détaillés sur la configuration des réseaux Docker pour la communication des conteneurs, reportez-vous au[Documentation Docker](https://docs.docker.com/network/#communication-between-containers).

Dans les environnements Kubernetes,[Politiques de réseau](https://kubernetes.io/docs/concepts/services-networking/network-policies/)peut être utilisé pour définir des règles qui régulent les interactions des pods au sein du cluster. Ces politiques fournissent un cadre robuste pour contrôler la manière dont les pods communiquent entre eux et avec d'autres points de terminaison du réseau. En plus,[Éditeur de stratégie réseau](https://networkpolicy.io/)simplifie la création et la gestion des politiques de réseau, rendant plus accessible la définition de règles de réseau complexes via une interface conviviale.

### RÈGLE#6 - Utilisez le module de sécurité Linux (seccomp, AppArmor ou SELinux)

**Tout d’abord, ne désactivez pas le profil de sécurité par défaut !**

Pensez à utiliser un profil de sécurité comme[seccomp](https://docs.docker.com/engine/security/seccomp/)ou[AppArmor](https://docs.docker.com/engine/security/apparmor/).

Les instructions sur la façon de procéder dans Kubernetes peuvent être trouvées sur[Configurer un contexte de sécurité pour un pod ou un conteneur](https://kubernetes.io/docs/tutorials/security/seccomp/).

### RÈGLE#7 - Limiter les ressources (mémoire, CPU, descripteurs de fichiers, processus, redémarrages)

La meilleure façon d’éviter les attaques DoS est de limiter les ressources. Vous pouvez limiter[mémoire](https://docs.docker.com/config/containers/resource_constraints/#memory),[CPU](https://docs.docker.com/config/containers/resource_constraints/#cpu), nombre maximum de redémarrages (`--restart=on-failure:<number_of_restarts>`), nombre maximum de descripteurs de fichiers (`--ulimit nofile=<number>`) et nombre maximum de processus (`--ulimit nproc=<number>`).

[Consultez la documentation pour plus de détails sur les ulimits](https://docs.docker.com/engine/reference/commandline/run/#set-ulimits-in-container---ulimit)

Vous pouvez également faire cela pour Kubernetes :[Attribuer des ressources de mémoire aux conteneurs et aux pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/),[Attribuer des ressources CPU aux conteneurs et aux pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)et[Attribuer des ressources étendues à un conteneur](https://kubernetes.io/docs/tasks/configure-pod-container/extended-resource/)

### RÈGLE#8 - Définir le système de fichiers et les volumes en lecture seule

**Exécuter des conteneurs avec un système de fichiers en lecture seule**en utilisant`--read-only`drapeau. Par exemple:

```bash
docker run --read-only alpine sh -c 'echo "whatever" > /tmp'
```

Si une application à l'intérieur d'un conteneur doit enregistrer temporairement quelque chose, combinez`--read-only`drapeau avec`--tmpfs`comme ça:

```bash
docker run --read-only --tmpfs /tmp alpine sh -c 'echo "whatever" > /tmp/file'
```

La composition Docker`compose.yml`l'équivalent serait :

```yaml
version: "3"
services:
  alpine:
    image: alpine
    read_only: true
```

Équivalent à Kubernetes dans[Contexte de sécurité](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: example
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      readOnlyRootFilesystem: true
```

De plus, si le volume est monté uniquement pour la lecture**montez-les en lecture seule**Cela peut être fait en ajoutant`:ro`au`-v`comme ça:

```bash
docker run -v volume-name:/path/in/container:ro alpine
```

Ou en utilisant`--mount`option:

```bash
docker run --mount source=volume-name,destination=/path/in/container,readonly alpine
```

### RÈGLE#9 - Intégrez les outils d'analyse de conteneurs dans votre pipeline CI/CD

[Pipelines CI/CD](CI_CD_Security_Cheat_Sheet.md)sont une partie cruciale du cycle de vie du développement logiciel et doivent inclure divers contrôles de sécurité tels que des contrôles de charpie, une analyse de code statique et une analyse de conteneurs.

De nombreux problèmes peuvent être évités en suivant certaines bonnes pratiques lors de l'écriture du Dockerfile. Cependant, l’ajout d’un linter de sécurité en tant qu’étape dans le pipeline de construction peut grandement contribuer à éviter d’autres problèmes. Certains problèmes couramment vérifiés sont :

-   Assurer un`USER`la directive est spécifiée
-   Assurez-vous que la version de l'image de base est épinglée
-   Assurez-vous que les versions des packages du système d'exploitation sont épinglées
-   Évitez l'utilisation de`ADD`en faveur de`COPY`
-   Évitez de dénigrer les boucles`RUN`directives

Les références:

-   [Lignes de base Docker sur DevSec](https://dev-sec.io/baselines/docker/)

-   [Utilisez la ligne de commande Docker](https://docs.docker.com/engine/reference/commandline/cli/)

-   [Présentation de la CLI Docker Compose v2](https://docs.docker.com/compose/reference/overview/)

-   [Configuration des pilotes de journalisation](https://docs.docker.com/config/containers/logging/configure/)

-   [Afficher les journaux d'un conteneur ou d'un service](https://docs.docker.com/config/containers/logging/)

-   [Meilleures pratiques de sécurité Dockerfile](https://cloudberry.engineering/article/dockerfile-security-best-practices/)

    Les outils d’analyse de conteneurs sont particulièrement importants dans le cadre d’une stratégie de sécurité réussie. Ils peuvent détecter les vulnérabilités connues, les secrets et les erreurs de configuration dans les images de conteneurs et fournir un rapport sur les résultats avec des recommandations sur la façon de les corriger. Voici quelques exemples d'outils d'analyse de conteneurs populaires :

-   Gratuit
    -   [Clair](https://github.com/coreos/clair)
    -   [ThreatMapper](https://github.com/deepfence/ThreatMapper)
    -   [Voyages](https://github.com/aquasecurity/trivy)

-   Commercial
    -   [Snyk](https://snyk.io/)**(open source et option gratuite disponible)**
    -   [Ancre](https://github.com/anchore/grype/)**(open source et option gratuite disponible)**
    -   [Éclaireur Docker](https://www.docker.com/products/docker-scout/)**(open source et option gratuite disponible)**
    -   [JFrog XRay](https://jfrog.com/xray/)
    -   [Qualys](https://www.qualys.com/apps/container-security/)

Pour détecter les secrets dans les images :

-   [bouclier à œufs](https://github.com/GitGuardian/ggshield)**(open source et option gratuite disponible)**
-   [Scanner secret](https://github.com/deepfence/SecretScanner)**(Open source)**

Pour détecter les erreurs de configuration dans Kubernetes :

-   [kubeaudit](https://github.com/Shopify/kubeaudit)
-   [kubesec.io](https://kubesec.io/)
-   [banc-cube](https://github.com/aquasecurity/kube-bench)

Pour détecter les erreurs de configuration dans Docker :

-   [inspec.io](https://www.inspec.io/docs/reference/resources/docker/)
-   [dev-sec.io](https://dev-sec.io/baselines/docker/)
-   [Banc Docker pour la sécurité](https://github.com/docker/docker-bench-security)

### RÈGLE#10 - Gardez le niveau de journalisation du démon Docker à`info`

Par défaut, le démon Docker est configuré pour avoir un niveau de journalisation de base de`info`. Cela peut être vérifié en vérifiant le fichier de configuration du démon`/etc/docker/daemon.json`pour le`log-level`clé. Si la clé n'est pas présente, le niveau de journalisation par défaut est`info`. De plus, si le démon Docker est démarré avec le`--log-level`option, la valeur du`log-level`clé dans le fichier de configuration sera remplacée. Pour vérifier si le démon Docker s'exécute avec un niveau de journalisation différent, vous pouvez utiliser la commande suivante :

```bash
ps aux | grep '[d]ockerd.*--log-level' | awk '{for(i=1;i<=NF;i++) if ($i ~ /--log-level/) print $i}'
```

La définition d'un niveau de journalisation approprié configure le démon Docker pour enregistrer les événements que vous souhaitez consulter ultérieurement. Un niveau de journalisation de base de « info » et supérieur capturerait tous les journaux à l'exception des journaux de débogage. Jusqu'à ce que cela soit nécessaire, vous ne devez pas exécuter le démon Docker au niveau de journalisation « débogage ».

### Règle#11 - Exécutez Docker en mode sans racine

Le mode sans racine garantit que le démon Docker et les conteneurs s'exécutent en tant qu'utilisateur non privilégié, ce qui signifie que même si un attaquant s'échappe du conteneur, il n'aura pas les privilèges root sur l'hôte, ce qui limite considérablement la surface d'attaque. C'est différent de[remappage des utilisateurs](#rule-2---set-a-user)mode, où le démon fonctionne toujours avec les privilèges root.

Évaluer le[exigences particulières](Attack_Surface_Analysis_Cheat_Sheet.md)et[posture de sécurité](Threat_Modeling_Cheat_Sheet.md)de votre environnement pour déterminer si le mode sans racine est le meilleur choix pour vous. Pour les environnements où la sécurité est une préoccupation primordiale et où[limitations du mode sans racine](https://docs.docker.com/engine/security/rootless/#known-limitations)n'interfère pas avec les exigences opérationnelles, c'est une configuration fortement recommandée. Vous pouvez également envisager d'utiliser[Tromperie](#podman-as-an-alternative-to-docker)comme alternative à Docker.

> Le mode sans racine permet d'exécuter le démon Docker et les conteneurs en tant qu'utilisateur non root afin d'atténuer les vulnérabilités potentielles du démon et du runtime du conteneur.
> Le mode sans racine ne nécessite pas de privilèges root même lors de l'installation du démon Docker, tant que le[conditions préalables](https://docs.docker.com/engine/security/rootless/#prerequisites)sont remplies.

En savoir plus sur le mode sans racine et ses limitations, les instructions d'installation et d'utilisation sur[Documentation Docker](https://docs.docker.com/engine/security/rootless/)page.

### RÈGLE#12 - Utiliser Docker Secrets pour la gestion des données sensibles

Docker Secrets offre un moyen sécurisé de stocker et de gérer des données sensibles telles que des mots de passe, des jetons et des clés SSH. L'utilisation de Docker Secrets permet d'éviter l'exposition de données sensibles dans les images de conteneurs ou dans les commandes d'exécution.

```bash
docker secret create my_secret /path/to/super-secret-data.txt
docker service create --name web --secret my_secret nginx:latest
```

Ou pour Docker Compose :

```yaml
  version: "3.8"
  secrets:
    my_secret:
      file: ./super-secret-data.txt
  services:
    web:
      image: nginx:latest
      secrets:
        - my_secret
```

Bien que les secrets Docker fournissent généralement un moyen sécurisé de gérer les données sensibles dans les environnements Docker, cette approche n'est pas recommandée pour Kubernetes, où les secrets sont stockés en texte brut par défaut. Dans Kubernetes, envisagez d'utiliser des mesures de sécurité supplémentaires telles que le chiffrement etcd ou des outils tiers. Se référer au[Aide-mémoire pour la gestion des secrets](Secrets_Management_Cheat_Sheet.md)pour plus d'informations.

### RÈGLE#13 - Améliorer la sécurité de la chaîne d'approvisionnement

S'appuyant sur les principes de[Règle#9](#rule-9---integrate-container-scanning-tools-into-your-cicd-pipeline), l'amélioration de la sécurité de la chaîne d'approvisionnement implique la mise en œuvre de mesures supplémentaires pour sécuriser l'ensemble du cycle de vie des images de conteneurs, de la création au déploiement. Certaines des pratiques clés comprennent :

-   [Image Provenance](https://slsa.dev/spec/v1.0/provenance): Documenter l'origine et l'historique des images de conteneurs pour garantir la traçabilité et l'intégrité.
-   [Génération SBOM](https://cyclonedx.org/guides/CycloneDX%20One%20Pager.pdf): Créez une nomenclature logicielle (SBOM) pour chaque image, détaillant tous les composants, bibliothèques et dépendances pour la transparence et la gestion des vulnérabilités.
-   [Signature d'images](https://github.com/notaryproject/notary): Signez numériquement les images pour vérifier leur intégrité et leur authenticité, établissant ainsi la confiance dans leur sécurité.
-   [Registre de confiance](https://snyk.io/learn/container-security/container-registry-security/) : stockez les images documentées et signées avec leurs SBOM dans un registre sécurisé qui applique des règles strictes.[contrôles d'accès](Access_Control_Cheat_Sheet.md)et prend en charge la gestion des métadonnées.
-   [Déploiement sécurisé](https://www.openpolicyagent.org/docs/latest/#overview): Mettez en œuvre des politiques de déploiement sécurisées, telles que la validation des images, la sécurité d'exécution et la surveillance continue, pour garantir la sécurité des images déployées.

## Podman comme alternative à Docker

[Tromperie](https://podman.io/)est un outil de gestion de conteneurs open source conforme à OCI développé par[chapeau rouge](https://www.redhat.com/en)qui fournit une interface de ligne de commande compatible Docker et une application de bureau pour gérer les conteneurs. Il est conçu pour être une alternative plus sécurisée et plus légère à Docker, en particulier pour les environnements où les valeurs par défaut sécurisées sont préférées. Certains des avantages de sécurité de Podman incluent :

1.  Architecture sans démon : contrairement à Docker, qui nécessite un démon central (dockerd) pour créer, exécuter et gérer des conteneurs, Podman utilise directement le modèle fork-exec. Lorsqu'un utilisateur demande de démarrer un conteneur, Podman quitte le processus en cours, puis le processus enfant s'exécute dans le runtime du conteneur.
2.  Conteneurs sans racine : le modèle fork-exec facilite la capacité de Podman à exécuter des conteneurs sans nécessiter de privilèges root. Lorsqu'un utilisateur non root lance un démarrage de conteneur, Podman se lance et s'exécute sous les autorisations de l'utilisateur.
3.  Intégration SELinux : Podman est conçu pour fonctionner avec SELinux, qui fournit une couche de sécurité supplémentaire en appliquant des contrôles d'accès obligatoires sur les conteneurs et leurs interactions avec le système hôte.

## Références et lectures complémentaires

[Top 10 des Dockers OWASP](https://github.com/OWASP/Docker-Security)[Bonnes pratiques de sécurité Docker](https://docs.docker.com/develop/security-best-practices/)[Sécurité du moteur Docker](https://docs.docker.com/engine/security/)[Aide-mémoire de sécurité Kubernetes](Kubernetes_Security_Cheat_Sheet.md)[SLSA - Niveaux de chaîne d'approvisionnement pour les artefacts logiciels](https://slsa.dev/)[Sigstore](https://sigstore.dev/)[Attestation de construction Docker](https://docs.docker.com/build/attestations/)[Confiance du contenu Docker](https://docs.docker.com/engine/security/trust/)
