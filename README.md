# .NET Core on Heroku

## Prerequirements

1. [.NET Core & SDK](https://www.microsoft.com/net/download) version 2.0+
2. Visual Studio Code 
3. [C# Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) for Visual Studio Code

On Terminal, check with `dotnet --version`

## Create repo

1. Create Github repo, select `.gitignore` for `Visual Studio`
2. Clone repo locally
3. Open with **Visual Code**

## Create .NET Core Webapi

Using **Visual Studio Code** Integrated Terminal, run:

1. Create new solution: `dotnet new webapi -n ForgeSample`
2. As a standard, adjust the port number under `Program.cs:Main()`, replace the following line: 

  `CreateWebHostBuilder(args).UseUrls("http://localhost:3000/").Build().Run();` (note how it just adds `UseUrls` 
3. Press <kbd>F5</kbd> (debug), at the top-center, where it prompts for **Select Environment** select **.NET Core**. A `.vscode` folder is created with `launch.json` and `task.json`. Press <kbd>F5</kbd> again, the app should run at `https://localhost:3000/api/values` (**Values** is the default controller on webapi projects template)

## Create a Heroku app

Requires [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli). 

1. Using the Integrated Terminal, run `heroku login`
2. Create app using [.NET Code Buildpack](https://elements.heroku.com/buildpacks/jincod/dotnetcore-buildpack): `heroku create ForgeSampleAppName --buildpack https://github.com/jincod/dotnetcore-buildpack.git`
There are many options of buildpack, this was one choice.
3. Go to [Heroku Dashboard](https://dashboard.heroku.com/apps) and select the newly create app. Under **Deploy** >> **Connect to GitHub**, connect to the github repo and **Enable Automatic Deploys**. 
4. Go to the app **Settings** (on [Heroku Dashboard](https://dashboard.heroku.com/apps)), under **Config Vars** click on **Reveal Config Vars** and add your **FORGE\_CLIENT\_ID**, **FORGE\_CLIENT\_SECRET** and **FORGE_CALLBACK_URL** (if 3-legged).

## Commit & Push

Now all news commits to the selected branch will automatically deploy to the Heroku app.

1. Add `.vscode` folder to `.gitignore`
2. Commit & Push to github

## Add Autodesk.Forge package

Prepare this project to use Forge:

1. On the Integrated Terminal, run `dotnet add ForgeSample package Autodesk.Forge`

   Confirm that under ForgeSample.csproj the newly added line `<PackageReference Include="Autodesk.Forge" Version="1.2.0" />` (the actual varion may be higher)

2. Under **.vscode** folder, open **launch.json**, search for `ASPNETCORE_ENVIRONMENT` env var and add the Forge variables, as shown below:

```xml
"env": {
    "ASPNETCORE_ENVIRONMENT": "Development",
    "FORGE_CLIENT_ID": "your client id here",
    "FORGE_CLIENT_SECRET": "your client secret here"
    "FORGE_CALLBACK_URL": "http://localhost:3000/api/forge/callback/oauth"
}
```

Now the project is ready for Forge! See [Controllers/OAuthController.cs](blob/master/ForgeSample/Controllers/OAuthController.cs) for an example (2-legged token).