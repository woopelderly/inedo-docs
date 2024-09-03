---
title: "Visual Studio Extension (.vsix)"
order: 7
---

A VSIX feed in ProGet stores Visual Studio extensions, which can then be installed directly from Visual Studio.

Visual Studio does not allow the default extension gallery to be disabled or removed, and this gallery uses an internal API that is different from the documented Atom VSIX feed that ProGet provides, so no connector to the Visual Studio Gallery can currently be created in ProGet. However, connectors to other Atom VSIX feeds are supported.

## Prerequisite Configuration

### Adding the Feed to Visual Studio

To add a VSIX feed to Visual Studio, an additional extension gallery must be added. To do this, navigate to "Tools" > "Options" > "Environment" > "Extensions" and click the "Add" button under "Additional Extension Galleries". Fill in the name and set the URL to the API endpoint URL of the VSIX feed.

![options](/resources/docs/visualstudio-options-extensions.png){height="" width="50%"}

This allows you to install the extensions of your VSIX feed in addition to the extensions in the built-in galleries using the "Manage Extensions" window, opened by navigating to "Extensions" > "Manage Extensions".

## Uploading Extensions

There is no official VSIX-defined API for adding packages. If you want to upload a .vsix extension, you can use Inedo's [pgutil](/docs/proget/reference-api/proget-pgutil) and run this command:

```bash
pgutil packages upload --feed=«vsix-feed-name» --input-file=«path-to-extension»
```

For example, to uploading the package `myExtension.vsix` located in `C:\visualstudio\extensions` to the feed `internal-vsix` you would enter:

```plaintext
pgutil packages upload --feed=internal-vsix --input-file=C:\visualstudio\extensions\myExtension.vsix
```

pgutil will require some [minor configuration](/docs/proget/reference-api/proget-pgutil#sources) before use.

### ProGet 2023 and earlier

To upload .vsix extensions in ProGet _____ and earlier, you can simply pass the extension to the feed API endpoint URL at PUT or POST.

```powershell
# PowerShell example
Invoke-RestMethod https://proget.example.com/vsix/FeedName `
-InFile .\MyExtension.vsix `
-Headers @{"Authorization" = "Basic " + [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("api:xxxxxxxxxxxxxx"))}
```

Note that PowerShell does not support TLS 1.2 (defined by [RFC 5246](https://tools.ietf.org/html/rfc5246), August 2008) by default. If your server does not allow TLS 1.0 connections (required for PCI compliance since June 2018), you will need to enable TLS 1.2 before running the above command.

### Bulk Importing Using a Drop Path

ProGet includes [Drop Path](/docs/proget/feeds/feed-overview/proget-bulk-import-with-droppath) support to Vsix feeds. 

## Installing Extensions
Installing extensions can be done through the Extension Manager, opened by navigating to "Extensions" > "Manage Extensions".

![extensions](/resources/docs/visualstudio-extensions-manager.png){height="" width="50%"}