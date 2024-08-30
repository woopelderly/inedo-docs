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

### ProGet ____ and earlier

To upload .vsix extensions in ProGet _____ and earlier, you can simply pass the extension to the feed API endpoint URL at PUT or POST.

```powershell
# PowerShell example
Invoke-RestMethod https://proget.example.com/vsix/FeedName `
-InFile .\MyExtension.vsix `
-Headers @{"Authorization" = "Basic " + [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("api:xxxxxxxxxxxxxx"))}
```

Note that PowerShell does not support TLS 1.2 (defined by [RFC 5246](https://tools.ietf.org/html/rfc5246), August 2008) by default. If your server does not allow TLS 1.0 connections (required for PCI compliance since June 2018), you will need to enable TLS 1.2 before running the above command.

#### TLS v1.2 Configuration

Inedo software doesn't connect with TLS v1.2 by default as our products are developed on the .NET platform, and the available protocols change depending on the targeted framework version when the application is built, not the actual framework or operating system installed on the server the product is running on. For more detailed information, see: [Transport Layer Security (TLS) best practices with the .NET Framework](https://docs.microsoft.com/en-us/dotnet/framework/network-programming/tls)

To resolve these errors, the following registry key can be added to the Inedo product server in order to force pre-v4.7-targeting .NET framework applications to use the most secure protocol established by the operating system:

```plaintext
Key: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
Value: SchUseStrongCrypto
Data: 1
```

Once this key is added (or its data set to 1), it will require the server to be rebooted. 

### Bulk Importing Using a Drop Path

ProGet includes [Drop Path](/docs/proget/feeds/feed-overview/proget-bulk-import-with-droppath) support to Vsix feeds. 

#### ProGet 2023.20 and Below

ProGet 2023.20 and below do not currently support import drop paths for Maven, but we are considering adding this in a future release. Please consider joining [the existing discussion in our forums](https://forums.inedo.com/topic/3128) if this feature would be of interest to you.

To work around this limitation, follow this approach:

1. Go through all directories and upload all POM files with a path relative to the root directory
2. Go through all directories again and upload all files that do not have POM and a checksum (like .md5)

Errors will occur, especially if you have invalid POM files or your directory structure does not conform to the required MAVEN convention. So check on a case-by-case basis if this matters (a bad artifact from 5 years ago can probably be ignored).

You can use a tool like `curl` or even `maven deploy:deploy` to deploy individual files.

## Installing Extensions
Installing extensions can be done through the Extension Manager, opened by navigating to "Extensions" > "Manage Extensions".

![extensions](/resources/docs/visualstudio-extensions-manager.png){height="" width="50%"}

## Technical Limitations

### Windows Integrated Authentication

Visual Studio requires "Anonymous" access to a ProGet instance. This is not possible if your instance of ProGet has built-in authentication enabled. To work around this problem, you can set up a second site in IIS without Windows Integrated Authentication enabled, pointing to the same path on disk.

