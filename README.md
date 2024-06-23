Modern software heavily relies on open-source libraries and tools. GitHub reports that over [90% of modern applications](https://github.blog/2022-11-17-octoverse-2022-10-years-of-tracking-open-source/#in-this-years-report-we-identified-three-big-trends-to-watch) leverage them to accelerate software development. While this speeds things up, it also introduces potential security risks.

Open-source tools and third-party libraries may contain security vulnerabilities or other malicious code, which can put your system at risk if not properly monitored or updated. For instance, in April 2023, so many spam packages were uploaded to NPM that [the repository briefly went offline](https://www.scmagazine.com/analysis/malicious-campaigns-overwhelm-open-source-ecosystems-dos-npm). 

As the saying goes, "You're only as secure as your weakest link."  Therefore, it is crucial to ensure your external libraries are up-to-date and secure. This is where the [Open Web Application Security Project's (OWASP) Dependency-Check tool](https://owasp.org/www-project-dependency-check/) comes in.

In this article, we will review OWASP Dependency-Check, how to set it up, and best practices for its use.

## What is OWASP Dependency-Check?

OWASP Dependency-Check is an open-source [software composition analysis (SCA)](https://blog.codacy.com/software-composition-analysis-sca) tool that detects publicly disclosed vulnerabilities in application dependencies. It was created by the OWASP organization to address one of the [OWASP Top 10 vulnerabilities: Vulnerable and outdated components](https://blog.codacy.com/owasp-top-10).

Dependency-Check currently provides comprehensive support for Java, Node.js, and .NET-based products, as well as experimental support for Python, Dart, SWIFT, and Golang. It can be executed via the command-line interface (CLI), as an Ant task, or through plugins integrated with Maven, Jenkins, or Gradle. 

OWASP Dependency-Check is usually used alongside other security scanning solutions to create a more holistic security strategy. 

## How Does OWASP Dependency-Check Work?

OWASP Dependency-Check identifies vulnerabilities using analyzers. These analyzers scan your code base for open-source libraries, check for vulnerabilities in them, and generate reports if a vulnerability is found.

Here's how analyzers work under the hood:

1. **Dependency Analysis:** Dependency-Check uses analyzers to examine your project's dependencies. These analyzers collect information (called evidence) about each dependency, like its name and version. For example, in a .NET project using NuGet packages, Dependency-Check would scan the project file to identify dependencies like **Newtonsoft.Json version 13.0.3** or **EntityFramework.Core version 8.0.0.**

2. **Common Platform Enumeration (CPE):** Once the dependencies are identified, Dependency-Check uses the collected information to determine each dependency's Common Platform Enumeration (CPE). The CPE provides a standardized name for software components and versions.

3. **Vulnerability Matching:** Dependency-Check then compares the CPE against the NVD database, which contains each package's Common Vulnerability and Exposure (CVE) entries. For example, if Dependency-Check scans a .NET project and identifies a dependency like **Newtonsoft.Json version 13.0.3**, it will create a CPE and compare it against the CVE in the NVD. If a match is found, it indicates a known vulnerability for that specific version of **Newtonsoft.Json**. Dependency-Check will flag this dependency as vulnerable in its report.  

4. **Report Generation:** After vulnerability matching, Dependency-Check generates a detailed report highlighting the vulnerable dependencies, specific vulnerabilities, and severity levels. See this example from the [Apache Struts 1 project](https://struts.apache.org/maven/dependency-check-report.html).

   ![](https://lh7-us.googleusercontent.com/dCR8_pSDhmdAIVG6evaTDxtDrgCFBPRKr1RtXiID6TVPLQInu9val4p4zoaZ2ACEpo6fkWPeeHQ5QRKFmPe_uYbeGgKWYKFI5ZoFaHO8IDSDxa3FNAQMYXgsI_6D1Iap7fDY6GRPORUl-0rvjoCjpWk)

It's important to note that the report only lists a dependency once, even if it's vulnerable in multiple locations within your project. So, you'll need to identify all affected areas within your project. Also, the vulnerability scores don't consider your specific application context. It's up to you to evaluate whether the vulnerabilities are relevant to your project's library usage.

**5\. Additional Data Sources:** For specific technologies, Dependency-Check may also use other data sources to identify a wider range of vulnerabilities that might not be listed in the NVD. For example:

-   The NPM Audit API is specialized in identifying vulnerabilities in NPM packages for JavaScript projects.
-   The OSS Index covers many open-source projects, including those commonly used in .NET development.
-   RetireJS is dedicated to detecting outdated JavaScript libraries with known security issues.
-   Bundler Audit focuses on vulnerabilities in Ruby Gem dependencies for Ruby on Rails applications.

**6\. Automatic Updates:** Dependency-Check automatically updates its vulnerability data using NVD API to ensure that reports reflect the most recent information. 

## Setting Up OWASP Dependency-Check in .NET 8.0 (Demo)

This guide demonstrates how to leverage OWASP Dependency-Check in a .NET 8.0 project to identify potential vulnerabilities within your dependencies.

**Prerequisites:**

-   .NET 8.0 SDK: Ensure you have installed the latest .NET 8.0 SDK on your system. You can download it from the official [Microsoft website](https://dotnet.microsoft.com/en-us/download)  

-   OWASP Dependency-Check: Download and install the [OWASP Dependency-Check tool](https://github.com/jeremylong/DependencyCheck)  

-   NVD API key: Get the [NVD API key](https://nvd.nist.gov/developers/api-key-requested)

### Creating a Sample .NET 8.0 Application

1.  Open a terminal or command prompt and create a new directory (dependency-check-demo) for your project.
2.  For this demo, we will be  initializing  a new .NET 8.0 console application using the following command:

```
<span><span>dotnet</span><span> </span><span>new</span><span> </span><span>console</span></span>
```

3\. Next, we add a dependency for our project. We'll use the Newtonsoft.Json library for demonstration purposes. Add it to your project using:

```
<span><span>dotnet</span><span> </span><span>add</span><span> </span><span>package</span><span> </span><span>Newtonsoft</span><span>.</span><span>Json</span></span>
```

4\. Open your preferred code editor and navigate to the project directory. In the Program.cs file, replace the existing content with the following code snippet that deserializes a JSON object:

```
<span><span>using</span><span> </span><span>System</span><span>;<br><span>using</span> <span>Newtonsoft</span>.<span>Json</span>;<br><br><span>public</span> <span>class</span> <span>Program</span><br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;<span>public</span> <span>static</span> <span>void</span> <span>Main</span>(<span>string</span>[] <span>args</span>)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>Console</span>.<span>WriteLine</span>(<span>"Hello, World!"</span>);<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>var</span> <span>json</span> = <span>@</span><span>"{'name':'John Smith','age':28}"</span>;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>dynamic</span> <span>obj</span> = <span>JsonConvert</span>.<span>DeserializeObject</span>(<span>json</span>);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span>Console</span>.<span>WriteLine</span>(<span>$</span><span>"Hello, {obj.name}!"</span>);<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>}</span></span>
```

5\. Build and compile the application using the following command:

```
<span><span>dotnet</span><span> </span><span>build</span></span>
```

6\. Run the application using the following command:

```
<span><span>dotnet</span><span> </span><span>run</span></span>
```

If everything functions correctly, you should see the "Hello, John!" message in the console.

### Scanning With OWASP Dependency-Check

Now that we have a functional .NET 8.0 application let's leverage Dependency-Check to scan its dependencies for vulnerabilities by running the following command from the command line. 

```
<span><span>&nbsp;</span><span>dependency</span><span>-</span><span>check</span><span> --</span><span>scan</span><span> . --</span><span>format</span><span> </span><span>HTML</span><span> --</span><span>project</span><span> </span><span>"dependency-check-demo"</span><span> --</span><span>out</span><span> . --</span><span>nvdApiKey</span><span> &lt;</span><span>your</span><span>-</span><span>api</span><span>-</span><span>key</span><span>&gt;</span></span>
```

-   Dependency-check: Invokes the Dependency-Check tool
-   \--scan .: Specifies the current directory (".") to be scanned for dependencies
-   \--format HTML: Defines the output format as HTML for a user-friendly report
-   \--project "dependency-check-demo": Sets the project name for reference in the report
-   \--out .: Instructs Dependency-Check to place the generated report in the current directory

### Viewing the OWASP Dependency-Check Report

After running the command, you should find a new file named dependency-check-report.html in your project directory. Open this file in a web browser to view the detailed report on identified vulnerabilities within your project's dependencies. In our case, the current version of **Newtonsoft.Json** does not have any reported vulnerabilities, so it shows as empty.

![](https://lh7-us.googleusercontent.com/aLBvSii2urIo7O_YfenX7jkY3KCccTaQKulsjSYY9bLA8JZzd6knoPhXAHc4dtSPAP_IA9gLT7Meq7z3UZep2ucpyhE2UiP-WGFcD-uc3K6jcZazPGXsgQccdJ4VOd23AtOtD1mEhJJv_tU0ipzZAsU)

## Best Practices for Using OWASP Dependency-Check

OWASP Dependency-Check is a valuable tool for identifying vulnerabilities within your project's external dependencies. Here are some best practices to maximize its effectiveness:

1.  **Create a Baseline Inventory:** The OWASP Dependency-Check is more effective if you have an internal inventory against which to compare. Start by creating a baseline inventory of all the open-source components and third-party libraries used in your software projects. This includes libraries directly used in your codebase and transitive dependencies. A baseline inventory helps prioritize which dependencies require the most attention and track how dependencies evolve.
2.  **Integrate Dependency-Check in CI/CDProcess:** Integrate dependency scanning into your CI/CD pipeline to automate scans on every code commit or scheduled build. Tools like Jenkins, GitLab CI/CD, and Azure DevOps often have plugins or scripts that facilitate this integration. Additionally, you can schedule regular manual scans of your project to proactively identify and address potential security risks.
3.  **Stay Up-to-Date:** Ensure you're using the latest version of Dependency-Check to benefit from the most recent vulnerability data.
4.  **Consider False Positives and Negatives:** While Dependency-Check strives for accuracy, there's a possibility of false positives (identifying a non-existent vulnerability) or false negatives (missing a real vulnerability). A manual review of flagged dependencies is always necessary.
5.  **Review Scan Results:** After each build, review the OWASP Dependency-Check report to identify and prioritize vulnerabilities for remediation. Integrate the results with your issue-tracking system to facilitate vulnerability management and tracking.
6.  **Consult Additional Security Measures:** Dependency-Check is a valuable tool, but it's not a silver bullet. Use [secure coding guidelines](https://blog.codacy.com/secure-coding-standards-in-agile-development) and other [application security testing](https://blog.codacy.com/application-security-testing-ast) methods to improve your overall application's security posture.

## Extend Your Dependency-Checks to Full Security Scans

OWASP Dependency-Check is a valuable tool for identifying third-party vulnerabilities in project dependencies — but it's not enough. Complementing Dependency-Check with other automated security testing tools can be highly beneficial in improving the quality of your [application security](https://blog.codacy.com/what-is-appsec) program.

Codacy Security goes beyond [Dependency-Check](https://blog.codacy.com/insecure-dependencies-detection) by providing an array of advanced features, including:

-   **Static Analysis:** Detect vulnerabilities and potential security issues in your codebase.
-   **Third-Party License Compliance Checks:** Ensure compliance with all licensing requirements for your open-source projects.
-   **Auditable Mitigation Workflows:** Document and track the entire process of addressing vulnerabilities.
-   **Real-Time Security Best Practice Recommendations:** Offer developers actionable insights and recommendations to enhance your code's overall security.

Get started for [free today](https://www.codacy.com/signup-codacy).







The Open Web Application Security Project ([OWASP](https://owasp.org/)), is an online community that produces free, publicly-available articles, methodologies, documentation, tools, and technologies in the field of [web application security](https://www.mend.io/blog/web-application-security/).

Open source components have become an integral part of software development. According to [Mend’s Risk Report](https://www.mend.io/risk-report/), 96.8% of developers rely on open source components. The increasingly widespread use of open source components requires that developers take a more proactive approach to open source security management. They need to make sure throughout the development process that the software products that they are creating and maintaining don’t contain vulnerable components.

In hopes of making working with open source components more secure, the good folks at OWASP have released a [free utility](https://owasp.org/www-project-dependency-check/) created for developers, that identifies project [dependencies](https://www.mend.io/blog/dependency-management-a-guide-and-3-tips-to-keep-you-sane/) and checks if they contain any known, publicly disclosed, open source vulnerabilities.

We’ve taken a look at the OWASP Dependency-Check’s functionality, along with its features and integrations, and I’m here to share what we found.

## Programming Languages and Integrations

The OWASP Dependency-Check currently supports five different programming languages. Java and .NET are fully supported and additional experimental support is provided for Ruby, Node.js, and Python.

The widespread adoption of open source requires developers concerned with the security of their software projects to integrate open source management tools into the [Software Development Lifecycle (SDLC)](https://www.mend.io/blog/sdlc-software-development-life-cycle/). Dependency-Check enables developers to stay on top of their open source components early in the development process with support for command-line integration. This allows seamless integration with other tools, build systems and APIs, helping developers to detect security vulnerabilities as early on in the CI/CD process as possible, without interfering with development time.

The OWASP’s tool also supports the Jenkins plugin, and can fail the build process, allowing you to make sure only approved code with no open source vulnerabilities is deployed to production.

## Vulnerability Database and Updates

IBM XForce report, the average time between a vulnerability being reported and then exploited shrank from 45 days to 15 days between 2006 and 2015. Another study from Tenable calculated the vulnerability exploit window to be 5.5 days. In these situations, the ability to properly detect vulnerabilities and fix them swiftly becomes even more important.

The OWASP Dependency-Check currently only covers vulnerabilities taken from the [NVD](https://nvd.nist.gov/). While this is a well-respected and popular vulnerabilities database, the process of assessing and verifying vulnerabilities does take some time. This means a vulnerability could be in the wild for a while before being added to the NVD.

While vulnerabilities in commercial software are routinely reported to the NVD, vulnerability research and disclosure in the open source community are less centralized processes. This means that some disclosed open source vulnerabilities might be found in public bug trackers, security advisories or forums, rather than the NVD.

Another factor is that as is typical for free tools, the vulnerability database for the OWASP Dependency-Check is stored locally. This requires users make sure that they update the local database frequently. This differs from databases that are stored in the cloud, which can be updated automatically. This means, again, that there’s a risk of missing vulnerabilities when the local machine is not up to date.

## Vulnerability Scanning

Scanning is the process of running the tool on the user’s code, to identify any vulnerable open source component. This is usually done by conducting a comparison between the user’s code and known open source vulnerabilities in the vulnerabilities database.  

The OWASP Dependency-Check uses a variety of analyzers to build a list of [Common Platform Enumeration (CPE)](https://en.wikipedia.org/wiki/Common_Platform_Enumeration) entries. CPE is a structured naming scheme, which includes a method for checking names against a system. The analyzer checks a combination of groupId, artifactId, and version (sometimes referred to as GAV) in the [Maven Project](https://www.mend.io/free-developer-tools/blog/maven-update-dependencies-automatically/) Object Model file (POM.XML file). This might lead to components being identified incorrectly and result in a high rate of false positives and false negatives, as opposed to calculating the [SHA-1](https://en.wikipedia.org/wiki/SHA-1) of the file, which is a lot more accurate because each component gets a unique identifier.

Reporting is extremely important when dealing with [vulnerability management](https://www.mend.io/blog/vulnerability-management/), since it provides all security and development teams with actionable insights, as well as giving stakeholders the metrics that they need. The OWASP Dependency-Check can support these needs and can generate reports and exports in a variety of formats: XML, CSV, JSON, and HTML.  

## OWASP Dependency-Check: Pros & Cons

 ![OWASP Dependency Check Pros and Cons](https://www.mend.io/wp-content/media/2021/04/aHViPTcyNTE0JmNtZD1pdGVtZWRpdG9yaW1hZ2UmZmlsZW5hbWU9aXRlbWVkaXRvcmltYWdlXzViZmZjNzMwODEzYzguanBnJnZlcnNpb249MDAwMCZzaWc9MWExYTIzYTEwZjY5NzlmOTk1ZTdlNzk5ODQ5M2VhMzQ.jpeg)

Developers are extremely concerned about open source security vulnerabilities, and OWASP’s dependency-check goes a long way in providing them with an easy-to-use tool for scanning their code.

Clearly, the fact that it’s free for developers is a great advantage. Developers don’t need to wait for their managers to approve and purchase OWASP’s free tool. There is no need for a long POC process as they can simply download it for themselves.

Which brings us to another advantage. The OWASP Dependency-Check is lightweight and very easy to download, install, and run. Users don’t need to spend time wading through a lengthy deployment process, working out all the kinks that might come up when adopting a new development tool. Installing and using the Dependency-Check is an effortless process, as long as users remember to update their local copy often.

The variety of reporting and export options are also a great advantage for users that want to keep a close eye on open source vulnerabilities security alerts and stay on top of them. The ability to easily export reports also enables teams to [collect metrics](https://www.mend.io/blog/vulnerability-management-metrics/) and get an overview of their open source vulnerability management capabilities over time.

The OWASP Dependency-Check provides development teams with a strong tool to start their journey towards managing their [open source security](https://www.mend.io/open-source-security/). However, like most free tools, it doesn’t provide all of the capabilities that a [Software Composition Analysis tool](https://www.mend.io/blog/software-composition-analysis/) can provide.

Dependency-Check doesn’t provide users the option of creating automatic rules or workflows to remediate vulnerabilities. So, once users get their report listing all of the open source security vulnerabilities in their code, it’s up to them to determine how to address them and schedule the vulnerability remediation or patching tasks into their already-tight schedule.

While developers can use the OWASP Dependency-Check for a report in a number of formats, the content of the report isn’t very modular. Users can’t create special reports to produce high-level data like analyzing the number of alerts over time, or according to other specific parameters. Any higher level type of analysis would need to be collected manually from the reports and then analyzed by the team.

Another aspect of reporting that is missing from the Dependency-Check is dashboards. Users don’t have a resource in the tool where they can go and see an overview of the state of their open source security status, vulnerability alerts, timeframes, etc. This is data that they will have to collect and organize manually.

The short answer to this question is yes. The OWASP Dependency-Check is great as a free tool for developers, providing them with some of the initial data that they need for open source vulnerability management.

That said, the tool’s scanning capabilities, the fact that it’s stored locally, and the number of false positives that its scans produce make it difficult to use for organizations that require a comprehensive open source security management solution.

Like all free tools, the OWASP Dependency-Check has its advantages and limitations. While we recommend that developers who are not using any technology to secure their open source usage to download it and try it for themselves, organizations seeking more automated controls and features that suit their specific needs may decide to look elsewhere for a solution.





### **What is OWASP?**

The Open Web Application Security Project (OWASP) is an online nonprofit making organization made up of volunteers from all over the world who seek to help security experts to protect their web applications from cyber-attacks. Founded in 2001, OWASP produces freely available tools, documentations, methodologies, articles, and technologies.

Today, OWASP has over 32,000 volunteers who are actively involved in security incidences assessments and research. One of their core principles that has enabled their popularity is that all of their materials are openly available for anyone to use to secure their web application.

This article shows how to use the OWASP Dependency Checker to scan for vulnerabilities in a Node-JS Code.

### **OWASP Dependency Checker**

OWASP Dependency Checker is an open source Software Composition Analysis (SCA) tool that identifies project dependencies on pen source code and checks for known vulnerabilities associated with that code. This tool is part of the solution to one of OWASP Top 10 of 2017 titled “A9 – Using Components with Known Vulnerabilities”.

Today the OWASP Dependency Checker offers full support for .NET and .java programming languages, experimental support for Node.JS, Python and Ruby, and limited support for C and C++ programming languages.

### **Importance of Code Security**

It is estimated that eighty percent of the code in today’s applications come from open source libraries and frameworks. What is more, approximately 27% of libraries available in the internet have well known and publicly disclosed vulnerabilities.

What is worrying, most organizations and individual developers continue to use the libraries in their code without addressing the vulnerabilities. Using a vulnerable library can allow malicious actors to access confidential data, perform transactions, and in some cases gain full control of an application.

As such, developers must be careful with the libraries they use in their code.

Check out this how-t0-video to watch the tutorial and follow along with the steps below.

### **Installing via CLI Version**

For this guide, I will be installing OWASP Dependency Checker in Ubuntu and using it to scan a Node.JS project. Before proceeding, ensure that you have java installed and running. You can confirm if your java installation is working using JAVA –HELP command

With that, here is the step by step process of installing OWASP Dependency Check:  

1.  Download the OWASP Dependency Check from the OWASP Website. Since we intend to deploy this SCA tool on a command line, we will download the CLI Version. This archive contains the files for Linux terminal and a windows command line.

![](https://static.wixstatic.com/media/37f7aa_17b26b0beb5f4233a080c7d5201474db~mv2.jpg/v1/fill/w_802,h_610,al_c,q_85,enc_auto/37f7aa_17b26b0beb5f4233a080c7d5201474db~mv2.jpg)

2\. The next step is to extract the files using the unzip command since we are on a Linux based terminal.

![](https://static.wixstatic.com/media/37f7aa_eb10f3f71d7e4a89a52dc174b98ca90b~mv2.jpg/v1/fill/w_883,h_280,al_c,lg_1,q_80,enc_auto/37f7aa_eb10f3f71d7e4a89a52dc174b98ca90b~mv2.jpg)

3\. The bin folder contains the dependency-check.sh for the Linux terminal and dependency-check.bat for Windows command prompt.

![](https://static.wixstatic.com/media/37f7aa_5158d5691e214c78aeb18e563f536b4a~mv2.jpg/v1/fill/w_925,h_233,al_c,lg_1,q_80,enc_auto/37f7aa_5158d5691e214c78aeb18e563f536b4a~mv2.jpg)

4\. You can test if you installation is working correctly by running the **./**dependency-check.sh command. It should give a list of arguments that can be used with the tool.  

![](https://static.wixstatic.com/media/37f7aa_946fa9f24434481e94adb6887a8065f7~mv2.jpg/v1/fill/w_925,h_243,al_c,lg_1,q_80,enc_auto/37f7aa_946fa9f24434481e94adb6887a8065f7~mv2.jpg)

**Installing via Brew**

If you are using a mac, you can install OWASP Dependency-Check by using the brew command.

```
<span><span>brew update &amp;&amp; brew install dependency-check
</span></span>
```

**Scanning Node JS Code** Before I proceed to scan the code, here are three basic arguments used with the OWASP Dependency-Check.

1\. --project <name> - Allows you to name the project you are scanning

2\. --scan <path> – This indicates the file or the folder that is to be scanned

3\. --out <path> – This is the path where the dependency checker will save the results

To scan some source code, run the dependency-check supplying it the project name, the files to scan and the path to the output location as shown;

For this project scan, we are using sample NodeJS code from the [HacWare Cybersecurity API](https://www.hacware.com/dev). This project is simple and only depends on the "axios" package. This is the output from the scan.

![](https://static.wixstatic.com/media/37f7aa_edf4919d487845f18c4e8672a6cc0565~mv2.png/v1/fill/w_925,h_643,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_edf4919d487845f18c4e8672a6cc0565~mv2.png)

![](https://static.wixstatic.com/media/37f7aa_cff12a9d4a5a4d48b9673552c99e900e~mv2.png/v1/fill/w_925,h_581,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_cff12a9d4a5a4d48b9673552c99e900e~mv2.png)

If you are running the scan for the first time, it will take some time to download and configure CVE details / Subsequent scans will be faster and can be performed without an active internet connection.

**Interpreting the Output**

Once the scan is complete you can review the output that is printed out on the command line and the analyzer provides a report titled "dependency-check-report.html". In the dependency-check-report, it provides a summary of its findings and a detailed report of each vulnerability and the severity rating.

![](https://static.wixstatic.com/media/37f7aa_6713e13a802848a3a098277a620bb6c0~mv2.png/v1/fill/w_925,h_536,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_6713e13a802848a3a098277a620bb6c0~mv2.png)

Each vulnerability explains where the file is found in your project, and the publicly disclosed problem with the vulnerability. The report shows the Common Vulnerability Scoring (CVS) Score to help developers prioritize responses and resources to threats.

![](https://static.wixstatic.com/media/37f7aa_130d0b10353c41b296fadf3a1e2936c2~mv2.png/v1/fill/w_925,h_504,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/37f7aa_130d0b10353c41b296fadf3a1e2936c2~mv2.png)

### **Automating Dependency Check Scans**

This tool can be run through command line interface as an Ant task or through plugins with Gradle, Maven or Jenkins. It does not offer any inbuilt automation but if required, an automation add on may be added.

### **Final Thoughts**

It is important for software developers to find efficient ways implement secure coding practices to build resilient applications and tools. The OWASP Dependency-Check tool shows you if to spot vulnerabilities in your dependent code and make decisions at development time to upgrade to a patched version or find alternative route to meeting the project requirements.

### **Want to Learn More about our Cybersecurity API?**

HacWare makes it stupid easy for software developers to launch next generation cybersecurity education programs to combat phishing attacks with a few lines of code. To learn more about our powerful security awareness API and developer program, [click here to apply](https://www.hacware.com/dev) .

### **References**

