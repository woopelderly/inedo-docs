---
title: "HOWTO: Create a Private Repository for Maven Packages (Optionally Offline)"
order: 1
---

By using ProGet, teams can take Maven Artifacts from [Maven Central](https://central.sonatype.com/), the primary repository for Java and other JVM platform libraries. They can then upload them to ProGet as packages, creating a feed that acts as a private extension gallery of curated packages that your team can access. 

If the Maven feed is hosted internally, this will also allow your team to access the packages in an offline environment. 

This article will run through a standard example scenario of a company, Kramerica, configuring ProGet to create a feed that acts as a private Maven repository for a team to access and consume a curated selection of Maven packages. This configuration would also allow the feed to be used offline.

## Prerequisites

You will need to make sure you have the following downloaded:

* [OpenJDK 22](https://jdk.java.net/22/) 
* [Maven](https://maven.apache.org/download.cgi)

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

## Step 2: Set Permissions

There are many ways to [configure security access controls for uses and groups](/docs/proget/administration-security) in ProGet. In this example, we want to permit only administrators and network engineers to upload packages to the `private-maven` feed, since they're trained to check the quality, licenses, and vulnerabilities of open-source packages. We also want to permit our developers to view the feed and download packages. To ensure this rule, we'll set up new permissions. By default, only administrators have assigned permissions.

To start, we navigate to "Settings"> "Manage Security".

![Manage Security](/resources/docs/proget-settings-managesecurity.png){height="" width="50%"}

We then navigate to the "Tasks / Permissions" tab, listing the currently configured permissions, and select "add permission".

![Tasks/Permissions](/resources/docs/proget-taskspermissions-add.png){height="" width="50%"}

Next, we will fill out the following dialog to give the "Network Engineers" user group permission to "Manage Feed" for the `private-maven` feed.

![Permit Manage Feed](/resources/docs/proget-maven-permissions-1.png){height="" width="50%"}

Now, following the same steps, we will create permissions for our developers to view and download from the feed.

![Permit View](/resources/docs/proget-maven-permissions-1.png){height="" width="50%"}

After saving these privileges, the task overview page looks like this:

![Overview](/resources/docs/proget-maven-permissions-1.png){height="" width="50%"}

To allow your developers to view and download your `private-maven` fee, you must configure Maven to authenticate with ProGet.

To do this, [create an API key](/docs/buildmaster/reference/api/buildmaster-administration-security-api-keys) that has the` View/Download` permissions for your Maven feed. Once you have that, you can edit the servers node of your settings.xml file as follows:

```xml
<servers>
    <server>
        <id>central</id>
        <username>api</username>
        <password>«api-key»</password>
    </server>
</servers>
```

## Step 3: Upload Artifacts as Packages to ProGet

Now we will populate our Maven feed `private-maven` with artifacts as packages. These can be downloaded from [Maven Central](https://central.sonatype.com/). 

ProGet allows you to upload artifacts from a local source through various means. This guide will offer three options; using [pgutil](/docs/proget/reference-api/proget-pgutil), through the UI, or bulk uploading.

### Option 1: Using Pgutil
You can use Inedo's [pgutil](/docs/proget/reference-api/proget-pgutil) tool to upload packages by running this command:

```bash
pgutil packages upload --feed=«maven-feed-name» --input-file=«path-to-artifact»
```

For example, to upload the file `myArtifact.pom` located in `C:\maven\projects` to the feed `private-maven` you would enter:

```plaintext
pgutil packages upload --feed=private-maven --input-file=C:\maven\projects\myArtifact.pom
```

pgutil will require some [minor configuration](/docs/proget/reference-api/proget-pgutil#sources) before use.

### Option 2: Through the UI
You can use the ProGet UI to upload packages. Navigate to "Feeds" > the `private-maven` feed and select "Add Artifact" from the drop-down menu.

![Add Package](/resources/docs/proget-maven-addartifact.png){height="" width="50%"}

Then select "Upload Maven Artifact".

![Upload Package](/resources/docs/proget-maven-uploadartifact.png){height="" width="50%"}

Finally, use the file browser to select the package and click "Upload".

![Upload File](/resources/docs/proget-vsix-uploadfile.png){height="" width="50%"}

### Option 3: Bulk Package Upload

ProGet 2023.21 adds [Drop Path](/docs/proget/feeds/feed-overview/proget-bulk-import-with-droppath) support to Maven Feeds. Maven packages are made up of a POM file and one or more artifacts, like jar, war, ear, xml, etc...  When adding Maven packages to a drop folder, each package should be its own folder that contains the POM file, and all related artifacts. When Maven packages are imported via a drop path, ProGet will first find all files that end with `.pom` and then will add any artifacts in the same folder as that POM file.

::: (Warning) (SNAPSHOTS are not supported)
Due to how Maven handles SNAPSHOT packages, packages using a SNAPSHOT version cannot be imported using a drop path.
:::

## Step 4: Configure ProGet Feed as Repositories

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

You will find the `«maven-feed-api-endpoint»` on the browse feed page in ProGet.

![Feed URL](/resources/docs/proget-uploadpackage.png){height="" width="50%"}

## Step 5: Add a Repository Mirror

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

#### Example: settings.xml

To put it all together, here is an example `settings.xml` that will push/pull artifacts and plugins from `http://progetsv:8624/maven2/my-maven/` using the API key `apiKey12345`.

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"          
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>central</id>
            <username>api</username>
            <password>apiKey12345</password>
        </server>
    </servers>
    <profiles>
        <profile>
            <id>ProGet</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://progetsv:8624/maven2/my-maven</url>
                    <snapshots><enabled>true</enabled></snapshots>
                    <releases><enabled>true</enabled></releases>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://progetsv:8624/maven2/my-maven</url>
                    <snapshots><enabled>true</enabled></snapshots>
                    <releases><enabled>true</enabled></releases>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>ProGet</activeProfile>
    </activeProfiles>
    <mirrors>
        <mirror>
            <id>proget</id>
            <name>ProGet</name>
            <url>http://progetsv:8624/maven2/my-maven</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>    
</settings>
```