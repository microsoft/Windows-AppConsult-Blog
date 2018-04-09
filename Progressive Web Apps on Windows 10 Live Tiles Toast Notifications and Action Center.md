<img src='https://msdnshared.blob.core.windows.net/media/2018/04/alpinecat_1920x1200.png' alt="Ninja Cat Loves PWAs" style="width: 350px;" align="left" hspace="10" /> Progressive Web Apps are gaining in popularity with web developers wanting to improve user engagement (via a Start menu presence, push notifications, live tile updates, etc.) and give their users an app like experience with support for offline and background sync. The 'Progressive' in Progressive Web Apps refers to the fact that the app adapts and takes advantage of the **features of the environment it is running on**. The great strength of PWAs on Windows 10, is that they can call Windows UWP APIs directly through JavaScript (or TypeScript). In this blog post, I will demo how you can add the necessary TypeScript to your existing web app, run it as a PWA, and call Windows APIs to update the app's tile, pop up a native toast and show the notification in Action Center.

## Prerequisites

* Visual Studio Community Edition ([http://visualstudio.com](http://visualstudio.com)) at least version 15.6.2.
* Install Node.js and NPM ([https://nodejs.org/](https://nodejs.org/ "https://nodejs.org/"))
* Install Angular CLI:

        npm install -g @angular/cli

* Install .NET Core templates for Angular, React and React with Redux:

        dotnet new --install Microsoft.DotNet.Web.Spa.ProjectTemplates::2.0.0

You can find the source code for this post here: [https://github.com/Microsoft/Windows-AppConsult-samples-PWA](https://github.com/Microsoft/Windows-AppConsult-samples-PWA)

## Create a simple .NET Core Core Angular + MVC web app

The name of the app will be MyNgApp

Open an admin command prompt and type:

```c#
md \MyPWA

cd \MyPWA

dotnet new angular -o MyNgApp
```

You now have an Angular + MVC web app called MyNgApp

## Add Windows Runtime UWP Type declarations to the project

Since we will be calling Windows Runtime APIs we need to add the type definitions to the project to get IntelliSense and avoid errors and warnings in the editor. The type definitions are maintained here: [https://www.npmjs.com/package/@types/winrt-uwp](https://www.npmjs.com/package/@types/winrt-uwp).

Open an admin command prompt

```javascript
cd \MyPWA\MyNgApp\ClientApp

npm install –-save @types/winrt-uwp
```

## Add an Angular component to call the Windows 10 APIs

Open an admin command prompt

```javascript
cd \MyPWA\MyNgApp\ClientApp

ng generate component notifications
```

More information: [https://angular.io/tutorial/toh-pt1](https://angular.io/tutorial/toh-pt1)

## Test it

1. Open the Angular ASP.NET MVC project in Visual Studio

```javascript
Cd \MyPWA\MyNgApp

Start Visual Studio

Open MyNgApp.csproj
```

The Solution view should look as follows:

[![MyNgApp Solution](https://msdnshared.blob.core.windows.net/media/2018/04/MyNgAppSol.png "MyNgApp Solution")](https://msdnshared.blob.core.windows.net/media/2018/04/MyNgAppSol.png)

1. F5 (Debug | Any CPU | IIS Express)

2. This may take a few minutes. Eventually, a browser session will start and the app will load. The load is slow on first run as the components are built.
3. Note: Two apps are created and running: the Angular and ASP.NET MVC - note the different ports. To the user this looks like one app.

You should see something like:

[![MyNgApp Running](https://msdnshared.blob.core.windows.net/media/2018/04/MyNgAppRun.png "MyNgApp Running")](https://msdnshared.blob.core.windows.net/media/2018/04/MyNgAppRun.png)

More information here: [https://docs.microsoft.com/en-us/aspnet/core/spa/angular?tabs=visual-studio](https://docs.microsoft.com/en-us/aspnet/core/spa/angular?tabs=visual-studio)

## Integrate the created Angular component 'Notifications' into the app

Update the navigation menu to display the Notifications component and routing path

1. Open **nav-menu.component.html**
2. Add the following as the last row in the nav navbar-nav table:

    ```html
    <li [routerLinkActive]='["link-active"]'>
        <a [routerLink]='["/notifications"]' (click)='collapse()'>
        <span class='glyphicon glyphicon-envelope'></span> Notifications
        </a>
    </li>
    ```
3. Update the RouterModule so that navigation loads the Notifications component

    a. Open **app.module.ts**

    b. Change:

    ```javascript
    RouterModule.forRoot([
     { path: '', component: HomeComponent, pathMatch: 'full' },
     { path: 'counter', component: CounterComponent },
     { path: 'fetch-data', component: FetchDataComponent },
     ])
    ```
    To:

    ```javascript
    RouterModule.forRoot([
    { path: '', component: HomeComponent, pathMatch: 'full' },
    { path: 'counter', component: CounterComponent },
    { path: 'fetch-data', component: FetchDataComponent },
    { path: 'notifications', component: NotificationsComponent },
    ])
    ```

4. Add Windows APIs to create toast and live tile notifications

   a. Open **notifications.component.ts**

   b. Add the following line to the top of the page:

    ```typescript
    ///<reference path='../../../node_modules/@types/winrt-uwp/index.d.ts' />
    ```

   c. Paste the following into the notifications component class:

```typescript
  public showToast(message, iconUrl) {
    if (typeof Windows !== 'undefined' &&
      typeof Windows.UI !== 'undefined' &&
      typeof Windows.UI.Notifications !== 'undefined') {
      var notifications = Windows.UI.Notifications;
      var template = notifications.ToastTemplateType.toastImageAndText01;
      var toastXml = notifications.ToastNotificationManager.getTemplateContent(template);
      var toastTextElements = toastXml.getElementsByTagName("text");
      toastTextElements[0].appendChild(toastXml.createTextNode(message));
      var toastImageElements = toastXml.getElementsByTagName("image");
      var newAttr = toastXml.createAttribute("src");
      newAttr.value = iconUrl;
      var altAttr = toastXml.createAttribute("alt");
      altAttr.value = "toast graphic";
      var attribs = toastImageElements[0].attributes;
      attribs.setNamedItem(newAttr);
      attribs.setNamedItem(altAttr);
      var toast = new notifications.ToastNotification(toastXml);
      var toastNotifier = notifications.ToastNotificationManager.createToastNotifier();
      toastNotifier.show(toast);
    }
  }

public updateTile(message, imgUrl, imgAlt) {
  if (typeof Windows !== 'undefined' &&
    typeof Windows.UI !== 'undefined' &&
    typeof Windows.UI.Notifications !== 'undefined') {
    var date = new Date();
    var notifications = Windows.UI.Notifications,
      tile = notifications.TileTemplateType.tileSquare150x150PeekImageAndText01,
      tileContent = notifications.TileUpdateManager.getTemplateContent(tile),
      tileText = tileContent.getElementsByTagName('text');
    tileText[0].appendChild(tileContent.createTextNode(message || date.toTimeString()));
    var tileImage = tileContent.getElementsByTagName('image');
    var newAttr = tileContent.createAttribute("src");
    newAttr.value = imgUrl || 'https://unsplash.it/150/150/?random';
    var altAttr = tileContent.createAttribute("alt");
    altAttr.value = imgAlt || 'Random demo image';
    var attribs = tileImage[0].attributes;
    attribs.setNamedItem(newAttr);
    attribs.setNamedItem(altAttr);
    var tileNotification = new notifications.TileNotification(tileContent);
    var currentTime = new Date();
    tileNotification.expirationTime = new Date(currentTime.getTime() + 600 * 1000);
    notifications.TileUpdateManager.createTileUpdaterForApplication().update(tileNotification);
  }
}

```

d. Replace the contents of **notifications.component.html** with:

```html
<button (click)="showToast('hello world','http://icons.iconarchive.com/icons/cute-little-factory/breakfast/128/toast-icon.png')">Show Toast</button><br />
<button (click)="updateTile('' ,'https://unsplash.it/150/150/?random','')">Update Tile</button>
```

e. Save all files and F5 to run under the debugger. Watch output window for errors. You should now see the Notifications menu option.

!["MyNgApp running w/ notifications menu"](https://msdnshared.blob.core.windows.net/media/2018/04/MyNGAppWithNotificationMenu.png "MyNgApp running w/ notifications menu")

f. Click on the Notification menu. Press the 'Show Toast' or 'Update Tile' button. Note that it does not work. This is because we are running in a browser context and not an application. The following JavaScript checks to see if we are in the application context. If it succeeds, it will run the WinRT JavaScript:

```javascript
if (typeof Windows !== 'undefined' && // Are we in Windows application context (a PWA or HWA?)
   typeof Windows.UI !== 'undefined' && // Is the Windows.UI namespace available on this device?
   typeof Windows.UI.Notifications !== 'undefined') // Is the Windows.UI.Notifications namespace available on this device?
```

Let's get this running in an application context in the next step.

## Create a Hosted Web App (essentially a Progressive Web App without the web manifest or service workers)

1. Add a Hosted Web App Project to the MyNGApp solution

    a. Right click the **MyNgApp** solution

    b. Add New Project

    c. Select JavaScript | Windows Universal | Hosted Web App (Universal Windows)

    d. Name the project **MyNgHWA**

2. Configure the **package.appxmanifest** to the home page of the web app (**MyNgApp**) and set the security to allow WinRT APIs to be called.

    a. Open **package.appxmanifest** in the **MyNgHWA** project.

    b. In Application | Start page change 'Start page:' to localhost and the port the ASP.NET app is being served from. You can see this in the output when you run the web app. For example: http://localhost:61438/

    Note: This will be updated to the server url where the app is finally deployed.

    c. In Content URIs | URI includes the same url as above. Make sure Rule is set to Include and WinRT Access is set to All.

    Note: In production, the URIs MUST be https.

3. Run the web app in the HWA.

    a. Start the web app without debugging.

        1. With the MyNgApp as the Startup Project, press Ctrl+F5 or Debug | Start Without Debugging

    c. Start the HWA with debugging.

        1. Set MyNgHWA as the Startup Project.

        2. Press F5 to Start Debugging.

After the splash screen is displayed, you will see the web app running inside the Hosted Web App.

### Test the Windows API Calls

With MyNgHWA running, select the Notification menu, click the 'Show Toast' button. Note a toast message is immediately displayed.

Click the Notification Center icon in the bottom right of Windows 10. Note that your toast message is listed in the Action Center list.

Press the Windows key to view your Start menu. Right click the MyNgApp in the app list and select 'Pin to Start'.

Back in the application, click the 'Update Tile' button. Press the Windows key to view your Start menu. After a few seconds, the tile will update with a new image and the time the tile was updated.

For more on Progressive Web Apps see here:

[https://blogs.windows.com/msedgedev/tag/progressive-web-apps](https://blogs.windows.com/msedgedev/tag/progressive-web-apps)

[http://www.pwabuilder.com](http://www.pwabuilder.com)

[https://blogs.windows.com/msedgedev/2017/03/10/progressive-web-apps-education](https://blogs.windows.com/msedgedev/2017/03/10/progressive-web-apps-education)