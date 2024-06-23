Les logiciels modernes s'appuient fortement sur des bibliothèques et des outils open source. GitHub rapporte que plus[90 % des applications modernes](https://github.blog/2022-11-17-octoverse-2022-10-years-of-tracking-open-source/#in-this-years-report-we-identified-three-big-trends-to-watch)exploitez-les pour accélérer le développement de logiciels. Même si cela accélère les choses, cela introduit également des risques potentiels pour la sécurité.

Les outils open source et les bibliothèques tierces peuvent contenir des failles de sécurité ou d'autres codes malveillants, qui peuvent mettre votre système en danger s'ils ne sont pas correctement surveillés ou mis à jour. Par exemple, en avril 2023, tant de packages de spam ont été téléchargés sur NPM que[le référentiel s'est brièvement déconnecté](https://www.scmagazine.com/analysis/malicious-campaigns-overwhelm-open-source-ecosystems-dos-npm). 

Comme le dit le proverbe : « Votre sécurité dépend de votre maillon le plus faible ».  Par conséquent, il est crucial de garantir que vos bibliothèques externes sont à jour et sécurisées. C'est là que le[Outil de vérification des dépendances du Open Web Application Security Project (OWASP)](https://owasp.org/www-project-dependency-check/)entre.

Dans cet article, nous passerons en revue OWASP Dependency-Check, comment le configurer et les meilleures pratiques pour son utilisation.

## Qu'est-ce que le contrôle de dépendance OWASP ?

OWASP Dependency-Check est un logiciel open source[analyse de la composition logicielle (SCA)](https://blog.codacy.com/software-composition-analysis-sca)outil qui détecte les vulnérabilités divulguées publiquement dans les dépendances des applications. Il a été créé par l'organisation OWASP pour répondre à l'un des[OWASP Top 10 des vulnérabilités : composants vulnérables et obsolètes](https://blog.codacy.com/owasp-top-10).

Dependency-Check fournit actuellement une prise en charge complète des produits basés sur Java, Node.js et .NET, ainsi qu'une prise en charge expérimentale de Python, Dart, SWIFT et Golang. Il peut être exécuté via l'interface de ligne de commande (CLI), en tant que tâche Ant ou via des plugins intégrés à Maven, Jenkins ou Gradle. 

OWASP Dependency-Check est généralement utilisé avec d'autres solutions d'analyse de sécurité pour créer une stratégie de sécurité plus globale. 

## Comment fonctionne le contrôle des dépendances OWASP ?

OWASP Dependency-Check identifie les vulnérabilités à l'aide d'analyseurs. Ces analyseurs analysent votre base de code à la recherche de bibliothèques open source, recherchent les vulnérabilités qu'elles contiennent et génèrent des rapports si une vulnérabilité est trouvée.

Voici comment fonctionnent les analyseurs sous le capot :

1.  **Analyse des dépendances :**Dependency-Check utilise des analyseurs pour examiner les dépendances de votre projet. Ces analyseurs collectent des informations (appelées preuves) sur chaque dépendance, comme son nom et sa version. Par exemple, dans un projet .NET utilisant des packages NuGet, Dependency-Check analyserait le fichier du projet pour identifier les dépendances telles que**Newtonsoft.Json version 13.0.3**ou**EntityFramework.Core version 8.0.0.**

2.  **Énumération des plates-formes communes (CPE) :**Une fois les dépendances identifiées, Dependency-Check utilise les informations collectées pour déterminer l'énumération de plate-forme commune (CPE) de chaque dépendance. Le CPE fournit un nom standardisé pour les composants et versions logiciels.

3.  **Correspondance des vulnérabilités :**Dependency-Check compare ensuite le CPE à la base de données NVD, qui contient les entrées CVE (Common Vulnerability and Exposure) de chaque package. Par exemple, si Dependency-Check analyse un projet .NET et identifie une dépendance telle que**Newtonsoft.Json version 13.0.3**, il créera un CPE et le comparera au CVE dans le NVD. Si une correspondance est trouvée, cela indique une vulnérabilité connue pour cette version spécifique de**Newtonsoft.Json**. Dependency-Check signalera cette dépendance comme vulnérable dans son rapport.

4.  **Génération de rapports:**Après la correspondance des vulnérabilités, Dependency-Check génère un rapport détaillé mettant en évidence les dépendances vulnérables, les vulnérabilités spécifiques et les niveaux de gravité. Voir cet exemple du[Projet Apache Struts 1](https://struts.apache.org/maven/dependency-check-report.html).

    ![](https://lh7-us.googleusercontent.com/dCR8_pSDhmdAIVG6evaTDxtDrgCFBPRKr1RtXiID6TVPLQInu9val4p4zoaZ2ACEpo6fkWPeeHQ5QRKFmPe_uYbeGgKWYKFI5ZoFaHO8IDSDxa3FNAQMYXgsI_6D1Iap7fDY6GRPORUl-0rvjoCjpWk)

Il est important de noter que le rapport ne répertorie une dépendance qu'une seule fois, même si elle est vulnérable à plusieurs endroits de votre projet. Vous devrez donc identifier toutes les zones concernées au sein de votre projet. De plus, les scores de vulnérabilité ne tiennent pas compte du contexte spécifique de votre application. C'est à vous d'évaluer si les vulnérabilités sont pertinentes pour l'utilisation de la bibliothèque de votre projet.

**5.Sources de données supplémentaires :**Pour des technologies spécifiques, Dependency-Check peut également utiliser d'autres sources de données pour identifier un plus large éventail de vulnérabilités qui pourraient ne pas être répertoriées dans le NVD. Par exemple:

-   L'API NPM Audit est spécialisée dans l'identification des vulnérabilités des packages NPM pour les projets JavaScript.
-   L'indice OSS couvre de nombreux projets open source, y compris ceux couramment utilisés dans le développement .NET.
-   RetireJS se consacre à la détection des bibliothèques JavaScript obsolètes présentant des problèmes de sécurité connus.
-   Bundler Audit se concentre sur les vulnérabilités des dépendances Ruby Gem pour les applications Ruby on Rails.

**6.Mises à jour automatiques:**Dependency-Check met automatiquement à jour ses données de vulnérabilité à l'aide de l'API NVD pour garantir que les rapports reflètent les informations les plus récentes. 

## Configuration de la vérification des dépendances OWASP dans .NET 8.0 (démo)

Ce guide montre comment exploiter OWASP Dependency-Check dans un projet .NET 8.0 pour identifier les vulnérabilités potentielles au sein de vos dépendances.

**Conditions préalables:**

-   SDK .NET 8.0 : assurez-vous d'avoir installé le dernier SDK .NET 8.0 sur votre système. Vous pouvez le télécharger depuis le site officiel[Site Web de Microsoft](https://dotnet.microsoft.com/en-us/download)

-   OWASP Dependency-Check : Téléchargez et installez le[Outil de vérification des dépendances OWASP](https://github.com/jeremylong/DependencyCheck)

-   Clé API NVD : obtenez le[Clé API NVD](https://nvd.nist.gov/developers/api-key-requested)

### Création d'un exemple d'application .NET 8.0

1.  Ouvrez un terminal ou une invite de commande et créez un nouveau répertoire (dependency-check-demo) pour votre projet.
2.  Pour cette démo, nous allons initialiser une nouvelle application console .NET 8.0 à l'aide de la commande suivante :


    <span><span>dotnet</span><span> </span><span>new</span><span> </span><span>console</span></span>

3.Ensuite, nous ajoutons une dépendance pour notre projet. Nous utiliserons la bibliothèque Newtonsoft.Json à des fins de démonstration. Ajoutez-le à votre projet en utilisant :

    <span><span>dotnet</span><span> </span><span>add</span><span> </span><span>package</span><span> </span><span>Newtonsoft</span><span>.</span><span>Json</span></span>

4.Ouvrez votre éditeur de code préféré et accédez au répertoire du projet. Dans le fichier Program.cs, remplacez le contenu existant par l'extrait de code suivant qui désérialise un objet JSON :

    <span><span>using</span><span> </span><span>System</span><span>;<br><span>using</span> <span>Newtonsoft</span>.<span>Json</span>;<br><br><span>public</span> <span>class</span> <span>Program</span><br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;<span>public</span> <span>static</span> <span>void</span> <span>Main</span>(<span>string</span>[] <span>args</span>)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>Console</span>.<span>WriteLine</span>(<span>"Hello, World!"</span>);<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>var</span> <span>json</span> = <span>@</span><span>"{'name':'John Smith','age':28}"</span>;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>dynamic</span> <span>obj</span> = <span>JsonConvert</span>.<span>DeserializeObject</span>(<span>json</span>);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>Console</span>.<span>WriteLine</span>(<span>$</span><span>"Hello, {obj.name}!"</span>);<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>}</span></span>

5.Créez et compilez l'application à l'aide de la commande suivante :

    <span><span>dotnet</span><span> </span><span>build</span></span>

6.Exécutez l'application à l'aide de la commande suivante :

    <span><span>dotnet</span><span> </span><span>run</span></span>

Si tout fonctionne correctement, vous devriez voir le message « Bonjour, John ! » message dans la console.

### Analyse avec la vérification des dépendances OWASP

Maintenant que nous disposons d'une application .NET 8.0 fonctionnelle, exploitons Dependency-Check pour analyser ses dépendances à la recherche de vulnérabilités en exécutant la commande suivante à partir de la ligne de commande. 

    <span><span>&nbsp;</span><span>dependency</span><span>-</span><span>check</span><span> --</span><span>scan</span><span> . --</span><span>format</span><span> </span><span>HTML</span><span> --</span><span>project</span><span> </span><span>"dependency-check-demo"</span><span> --</span><span>out</span><span> . --</span><span>nvdApiKey</span><span> &lt;</span><span>your</span><span>-</span><span>api</span><span>-</span><span>key</span><span>&gt;</span></span>

-   Dependency-check : appelle l'outil Dependency-Check
-   \--scan .: Spécifie le répertoire actuel ("".") à analyser pour les dépendances
-   \--format HTML : définit le format de sortie au format HTML pour un rapport convivial
-   \--project "dependency-check-demo" : définit le nom du projet pour référence dans le rapport
-   \--out . : demande à Dependency-Check de placer le rapport généré dans le répertoire actuel

### Affichage du rapport de vérification des dépendances OWASP

Après avoir exécuté la commande, vous devriez trouver un nouveau fichier nommé dependency-check-report.html dans le répertoire de votre projet. Ouvrez ce fichier dans un navigateur Web pour afficher le rapport détaillé sur les vulnérabilités identifiées dans les dépendances de votre projet. Dans notre cas, la version actuelle de**Newtonsoft.Json**n'a aucune vulnérabilité signalée, il apparaît donc comme vide.

![](https://lh7-us.googleusercontent.com/aLBvSii2urIo7O_YfenX7jkY3KCccTaQKulsjSYY9bLA8JZzd6knoPhXAHc4dtSPAP_IA9gLT7Meq7z3UZep2ucpyhE2UiP-WGFcD-uc3K6jcZazPGXsgQccdJ4VOd23AtOtD1mEhJJv_tU0ipzZAsU)

## Meilleures pratiques d'utilisation du contrôle de dépendance OWASP

OWASP Dependency-Check est un outil précieux pour identifier les vulnérabilités au sein des dépendances externes de votre projet. Voici quelques bonnes pratiques pour maximiser son efficacité :

1.  **Créez un inventaire de référence :**Le contrôle de dépendance de l'OWASP est plus efficace si vous disposez d'un inventaire interne auquel comparer. Commencez par créer un inventaire de référence de tous les composants open source et bibliothèques tierces utilisés dans vos projets logiciels. Cela inclut les bibliothèques directement utilisées dans votre base de code et les dépendances transitives. Un inventaire de référence permet de prioriser les dépendances qui nécessitent le plus d’attention et de suivre leur évolution.
2.  **Intégrer le contrôle des dépendances dans CI/CDProcess :**Intégrez l'analyse des dépendances dans votre pipeline CI/CD pour automatiser les analyses à chaque validation de code ou build planifiée. Des outils tels que Jenkins, GitLab CI/CD et Azure DevOps disposent souvent de plugins ou de scripts qui facilitent cette intégration. De plus, vous pouvez planifier des analyses manuelles régulières de votre projet pour identifier et traiter de manière proactive les risques de sécurité potentiels.
3.  **Tiens-toi à jour:**Assurez-vous d'utiliser la dernière version de Dependency-Check pour bénéficier des données de vulnérabilité les plus récentes.
4.  **Considérez les faux positifs et négatifs :**Bien que Dependency-Check s'efforce d'être précis, il existe une possibilité de faux positifs (identifiant une vulnérabilité inexistante) ou de faux négatifs (manquant une vulnérabilité réelle). Un examen manuel des dépendances signalées est toujours nécessaire.
5.  **Examiner les résultats de l'analyse :**Après chaque build, consultez le rapport OWASP Dependency-Check pour identifier et hiérarchiser les vulnérabilités à corriger. Intégrez les résultats à votre système de suivi des problèmes pour faciliter la gestion et le suivi des vulnérabilités.
6.  **Consultez les mesures de sécurité supplémentaires :**Dependency-Check est un outil précieux, mais ce n’est pas une solution miracle. Utiliser[directives de codage sécurisé](https://blog.codacy.com/secure-coding-standards-in-agile-development)et autre[tests de sécurité des applications](https://blog.codacy.com/application-security-testing-ast)méthodes pour améliorer la posture de sécurité globale de votre application.

## Étendez vos contrôles de dépendance aux analyses de sécurité complètes

OWASP Dependency-Check est un outil précieux pour identifier les vulnérabilités tierces dans les dépendances du projet, mais ce n'est pas suffisant. Compléter Dependency-Check avec d'autres outils de tests de sécurité automatisés peut être très bénéfique pour améliorer la qualité de votre[sécurité des applications](https://blog.codacy.com/what-is-appsec)programme.

La sécurité de Codacy va au-delà[Contrôle de dépendance](https://blog.codacy.com/insecure-dependencies-detection)en fournissant une gamme de fonctionnalités avancées, notamment :

-   **Analyse statique :**Détectez les vulnérabilités et les problèmes de sécurité potentiels dans votre base de code.
-   **Vérifications de conformité des licences tierces :**Assurez le respect de toutes les exigences de licence pour vos projets open source.
-   **Flux de travail d’atténuation vérifiables :**Documentez et suivez l’ensemble du processus de résolution des vulnérabilités.
-   **Recommandations sur les meilleures pratiques de sécurité en temps réel :**Offrez aux développeurs des informations et des recommandations exploitables pour améliorer la sécurité globale de votre code.

Commencez pour[gratuit aujourd'hui](https://www.codacy.com/signup-codacy).

Le projet de sécurité des applications Web ouvertes ([OWASP](https://owasp.org/)), est une communauté en ligne qui produit des articles, des méthodologies, de la documentation, des outils et des technologies gratuits et accessibles au public dans le domaine de[sécurité des applications Web](https://www.mend.io/blog/web-application-security/).

Les composants open source font désormais partie intégrante du développement logiciel. Selon[Rapport sur les risques de Mend](https://www.mend.io/risk-report/), 96,8 % des développeurs s'appuient sur des composants open source. L'utilisation de plus en plus répandue des composants open source nécessite que les développeurs adoptent une approche plus proactive en matière de gestion de la sécurité open source. Ils doivent s’assurer tout au long du processus de développement que les produits logiciels qu’ils créent et maintiennent ne contiennent pas de composants vulnérables.

Dans l'espoir de rendre le travail avec des composants open source plus sécurisé, les bonnes personnes de l'OWASP ont publié un[utilitaire gratuit](https://owasp.org/www-project-dependency-check/)créé pour les développeurs, qui identifie le projet[dépendances](https://www.mend.io/blog/dependency-management-a-guide-and-3-tips-to-keep-you-sane/)et vérifie s’ils contiennent des vulnérabilités open source connues et divulguées publiquement.

Nous avons examiné la fonctionnalité de l'OWASP Dependency-Check, ainsi que ses fonctionnalités et intégrations, et je suis ici pour partager ce que nous avons trouvé.

## Langages de programmation et intégrations

L'OWASP Dependency-Check prend actuellement en charge cinq langages de programmation différents. Java et .NET sont entièrement pris en charge et une prise en charge expérimentale supplémentaire est fournie pour Ruby, Node.js et Python.

L'adoption généralisée de l'open source oblige les développeurs soucieux de la sécurité de leurs projets logiciels à intégrer des outils de gestion open source dans leur[Cycle de vie du développement logiciel (SDLC)](https://www.mend.io/blog/sdlc-software-development-life-cycle/). Dependency-Check permet aux développeurs de rester au top de leurs composants open source dès le début du processus de développement grâce à la prise en charge de l'intégration en ligne de commande. Cela permet une intégration transparente avec d'autres outils, systèmes de construction et API, aidant ainsi les développeurs à détecter les vulnérabilités de sécurité le plus tôt possible dans le processus CI/CD, sans interférer avec le temps de développement.

L'outil OWASP prend également en charge le plugin Jenkins et peut faire échouer le processus de construction, vous permettant de vous assurer que seul le code approuvé sans vulnérabilités open source est déployé en production.

## Base de données de vulnérabilités et mises à jour

Selon le rapport IBM XForce, le délai moyen entre le signalement d'une vulnérabilité et son exploitation est passé de 45 jours à 15 jours entre 2006 et 2015. Une autre étude de Tenable a calculé que la fenêtre d'exploitation de la vulnérabilité était de 5,5 jours. Dans ces situations, la capacité à détecter correctement les vulnérabilités et à les corriger rapidement devient encore plus importante.

Le Dependency-Check OWASP ne couvre actuellement que les vulnérabilités extraites du[MVN](https://nvd.nist.gov/). Bien qu'il s'agisse d'une base de données de vulnérabilités très respectée et populaire, le processus d'évaluation et de vérification des vulnérabilités prend un certain temps. Cela signifie qu'une vulnérabilité peut rester présente pendant un certain temps avant d'être ajoutée au NVD.

Alors que les vulnérabilités des logiciels commerciaux sont régulièrement signalées au NVD, la recherche et la divulgation des vulnérabilités dans la communauté open source sont des processus moins centralisés. Cela signifie que certaines vulnérabilités open source divulguées peuvent être trouvées dans les outils de suivi de bogues publics, les avis de sécurité ou les forums, plutôt que dans le NVD.

Un autre facteur est que, comme c'est généralement le cas pour les outils gratuits, la base de données de vulnérabilités pour le contrôle de dépendance OWASP est stockée localement. Cela nécessite que les utilisateurs s'assurent qu'ils mettent fréquemment à jour la base de données locale. Cela diffère des bases de données stockées dans le cloud, qui peuvent être mises à jour automatiquement. Cela signifie, encore une fois, qu’il existe un risque de manquer des vulnérabilités lorsque la machine locale n’est pas à jour.

## Analyse des vulnérabilités

L'analyse est le processus d'exécution de l'outil sur le code de l'utilisateur, pour identifier tout composant open source vulnérable. Cela se fait généralement en effectuant une comparaison entre le code de l’utilisateur et les vulnérabilités open source connues dans la base de données des vulnérabilités.  

L'OWASP Dependency-Check utilise une variété d'analyseurs pour construire une liste de[Énumération des plates-formes communes (CPE)](https://en.wikipedia.org/wiki/Common_Platform_Enumeration)entrées. CPE est un schéma de dénomination structuré, qui inclut une méthode de vérification des noms par rapport à un système. L'analyseur vérifie une combinaison de groupId, d'artefactId et de version (parfois appelée GAV) dans le fichier[Projet Maven](https://www.mend.io/free-developer-tools/blog/maven-update-dependencies-automatically/)Fichier de modèle d'objet (fichier POM.XML). Cela pourrait conduire à une identification incorrecte des composants et entraîner un taux élevé de faux positifs et de faux négatifs, par opposition au calcul du[SHA-1](https://en.wikipedia.org/wiki/SHA-1)du fichier, ce qui est beaucoup plus précis car chaque composant obtient un identifiant unique.

Le signalement est extrêmement important lorsqu’il s’agit de[gestion des vulnérabilités](https://www.mend.io/blog/vulnerability-management/), car il fournit à toutes les équipes de sécurité et de développement des informations exploitables, tout en donnant aux parties prenantes les mesures dont elles ont besoin. L'OWASP Dependency-Check peut répondre à ces besoins et générer des rapports et des exportations dans une variété de formats : XML, CSV, JSON et HTML.  

## Vérification des dépendances OWASP : avantages et inconvénients

![OWASP Dependency Check Pros and Cons](https://www.mend.io/wp-content/media/2021/04/aHViPTcyNTE0JmNtZD1pdGVtZWRpdG9yaW1hZ2UmZmlsZW5hbWU9aXRlbWVkaXRvcmltYWdlXzViZmZjNzMwODEzYzguanBnJnZlcnNpb249MDAwMCZzaWc9MWExYTIzYTEwZjY5NzlmOTk1ZTdlNzk5ODQ5M2VhMzQ.jpeg)

Les développeurs sont extrêmement préoccupés par les vulnérabilités de sécurité open source, et le contrôle des dépendances de l’OWASP contribue grandement à leur fournir un outil facile à utiliser pour analyser leur code.

Clairement, le fait qu’il soit gratuit pour les développeurs est un grand avantage. Les développeurs n'ont pas besoin d'attendre que leurs responsables approuvent et achètent l'outil gratuit d'OWASP. Il n’y a pas besoin d’un long processus POC car ils peuvent simplement le télécharger eux-mêmes.

Ce qui nous amène à un autre avantage. L'OWASP Dependency-Check est léger et très facile à télécharger, à installer et à exécuter. Les utilisateurs n’ont pas besoin de passer du temps à parcourir un long processus de déploiement, à résoudre tous les problèmes qui pourraient survenir lors de l’adoption d’un nouvel outil de développement. L'installation et l'utilisation de Dependency-Check sont un processus sans effort, à condition que les utilisateurs n'oublient pas de mettre à jour souvent leur copie locale.

La variété des options de reporting et d'exportation constitue également un grand avantage pour les utilisateurs qui souhaitent garder un œil attentif sur les alertes de sécurité des vulnérabilités open source et en rester au courant. La possibilité d'exporter facilement des rapports permet également aux équipes de[collecter des métriques](https://www.mend.io/blog/vulnerability-management-metrics/)et obtenez un aperçu de leurs capacités de gestion des vulnérabilités open source au fil du temps.

L'OWASP Dependency-Check fournit aux équipes de développement un outil puissant pour commencer leur parcours vers la gestion de leur[sécurité open source](https://www.mend.io/open-source-security/). Cependant, comme la plupart des outils gratuits, il n’offre pas toutes les fonctionnalités qu’un[Outil d'analyse de la composition logicielle](https://www.mend.io/blog/software-composition-analysis/)peut fournir.

Dependency-Check n'offre pas aux utilisateurs la possibilité de créer des règles ou des flux de travail automatiques pour corriger les vulnérabilités. Ainsi, une fois que les utilisateurs reçoivent leur rapport répertoriant toutes les vulnérabilités de sécurité open source dans leur code, c'est à eux de déterminer comment les résoudre et de planifier les tâches de correction des vulnérabilités ou de correctifs dans leur calendrier déjà serré.

Bien que les développeurs puissent utiliser OWASP Dependency-Check pour un rapport dans un certain nombre de formats, le contenu du rapport n'est pas très modulaire. Les utilisateurs ne peuvent pas créer de rapports spéciaux pour produire des données de haut niveau, comme l'analyse du nombre d'alertes au fil du temps ou en fonction d'autres paramètres spécifiques. Tout type d'analyse de niveau supérieur devra être collecté manuellement à partir des rapports, puis analysé par l'équipe.

Un autre aspect du reporting qui manque dans le Dependency-Check concerne les tableaux de bord. Les utilisateurs ne disposent pas d'une ressource dans l'outil où ils peuvent consulter un aperçu de l'état de leur sécurité open source, des alertes de vulnérabilité, des délais, etc. Il s'agit de données qu'ils devront collecter et organiser manuellement.

La réponse courte à cette question est oui. L'OWASP Dependency-Check est un excellent outil gratuit pour les développeurs, leur fournissant certaines des données initiales dont ils ont besoin pour la gestion des vulnérabilités open source.

Cela dit, les capacités d’analyse de l’outil, le fait qu’il soit stocké localement et le nombre de faux positifs générés par ses analyses rendent son utilisation difficile pour les organisations qui ont besoin d’une solution complète de gestion de la sécurité open source.

Comme tous les outils gratuits, l’OWASP Dependency-Check a ses avantages et ses limites. Bien que nous recommandions aux développeurs qui n'utilisent aucune technologie pour sécuriser leur utilisation de l'open source de le télécharger et de l'essayer par eux-mêmes, les organisations recherchant des contrôles plus automatisés et des fonctionnalités adaptées à leurs besoins spécifiques peuvent décider de chercher une solution ailleurs.

### **Qu’est-ce que l’OWASP ?**

L'Open Web Application Security Project (OWASP) est une organisation en ligne à but non lucratif composée de bénévoles du monde entier qui cherchent à aider les experts en sécurité à protéger leurs applications Web contre les cyberattaques. Fondée en 2001, OWASP produit des outils, documentations, méthodologies, articles et technologies disponibles gratuitement.

Aujourd'hui, l'OWASP compte plus de 32 000 bénévoles qui participent activement aux évaluations et à la recherche sur les incidents de sécurité. L'un de leurs principes fondamentaux qui a permis leur popularité est que tous leurs documents sont librement accessibles à tous pour sécuriser leur application Web.

Cet article montre comment utiliser le vérificateur de dépendances OWASP pour rechercher les vulnérabilités dans un code Node-JS.

### **Vérificateur de dépendances OWASP**

OWASP Dependency Checker est un outil open source d'analyse de la composition logicielle (SCA) qui identifie les dépendances du projet sur le code source du stylo et vérifie les vulnérabilités connues associées à ce code. Cet outil fait partie de la solution à l'un des Top 10 OWASP de 2017 intitulé « A9 – Utilisation de composants présentant des vulnérabilités connues ».

Aujourd'hui, l'OWASP Dependency Checker offre une prise en charge complète des langages de programmation .NET et .java, une prise en charge expérimentale de Node.JS, Python et Ruby, ainsi qu'une prise en charge limitée des langages de programmation C et C++.

### **Importance de la sécurité du code**

On estime que quatre-vingts pour cent du code des applications actuelles provient de bibliothèques et de frameworks open source. De plus, environ 27 % des bibliothèques disponibles sur Internet présentent des vulnérabilités bien connues et divulguées publiquement.

Ce qui est inquiétant, c'est que la plupart des organisations et des développeurs individuels continuent d'utiliser les bibliothèques dans leur code sans remédier aux vulnérabilités. L'utilisation d'une bibliothèque vulnérable peut permettre à des acteurs malveillants d'accéder à des données confidentielles, d'effectuer des transactions et, dans certains cas, de prendre le contrôle total d'une application.

En tant que tel, les développeurs doivent faire attention aux bibliothèques qu’ils utilisent dans leur code.

Regardez cette vidéo how-t0 pour regarder le didacticiel et suivez les étapes ci-dessous.

### **Installation via la version CLI**

Pour ce guide, j'installerai OWASP Dependency Checker dans Ubuntu et je l'utiliserai pour analyser un projet Node.JS. Avant de continuer, assurez-vous que Java est installé et exécuté. Vous pouvez confirmer si votre installation Java fonctionne à l'aide de la commande JAVA –HELP

Sur ce, voici le processus étape par étape d'installation d'OWASP Dependency Check :

1.  Téléchargez le contrôle de dépendance OWASP sur le site Web de l'OWASP. Puisque nous avons l'intention de déployer cet outil SCA sur une ligne de commande, nous allons télécharger la version CLI. Cette archive contient les fichiers pour le terminal Linux et une ligne de commande Windows.

![](https://static.wixstatic.com/media/37f7aa_17b26b0beb5f4233a080c7d5201474db~mv2.jpg/v1/fill/w_802,h_610,al_c,q_85,enc_auto/37f7aa_17b26b0beb5f4233a080c7d5201474db~mv2.jpg)

2.L'étape suivante consiste à extraire les fichiers à l'aide de la commande unzip puisque nous sommes sur un terminal basé sur Linux.

![](https://static.wixstatic.com/media/37f7aa_eb10f3f71d7e4a89a52dc174b98ca90b~mv2.jpg/v1/fill/w_883,h_280,al_c,lg_1,q_80,enc_auto/37f7aa_eb10f3f71d7e4a89a52dc174b98ca90b~mv2.jpg)

3.Le dossier bin contient dependency-check.sh pour le terminal Linux et dependency-check.bat pour l'invite de commande Windows.

![](https://static.wixstatic.com/media/37f7aa_5158d5691e214c78aeb18e563f536b4a~mv2.jpg/v1/fill/w_925,h_233,al_c,lg_1,q_80,enc_auto/37f7aa_5158d5691e214c78aeb18e563f536b4a~mv2.jpg)

4.Vous pouvez tester si votre installation fonctionne correctement en exécutant le**./**Commande dependency-check.sh. Il doit donner une liste d’arguments pouvant être utilisés avec l’outil.

![](https://static.wixstatic.com/media/37f7aa_946fa9f24434481e94adb6887a8065f7~mv2.jpg/v1/fill/w_925,h_243,al_c,lg_1,q_80,enc_auto/37f7aa_946fa9f24434481e94adb6887a8065f7~mv2.jpg)

**Installation via Brew**

Si vous utilisez un Mac, vous pouvez installer OWASP Dependency-Check en utilisant la commande Brew.

    <span><span>brew update &amp;&amp; brew install dependency-check
    </span></span>

**Analyse du code JS du nœud**Avant de procéder à l'analyse du code, voici trois arguments de base utilisés avec le contrôle de dépendance OWASP.

1.--projet<name>- Vous permet de nommer le projet que vous numérisez

2.--analyse<path>– Ceci indique le fichier ou le dossier à analyser

3.--dehors<path>– C'est le chemin où le vérificateur de dépendances enregistrera les résultats

Pour analyser du code source, exécutez la vérification des dépendances en lui fournissant le nom du projet, les fichiers à analyser et le chemin d'accès à l'emplacement de sortie, comme indiqué ;

Pour cette analyse de projet, nous utilisons un exemple de code NodeJS du[API de cybersécurité HacWare](https://www.hacware.com/dev). Ce projet est simple et ne dépend que du package "axios". C'est le résultat de l'analyse.

![](https://static.wixstatic.com/media/37f7aa_edf4919d487845f18c4e8672a6cc0565~mv2.png/v1/fill/w_925,h_643,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_edf4919d487845f18c4e8672a6cc0565~mv2.png)

![](https://static.wixstatic.com/media/37f7aa_cff12a9d4a5a4d48b9673552c99e900e~mv2.png/v1/fill/w_925,h_581,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_cff12a9d4a5a4d48b9673552c99e900e~mv2.png)

Si vous exécutez l'analyse pour la première fois, le téléchargement et la configuration des détails CVE prendront un certain temps. Les analyses ultérieures seront plus rapides et pourront être effectuées sans connexion Internet active.

**Interprétation du résultat**

Une fois l'analyse terminée, vous pouvez consulter le résultat imprimé sur la ligne de commande et l'analyseur fournit un rapport intitulé "dependency-check-report.html". Dans le rapport de vérification des dépendances, il fournit un résumé de ses conclusions et un rapport détaillé de chaque vulnérabilité et de son indice de gravité.

![](https://static.wixstatic.com/media/37f7aa_6713e13a802848a3a098277a620bb6c0~mv2.png/v1/fill/w_925,h_536,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_6713e13a802848a3a098277a620bb6c0~mv2.png)

Chaque vulnérabilité explique où se trouve le fichier dans votre projet et le problème révélé publiquement avec la vulnérabilité. Le rapport présente le score CVS (Common Vulnerability Scoring) pour aider les développeurs à prioriser les réponses et les ressources aux menaces.

![](https://static.wixstatic.com/media/37f7aa_130d0b10353c41b296fadf3a1e2936c2~mv2.png/v1/fill/w_925,h_504,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_130d0b10353c41b296fadf3a1e2936c2~mv2.png)

### **Automatisation des analyses de vérification des dépendances**

Cet outil peut être exécuté via l'interface de ligne de commande en tant que tâche Ant ou via des plugins avec Gradle, Maven ou Jenkins. Il n'offre aucune automatisation intégrée, mais si nécessaire, un module complémentaire d'automatisation peut être ajouté.

### **Dernières pensées**

Il est important que les développeurs de logiciels trouvent des moyens efficaces de mettre en œuvre des pratiques de codage sécurisées pour créer des applications et des outils résilients. L'outil OWASP Dependency-Check vous indique s'il faut détecter les vulnérabilités dans votre code dépendant et prendre des décisions au moment du développement pour mettre à niveau vers une version corrigée ou trouver un autre moyen de répondre aux exigences du projet.

### **Vous souhaitez en savoir plus sur notre API de cybersécurité ?**

HacWare permet aux développeurs de logiciels de lancer facilement des programmes éducatifs sur la cybersécurité de nouvelle génération pour lutter contre les attaques de phishing avec quelques lignes de code. Pour en savoir plus sur notre puissante API de sensibilisation à la sécurité et notre programme pour développeurs,[cliquez ici pour postuler](https://www.hacware.com/dev).

### **Les références**
