---
title: "HOWTO: Create and Publish Maven Packages to a Private Feed"
order: 2
---

ProGet lets you to connect to [Maven Central](https://central.sonatype.com/) to proxy open source Maven packages. However, it also lets you host and manage your own private Maven packages internally. This article will guide you through the steps to create and publish Maven packages to a ProGet Maven Feed.

If the Maven feed is hosted internally, this will also allow your team to access the packages in an offline environment. 

This article will run through a standard example scenario of a company, Kramerica, configuring ProGet to create a private Maven feed that your team can use to access and consume your private Maven packages. This configuration would also allow the feed to be used offline.

## Step 1: Creating and Naming a New Maven Feed

The first thing we need to do is create a "Maven" feed. We start by selecting "Feeds" and "Create New Feed".

![Create New Feed](/resources/docs/proget-feeds-createnewfeed.png){height="" width="50%"}

Next, we need to select "Maven" as we will be using Maven packages.

![Create Maven Feed](/resources/docs/proget-maven-createfeed.png){height="" width="50%"}

We will then select "No Connectors (Private Packages only)" as this feed is intended for our own private packages.

![No Connectors](/resources/docs/proget-maven-noconnectors.png){height="" width="50%"}

From here, we name our feed as specified below, and then select "Create Feed". 

![Name Feed](/resources/docs/proget-maven-namefeed.png){height="" width="50%"}

Finally, we will keep the "Track Package Usage" box checked, and then select "Set Feed Features", which will create the feed, 

![Feed Features](/resources/docs/proget-maven-feedfeatures.png){height="" width="50%"}

We will now be redirected to our `internal-maven` feed, which is currently empty and will be populated with packages later.

![Feed Detail](/resources/docs/proget-maven-emptyfeed.png){height="" width="50%"}


## Step 2: Create Your Package

Maven inherently includes a lifecycle for creating a package. You can create a package by running the following commands:

```bash
mvn install
mvn compile
mvn package
```

## Step 3: Publish Your Package to ProGet

Publishing Maven packages requires a distribution repository to be added to your `pom.xml`  as follows:

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