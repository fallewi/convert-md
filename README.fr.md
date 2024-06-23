## Un tutoriel sur l'utilisation de Snyk pour trouver les failles de sécurité dans notre code et les corriger.

## Qu’est-ce que Snyk ?

Snyk (prononcé_se faufiler_) est une plateforme de sécurité pour développeurs permettant de sécuriser le code, les dépendances, les conteneurs et l'infrastructure en tant que code. Donc, ce qu'il fait, c'est qu'il analyse votre code, le lit et vous indique si vous avez des vulnérabilités dans votre code. Désormais, il ne vérifie plus seulement votre code, il peut également vérifier les dépendances installées, votre conteneur Docker, votre infrastructure as code et quelques autres éléments. Il est compatible avec de nombreux langages et est livré avec des plugins pris en charge par différents IDE. Il s’agit donc essentiellement de Grammarly pour votre code.

## Premiers pas avec Snyk

Pour commencer, vous devez créer un compte sur Snyk. Rendez-vous sur<https://snyk.io/>et inscrivez-vous pour un compte gratuit. Je vous recommande de vous connecter via Github. Une fois inscrit, vous pouvez vous connecter à votre compte. Après vous être connecté, vous pourrez voir un tableau de bord similaire.

![](https://miro.medium.com/v2/resize:fit:875/0*8zTx1RctHKorAG9J)

Maintenant tu peux aller à[ce lien](https://docs.snyk.io/snyk-cli/install-the-snyk-cli)et suivez les instructions pour télécharger la CLI Snyk. Il existe différentes méthodes pour télécharger la CLI Snyk. Vous pouvez continuer avec n’importe lequel d’entre eux.

Maintenant, si vous êtes ici, je suppose que vous avez déjà installé Snyk CLI en utilisant l'une des méthodes disponibles. Maintenant, ce que nous devons faire est de nous authentifier auprès de Snyk CLI. Pour ce faire, exécutez la commande suivante dans le terminal :

    <span id="7806" data-selectable-paragraph="">snyk auth</span>

Lorsque vous exécutez la commande, une page d'authentification s'ouvrira dans votre navigateur par défaut comme ci-dessous :

![](https://miro.medium.com/v2/resize:fit:875/0*ksUJI8o-UoQbsFdv)

Cliquez simplement sur le**Authenticate**et attendez que la page affiche un message de réussite. Une fois que vous voyez le message, vous pouvez accéder à votre terminal où vous trouverez une sortie similaire à celle ci-dessous :

![](https://miro.medium.com/v2/resize:fit:875/0*6DVd2KSWT_G-WvSr)

Maintenant, la Synk CLI a été connectée à votre compte.

## Recherche de vulnérabilités dans l'application de démonstration

À des fins de démonstration, nous allons utiliser une application Web appelée PyGoat écrite en Django. De nombreuses vulnérabilités ont été ajoutées intentionnellement à l'application, afin que nous puissions avoir une bonne démo de Snyk autour d'elle.

Voici le lien vers le dépôt Github :<https://github.com/purpledobie/pygoat>. Ouvrez le lien du référentiel, cliquez sur Fork, puis clonez le référentiel forké sur votre machine locale. Lorsque vous parcourez le référentiel, vous trouverez Dockerfile, le fichier Infrastructure as Code ainsi que les fichiers Python standards. Nous passerons en revue les fichiers plus tard. Vous pouvez installer les dépendances Python à partir du**exigences.txt**déposer.

    <span id="8d49" data-selectable-paragraph="">pip install -r requirements.txt</span>

## Plugins Snyk

Snyk propose des plugins disponibles pour différents IDE tels que Eclipse, VS Code et Jetbrains (PyCharm, IntelliJ, etc.). Depuis que je suis sur VS Code, j'ai installé l'extension Snyk sur mon IDE. Vous pouvez faire la même chose pour votre IDE.

![](https://miro.medium.com/v2/resize:fit:875/0*px1G7p0GCClqJ8YK)

Une fois l’extension installée, vous devrez peut-être vous authentifier à nouveau. Une fois authentifié, le plugin commencera à analyser le code automatiquement, après quelques secondes, il affichera les résultats similaires à ceux ci-dessous :

![](https://miro.medium.com/v2/resize:fit:875/0*BTlgkPJt6T8ELada)

Vous pouvez voir qu'il existe de nombreuses 18 vulnérabilités de sécurité du code et 2 problèmes de qualité du code dans le code. Chaque problème ou vulnérabilité est accompagné d'une icône. Ça peut être**C**,**H**,**M.**, et**L**signification**Critique**,**Haut**,**Moyen**, et**Faible**respectivement. Vous pouvez cliquer sur l'un d'entre eux pour en savoir plus et cela suggérera même des correctifs pour le problème ou la vulnérabilité.

## Commandes CLI Snyk

Nous avons déjà exécuté une commande Synk CLI, c'est-à-dire`**snyk auth**`pour nous authentifier auprès de Snyk. Examinons maintenant quelques autres commandes importantes.

## 1.`snyk test`

Cette commande analysera le code et vous montrera les vulnérabilités. Exécutons ceci et voyons quel résultat nous obtenons :

![](https://miro.medium.com/v2/resize:fit:875/0*XhbNDc8on0kwDu9f)

Vous pouvez voir qu’il a terminé l’analyse et qu’il a trouvé les mêmes vulnérabilités. Les vulnérabilités sont à nouveau marquées comme faibles, moyennes, élevées et critiques. En dehors de cela, il nous fournit également des suggestions pour résoudre le problème. Par exemple, si vous voyez l'image ci-dessus, elle nous suggère de mettre à niveau Django de la version 3.1.12 vers la version 3.2.13 pour résoudre de nombreux problèmes. Mettons à niveau Django, puis réanalysons l'application pour voir si ces vulnérabilités ont été corrigées ou non.

Commençons par mettre à jour la version de Django vers la 3.2.13 à l'aide de la commande :

    <span id="ffa7" data-selectable-paragraph="">pip install django==3.2.13</span>

Vous obtiendrez un résultat similaire :

![](https://miro.medium.com/v2/resize:fit:875/0*xD3l2YUK2s8llnbd)

Maintenant, analysons à nouveau le code en utilisant`snyk test`commande.

![](https://miro.medium.com/v2/resize:fit:875/0*rU21WqB2CSlu2BVY)

Maintenant, si vous le remarquez, nous n’avons pas ces vulnérabilités telles que l’injection SQL.

## 2.`snyk monitor`

Cette commande analyse le code et en télécharge un instantané sur l'interface utilisateur Snyk ou la plate-forme Snyk. Commençons par exécuter la commande :

![](https://miro.medium.com/v2/resize:fit:875/0*R56SEhUq_0mJhZYJ)

La commande a pris un instantané du projet et l'a téléchargé sur la plateforme Snyk. Il nous donne ensuite une URL où nous pouvons voir de nombreuses autres informations concernant le projet. Si vous ouvrez l'URL, vous verrez une page similaire :

![](https://miro.medium.com/v2/resize:fit:875/0*OuwKhskkH49j_Qv1)

Il est désormais plus facile de voir les vulnérabilités de l'application. Vous pouvez également retester l'application en cliquant sur le**Retester maintenant**lien. Vous pouvez également voir le**Correctifs**et**Dépendances**de la demande.

## 3.Analyser l'infrastructure en tant que code

Si vous regardez le projet, vous trouverez un dossier appelé**Infrastructure**. À l'intérieur de cela, nous avons un**application-load-balancer**dossier. Ainsi, ce projet peut être déployé sur l'équilibreur de charge AWS. Il existe un fichier Python`app.py`qui génère en fait un modèle pour la configuration de l'équilibreur de charge sur AWS. Puis dans le**cdk.out**dossier, vous pouvez trouver un**LoadBalancerStack.template.json**fichier généré à partir du code Python.

Pour rechercher d'éventuelles erreurs de configuration avant le déploiement, nous pouvons réellement tester ce fichier à l'aide de Snyk. La commande pour la même chose est :

    <span id="94d9" data-selectable-paragraph="">snyk iac test &lt;template-file-path&gt;</span>

Lançons-nous et voyons le résultat :

![](https://miro.medium.com/v2/resize:fit:875/0*lcpEwq5BdEg5cf5T)

Il montre tous les problèmes et vulnérabilités dans le fichier modèle.

## 4.Analyser le fichier Docker et les images Docker

Dans le projet, nous avons un Dockerfile. Vous pouvez créer l'image Docker à partir du Dockerfile à l'aide de la commande :

    <span id="4f3f" data-selectable-paragraph="">docker build -t pygoat .</span>

Vous pouvez voir l'image en cours de création ci-dessous :

![](https://miro.medium.com/v2/resize:fit:875/0*w_RjI-IFo277ozpC)

Une fois l'image créée, vous pouvez rechercher les vulnérabilités à l'aide de la commande :

    <span id="b23f" data-selectable-paragraph="">docker scan pygoat</span>

L'intégration du Snyk avec Docker rend l'analyse incroyablement simple.

Vous obtiendrez le résultat comme ci-dessous :

![](https://miro.medium.com/v2/resize:fit:875/0*pfu5C7mWL0KHuThL)

Le résultat est très volumineux et il n’est pas possible de montrer tout ce qui s’y trouve. Mais nous pouvons voir les vulnérabilités dans l’image.

## Intégration de Snyk avec Github

Snyk peut résoudre automatiquement les problèmes pour vous. Lorsque vous liez un référentiel Github à Snyk, il analysera l'intégralité du projet et s'il dispose d'un correctif concernant une vulnérabilité, il créera une Pull Request avec le correctif. N'est-ce pas incroyable ?

Puisque nous nous sommes déjà connectés via Github, Snyk a déjà accès à nos référentiels. Il suffit de sélectionner le référentiel que nous souhaitons analyser.

Cliquez sur le bouton Ajouter un projet, puis cliquez sur Github et sélectionnez votre référentiel.

![](https://miro.medium.com/v2/resize:fit:875/0*KlH7J6rU0G9LGhdV)

Une fois que vous avez ajouté le projet, vous pouvez le trouver sur le tableau de bord. Snyk analysera automatiquement le projet. Une fois que vous aurez vu les problèmes, vous verrez un**Corriger cette vulnérabilité**(pour chaque vulnérabilité) ou**Corrigez ces vulnérabilités**(pour corriger toutes les vulnérabilités).

![](https://miro.medium.com/v2/resize:fit:875/0*gygoLoyNQ5GhqYIi)

Lorsque vous cliquez dessus, vous verrez cette page :

![](https://miro.medium.com/v2/resize:fit:875/0*V4hC54cf0f-TkiRw)

Vous pouvez cocher les cases pour corriger les vulnérabilités que vous souhaitez corriger, puis cliquer sur le**Ouvrir un correctif PR**bouton. Une fois que vous cliquez dessus, un PR est créé sur votre référentiel avec le correctif.

![](https://miro.medium.com/v2/resize:fit:875/0*pWpCNzqEaFdKkMUh)

Vous êtes désormais libre de fusionner ou de rejeter la pull request.

## Emballer

Dans cet article, nous avons découvert Snyk, un outil qui peut nous aider à trouver les vulnérabilités et à les corriger. Ce n’était qu’un aperçu de base. Il y a beaucoup plus à apprendre à ce sujet.

Merci d'avoir lu! Vous pouvez me suivre[Twitter](https://twitter.com/ashutoshkrris)ou consultez[mon blog](https://ireadblog.com/).
