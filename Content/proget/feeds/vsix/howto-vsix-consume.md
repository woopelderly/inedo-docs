---
title: "HOWTO: Create a Private Extension Gallery for Visual Studio and Visual Studio Code"
order: 1
---

By using ProGet, teams can take extension packages from Microsoft’s public Visual Studio Extension repository, the [Visual Studio Marketplace](https://marketplace.visualstudio.com/), and upload them to ProGet as packages, creating a feed that acts as a private extension gallery of curated packages that your team can access. 

If the Visual Studio Extension feed is hosted internally, this will also allow your team to access the packages in an offline environment. 

This article will run through a standard example scenario of a company, Kramerica, configuring ProGet to create a feed that acts as a private Visual Studio Extension gallery for a team using either Visual Studio and Visual Studio Code to access and consume a curated selection of Visual Studio Extension packages. This configuration would also allow the feed to be used offline.

## Step 1: Creating and Naming a New Feeds

The first thing we need to do is create a "Visual Studio Extension" feed. We start by selecting "Feeds" and "Create New Feed".

![Create New Feed](/resources/docs/proget-feeds-createnewfeed.png){height="" width="50%"}

Next, we need to select "Visual Studio Extensions" as we will be using Visual Studio Extension packages.

![Create Vsix Feed](/resources/docs/proget-newfeed-vsix.png){height="" width="50%"}

From here, we name our feed as specified below.

![Name Feed](/resources/docs/proget-vsix-naming.png){height="" width="50%"}

Finally, we select "Create Feed", which will create the feed, and redirect us to our `private-vsix` feed, which is currently empty and will be populated with packages later.

![Feed Detail](/resources/docs/proget-vsix-empty.png){height="" width="50%"}

## Step 2: Set Permissions to Upload and Manage Packages

There are many ways to [configure security access controls for uses and groups](/docs/proget/administration-security) in ProGet. In this example, we want to permit only administrators and network engineers to upload packages to the `private-vsix` feed, since they're trained to check the quality, licenses, and vulnerabilities of open-source packages. To ensure this rule, we'll set up a new permission. By default, only administrators have assigned permissions.

To start, we navigate to "Settings"> "Manage Security".

![Manage Security](/resources/docs/proget-settings-managesecurity.png){height="" width="50%"}

We then navigate to the "Tasks / Permissions" tab, listing the currently configured permissions, and select "add permission".

![Tasks/Permissions](/resources/docs/proget-taskspermissions-add.png){height="" width="50%"}

Next, we will fill out the following dialog to give the "Network Engineers" user group permission to "Manage Feed" for the `private-vsix` feed.

![Permit Manage Feed](/resources/docs/proget-vsix-permissions-managefeed.png){height="" width="50%"}

After saving these privileges, the task overview page looks like this:

![Overview](/resources/docs/proget-vsix-permissions-overview.png){height="" width="50%"}

## Step 3: Set Permissions to View/Download Packages (Optional)

To allow your developers to access your Visual Studio Extension Feed through Visual Studio, it will require "Anonymous" access to a ProGet instance. This is not possible if your instance of ProGet has built-in authentication enabled. To work around this problem, you can set up a second site in [IIS](/docs/installation/installing-on-iis/various-iis-configuring-iis-roles-and-features) without Windows Integrated Authentication enabled, pointing to the same path on disk.

If you want to add permissions this way, first create

## Step 3: Upload Extensions as Packages to ProGet

Now we will populate our Visual Studio Extensions feed `private-vsix` with extensions. These can be downloaded from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/). 

There is no official VSIX-defined API for adding packages. ProGet allows you to upload packages from a local source through various means. This guide will offer four options; using [pgutil](/docs/proget/reference-api/proget-pgutil), through the UI, bulk uploading or by using PowerShell (For ProGet 2023 and below).

### Option 1: Using Pgutil
You can use Inedo's [pgutil](/docs/proget/reference-api/proget-pgutil) tool to upload packages by running this command:

```bash
pgutil packages upload --feed=«vsix-feed-name» --input-file=«path-to-extension»
```

For example, to uploading the package `myExtension.vsix` located in `C:\visualstudio\extensions` to the feed `private-vsix` you would enter:

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

### Option 3: Bulk Uploading

ProGet allows you to bulk upload by configuring a [Drop Path](/docs/proget/feeds/feed-overview/proget-bulk-import-with-droppath) for your Visual Studio Extension feed. 

## Step 4.1: Adding the Feed to Visual Studio

To add a VSIX feed to Visual Studio, an additional extension gallery must be added. To do this, navigate to "Tools" > "Options" > "Environment" > "Extensions" and click the "Add" button under "Additional Extension Galleries". Fill in the name and set the URL to the API endpoint URL of the VSIX feed.

![options](/resources/docs/visualstudio-options-extensions.png){height="" width="50%"}

This allows you to install the extensions of your VSIX feed in addition to the extensions in the built-in galleries using the "Extension Manager" window, opened by navigating to "Extensions" > "Manage Extensions".

![extensions](/resources/docs/visualstudio-extensions-manager.png){height="" width="50%"}

## Step 4.1: Using Packages from a Feed with Visual Studio Code

Currently Visual Studio Code does not support private galleries, despite there being a [request](https://github.com/microsoft/vscode/issues/21839) for it that has been open since 2017. You can still upload extensions to a VSIX feed, but users will need to manually download them and then import them into their Visual Studio Code.

To download, navigate to a version of the package and select "Download Package"

![download](/resources/docs/proget-vsix-downloadpackage.png){height="" width="50%"}

Once the file is downloaded, in VS Code navigate to "Extensions" > "..." > "Install from VSIX" and then using the file browser to locate the `.vix` VS Code extension locally.

![install](/resources/docs/vscode-installpackage.png){height="" width="50%"}

### Adding Custom Feed Instructions (Optional)

To provide developers using VS Code with the above guidance when using VS Code, you can add [Custom Feed Instructions](/docs/proget/feeds/feed-overview/proget-usage-instructions)

To create custom feed instructions, navigate to your Visual Studio Extensions feed and select "Manage Feed".