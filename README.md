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
By default dotnet binds on port 5000.

    > dotnet run
    > curl http://localhost:5000

Creating a docker image
-----------------------
Create a .dockerignore file with the following content.

    Dockerfile
    bin
    obj

Create a Dockerfile file with the following content.

    FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
    WORKDIR /src
    COPY . .
    RUN dotnet restore
    RUN dotnet publish -c release -o /app

    FROM mcr.microsoft.com/dotnet/aspnet:7.0
    WORKDIR /app
    COPY --from=build /app .
    CMD ["dotnet", "Learning-DotNet7.dll"]

Run the following commands.  
The base image forces dotnet to bind on port 80 in the container.

    > docker image build -t learning-dotnet7 .
    > docker container run -p 5000:80 learning-dotnet7
    > curl http://localhost:5000

Deploying on AWS
----------------
Create a private ECR registry.

    Name: learning-dotnet7

Run the commands shown when clicking on View push commands.

    > aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin 00000.dkr.ecr.ca-central-1.amazonaws.com
    > docker tag learning-dotnet7:latest 00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7:latest
    > docker push 00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7:latest

Copy the repository URL.

    00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7

Running on AWS
--------------
Create an ECS Cluster.

    Name: learning-dotnet7

Create a Task definition.
  
    Family: learning-dotnet7
    Container name: learning-dotnet7
    Image URI: Repository URL copied above

Create a Service in the Cluster.

    Launch type: Fargate
    Application type: Service
    Task definition family: learning-dotnet7
    Service name: learning-dotnet7
    Desired tasks: 1
    Create a new security group (learning-dotnet7, HTTP, Anywhere)
    Public IP: Turned on

Click on the Task in the Cluster.  
Get its public IP on the Network tab.

Run the following command.

    > curl http://task-public-ip

Managing configuration
----------------------
Modify the Program.cs file.

    var greeting = app.Configuration.GetValue<string>("Greeting");
    app.MapGet("/", () => greeting);

Configuration from the command line.

    > dotnet run --Greeting "Hello from the command line."

Configuration from the environment.

    > $Env:Greeting = "Hello from the environment"
    > dotnet run

Configuration from the appsettings.json file.  

    ++++ "Greeting": "Hello from appsettings."
    > dotnet run
