---
title: "HOWTO: Create a Private Repository for Maven Packages"
order: 1
---

By using ProGet, teams can take Maven Artifacts from [Maven Central](https://central.sonatype.com/), the primary repository for Java and other JVM platform libraries. They can then upload them to ProGet as packages, creating a feed that acts as a private extension gallery of curated packages that your team can access. 

If the Maven feed is hosted internally, this will also allow your team to access the packages in an offline environment. 

This article will run through a standard example scenario of a company, Kramerica, configuring ProGet to create a feed that acts as a private Maven repository for a team to access and consume a curated selection of Maven packages. This configuration would also allow the feed to be used offline.

## Step 1: Creating and Naming a New Feed

The first thing we need to do is create a Maven feed. We start by selecting "Feeds" and "Create New Feed".

![Create New Feed](/resources/docs/proget-feeds-createnewfeed.png){height="" width="50%"}

Next, we need to select "Maven" as we will be using Maven packages.

![Create Maven Feed](/resources/docs/proget-newfeed-vsix.png){height="" width="50%"}

From here, we name our feed, which we will call `private-maven` in this example.

![Name Feed](/resources/docs/proget-vsix-naming.png){height="" width="50%"}

Finally, we select "Create Feed", which will create the feed, and redirect us to our `private-maven` feed, which is currently empty and will be populated with packages later.

![Feed Detail](/resources/docs/proget-vsix-empty.png){height="" width="50%"}

## Step 2: Set Permissions

There are many ways to [configure security access controls for uses and groups](/docs/proget/administration-security) in ProGet. In this example, we want to permit only administrators and network engineers to upload packages to the `private-maven` feed, since they're trained to check the quality, licenses, and vulnerabilities of open-source packages. To ensure this rule, we'll set up a new permission. By default, only administrators have assigned permissions.

To start, we navigate to "Settings"> "Manage Security".

![Manage Security](/resources/docs/proget-settings-managesecurity.png){height="" width="50%"}

We then navigate to the "Tasks / Permissions" tab, listing the currently configured permissions, and select "add permission".

![Tasks/Permissions](/resources/docs/proget-taskspermissions-add.png){height="" width="50%"}

Next, we will fill out the following dialog to give the "Network Engineers" user group permission to "Manage Feed" for the `private-maven` feed.

![Permit Manage Feed](/resources/docs/proget-vsix-permissions-managefeed.png){height="" width="50%"}

After saving these privileges, the task overview page looks like this:

![Overview](/resources/docs/proget-vsix-permissions-overview.png){height="" width="50%"}

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

ProGet allows you to upload packages from a local source through various means. This guide will offer three options; using [pgutil](/docs/proget/reference-api/proget-pgutil), through the UI, or bulk uploading.

### Option 1: Using Pgutil
You can use Inedo's [pgutil](/docs/proget/reference-api/proget-pgutil) tool to upload packages by running this command:

```bash
pgutil packages upload --feed=«vsix-feed-name» --input-file=«path-to-extension»
```

For example, to upload the package `myExtension.vsix` located in `C:\visualstudio\extensions` to the feed `private-vsix` you would enter:

```plaintext
pgutil packages upload --feed=private-vsix --input-file=C:\visualstudio\extensions\myExtension.vsix
```

pgutil will require some [minor configuration](/docs/proget/reference-api/proget-pgutil#sources) before use.

### Option 2: Through the UI
You can use the ProGet UI to upload packages. Navigate to "Feeds" > the `private-vsix` feed and select "Add Package" from the drop-down menu.

![Add Package](/resources/docs/proget-vsix-addpackage.png){height="" width="50%"}

Then select "Upload Package".

![Upload Package](/resources/docs/proget-uploadpackage.png){height="" width="50%"}

Finally, use the file browser to select the package and click "Upload File".

![Upload File](/resources/docs/proget-vsix-uploadpackage.png){height="" width="50%"}

### Option 3: Bulk Package Upload

ProGet allows you to [bulk upload](/docs/proget/feeds/feed-overview/proget-bulk-import-with-droppath) extensions to your Maven feed. 

## Step 4.1: Adding the Feed to Visual Studio

To add a VSIX feed to Visual Studio, an additional extension gallery must be added. To do this, navigate to "Tools" > "Options" > "Environment" > "Extensions" and click the "Add" button under "Additional Extension Galleries". Fill in the name and set the URL to the API endpoint URL of the VSIX feed.

![options](/resources/docs/visualstudio-options-extensions.png){height="" width="50%"}

This allows you to install the extensions of your VSIX feed in addition to the extensions in the built-in galleries using the "Extension Manager" window, opened by navigating to "Extensions" > "Manage Extensions".

![extensions](/resources/docs/visualstudio-extensions-manager.png){height="" width="50%"}

### Managing Galleries with Registry Settings

To [manage multiple machines or control access to extension galleries in Visual Studio](https://learn.microsoft.com/en-us/visualstudio/extensibility/how-to-manage-a-private-gallery-by-using-registry-settings?view=vs-2022), you can use a `.pkgdef file`. This allows you to add, prioritize, or disable galleries, including your private ProGet feed, by modifying registry settings. You can also disable public galleries or set your private feed to appear first.

## Step 4.2: Adding Packages from a Feed to VS Code

Currently, Visual Studio Code does not support private galleries, despite there being a [request](https://github.com/microsoft/vscode/issues/21839) for it that has been open since 2017. You can still upload extensions to a VSIX feed, but users will need to manually download them and then import them into their Visual Studio Code.

To download, navigate to a version of the package and select "Download Package"

![download](/resources/docs/proget-vsix-downloadpackage.png){height="" width="50%"}

Once the file is downloaded, in VS Code navigate to "Extensions" > "..." > "Install from VSIX" and then using the file browser to locate the `.vsix` VS Code extension locally.

![install](/resources/docs/vscode-installpackage.png){height="" width="50%"}

### Adding Custom Feed Instructions (Optional)

To provide developers using VS Code with the above guidance when using VS Code, you can add [Custom Feed Instructions](/docs/proget/feeds/feed-overview/proget-usage-instructions)

To create custom feed instructions, navigate to your Visual Studio Extensions feed and select "Manage Feed".

![manage feed](/resources/docs/proget-vsix-managefeed.png){height="" width="50%"}

Then select "Create" under "Feed Usage Instructions". 


Give the instructions a title and then enter them using MarkDown syntax. For the instructions given above, you can use this pre-written example:

```markdown
Currently, Visual Studio Code does not support private galleries. To install extensions users will need to manually download them and then import them into their Visual Studio Code by following these steps:

1. Navigate to a version of a package
2. Select "Download Package"
3. In VS Code, navigate to "Extensions" > "..." > "Install from VSIX"
4. Using the file browser, locate the `.vsix` VS Code extension and select it.

```

After entering title and instructions, select "Save". The instructions will then be found by navigating to your Visual Studio Extensions feed, and selecting the title from the drop-down menu.

![instructions](/resources/docs/proget-vsix-instructions.png){height="" width="50%"}