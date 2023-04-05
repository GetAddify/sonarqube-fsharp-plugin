# Instructions to install and run SonarQube with F# security plugin

## Prerequisites

### To compile the project:

* Windows, Linux or Mac
* A recent Java JDK
* Maven
* Visual Studio 2019 or 2022 or Visual Studio Code
* [.NET 6](https://dotnet.microsoft.com/en-us/download/dotnet/6.0) (comes with installing VS)
* For Mac/Linux: [.NET Core 3.1](https://dotnet.microsoft.com/en-us/download/dotnet/3.1) (usually comes with installing VS)

### To run:

* Windows or Linux (Mac is currently not supported)
* [.NET 6](https://dotnet.microsoft.com/en-us/download/dotnet/6.0)
* [.NET Core 3.1](https://dotnet.microsoft.com/en-us/download/dotnet/3.1) (not included by default in prev. download, unless you install Visual Studio)

## Compile sonar-fsharp-security-plugin

Compilation instructions can be found in the [contributing-plugin.md](sonar-fsharp-security-plugin/docs/contributing-plugin.md) 
of the original plugin. Compiling requires a recent Java + SDK and Maven installed.

## How the plugin works

See [how the plugin works](sonar-fsharp-security-plugin/docs/contributing-plugin.md).


## Install SonarQube

Use the docker image named `sonarqube` from https://hub.docker.com/_/sonarqube/.

Or follow the [quickstart instructions here](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/). 

NOTE: I didn't manage to get the zip file install running, but the docker image worked just fine and can be used with plugins.

## Install the plugin

With a running docker image named `sonarqube`, do as follows.

### Uninstall first 

Only needed if you are updating an existing plugin.

* On your SQ webpage at http://localhost:9000 go to Administration > Marketplace
* Select "Installed", this will show the locally installed plugins
* Click Uninstall
* Restart the server, for instance with `docker restart sonarqube`
* Go to http://localhost:9000 and wait for the server to come backup
* Login as administrator to SonarQube
* Go to Administration > Marketplace and verify that the plugin is removed (the list of installed plugins is empty)

### Install the plugin

The JAR file named `sonar-fsharpsecurity-plugin-0.0.0.1` will be in `sonar-fsharpsecurity-plugin\target` after being
compiled and packaged with `mvn clean package` or `mvn package`.

* Rename the file without version, i.e. rename it to `sonar-fsharpsecurity-plugin.jar`
* Copy the file to the download directory of the docker image: 
    ```bash
    docker cp sonar-fsharpsecurity-plugin.jar sonarqube:/opt/sonarqube/extensions/downloads
	```
* To verify that the file is in the download dir just go to the docker console and check, i.e.: 
    ```bash
    # ls extensions/downloads/
    sonar-fsharpsecurity-plugin.jar
    ```
* Docker won't pick up the downloaded plugin automatically until restart the server, for instance with `docker restart sonarqube`
* Go to http://localhost:9000 and wait for the server to come up
* Login as administrator
* It may show a question about whether you trust the new plugin. Click Yes.
* Verify successful installation by going to Administration > Marketplace and select "Installed".

The plugin is now installed and ready for use.

## Running SonarQube with plugin locally

This assumes a docker container named `sonarqube` exists.

### Create a project and token
To run the analysis locally you need to create a token. If you didn't create one the first time you started the SonarQube instance, 
go to Administration > Security (dropdown) > Users and create a token there. Copy this token somewhere as you'll need
it as the `sonar.login` commandline variable.

Optionally, set env var `SONAR_TOKEN` to your user token.

Optionally, create a project in SonarQube. If you don't, a default project will be created with the name you give on the commandline.

### Run the `begin` command
From the sln dir, run the following command. Replace `<ProjectName>` with your project name and `%SONAR_TOKEN%` with your token, or set the 
environment variable.

```bash
dotnet sonarscanner begin /k:"<ProjectName>" /d:sonar.host.url="http://localhost:9000"  /d:sonar.login="%SONAR_TOKEN%"
```

This command will add instrumentation to the output of the build command in the next step.

### Run `dotnet build`
Run the full build, usually: `dotnet build`.

### Run the `end` command

Even though it is called `end`, it actually starts the analysis. This step will download the plugin, which in turn will
extract the contained zip file, which is a .NET executable that will run the analysis.

```bash
dotnet sonarscanner end /d:sonar.login="%SONAR_TOKEN%"
```

If the FsSonarScanner does its job, you will now see lines like the following in the command window:

```text
INFO: 12:34:39 [Information] ["FsSonarRunner"] Starting analysis
INFO: 12:34:39 [Information] ["SonarAnalyzer.FSharp.RuleRunner"] Analyzing 16 rules
INFO: 12:34:39 [Information] ["SonarAnalyzer.FSharp.RuleRunner"] Analyzing 22 files
INFO: 12:34:39 [Information] ["SonarAnalyzer.FSharp.RuleRunner"] Analyzing "D:\\Projects\\Lula\\risk-assessment\\Lula.RiskAssessment\\Score.fs"
INFO: 12:34:44 [Information] ["SonarAnalyzer.FSharp.RuleRunner"] ... 0 issues found
```

And it should end with something like (where `SonarAnalyzerTest` is replaced by your project name):

```text
INFO: ANALYSIS SUCCESSFUL, you can find the results at: http://localhost:9000/dashboard?id=SonarAnalyzerTest
INFO: Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
INFO: More about the report processing at http://localhost:9000/api/ce/task?id=AYB1H2i1HasxVEoR8T6b
INFO: Analysis total time: 2:31.394 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
INFO: Total time: 2:34.314s
INFO: Final Memory: 17M/112M
INFO: ------------------------------------------------------------------------
The SonarScanner CLI has finished
13:40:45.455  Post-processing succeeded.
```





