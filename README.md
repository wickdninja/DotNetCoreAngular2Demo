# ASP.NET Core / Angular 2 via CLI Quick Start

## Install Project Dependencies

- [ASP.NET Core](https://www.microsoft.com/net/core)
- [NodeJS](https://nodejs.org)
  - This guide created with v7.4.0
- Yoeman
  - `$ npm i -g yo`
- ASP.NET Core Generator
  - `$ npm i -g generator-aspnet`
- Angular CLI
  - `$ npm i -g angular-cli`


## Generate Project

- `$ yo aspnet`
  - Select Web Api Project
  - Name application `DemoApp`
  - `$ cd DemoApp`

## Server Setup

- Open project.json
  - Exclude `node_modules` from compilation
    - Add the following to `"buildOptions"`
        ```json
        "compile": {
          "exclude": [
            "node_modules"
          ]
        }
        ```
  - Add StaticFiles dependency
    - Add `"Microsoft.AspNetCore.StaticFiles": "1.0.0"` to `dependencies`
- Restore dependencies
  - `$ dotnet restore`
- Configure serve to handle api routes and client side routes
  - Open `Startup.cs` 
    - Locate the `Configure` method add the following to it
      - Configure CORS
        ```C#
        app.UseCors(cors =>
          cors
          .AllowAnyHeader()
          .AllowAnyMethod()
          .AllowAnyOrigin()
        );
        ```
      - Route requests to root for client side spa
        ```C#
        app.Use(async (context, next) =>
        {
            await next();
            if (context.Response.StatusCode == 404)
            {
                Console.WriteLine("passing to client");
                context.Request.Path = "/";
                await next();
            }
        });
        ```
      - Enable static files (without allow directory browsing)
        ```C#
        app.UseFileServer(enableDirectoryBrowsing: false);
        ```
      - Setup routes for API
        ```C#
        app.UseMvc(routes =>
        {
            routes.MapRoute(
                  name: "api",
                  template: "api/{controller=Version}/{action=Index}/{id?}");
        });
        ```
- Verify changes by building server
  - `$ dotnet build`

## Client Setup

- Create a new angular 2 app using the angular CLI
  - `$ ng new DemoApp`
- Move angular app to project's root directory
  - `$ mv DemoApp/* .`
    - This command moves everything except "dot files"
  - Move `.editorconfig`
    - `$ mv DemoApp/.editorconfig .`
  - Move git repo
    - `$ mv DemoApp/.git .`
  - Copy contents of `DemoApp/.gitignore` into `./gitignore`
    - Find and uncomment `#wwwroot`
      - Remove the `#` character
  - Remove angular generated app root
    - `$ rm -rf DemoApp`
- Configure Angular CLI build output
  - Open `angular-cli.json`
    - Update `outDir`
      - `"outDir": "wwwroot"`
- Test client app build
  - `$ ng build`

## Build Scripts

- Open `project.json`
  - Paste the following into `scripts`
  ```json
  "scripts": {
    "ng": "ng",
    "start": "dotnet run",
    "prestart": "npm run build",
    "client": "ng serve",
    "lint": "tslint \"src/**/*.ts\"",
    "test": "ng test",
    "pree2e": "webdriver-manager update --standalone false --gecko false",
    "e2e": "protractor",
    "clean": "rimraf -- wwwroot",
    "postclean": "ng build",
    "prebuild": "npm run clean",
    "build": "dotnet build",
    "clean:prod": "rimraf -- wwwroot",
    "postclean:prod": "ng build --prod",
    "prebuild:prod": "npm run clean:prod",
    "build:prod": "dotnet publish -c release"
  }
  ```

## Run App

- `$ npm start`
  - Wait for server to start
    - `Now listening on: http://localhost:5000`
  - Verify client application works
    - Open browser and navigate to `http://localhost:5000`
      - Notice `app works!` text from our AppComponent is displayed on the screen
  - Verify API is working
    - Navigate to `http://localhost://5000/api/values`

