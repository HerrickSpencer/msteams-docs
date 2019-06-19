---
title: "Quickstart: Create a Group and Channel Tab with the Teams Yeoman Generator"
author: laujan
description: A quickstart guide to creating a group and channel tab with the Teams Yeoman Generator.
ms.topic: quickstart
ms.author: laujan
---
# Quickstart: build a custom group or channel tabs with Node.js and the Microsoft Teams Yeoman Generator

[!INCLUDE [build-custom-tab-node-js-common](~/includes/build-custom-tab-node-js-common.md)]

## Add code to your group/channel tab

Your tab logic is located in the `./src/app/scripts/<yourDefaultTabNameTab>/<yourDefaultTabNameTab>.tsx` TypeScript JSX file . Locate the `render()` method and add the following <div\><&#47;div\> at the top of the `<PanelBody>` container code:

```html
    <PanelBody>
        <div style={styles.section}>
        Hello World! Yo Teams rocks!
        </div>
    </PanelBody>
```

>[!NOTE]
>Open a command prompt in your app's project folder to complete the project's gulp tasks.

## Create a Teams App manifest

Now that your tab code is complete, you can build your project:
>The [Teams Manifest](foo.md) will be part of your app package zip file (along with your two app icons) and will be uploaded into Microsoft Teams. This is achieved through a gulp task that validates the manifest and creates the zip file in the `./package directory`. In the command prompt, type the following:

```bash
gulp manifest
```

## Build your app

>The build command compiles your solution into the `./dist` folder. In the command prompt, type the following:

```bash
gulp build
```

## Run your app in localhost

>To build and start a local web server, in the command prompt, type the following:

```bash
gulp serve
```

>Enter `http://localhost:3007/<yourDefaultAppNameTab>/` in your browser and view your configurable tab's content page:
>>![content page screenshot](/msteams-platform/assets/configTab.PNG)
>To view your personal tab, remain in the current browser and add `static.html` to the app's file path: `http://localhost:3007/<yourDefaultAppNameTab>/static.html` Press Enter.<br>
>![static tab screenshot](/msteams-platform/assets/staticTab.PNG)

>[!NOTE]
>To view your configuration page, add  `config.html` to the file path: `http://localhost:3007/<yourDefaultAppNameTab>/config.html`.

## Package your app for Microsoft Teams

Microsoft Teams is an entirely cloud-based product, and thus requires your app to be available from the cloud using HTTPS endpoints. Microsoft Teams does not allow apps to be hosted on localhost. Therefore, you need to either publish your app to a public URL or use a proxy which will expose your local port to an internet-facing URL.

The [ngrok](https://ngrok.com/docs) tool, which is built into this project, is a reverse proxy software application that will create a tunnel to your locally running web server's publicly-available HTTPS endpoints. Your server's web endpoints will be available during the current session on your local machine. When the machine is shut down or goes to sleep the service will no longer be available.

>In your command prompt, enter the following:

```bash
gulp ngrok-serve
```

## Upload and run your app in Microsoft Teams

Open Microsoft Teams. In the **YourTeams** panel click (**&#8943;**) *More options* next to the team that you are using to test your app's tabs and Select *Manage team*.  In the main panel click on *Apps* from the tab bar and click on *Upload a custom app* located in the lower right-hand corner of the page. Open your project folder, browse to the `./package` folder, select the zip file in the `./package` folder, right-click, and choose open. Your app will upload into Microsoft Teams.

Return to your team's General channel and select ➕ to add your tab from the list of tabs. Follow the directions for adding a tab. Note that there is a custom configuration dialog for your group/channel tab. Select *Save* and your tabs should be loaded in Microsoft Teams.

## Add and view your group/channel tab

1. Choose ➕ *Add a tab*  from the tab bar.
2. Select your tab from the gallery.
3. Accept the consent prompt.
4. Enter a value for the configuration page.
5. *Save*.
6. To view, select your new tab from the tab bar.

### Nice work! You just extended Microsoft Teams with custom tabs.