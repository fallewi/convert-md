## A tutorial on using Snyk to find security vulnerabilities in our code and fix them.



## What is Snyk?

Snyk (pronounced _sneak_) is a developer security platform for securing code, dependencies, containers, and infrastructure as code. So what it does is, it scans your code, reads through it and tells you if you have any vulnerabilities in your code. Now it doesn’t only check your code, it can check the installed dependencies, your Docker container, your Infrastructure as Code, and a few other things too. It is compatible with a lot of languages and comes with plugins supported by different IDEs. So, it is basically Grammarly for your code.

## Getting Started with Snyk

To get started, you need to create an account on Snyk. Head over to [https://snyk.io/](https://snyk.io/) and register for a free account. I’d recommend you log in through Github. Once registered, you can log in to your account. After logging in, you will be able to see a similar dashboard.

![](https://miro.medium.com/v2/resize:fit:875/0*8zTx1RctHKorAG9J)

Now you can go to [this link](https://docs.snyk.io/snyk-cli/install-the-snyk-cli) and follow the instructions to download the Snyk CLI. There are various methods to download the Snyk CLI. You can go ahead with any one of them.

Now if you’re here, I assume you have already installed Snyk CLI using any of the available methods. Now what we need to do is to authenticate ourselves with Snyk CLI. To do that, run the following command in the terminal:

```
<span id="7806" data-selectable-paragraph="">snyk auth</span>
```

When you run the command, an authentication page will be opened in your default browser as below:

![](https://miro.medium.com/v2/resize:fit:875/0*ksUJI8o-UoQbsFdv)

Just click on the **Authenticate** button and wait for the page to show a success message. Once you see the message, you can go to your terminal where you’ll find a similar output as below:

![](https://miro.medium.com/v2/resize:fit:875/0*6DVd2KSWT_G-WvSr)

Now, the Synk CLI has been connected to your account.

## Finding Vulnerabilities in Demo App

For the demo purpose, we’re going to use a web application called PyGoat written in Django. A lot of vulnerabilities have been added to the app intentionally, so we can have a good demo of Snyk around it.

Here is the link to the Github repository: [https://github.com/purpledobie/pygoat](https://github.com/purpledobie/pygoat). Open the repository link, click on Fork and then clone the forked repository to your local machine. When you go through the repository, you will find Dockerfile, Infrastructure as Code file as well as standard Python files. We’ll go through the files later. You can install the Python dependencies from the **requirements.txt** file.

```
<span id="8d49" data-selectable-paragraph="">pip install -r requirements.txt</span>
```

## Snyk Plugins

Snyk has plugins available for different IDEs such as Eclipse, VS Code, and Jetbrains (PyCharm, IntelliJ, etc). Since I am on VS Code, I have installed the Snyk extension on my IDE. You can do the same for your IDE.

![](https://miro.medium.com/v2/resize:fit:875/0*px1G7p0GCClqJ8YK)

Once you install the extension, you may need to authenticate again. Once authenticated, the plugin will start scanning the code automatically, after a few seconds, it will show the results similar to below:

![](https://miro.medium.com/v2/resize:fit:875/0*BTlgkPJt6T8ELada)

You can see there are many 18 code security vulnerabilities and 2 code quality issues in the code. Each issue or vulnerability has an icon beside it. It can be **C**, **H**, **M**, and **L** meaning **Critical**, **High**, **Medium**, and **Low** respectively. You can click on any of them to learn more and it would even suggest fixes for the issue or vulnerability.

## Snyk CLI Commands

We have already run one Synk CLI command i.e. `**snyk auth**` to authenticate ourselves with Snyk. Now let us look at some other important commands.

## 1\. `snyk test`

This command will scan the code and show you any vulnerabilities. Let’s run this and see what output we get:

![](https://miro.medium.com/v2/resize:fit:875/0*XhbNDc8on0kwDu9f)

You can see that it has finished scanning and has found the same vulnerabilities. The vulnerabilities are again marked as Low, Medium, High and Critical. Apart from that, it also provides us with suggestions to fix the issue. For example, if you see the above image, it suggests us to upgrade Django from version 3.1.12 to 3.2.13 to resolve a lot of issues. Let us upgrade the Django and then rescan the application to see if those vulnerabilities have been fixed or not.

Let us first upgrade the Django version to 3.2.13 using the command:

```
<span id="ffa7" data-selectable-paragraph="">pip install django==3.2.13</span>
```

You will get a similar output:

![](https://miro.medium.com/v2/resize:fit:875/0*xD3l2YUK2s8llnbd)

Now let us rescan the code using `snyk test` command.

![](https://miro.medium.com/v2/resize:fit:875/0*rU21WqB2CSlu2BVY)

Now, if you notice, we don’t have those vulnerabilities such as SQL Injection.

## 2\. `snyk monitor`

What this command does is, it scans through the code and uploads a snapshot of it on the Snyk UI or Snyk Platform. Let’s first run the command:

![](https://miro.medium.com/v2/resize:fit:875/0*R56SEhUq_0mJhZYJ)

The command has taken a snapshot of the project and uploaded it to the Snyk Platform. It then gives us a URL where we can see a lot of other information regarding the project. If you open the URL, you will see a similar page:

![](https://miro.medium.com/v2/resize:fit:875/0*OuwKhskkH49j_Qv1)

It is now easier to see the vulnerabilities in the application. You can also retest the application by clicking on the **Retest Now** link. You can also see the **Fixes** and **Dependencies** of the application.

## 3\. Scan Infrastructure as Code

If you look at the project, you will find a folder called **infrastructure**. Within that, we have an **application-load-balancer** folder. So, this project can be deployed to AWS load balancer. There is a Python file `app.py` that actually generates a template for the configuration of the load balancer on AWS. Then in the **cdk.out** folder, you can find a **LoadBalancerStack.template.json** file generated from the Python code.

To scan for any misconfigurations before deployment, we can actually test this file using Snyk. The command for the same is:

```
<span id="94d9" data-selectable-paragraph="">snyk iac test &lt;template-file-path&gt;</span>
```

Let us run and see the output:

![](https://miro.medium.com/v2/resize:fit:875/0*lcpEwq5BdEg5cf5T)

It shows all the issues and vulnerabilities in the template file.

## 4\. Scan Dockerfile and Docker Images

In the project, we do have a Dockerfile. You can build the Docker image from the Dockerfile using the command:

```
<span id="4f3f" data-selectable-paragraph="">docker build -t pygoat .</span>
```

You can see the image being created below:

![](https://miro.medium.com/v2/resize:fit:875/0*w_RjI-IFo277ozpC)

Once the image is built, you can scan for vulnerabilities using the command:

```
<span id="b23f" data-selectable-paragraph="">docker scan pygoat</span>
```

The integration of the Snyk with Docker makes it incredibly simple to scan.

You will get the output as below:

![](https://miro.medium.com/v2/resize:fit:875/0*pfu5C7mWL0KHuThL)

The output is very large and it is not possible to show what all is there. But we can see the vulnerabilities in the image.

## Snyk Integration with Github

Snyk can automatically fix issues for you. When you link a Github repository to Snyk, it will scan the entire project and if it has a fix around any vulnerability, it will create a Pull Request with the fix. Isn’t that amazing?

Since we already logged in using Github, Snyk has already access to our repositories. We just need to select the repository we wish to scan.

Click on the Add project button, then click on Github and select your repository.

![](https://miro.medium.com/v2/resize:fit:875/0*KlH7J6rU0G9LGhdV)

Once you add the project, you can find it on the Dashboard. Snyk will automatically scan the project. Once you see the issues, you will see a **Fix this vulnerability** (for each vulnerability) or **Fix these vulnerabilities** (for fixing all vulnerabilities) buttons.

![](https://miro.medium.com/v2/resize:fit:875/0*gygoLoyNQ5GhqYIi)

When you click on it, you will see this page:

![](https://miro.medium.com/v2/resize:fit:875/0*V4hC54cf0f-TkiRw)

You can select the checkboxes to fix the vulnerabilities you want to fix and then click on the **Open a Fix PR** button. Once you click on it, a PR is created on your repository with the fix.

![](https://miro.medium.com/v2/resize:fit:875/0*pWpCNzqEaFdKkMUh)

Now you are free to merge or reject the pull request.

## Wrapping Up

In this article, we learned about Snyk, a tool that can help us find vulnerabilities and fix them. This was just a basic overview of it. There’s much more to learn about it.

Thanks for reading! You can follow me on [Twitter](https://twitter.com/ashutoshkrris) or check out [my blog](https://ireadblog.com/).
