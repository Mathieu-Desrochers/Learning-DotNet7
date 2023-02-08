Creating a WebApi
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

Delete the appsettings.Development.json file.  
Cleanup the appsettings.json file.

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

Creating a Docker Image
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
The base image forces dotnet to bind on port 80.

    > docker image build -t learning-dotnet7 .
    > docker container run -p 5000:80 learning-dotnet7
    > curl http://localhost:5000

Running on AWS
--------------
Create a private ECR Registry.

    Name: learning-dotnet7

Click on View push commands.  
Run the commands shown.

    > aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin 00000.dkr.ecr.ca-central-1.amazonaws.com
    > docker tag learning-dotnet7:latest 00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7:latest
    > docker push 00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7:latest

Copy the repository URL.

    00000.dkr.ecr.ca-central-1.amazonaws.com/learning-dotnet7

Create an ECS Cluster.

    Name: learning-dotnet7

Create a Task Definition.
  
    Family: learning-dotnet7
    Container name: learning-dotnet7
    Image URI: repository URL copied above

Create a Service in the Cluster.

    Launch type: Fargate
    Application type: Service
    Task definition family: learning-dotnet7
    Service name: learning-dotnet7
    Desired tasks: 1
    Create a new security group (learning-dotnet7, HTTP, Anywhere)
    Public IP: Turned on

Click on the Task in the Cluster.  
Get its public IP from the Network tab.  
Run the following command.

    > curl http://public-ip

Managing Configuration
----------------------
Modify the Program.cs file.

    var greeting = app.Configuration.GetValue<string>("Greeting");
    app.MapGet("/", () => greeting);

Configuration from the command line.

    > dotnet run --Greeting "Hello from the command line."

Configuration from the environment.  
Run the following command.

    > $Env:Greeting = "Hello from the environment."

Configuration from the appsettings.json file.

    {
      "Greeting": "Hello from appsettings."
    }

Configuration from AWS.  
Create a Parameter in Systems Manager.

    Name: /learning-dotnet7/Greeting
    Value: Hello from AWS.

Run the following command.

    > dotnet add package Amazon.Extensions.Configuration.SystemsManager

Modify the Program.cs file.

    builder.Configuration.AddSystemsManager("/learning-dotnet7/");

Run the following commands.

    > dotnet run
    > curl http://localhost:5000

Further reading:

    The IOptions<T> pattern and dependency injection.

Producing Logs
--------------
Logging to the console.  
Modify the Program.cs file.

    app.MapGet("/", () => {
        app.Logger.LogInformation("Saying hello.");
        return "This hello was logged.";
    });

Run the following commands.

    > dotnet run
    > curl http://localhost:5000

Logging to OpenTelemetry.  
Run the following commands.

    > dotnet add package OpenTelemetry.Exporter.Console

Modify the Program.cs file.

    builder.Logging.ClearProviders();
    builder.Logging.AddOpenTelemetry(options =>
    {
        options.AddConsoleExporter();
    });

Run the following commands.

    > dotnet run
    > curl http://localhost:5000

Further reading:

    The ILogger<T> pattern and dependency injection.
    The dotnet-trace tool.

Producing Metrics
-----------------
Run the following commands.

    > dotnet add package OpenTelemetry.Exporter.Console
    > dotnet add package OpenTelemetry.Extensions.Hosting

Modify the Program.cs file.

    builder.Services.AddOpenTelemetryMetrics(builder =>
    {
        builder.AddConsoleExporter();
        builder.AddMeter("learning-dotnet7");
    });

    var meter = new Meter("learning-dotnet7");
    var greetingsCounter = meter.CreateCounter<int>("greetings_count");

    app.MapGet("/", () => {
        greetingsCounter.Add(1);
        return "This hello was counted.";
    });

Run the following commands.

    > dotnet run
    > curl http://localhost:5000

Further reading:

    The dotnet-counters tool.
