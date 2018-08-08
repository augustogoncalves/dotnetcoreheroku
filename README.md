# .NET Core on Heroku (auto-deploy from Github)

This is a step-by-step to setup a .NET Core (2.0+) app using Visual Studio Code (non-Windows environment, e.g. MacOS), and how to control code with Github and auto-deploy to Heroku. 

Found anything wrong or that can be improved? Please contribute! 

## Prerequirements

1. [.NET Core & SDK](https://www.microsoft.com/net/download) version 2.0+
2. [Visual Studio Code](https://code.visualstudio.com/)
3. [C# Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) for Visual Studio Code

On Terminal, run `dotnet --version` to check the .NET version. 

## Create repo

1. Create a [new Github repo](https://github.com/augustogoncalves?tab=repositories), add `.gitignore` for `Visual Studio`
2. Clone repo locally
3. Open with **Visual Studio Code**

## Create .NET Core Webapi

1. Using **Visual Studio Code** Integrated Terminal, run `dotnet new webapi -n ForgeSample` to create a new project.
2. As a standard on Forge samples, adjust the port number `3000`. Under `.vscode\launch.json` file, add `ASPNETCORE_URLS`:

   ```json
	"env": {
	    "ASPNETCORE_ENVIRONMENT": "Development",
	    "ASPNETCORE_URLS" : "http://localhost:3000",
	},
	```
  
3. Add support for static files, like `.html`, `.js` and `.css`. Under `Startup.cs:Configure()`, add the following ([see sample](ForgeSample/Startup.cs)):

   ```csharp
   app.UseDefaultFiles();
   app.UseStaticFiles();
   ```
   
3. Press <kbd>F5</kbd> (debug), the application should compile and open the browser. If the debugger is not set, at the top-center, where it prompts for **Select Environment** select **.NET Core**. A `.vscode` folder is created with `launch.json` and `task.json`. Press <kbd>F5</kbd> again, the app should run at `https://localhost:3000/api/values` (**Values** is the default controller on webapi projects template)

## Create a Heroku app

Requires [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli). 

Using the Integrated Terminal, run:

> It may be required to stop debugging to unlock the terminal, press <kbd>Ctrl</kbd> + <kbd>C</kbd>

1. `heroku login`
2. Create app using [.NET Code Buildpack](https://elements.heroku.com/buildpacks/jincod/dotnetcore-buildpack) (there are many options of buildpack, this was one choice): 

   `heroku create ForgeSampleAppName --buildpack https://github.com/jincod/dotnetcore-buildpack.git`
3. Go to [Heroku Dashboard](https://dashboard.heroku.com/apps) and select the newly create app. Under **Deploy** menu, go to **Deployment method** and select **Connect to Github**. Next, go to **Connect to Github** and search the respective repository and click **Connect**. Next, go to **Automatic deploys** and and **Enable Automatic Deploys** for the brach (e.g. `master`). 
4. Go to the app **Settings** (on [Heroku Dashboard](https://dashboard.heroku.com/apps)), under **Config Vars** click on **Reveal Config Vars** and add your **FORGE\_CLIENT\_ID**, **FORGE\_CLIENT\_SECRET** and **FORGE\_CALLBACK\_URL** (if 3-legged).

## Commit & Push to Github

As the Github repo is connected to Heroku, all new commits to the selected branch will automatically deployed to the Heroku app.

1. As `launch.json` contains the ID/Secret, add `.vscode` folder to `.gitignore`
2. Commit & Push to Github

At the Heroku dashboard, for this app, see **Logs**, it should say: _Build started by username_, then _Build succeeded_. Now the app should live, try the same **Values** endpoint.

## Add Autodesk.Forge package

Prepare this project to use Forge:

1. On the Integrated Terminal, run `dotnet add ForgeSample package Autodesk.Forge`

   A popup should appear asking to **Restore**, which downloads the newly added package. Confirm that under ForgeSample.csproj the newly added line `<PackageReference Include="Autodesk.Forge" Version="1.2.0" />` (the actual varion may be higher)

2. Under **.vscode** folder, open **launch.json**, search for `ASPNETCORE_ENVIRONMENT` env var and add the Forge variables, as shown below:

```xml
"env": {
    "ASPNETCORE_ENVIRONMENT": "Development",
    "ASPNETCORE_URLS" : "http://localhost:3000",
    "FORGE_CLIENT_ID": "your client id here",
    "FORGE_CLIENT_SECRET": "your client secret here",
    "FORGE_CALLBACK_URL": "http://localhost:3000/api/forge/callback/oauth"
}
```

Now the project is ready for Forge! See [Controllers/OAuthController.cs](ForgeSample/Controllers/OAuthController.cs) for an example (2-legged token). With this controller, the `/api/forge/oauth/token` should also work (localhost and live on Heroku). Static files should be placed on `wwwroot` folder.

## Tips & Tricks

### Enable "Deploy to Heroku" button

Create a `app.json` and specify the buildpack for .NET Core:

```json
"buildpacks": [
  {
    "url": "https://github.com/jincod/dotnetcore-buildpack.git"
  }
]
```

### Open browser (or new tab) on debug

This is a personal choice, I prefer to NOT open a new browser/tab when start debugging, this can be disabled under `launch.json`, change to `false`. You can start debugging with <kbd>Cmd</kbd> + <kbd>Shift</kbd> + <kbd>F5</kbd>.

```
"launchBrowser": {
    "enabled": false,
    ...
}
```

### Console output

If you want to reduce the output, here are a few tips:

1. To ignore messages when modules are loading, open `launch.json` and add (under `configurations`):

```json
"logging":{
  "moduleLoad": false  
},
```

2. Change the logging level to `error` at the `appsettings.development.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Error",
      "Microsoft": "Error"
    }
  }
}
```

### dotnet process consuming 100% of CPU 

Follow [this workaround](https://github.com/dotnet/roslyn/issues/24137#issuecomment-388494024). In summary, create the file `~/Library/LaunchAgents/UseSharedCompilation.plist` with the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>UseSharedCompilation</string>
  <key>ProgramArguments</key>
  <array>
    <string>sh</string>
    <string>-c</string>
    <string>launchctl setenv UseSharedCompilation false</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

## License

This sample is licensed under the terms of the [MIT License](http://opensource.org/licenses/MIT).
Please see the [LICENSE](LICENSE) file for full details.

## Written by

Augusto Goncalves [@augustomaia](https://twitter.com/augustomaia), [Forge Partner Development](http://forge.autodesk.com)