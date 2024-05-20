# Comment chiffrer les secrets Kubernetes à l'aide de secrets scellés

## Introduction

Dans ce didacticiel, vous apprendrez à déployer et`encrypt`générique`Kubernetes Secrets`en utilisant le[Contrôleur de secrets scellés](https://github.com/bitnami-labs/sealed-secrets).

Quoi`Sealed Secrets`vous permet de faire est :

-   Magasin`encrypted`secrets dans un`Git`référentiel (même dans`public`ceux).
-   Appliquer`GitOps`principes pour`Kubernetes Secrets`aussi ([Section 15 - Livraison continue à l'aide de GitOps](../15-continuous-delivery-using-gitops/README.md)vous donne des exemples plus pratiques sur ce sujet).

### Comprendre le fonctionnement des secrets scellés

Le`Sealed Secrets Controller`crée du générique (classique)`Kubernetes`secrets dans votre`DOKS`cluster, à partir de manifestes secrets scellés. Secrets scellés`decryption`arrive`server side`seulement, donc tant que le`DOKS`le cluster est sécurisé (`etcd`base de données,`RBAC`correctement réglé), tout devrait être en sécurité.

Deux composants sont impliqués :

1.  A client side utility called `kubeseal`, utilisé pour`encrypting`générique`Kubernetes`secrets. Le`kubeseal`Utilisations de la CLI`asymmetric crypto`pour chiffrer les secrets qui`only`le`Sealed Secrets Controller`peut`decrypt`.
2.  Un composant côté serveur appelé`Sealed Secrets Controller`qui fonctionne sur votre`DOKS`cluster, et s'occupe de`decrypting`objets secrets scellés que les applications peuvent utiliser.

Le véritable avantage vient lorsque vous utilisez`Sealed Secrets`dans un`GitOps`couler. Après vous`commit`le secret scellé`manifest`à vos candidatures`Git`référentiel, le`Continuous Delivery`système (par ex.`Flux CD`) est informé du changement et crée un`Sealed Secret`ressource dans votre`DOKS`grappe. Puis le`Sealed Secrets Controller`entre en jeu, et`decrypts`votre objet secret scellé à l'original`Kubernetes`secrète. Ensuite, les applications peuvent consommer le secret comme d'habitude.

Par rapport à d'autres solutions comme`Vault`, Sealed Secrets ne dispose pas des fonctionnalités suivantes :

-   `Multiple`prise en charge du backend de stockage (comme`Consul`,`S3`,`Filesystem`,`SQL databases`, etc).
-   `Dynamic Secrets` : Les secrets scellés ne peuvent pas créer d'informations d'identification d'application sur`demand`pour accéder à d'autres systèmes, comme`S3`stockage compatible (par ex.`DO Spaces`), et`automatically revoke credentials`plus tard, lorsque le`lease`expire.
-   `Leasing`et`renewal`de secrets : Sealed Secrets ne fournit pas de`client API`pour`renewing leases`, et il ne fournit pas non plus de`lease`associé à chacun`secret`.
-   `Revoking`anciennes clés/secrets : les secrets scellés peuvent`rotate`la clé de cryptage`automatically`, mais c'est assez limité à cet égard.**Les anciennes clés et secrets ne sont pas révoqués automatiquement - vous devez révoquer manuellement les anciennes clés et tout refermer.**
-   `Pluggable`architecture qui étend les fonctionnalités existantes telles que la configuration`ACLs`via`identity based access control`plugins (`Okta`,`AWS`, etc).

Bien que`Vault`est plus performant en fonctionnalités, il s'accompagne d'un compromis :`increased complexity`et`costs`en termes d'entretien. Où`Sealed Secrets`qui brille vraiment, c'est :`simplicity`et`low`entretien`overhead`et`costs`.

Pour`enterprise`grade`production`ou`HIPAA`des systèmes conformes,`Vault`est certainement l'un des meilleurs candidats. Pour`small`des projets et`development`environnements,`Sealed Secrets`suffira dans la plupart des cas.

Après avoir terminé ce tutoriel, vous pourrez :

-   `Create`et déployer scellé`Kubernetes`secrets pour votre`DOKS`grappe.
-   `Manage`et`update`secrets scellés.
-   `Configure`secrets scellés`scope`.

### Présentation de la configuration du contrôleur Sealed Secrets

![Sealed Secrets Controller Setup Overview](assets/images/sealed_secrets_flow.png)

## Table des matières

-   [Introduction](#introduction)
    -   [Comprendre le fonctionnement des secrets scellés](#understanding-how-sealed-secrets-work)
    -   [Présentation de la configuration du contrôleur Sealed Secrets](#sealed-secrets-controller-setup-overview)
-   [Conditions préalables](#prerequisites)
-   [Étape 1 - Installation du contrôleur Sealed Secrets](#step-1---installing-the-sealed-secrets-controller)
-   [Étape 2 - Chiffrer un secret Kubernetes](#step-2---encrypting-a-kubernetes-secret)
-   [Étape 3 - Gestion des secrets scellés](#step-3---managing-sealed-secrets)
    -   [Gestion des secrets existants](#managing-existing-secrets)
    -   [Mise à jour des secrets existants](#updating-existing-secrets)
-   [Étape 4 - Sauvegarde de la clé privée du contrôleur de secrets scellés](#step-4---sealed-secrets-controller-private-key-backup)
-   [Meilleures pratiques de sécurité](#security-best-practices)
-   [Conclusion](#conclusion)
    -   [Avantages](#pros)
    -   [Les inconvénients](#cons)
    -   [Apprendre encore plus](#learn-more)

## Conditions préalables

Pour réaliser ce tutoriel, vous aurez besoin de :

1.  UN[Git](https://git-scm.com/downloads)client, pour cloner le`Starter Kit`dépôt.
2.  [C'était un sceau](https://github.com/bitnami-labs/sealed-secrets/releases/tag/v0.18.1), pour chiffrer les secrets et`Sealed Secrets Controller`interaction.
3.  [Barre](https://www.helms.sh), pour gérer`Sealed Secrets Controller`versions et mises à niveau.
4.  [Kubectl](https://kubernetes.io/docs/tasks/tools), pour`Kubernetes`interaction.

## Étape 1 - Installation du contrôleur Sealed Secrets

Dans cette étape, vous apprendrez à déployer le`Sealed Secrets Controller`en utilisant`Helm`. La carte d'intérêt s'appelle`sealed-secrets`et il est fourni par le`bitnami-labs`dépôt.

Tout d'abord, clonez le`Starter Kit`Dépôt Git et remplacez le répertoire par votre copie locale :

```shell
git clone https://github.com/digitalocean/Kubernetes-Starter-Kit-Developers.git

cd Kubernetes-Starter-Kit-Developers
```

Ensuite, ajoutez les secrets scellés`bitnami-labs`référentiel pour`Helm`:

```shell
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
```

Ensuite, mettez à jour le`sealed-secrets`référentiel de graphiques :

```shell
helm repo update sealed-secrets
```

Ensuite, recherchez le`sealed-secrets`référentiel pour les graphiques disponibles à installer :

```shell
helm search repo sealed-secrets
```

Le résultat ressemble à :

```text
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
sealed-secrets/sealed-secrets   2.4.0           v0.18.1         Helm chart for the sealed-secrets controller.
```

Maintenant, ouvrez et inspectez le`06-kubernetes-secrets/assets/manifests/sealed-secrets-values-v2.4.0.yaml`fichier fourni dans le`Starter kit`dépôt, en utilisant un éditeur de votre choix (de préférence avec`YAML`support anti-peluches). Vous pouvez utiliser[Code VS](https://code.visualstudio.com), Par exemple:

```shell
code 06-kubernetes-secrets/assets/manifests/sealed-secrets-values-v2.4.0.yaml
```

Ensuite, installez le`sealed-secrets/sealed-secrets`graphique, en utilisant`Helm`(notez qu'un dédié`sealed-secrets`l'espace de noms est également créé) :

```shell
HELM_CHART_VERSION="2.4.0"

helm install sealed-secrets-controller sealed-secrets/sealed-secrets --version "${HELM_CHART_VERSION}" \
  --namespace sealed-secrets \
  --create-namespace \
  -f "06-kubernetes-secrets/assets/manifests/sealed-secrets-values-v${HELM_CHART_VERSION}.yaml"
```

**Remarques:**

-   UN`specific`version pour le`Helm`graphique est utilisé. Dans ce cas`2.4.0`est sélectionné, ce qui correspond au`0.18.1`version de l'application. C'est une bonne pratique en général de verrouiller une version spécifique. Cela permet d'avoir des résultats prévisibles et permet le contrôle des versions via`Git`.
-   Vous voudrez`restrict`accès aux secrets scellés`namespace`pour les autres utilisateurs qui ont accès à votre`DOKS`cluster, pour éviter`unauthorized`accès au`private key`(par exemple, utilisez`RBAC`Stratégies).

Ensuite, répertoriez l'état de déploiement pour`Sealed Secrets`contrôleur (le`STATUS`la valeur de la colonne doit être`deployed`):

```shell
helm ls -n sealed-secrets
```

Le résultat ressemble à :

```text
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                  APP VERSION
sealed-secrets-controller       sealed-secrets  1               2021-10-04 18:25:03.594564 +0300 EEST   deployed        sealed-secrets-2.4.0   v0.18.1
```

Enfin, inspectez le`Kubernetes`ressources créées par le`Sealed Secrets`Déploiement de la barre :

```shell
kubectl get all -n sealed-secrets
```

La sortie ressemble à (notez l'état du`sealed-secrets-controller`pod et service - doivent être`UP`et`Running`):

```text
NAME                                             READY   STATUS    RESTARTS   AGE
pod/sealed-secrets-controller-7b649d967c-mrpqq   1/1     Running   0          2m19s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/sealed-secrets-controller   ClusterIP   10.245.105.164   <none>        8080/TCP   2m20s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sealed-secrets-controller   1/1     1            1           2m20s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/sealed-secrets-controller-7b649d967c   1         1         1       2m20s
```

Dans l'étape suivante, vous apprendrez comment`seal`ton`secrets`. Seulement`your DOKS`le cluster peut`decrypt`les secrets scellés, car c'est le seul à avoir le`private`clé.

## Étape 2 - Chiffrer un secret Kubernetes

Dans cette étape, vous apprendrez à chiffrer votre générique`Kubernetes`secret, en utilisant`kubeseal`CLI. Ensuite, vous le déployerez sur votre`DOKS`cluster et voyez comment le`Sealed Secrets`manette`decrypts`pour que vos applications puissent l'utiliser.

Supposons que vous deviez sceller un secret générique pour votre application, enregistré dans le fichier suivant :`your-app-secret.yaml`. Remarquez le`your-data`champ qui est`base64`codé (c'est`vulnerable`aux attaques, car cela peut être très facilement`decoded`en utilisant des outils gratuits) :

```yaml
apiVersion: v1
data:
  your-data: ZXh0cmFFbnZWYXJzOgogICAgRElHSVRBTE9DRUFOX1RPS0VOOg== # base64 encoded application data
kind: Secret
metadata:
  name: your-app
```

Tout d'abord, vous devez récupérer le`public key`du`Sealed Secrets Controller`(effectué`only once`par cluster, et sur chaque`fresh`installer):

```shell
kubeseal --fetch-cert --controller-namespace=sealed-secrets > pub-sealed-secrets.pem
```

**Remarques:**

-   Si vous déployez le`Sealed Secrets`contrôleur vers un autre espace de noms (par défaut`kube-system`), vous devez préciser au`kubeseal`CLI l'espace de noms, via le`--controller-namespace`drapeau.
-   Le`public key`peut être`safely`stocké dans un`Git`référentiel par exemple, voire donné au monde. Le mécanisme de cryptage utilisé par le`Sealed Secrets`Le contrôleur ne peut pas être inversé sans le`private key`(stocké dans votre`DOKS`cluster uniquement).

Ensuite, créez un`sealed`fichier du`Kubernetes`secret, en utilisant le`pub-sealed-secrets.pem`clé:

```shell
kubeseal --format=yaml \
  --cert=pub-sealed-secrets.pem \
  --secret-file your-app-secret.yaml \
  --sealed-secret-file your-app-sealed.yaml
```

Le contenu du fichier ressemble à (notez le`your-data`champ qui est`encrypted`maintenant, en utilisant un Bitnami`SealedSecret`objet):

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: your-app
  namespace: default
spec:
  encryptedData:
    your-data: AgCFNTLd+KD2IGZo3YWbRgPsK1dEhxT3NwSCU2Inl8A6phhTwMxKSu82fu0LGf/AoYCB35xrdPl0sCwwB4HSXRZMl2WbL6HrA0DQNB1ov8DnnAVM+6TZFCKePkf9yqVIekr4VojhPYAvkXq8TEAxYslQ0ppNg6AlduUZbcfZgSDkMUBfaczjwb69BV8kBf5YXMRmfGtL3mh5CZA6AAK0Q9cFwT/gWEZQU7M1BOoMXUJrHG9p6hboqzyEIWg535j+14tNy1srAx6oaQeEKOW9fr7C6IZr8VOe2wRtHFWZGjCL3ulzFeNu5GG0FmFm/bdB7rFYUnUIrb2RShi1xvyNpaNDF+1BDuZgpyDPVO8crCc+r2ozDnkTo/sJhNdLDuYgIzoQU7g1yP4U6gYDTE+1zUK/b1Q+X2eTFwHQoli/IRSv5eP/EAVTU60QJklwza8qfHE9UjpsxgcrZnaxdXZz90NahoGPtdJkweoPd0/CIoaugx4QxbxaZ67nBgsVYAnikqc9pVs9VmX/Si24aA6oZbtmGzkc4b80yi+9ln7x/7/B0XmyLNLS2Sz0lnqVUN8sfvjmehpEBDjdErekSlQJ4xWEQQ9agdxz7WCSCgPJVnwA6B3GsnL5dleMObk7eGUj9DNMv4ETrvx/ZaS4bpjwS2TL9S5n9a6vx6my3VC3tLA5QAW+GBIfRD7/CwyGZnTJHtW5f6jlDWYS62LbFJKfI9hb8foR/XLvBhgxuiwfj7SjjAzpyAgq
  template:
    data: null
    metadata:
      creationTimestamp: null
      name: your-app
      namespace: default
```

**Note:**

Si vous ne précisez pas de`namespace`, le`default`on est supposé (utilisez Kubeseal`--namespace`flag, pour modifier l'espace de noms ciblé). Défaut`scope`utilisé par`kubeseal`est`strict`- veuillez vous référer aux portées dans[Meilleures pratiques de sécurité](#security-best-practices).

Ensuite, vous pouvez supprimer le`Kubernetes`fichier secret, car il n'est plus nécessaire :

```shell
rm -f your-app-secret.yaml
```

Enfin,`deploy`le`sealed secret`à votre cluster :

```shell
kubectl apply -f your-app-sealed.yaml
```

Vérifiez que le`Sealed Secrets Controller`décrypté votre`Kubernetes`secret dans le`default`espace de noms :

```shell
kubectl get secrets
```

Le résultat ressemble à :

```text
NAME                  TYPE                                  DATA   AGE
your-app              Opaque                                1      31s
```

Inspectez le secret :

```shell
kubectl get secret your-app -o yaml
```

Le résultat ressemble à (`your-data`clé`value`devrait être`decrypted`à l'original`base64`codé`value`):

```yaml
apiVersion: v1
data:
  your-data: ZXh0cmFFbnZWYXJzOgogICAgRElHSVRBTE9DRUFOX1RPS0VOOg==
kind: Secret
metadata:
  creationTimestamp: "2021-10-05T08:34:07Z"
  name: your-app
  namespace: default
  ownerReferences:
  - apiVersion: bitnami.com/v1alpha1
    controller: true
    kind: SealedSecret
    name: your-app
    uid: f6475e74-78eb-4c6a-9f19-9d9ceee231d0
  resourceVersion: "235947"
  uid: 7b7d2fee-c48a-4b4c-8f16-2e58d25da804
type: Opaque
```

## Étape 3 - Gestion des secrets scellés

### Gestion des secrets existants

Si tu veux`SealedSecret`contrôleur pour prendre la gestion d'un`existing`Secret (c'est-à-dire l'écraser lors de la descellement d'un SealedSecret avec le même nom et le même espace de noms), alors vous devez`annotate`que`Secret`avec l'annotation`sealedsecrets.bitnami.com/managed: "true"`postuler à l'avance[Étape 2 - Chiffrer un secret Kubernetes](#step-2---encrypting-a-kubernetes-secret).

### Mise à jour des secrets existants

Si tu veux`add`ou`update`secrets scellés existants sans avoir le texte clair pour les autres éléments, vous pouvez simplement`copy&paste`les nouvelles données chiffrées et`merge`dans un`existing`secret scellé.

Vous devez prendre soin de sceller les éléments mis à jour avec un`name`et`namespace`(voir la note sur les étendues ci-dessus).

Vous pouvez utiliser le`--merge-into`commande pour mettre à jour un secret scellé existant si vous ne souhaitez pas copier-coller :

```shell
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json \
  | kubeseal --controller-namespace=sealed-secrets > mysealedsecret.json

echo -n baz | kubectl create secret generic mysecret --dry-run=client --from-file=bar=/dev/stdin -o json \
  | kubeseal --controller-namespace=sealed-secrets --merge-into mysealedsecret.json
```

Si vous utilisez`VS Code`il existe une extension qui vous permet d'utiliser le`GUI`mode pour effectuer les opérations ci-dessus -[Il a été scellé pour vscode](https://marketplace.visualstudio.com/items?itemName=codecontemplator.kubeseal).

## Étape 4 - Sauvegarde de la clé privée du contrôleur de secrets scellés

Si vous souhaitez effectuer un`manual backup`des clés privées et publiques, vous pouvez le faire via :

```shell
kubectl get secret -n sealed-secrets -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > master.key
```

Ensuite, stockez le`master.key`déposer dans un endroit sûr. Pour restaurer à partir d'une sauvegarde après un sinistre, remettez simplement ces secrets avant de démarrer le contrôleur - ou si le contrôleur a déjà été démarré, remplacez les secrets nouvellement créés et redémarrez le contrôleur :

```shell
kubectl apply -f master.key

kubectl delete pod -n sealed-secrets -l name=sealed-secrets-controller
```

La meilleure approche consiste à effectuer des sauvegardes régulières par exemple, comme vous l'avez déjà appris dans[Section 6 - Configurer la sauvegarde et la restauration](../06-setup-backup-restore/README.md).`Velero`ou`Trilio`vous aide à restaurer le`Sealed Secrets`état du contrôleur en cas de sinistre également (sans qu'il soit nécessaire de récupérer le`master`clé, puis`inserting`il est de retour dans le`cluster`).

## Meilleures pratiques de sécurité

En termes de sécurité,`Sealed Secrets`te permet de`restrict`d'autres utilisateurs pour déchiffrer vos secrets scellés à l'intérieur du cluster. Il ya trois`scopes`que vous pouvez utiliser (`kubeseal`CLI`--scope`drapeau):

1.  `strict`(par défaut) : le secret doit être scellé avec`exactly`le même`name`et`namespace`. Ces`attributes`devenir`part`de la`encrypted data`Et ainsi`changing name`et/ou`namespace`conduirait à**"erreur de décryptage"**.
2.  `namespace-wide`: vous pouvez librement`rename`le secret scellé dans un domaine donné`namespace`.
3.  `cluster-wide`: le`secret`peut être`unsealed`dans tous`namespace`et peut recevoir n'importe quel`name`.

Ensuite, vous pouvez appliquer certaines des meilleures pratiques soulignées ci-dessous :

-   Assurez-vous de changer**les deux**`secrets`périodiquement (comme les mots de passe, les jetons, etc.), et le`private key`utilisé pour`encryption`. De cette façon, si le`encryption key`est toujours`leaked`, les données sensibles ne sont pas exposées. Et même si c’est le cas, les secrets ne sont plus valables. Vous pouvez en savoir plus sur le sujet en vous référant au[Rotation secrète](https://github.com/bitnami-labs/sealed-secrets#secret-rotation)chapitre, de la documentation officielle.
-   Vous pouvez exploiter la puissance de`RBAC`pour votre`Kubernetes`regrouper à`restrict`accès à`namespaces`. Donc, si vous stockez tous vos secrets Kubernetes dans un`specific namespace`, Ensuite vous pouvez`restrict`accès à`unwanted users`et`applications`pour ça`specific namespace`. C'est important, car il est clair`Kubernetes Secrets`sont`base64`codé et peut être`decoded`très facile pour n'importe qui.`Sealed Secrets`fournit un`encryption`couche sur`encoding`, mais dans votre`DOKS`les secrets scellés du cluster sont reconvertis en`generic`Secrets Kubernetes.
-   Éviter`private key leaks`, veuillez vous assurer que le`namespace`où vous avez déployé le`Sealed Secrets`Le contrôleur est également protégé, via correspondant`RBAC`règles.

## Conclusion

Dans ce didacticiel, vous avez appris à utiliser le générique`Kubernetes secrets`dans un`secure`chemin. Vous avez également appris que le`encryption key`est stocké et les secrets sont`decrypted`dans le`cluster`(le client n'a pas accès à la clé de cryptage).

Ensuite, vous avez découvert comment utiliser`kubeseal`CLI, pour générer`SealedSecret`manifestes contenant du contenu sensible`encrypted`. Après`applying`le fichier manifeste des secrets scellés à votre`DOKS`cluster, le`Sealed Secrets Controller`le reconnaîtra comme une nouvelle ressource secrète scellée, et`decrypt`c'est générique`Kubernetes Secret`Ressource.

### Avantages

-   `Lightweight`, ce qui signifie que les coûts de mise en œuvre et de gestion sont faibles.
-   `Transparent` integration with `Kubernetes Secrets`.
-   `Decryption`arrive`server side`(grappe DOKS).
-   Fonctionne très bien dans un`GitOps`installation (`encrypted`les fichiers peuvent être stockés en utilisant`public Git`référentiels).

### Les inconvénients

-   Pour`each DOKS cluster`un séparé`private`et`public key`la paire doit être`created`et`maintained`.
-   `Private keys`doit être`backed`vers le haut (par exemple en utilisant`Velero`) pour`disaster`récupération.
-   `Updating`et`re-sealing`secrets, ainsi que`adding`ou`merging`les nouvelles clés/valeurs ne sont pas tout à fait simples.

Même s'il y a quelques inconvénients à utiliser`Sealed Secrets`, le`ease`de`management`et`transparent`intégration avec`Kubernetes`et`GitOps`les flux en font un bon candidat dans la pratique.

### Apprendre encore plus

-   [Mise à niveau](https://github.com/bitnami-labs/sealed-secrets#upgrade)étapes et notes.
-   [FAQ sur les secrets scellés](https://github.com/bitnami-labs/sealed-secrets#faq), pour les questions fréquemment posées sur`Sealed Secrets`.

Ensuite, vous apprendrez comment faire évoluer automatiquement les charges de travail de vos applications en fonction de la charge externe (ou du trafic). Vous apprendrez à tirer parti`metrics-server`ainsi que`Prometheus`via`prometheus-adapter`pour faire le travail et laisser le système de mise à l'échelle automatique des pods horizontal (ou vertical) Kubernetes prendre des décisions intelligentes.

Aller à[Section 7 - Mise à l'échelle des charges de travail des applications](../07-scaling-application-workloads/README.md).
