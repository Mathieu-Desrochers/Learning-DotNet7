Creating a webapi
-----------------
Run the following commands.

    > dotnet new web

Cleanup the launchSettings.json file.

    {
      "profiles": {
        "Default": {
          "commandName": "Project"
        }
      }
    }

Cleanup the appsettings.json file.  
Delete the appsettings.Development.json file.

    {
      "Logging": {
        "LogLevel": {
          "Default": "Information"
        }
      }
    }

Run the following commands.  
By default dotnet binds to port 5000.

    > dotnet run
    > curl http://localhost:5000

Creating a docker image
-----------------------
Create a file .dockerignore with the following content.

    Dockerfile
    bin
    obj

Create a file Dockerfile with the following content.  

    FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
    WORKDIR /src
    COPY . .
    RUN dotnet publish -c release -o /app

    FROM mcr.microsoft.com/dotnet/aspnet:7.0
    WORKDIR /app
    COPY --from=build /app .
    CMD ["dotnet", "Learning-DotNet7.dll"]

Run the following commands.  
The base image forces port 80 in the container.

    > docker image build -t learning-dotnet7 .
    > docker container run -p 5000:80 learning-dotnet7
    > curl http://localhost:5000

Configuration
-------------
Modify the Program.cs file.

    var greeting = app.Configuration.GetValue<string>("Greeting");
    app.MapGet("/", () => greeting);

Configuration from the command line.

    > dotnet run --Greeting "Hello from command line."

Configuration from the environment.

    > $Env:Greeting = "Hello from environment"
    > dotnet run
    > $Env:Greeting = $null

Configuration from the appsettings.json file.

    ++++ "Greeting": "Hello from appsettings."
    > dotnet run
    ---- "Greeting": "Hello from appsettings."
