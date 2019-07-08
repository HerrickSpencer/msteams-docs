---
title: "Quickstart: Create a Personal tab with ASP.NET Core" 
author: laujan 
description: A quickstart guide to creating a custom personal tab with ASP.NET Core
ms.topic: quickstart 
ms.author: laujan 
---
# Quickstart: Create a custom Personal Tab with ASP.NET Core

Custom tabs enable you to embed your hosted web content directly into Microsoft Teams and add Teams-specific functionality via your [Teams App Package](foo.md). See [What are custom tabs in Microsoft Teams?](/msteams-platform/tabs/what-are-custom-tabs.md). There are two types of tabs available in Teams&mdash;channel/group and personal. A channel/group tab delivers content to channels and group chats. Personal tabs, along with direct conversation bots, are part of personal apps and are scoped to a single user. Personal tabs can be pinned to the left navigation bar and promote increased productivity by making your service available directly inside the Teams client. An app can have up to sixteen (16) personal tabs.

In this quickstart we'll walk-through creating a custom personal tab with C# and[ASP.Net Core](AspNetCore.Docs/aspnetcore/index.md). We'll also use [App Studio for Microsoft Teams](/msteams-platform/get-started/get-started-app-studio.md) to test your tab's Teams integration.

## Prerequisites

- To complete this quickstart you will need an Office 365 tenant and a team configured with *Allow uploading custom apps* enabled. To learn more, see [Manage Microsoft Teams settings for your organization](/OfficeDocs-SkypeForBusiness/Teams/enable-features-office-365.md). If you don't currently have an Office 365 account, you can sign up for a free subscription through the Office 365 Developer Program. The subscription will remain active as long as you're using it for ongoing development. See [Welcome to the Office 365 Developer Program](/OfficeDev/office-dev-program-docs/docs/office-365-developer-program.md).

- You'll use App Studio for Microsoft Teams to import your app package to Teams. To install App Studio in Teams select **Apps** ![Store App](/microsoftteams/platform/assets/images/tab-images/storeApp.png) at the bottom-left corner of the Teams app, and search for App Studio. Once you find the tile for App Studio, select it and choose install in the pop-up window dialog box.

- You'll also need a [GitHub](https://github.com) account so that you can get a copy of the source code for for this project.

In addition, this project requires that you have the following installed in your development environment:

- The Visual Studio 2019 IDE with the `.NET CORE cross-platform development` workload installed:

![screenshot: visual studio install options](/microsoftteams/platform/assets/images/tab-images/workloads.png)

If you don't already have Visual Studio, you can download and install the latest [Microsoft Visual Studio Community](https://visualstudio.microsoft.com/downloads) version for free.

- The [ngrok](https://ngrok.com/docs) reverse proxy tool. You'll use ngrok to create a tunnel to your locally running web server's publicly-available HTTPS endpoints. Go to https://ngrok.com/download to get the download for your environment.

## Get the source code for this project

Open a command prompt and create a new directory for your tab project. You can find the GitHub repository for this quickstart at [GitHubRepository foo.md](https:///github.com/MicrosoftDocs). Navigate to your new directory and type the following command to [clone](https://help.github.com/en/articles/cloning-a-repository) the sample repository to your local machine:

```bash
git clone https:///github.com/MicrosoftDocs/foo.md
```

Once the repository is cloned, open the solution file, `foo.md`, in Visual Studio and select `Build Solution` from the `Build` menu. Run the sample by pressing `F5` or choosing `Start Debugging` from the `Debug` menu and navigate to the following URLs to verify that the app URLS are loading:

- `http://localhost:44311`
- `http://localhost:44311/privacy`
- `http://localhost:44311/tou`

## Review the source code files

&#9989; Startup.cs

This project was created from an ASP.NET Core web application empty template. Since Razor pages are a subset of the ASP.NET Core MVC framework, we needed to register the MVC services for the components to the service collection that MVC and, therefore, Razor Pages require. To do so, we added the dependency injection framework to the  `ConfigureServices()` method. Additionally,
the empty template doesn't enable serving static content by default, however, since your project will be serving static files&mdash;HTML, CSS, and images, we added the static files middleware to the `Configure()` method:

```csharp
public void ConfigureServices(IServiceCollection services)
        {
           services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
        }
public void Configure(IApplicationBuilder app)
        {
        app.UseStaticFiles();
        app.UseMvc();
        }
```

&#9989; wwwroot folder

In the ASP.NET Core application, the web root (wwwroot) folder is the default static file location and isn't included with the empty template. We added a new folder to the root of our project, and named it wwwroot. When the newly created folder displayed in Solution Explorer it had the proper appearance and &#127760; icon. With the static files middleware added to the project, ASP.NET Core will look in the wwwroot folder of your project for static files and return them if the filename matches the request. The wwwroot folder contains the CSS styling for your project, `site.css`, an images folder containing several icons, and a library folder containing `bootstrap.css` files.

&#9989; Index.cshtml

ASP.NET Core treats files called `Index` as the default page for the site. When your browser URL points to the root of the site, the Index.cshtml will be displayed. This will be the home page for your tab.

&#9989; Manifest folder

This folder contains the following required app package files:

- A full color icon measuring 192 x 192 pixels.
- A transparent outline icon measuring 32 x 32 pixels.
- A manifest.json file which specifies the attributes of your tab and points to required resources like the channelGroup page.

These files will need to be zipped in an app package for use in uploading your tab to Teams. When a user chooses to add or update your tab, Microsoft Teams will load the configurationUrl, specified in your manifest, load it in an IFrame, and render it in your channel or group chat.

In the Solution Explorer window right-click on the foo.md project and select `Edit Project File`. At the bottom of the file you'll see the code that creates/updates your zip file when the project builds:

```xml
<PropertyGroup>
    <PostBuildEvent>powershell.exe Compress-Archive -Path \"$(ProjectDir)Manifest\*\" -DestinationPath \"$(TargetDir)tab.zip\" -Force</PostBuildEvent>
  </PropertyGroup>

  <ItemGroup>
    <EmbeddedResource Include="Manifest\icon-outline.png">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </EmbeddedResource>
    <EmbeddedResource Include="Manifest\icon-color.png">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </EmbeddedResource>
    <EmbeddedResource Include="Manifest\manifest.json">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </EmbeddedResource>
  </ItemGroup>
```

## Establish a secure tunnel to your tab content

Microsoft Teams is an entirely cloud-based product, and thus requires that your tab content be available from the cloud using HTTPS endpoints. Teams doesn't allow local hosting, therefore, you need to either publish your tab to a public URL, or use a proxy that will expose your local port to an internet-facing URL.

To test your tab you'll use [ngrok](https://ngrok.com/docs). Your server's web endpoints will be available during the current session on your local machine. When the machine is shut down or goes to sleep the service will no longer be available.

- Open a command prompt in the root of your project folder and run the following command:

```bash
ngrok http https://localhost:44311 -host-header="localhost:44311"
```

- Ngrok will listen to requests from the internet and will route them to your project when it is running on port 44311.  It should resemble `https://yo8urGro7upChann3elTa2b.ngrok.io/` where `yo8urGro7upChann3elTa2b` is replaced by the ngrok alpha-numeric HTTPS URL.

- Make note of the HTTPS ngrok URL - you can copy it to `Notepad for Windows`. You'll need the ngrok HTTPS URL to test your tab in Teams.

## Update your personal tab page for Teams

For your personal tab to display within Microsoft Teams, you must include the `Microsoft Teams JavaScript client SDK` and include a call to the Teams SDK&mdash;`microsoftTeams.initialize()`&mdash;within your channel/group page &#60;`script`&#62; tags. This is how your tab and the Teams app communicate.

- The `Pages` folder is where the framework looks for Razor Pages by default. To reference the [Microsoft Teams Library](https://github.com/OfficeDev/microsoft-teams-library-js), select the `Pages` folder, open the  `ChannelGroup.cshtml` file, and add the markup for the latest version of the`jQuery Library` and the `MicrosoftTeams SDK` below the following Razor page shared layout reference:

```html
@{
Layout= #_Layout";
}
```

The markup should resemble the following with the **latest versions** referenced:

```html
`<script src="https://ajax.aspnetcdn.com/ajax/jQuery/jquery-3.4.1.min.js"></script>`
`<script src='https://statics.teams.microsoft.com/sdk/v1.4.3/js/MicrosoftTeams.min.js'></script>`
```

>[!IMPORTANT]
>Don't copy/paste the &#60;script src="..." URLs from this page, they may not represent the latest version. To get the latest version of the SDK markup, always go to:
`foo.md(SDK)` and [jQuery CDN - Latest Stable Versions](https://code.jquery.com) or [Microsoft jQuery Releases on the CDN.](/aspnet/ajax/cdn/overview#jquery-releases-on-the-cdn)

- Within the first set of &#60;`script`&#62; tags, call the `initialize()` method on `microsoftTeams` as follows:

```javascript
    microsoftTeams.initialize();
```
Your tab code is complete. Now you can build your project. But first, *Save all* for good measure.

### Run your project from Visual Studio

- You can run the project by pressing `F5` or choosing `Start Debugging` from the `Debug` menu. Verify that `ngrok` is running and working properly by opening your browser and going to the HTTPS URL supplied by `ngrok` in your command prompt window.

>[!TIP]
>You need to have both your project in Visual Studio and ngrok running to complete this quickstart. If you need to stop running your project in Visual Studio to work on it **keep ngrok running**. Ngrok will continue to listen and will resume routing your project's request when your restarts in Visual Studio. If you have to restart the ngrok service it will return a new URL and you'll have to update every place that uses that URL.

### Upload your tab in Microsoft Teams with App Studio

- Open `Microsoft Teams` using the [web based version](https://teams.microsoft.com) so that you can inspect your front-end code using your browser's developer tools.

- Open App studio and select the `Manifest editor` tab.

- Select the *Import an existing app* tile in the Manifest editor to begin updating the app package for your tab. Recall that the source code comes with its own pre-made manifest and the `.csproj file` contains code to create an app package when the project is built. The name of your app package is *tab.zip*. You can search your local machine's file explorer or switch to Visual Studio `Folder View` to find your zip file's location. It should be found here:

```bash
 /bin/Debug/netcoreapp2.2/tab.zip
```

- Upload `tab.zip` to App Studio.

### Update your app package with Manifest editor

- Once you've uploaded your tab into Teams, you'll need to configure it to show content.

- Select the tile for your newly imported tab in the right panel of the Manifest editor welcome page.

There's a list of steps in the left-hand side of the Manifest editor, and on the right a list of properties that need to be filled in for each of those steps. Much of the information has been provided by your `manifest.json` file but there are a few fields that you'll need to update:

#### Details: App details

- Under *Identification* select ![generate](/microsoftteams/platform/assets/images/tab-images/generate-button.png) to replace the placeholder id with a required Microsoft-generated GUID for your tab.

- In the **Developer information** field update the `Website URL` with your `ngrok` HTTPS URL.

- In the **App URLs** field update the `Privacy statement` and `Terms of use` URLs with your `ngrok` HTTPS URL. Remember to include the */privacy* and */tou* parameters at the end of the URLs.

#### Capabilities

- Select *Tabs*.

- To clear placeholder URLs from the manifest, scroll to the bottom of the Tabs section (*Select a tab from the following list to include any additional domains*) select the (•••) button and choose &#x1F5D1; Delete.

##### Add a personal tab

- Scroll up to *Add a personal tab*, select ![add button](/microsoftteams/platform/assets/images/tab-images/add-button.png).

- Enter a complete the *Name* (The display name of the tab) and *Entity ID* (The ID for the item in the tab, for example, "redIcon123") fields and select ![Save](/microsoftteams/platform/assets/images/tab-images/save-button.png).

#### Finish

##### Domains and permissions

The `Domains from your tabs` should contain only your ngrok URL without the HTTPS prefix&mdash;`y8urGr7pChan3Ta2b.ngrok.io/`. If the *Additional valid domains* field is completed, select the (•••) button and choose 🗑 Delete.

##### Test and distribute

- Select ![install](/microsoftteams/platform/assets/images/tab-images/install-button.png).

>[!IMPORTANT]
>In the `Description` field on the right you'll see the following warning:<br><br>
&#9888; "**The 'validDomains' array cannot contain a tunneling site...**" <br><br>**The warning can be ignored while you're testing your tab.**<br><br>
After your personal tab has been uploaded to Microsoft teams, via ngrok, and successfully saved, you can view its content until your tunnel session ends .<br><br>
**Remember to serve your tab on your hosted website prior to submission to the Teams app store for approval**.

- In the pop-up window select ![install](/microsoftteams/platform/assets/images/tab-images/install-button.png).

- In the final pop-up window select ![open](/microsoftteams/platform/assets/images/tab-images/open-button.png) and your personal tab will be displayed.
 
## View your personal tabs

- In the navbar located at the far-left of the Teams App, select (**&#8943;**) *More added apps*. You'll be presented with a list of personal view apps.

- Select your personal tab from the list to view.

### Nice work! You just extended Microsoft Teams with a custom personal tab.