---
title: "HOWTO: Create and Publish Maven Packages"
order: 2
---

When working with Java projects, managing and distributing packages efficiently is crucial for maintaining a streamlined development process. Maven is a popular tool for building and managing Java-based projects, and ProGet offers a powerful platform to host and manage those Maven packages internally. This article will guide you through the steps to create and publish Maven packages using ProGet, ensuring that your development teams can easily access and share your Java components securely and efficiently. Whether you're setting up your first Maven package or looking to optimize your current workflow, this guide will provide practical insights and actionable steps to help you make the most of ProGet's capabilities.

## Step 1: Create a Maven Feed


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