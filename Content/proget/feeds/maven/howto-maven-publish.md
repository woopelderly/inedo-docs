---
title: "HOWTO: Create and Publish Maven Packages to ProGet"
order: 2
---

A Maven feed in ProGet acts as a "maven2-compatible" artifact repository. Clients such as Maven, Gradle, and SBT can publish and consume artifacts in a Maven feed, and you can browse artifacts using the ProGet web application.

This article will run through a standard example scenario of a company, Kramerica, configuring ProGet to create a feed so that a team can create Maven artifacts and then publish them to the feed.

## Prerequisites

You will need to make sure you have the following downloaded:

* [OpenJDK 22](https://jdk.java.net/22/) 
* [Maven](https://maven.apache.org/download.cgi)

We recommend using newer versions of the Maven client; see [Maven Releases History](https://maven.apache.org/docs/history.html) to learn which versions are considered end-of-life.

## Step 1: Creating and Naming a New Feed

The first thing we need to do is create a Maven feed. We start by selecting "Feeds" and "Create New Feed".

![Create New Feed](/resources/docs/proget-feeds-createnewfeed.png){height="" width="50%"}

Next, we need to select "Maven Artifacts" as we will be creating a Maven feed.

![Create Maven Feed](/resources/docs/proget-maven-createfeed-1.png){height="" width="50%"}

Then we will select No Connectors (private artifacts only) as we will be uploading our artifacts directly.

![Create Maven Feed](/resources/docs/proget-maven-createfeed-2.png){height="" width="50%"}

From here, we name our feed, which we will call `private-maven` in this example.

![Name Feed](/resources/docs/proget-maven-createfeed-3.png.png){height="" width="50%"}

We are then presented with several options. More information on these can be found in the [Vulnerability Scanning and Blocking](https://docs.inedo.com/docs/proget/sca/vulnerabilities) documentation.

![Options](/resources/docs/proget-maven-createfeed-4.png.png){height="" width="50%"}

Finally, we select "Create Feed", which will create the feed, and redirect us to our `private-maven` feed, which is currently empty and will be populated with packages later.

![Feed Detail](/resources/docs/proget-maven-feed.png){height="" width="50%"}

## Step 2: Edit your Settings.xml File

To install packages and plugins from ProGet, you need to modify Maven's `settings.xml` file. This is the Maven configuration file, which is described in detail in the [Maven's Settings Reference Documentation](https://maven.apache.org/settings.html). 

The `settings.xml` file may exist in two different places:
- `«maven-install-directory»/conf/settings.xml` - global-level settings for all users on the machine
- `«user-home-directory»/.m2/settings.xml` - user-level settings that can override any global settings

You can find your `«maven-install-directory»` by running `mvn --version`, and the `«user-home-directory»` is `~/` on Linux or `%UserProfile%` on Windows. If both files are available, we generally recommend editing the global settings file,

If you can't find a `settings.xml` file in either location, you can create one using the template below.

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository/>
  <interactiveMode/>
  <offline/>
  <pluginGroups/>
  <servers/>
  <mirrors/>
  <proxies/>
  <profiles/>
  <activeProfiles/>
</settings>
```

## Step 3: Add a ProGet profile

Profiles are used by Maven to store repository settings and we recommend adding an active profile called `ProGet`. To do this, you need to edit the `profiles` and `activeProfiles` nodes as follows:

```xml
<profiles>
    <profile>
        <id>ProGet</id>
        <repositories />
        <pluginRepositories />
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>ProGet</activeProfile>
</activeProfiles>
```

### Step 4: Configure ProGet Feed as Repositories

Maven uses two types of repositories:
* normal repositories, which are used for libraries and other artifacts
* plugin repositories, which are used for the many plugins that Maven requires

When searching for content (i.e. ordinary artifacts or plugins), Maven will look in the `settings.xml` files for a repository named `central`. If this isn't found, the default repository on `maven.org` will be used.

Therefore, we recommend adding a `central` repository to your `settings.xml` by editing the `repositories` and `pluginRepositories` nodes as follows:

```xml
<repositories>
    <repository>
        <id>central</id>
        <url>«maven-feed-api-endpoint»</url>
        <snapshots><enabled>true</enabled></snapshots>
        <releases><enabled>true</enabled></releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <url>«maven-feed-api-endpoint»</url>
        <snapshots><enabled>true</enabled></snapshots>
        <releases><enabled>true</enabled></releases>
    </pluginRepository>
</pluginRepositories>
```

The `«maven-feed-api-endpoint»` can be found on the browse feed page in ProGet.

![Feed URL](/resources/docs/proget-uploadpackage.png){height="" width="50%"}

### Step 5: Add a Repository Mirror

A POM file can reference a Maven repository directly, bypassing your `settings.xml` configuration. To prevent this, we recommend also configuring a mirror to ensure that all artifact requests go to your feed.

```xml
<mirrors>
	<mirror>
	  <id>proget</id>
	  <name>ProGet</name>
	  <url>«maven-feed-api-endpoint»</url>
	  <mirrorOf>*</mirrorOf>
	</mirror>
</mirrors>
```

If you want to mirror a specific repository, you can change the `mirrorOf` to be the ID of a repository.

### Step 6: (Optional) Authenticate to Your Feeds

If you've [configured a restricted feed](/docs/proget/administration-security/proget-howto-configure-permissions-and-restrictions-on-feeds), you must configure Maven to authenticate with ProGet.

To do this, we recommend that you first [create an API key](/docs/proget/reference-api/proget-apikeys) that has at least the `View/Download`, `Add/Repackage` and `Overwrite/Delete` permissions for your Maven feed. Once you have that, you can edit the `servers` node of your `settings.xml` file as follows:

```xml
<servers>
    <server>
        <id>central</id>
        <username>api</username>
        <password>«api-key»</password>
    </server>
</servers>
```

## Step 6: Create Your Maven Package

Maven inherently includes a lifecycle for creating a package. You can create a package by running the following commands:

```bash
mvn install
```
Executes the phases up to install, meaning it compiles the code, runs tests, packages the project, and installs it in the local Maven repository (~/.m2/repository by default). 

```bash
mvn compile
```
Compiles the project's source code but does not create a packaged artifact (such as a JAR or WAR file). This is useful when you want to check if the code compiles successfully before packaging.

```bash
mvn package
```
Packages the compiled code into its distributable format, which could be a JAR, WAR, or other artifact, depending on the project setup.

## Step 7: Publish Your Package

Publishing Maven packages requires a distribution repository to be added to your `pom.xml` as follows:

```xml
<distributionManagement>
    <repository>
        <id>central</id>
        <name>ProGet</name>   
        <url>«maven-feed-api-endpoint»</url>
        <layout>default</layout>
    </repository>
</distributionManagement>
```

This distribution repository will use the authentication credentials specified in your `settings.xml`, under the `servers` node. In the case above, this will use the `central` server. 

To publish a Maven package and deploy it to ProGet, just run the `mvn deploy` operation. 

### Publishing Snapshot Dependencies

[Snapshot versions](https://maven.apache.org/guides/getting-started/index.html#What_is_a_SNAPSHOT_version) allow you to work with unstable (i.e. pre-release) versions of a library. This is common when you're developing an application—and libraries that the application consumes—at the same time.

To publish (deploy) an unstable snapshot version of a library, simply append `-SNAPSHOT` to the version number in your POM file. For example:

```xml
<project>
  ... snip ...
   <groupId>corp.kramerica</groupId>
   <artifactId>common-library</artifactId>
   <version>2.2-SNAPSHOT</version>
  ... snip ...   
</project>
```

After doing this, when you run the `mvn deploy` command, Maven will simply replace the `-SNAPSHOT` with a timestamp of when it was published. For example, if your POM file specified `2.2-SNAPSHOT`, then Maven will deploy something like `2.2-20230308.201920-3` to the repository instead.

:::(Error) (Don't Use `deploy:deploy-file` with `-SNAPSHOT` Artifacts)
Maven has a [deploy:deploy "mojo"](https://maven.apache.org/plugins/maven-deploy-plugin/usage.html) that publishes (deploys) a file directly to a repository. You can specify a version number with this command; if you do, Maven will **not** replace it with a timestamp. This leads to unpredictable behavior in both Maven and ProGet.
:::

:::(Warning) (Use Standardized Versioning with SNAPSHOT Versions)
Maven uses a 5-part scheme for version numbers: major, minor, incremental, build, and qualifier. This is "documented" in the Maven source code, and explained in detail in an [archived Codehaus article](https://web.archive.org/web/20150308195657/http://docs.codehaus.org/display/MAVEN/Versioning).

If you don't use "standard" versions (for example, if you were to use a 6-part version number), then both Maven and ProGet will behave in unpredictable ways.
:::

## Step 8: (Optional) Use Properties for Repository URL

Instead of specifying the repository URL directly, we recommend using a property. This not only allows you to override the setting when you run the `mvn deploy` command, but also allows you to specify the property in the `settings.xml` file instead of in each project.

To do this, add (or edit) the `properties` node under the ProGet profile in your `settings.xml` file:

```xml
<profiles>
    <profile>
        <id>ProGet</id>
        <properties>
            <distribution-repository>«maven-feed-api-endpoint»</distribution-repository>
        </properties>
        ... snip ...
```

You can then use a property in your POM file, like this:

```xml
<distributionManagement>
    <repository>
        <id>central</id>
        <name>ProGet</name>   
        <url>${distribution-repository}</url>
        <layout>default</layout>
    </repository>
</distributionManagement>
```