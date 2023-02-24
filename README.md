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

Managing Configuration
----------------------
Modify the Program.cs file.

    var greeting = app.Configuration.GetValue<string>("Greeting");
    app.MapGet("/", () => greeting);

From the command line.

    > dotnet run --Greeting "Hello from the command line."

From the environment.  

    > $Env:Greeting = "Hello from the environment."

From the appsettings.json file.

    {
      "Greeting": "Hello from appsettings."
    }

Configuration using the IOptions pattern.  
Create the following class.

    public class GreetingOptions
    {
        public string Greeting { get; set; } = String.Empty;
    }

Modify the Program.cs file.

    var section = builder.Configuration.GetSection(nameof(GreetingOptions));
    builder.Services.Configure<GreetingOptions>(section);

    app.MapGet("/", (IOptions<GreetingOptions> options) => options.Value.Greeting);

Managing Logs
-------------
Logging to the console.  
Modify the Program.cs file.

    app.MapGet("/", () =>
    {
        app.Logger.LogInformation("Saying hello.");
        return "This hello was logged.";
    });

Logging using the ILogger pattern.  
Modify the Program.cs file.

    app.MapGet("/", (ILogger<Program> logger) =>
    {
        logger.LogInformation("Saying hello using ILogger.");
        return "This hello was logged.";
    });

Sending logs to OpenTelemetry.  
Run the following commands.

    > dotnet add package OpenTelemetry.Exporter.Console

Modify the Program.cs file.

    builder.Logging.ClearProviders();
    builder.Logging.AddOpenTelemetry(options =>
    {
        options.AddConsoleExporter();
    });

Managing Metrics
----------------
Modify the Program.cs file.

    var meter = new Meter("learning-dotnet7");
    var greetingsCounter = meter.CreateCounter<int>("greetings_count");

    app.MapGet("/", () =>
    {
        greetingsCounter.Add(1);
        return "This hello was counted.";
    });

Run the following commands once the program is running.

    > dotnet counters monitor --name Learning-DotNet7 --counters learning-dotnet7

Sending metrics to OpenTelemetry.  
Run the following commands.

    > dotnet add package OpenTelemetry.Exporter.Console
    > dotnet add package OpenTelemetry.Extensions.Hosting

Modify the Program.cs file.

    builder.Services.AddOpenTelemetry().WithMetrics(configure =>
    {
        configure.AddConsoleExporter();
        configure.AddMeter("learning-dotnet7");
    });

Managing Traces
---------------
Modify the Program.cs file.

    var activitySource = new ActivitySource("learning-dotnet7");

    app.MapGet("/", () =>
    {
        using (var activity = activitySource.StartActivity("Greeting"))
        {
            Thread.Sleep(500);
            return "This hello was timed.";
        }
    });

Sending traces to OpenTelemetry.  
Run the following commands.

    > dotnet add package OpenTelemetry.Exporter.Console
    > dotnet add package OpenTelemetry.Extensions.Hosting
    > dotnet add package OpenTelemetry.Instrumentation.AspNetCore

Modify the Program.cs file.

    builder.Services.AddOpenTelemetry().WithTracing(configure =>
    {
        configure.AddAspNetCoreInstrumentation();
        configure.AddConsoleExporter();
        configure.AddSource("learning-dotnet7");
    });

Managing Authentification
-------------------------

Managing Localization
---------------------

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
Create a private ECR registry.

    Name: learning-dotnet7

Click on View push commands and run them.  
Copy the repository URL.

Create an ECS cluster.

    Name: learning-dotnet7

Create a task definition.
  
    Family: learning-dotnet7
    Container name: learning-dotnet7
    Image URI: repository URL copied above

Create a service in the cluster.

    Launch type: Fargate
    Application type: Service
    Task definition family: learning-dotnet7
    Service name: learning-dotnet7
    Desired tasks: 1
    Create a new security group (learning-dotnet7, HTTP, Anywhere)
    Public IP: Turned on

Click on the task in the cluster.  
Get its public IP from the Network tab.  
Run the following command.

    > curl http://public-ip
