Creating a Web Api
------------------
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

Configuration
-------------
Modify the Program.cs file.

    var greeting = app.Configuration.GetValue<string>("Greeting");
    app.MapGet("/", () => greeting);

From the command line.

    > dotnet run --Greeting "Hello from the command line."

From the environment.  

    > $Env:Greeting = "Hello from the environment."

From the appsettings.json file.

    {
      "Greeting": "Hello from appsettings.json."
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

    app.MapGet("/", (IOptions<GreetingOptions> options) =>
    {
        return options.Value.Greeting;
    });

Logging
-------
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
    builder.Logging.AddOpenTelemetry(configure =>
    {
        configure.AddConsoleExporter();
    });

Metrics
-------
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

    > dotnet add package OpenTelemetry.Extensions.Hosting
    > dotnet add package OpenTelemetry.Exporter.Console

Modify the Program.cs file.

    builder.Services.AddOpenTelemetry().WithMetrics(configure =>
    {
        configure.AddMeter("learning-dotnet7");
        configure.AddConsoleExporter();
    });

Traces
------
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

    > dotnet add package OpenTelemetry.Extensions.Hosting
    > dotnet add package OpenTelemetry.Exporter.Console

Modify the Program.cs file.

    builder.Services.AddOpenTelemetry().WithTracing(configure =>
    {
        configure.AddSource("learning-dotnet7");
        configure.AddConsoleExporter();
    });

Localization
------------
Modify the Program.cs file.

    builder.Services.AddLocalization();

    var supportedCultures = new[] { "en-US", "fr-CA" };

    app.UseRequestLocalization(
        new RequestLocalizationOptions()
            .SetDefaultCulture(supportedCultures[0])
            .AddSupportedCultures(supportedCultures)
            .AddSupportedUICultures(supportedCultures));

    app.MapGet("/", () => Thread.CurrentThread.CurrentCulture.Name);

From the query string.

    > curl http://localhost:5000?culture=fr-CA

From the cookies.

    > $cookieValue = [uri]::EscapeDataString("c=fr-CA|uic=fr-CA")
    > curl -b ".AspNetCore.Culture=$cookieValue" http://localhost:5000

From the headers.

    > curl -H "Accept-Language: fr-CA" http://localhost:5000

Localization using the IStringLocalizer pattern.  
Create the following class.

    public class SharedResources { }

Create the SharedResources.en-US.resx file.

    <root>
      <data name="Greeting">
        <value>Hello from the resx.</value>
      </data>
    </root>

Create the SharedResources.fr-CA.resx file.

    <root>
      <data name="Greeting">
        <value>Bonjour du resx.</value>
      </data>
    </root>

Modify the Program.cs file.

    app.MapGet("/", string (IStringLocalizer<SharedResources> stringLocalizer) =>
    {
        return stringLocalizer["Greeting"];
    });

Authentication
--------------
Run the following commands.

    > dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

Modify the Program.cs file.

    builder.Services
        .AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.Authority = "https://********.us.auth0.com/";
            options.Audience = "learning-dotnet7-api";
        });

    builder.Services.AddAuthorization();

    app.UseAuthentication();
    app.UseAuthorization();

    app.MapGet("/", () => "Hello World!").RequireAuthorization();

Register to auth0.com.

    Create an application
    Name: learning-dotnet7-application
    Type: Regular Web Applications
    Allowed Callback URLs: https://localhost/callback
    
    Take note of the following values
    Domain: ********.us.auth0.com
    ClientID: ********
    ClientSecret: ********

    Create an API
    Name: learning-dotnet7-api
    Identifier: learning-dotnet7-api

    Create a database connection
    Name: learning-dotnet7-database
    Applications: learning-dotnet7-application

    Create a user
    Email: learning@dotnet7.com
    Password: ********
    Connection: learning-dotnet7-database

Perform a user login.  
Browse to the following url.  
Recover the authorization code in the callback URL.

    https://********.us.auth0.com/authorize?
        response_type=code&
        client_id=********&
        audience=learning-dotnet7-api&
        redirect_uri=https://localhost/callback

Simulate the server side callback.  
Run the following commands.  
Recover the access token in the response body.

    > curl -X POST 'https://********.us.auth0.com/oauth/token'
        -H 'content-type: application/x-www-form-urlencoded'
        -d 'grant_type=authorization_code'
        -d 'client_id=********'
        -d 'client_secret=********'
        -d 'code=********'
        -d 'redirect_uri=https://localhost/callback'

Run the following commands.

    > curl http://localhost:5000 -H 'Authorization: Bearer ********'

Background Services
-------------------
Run the following commands.

    > dotnet add package Microsoft.Extensions.Hosting

Create the following class.

    public class CheeringBackgroundService : BackgroundService
    {
        private readonly ILogger<CheeringBackgroundService> logger;

        public CheeringBackgroundService(ILogger<CheeringBackgroundService> logger)
        {
            this.logger = logger;
        }

        protected override async Task ExecuteAsync(CancellationToken cancellationToken)
        {
            while (!cancellationToken.IsCancellationRequested)
            {
                await Task.Delay(5000, cancellationToken);
                logger.LogInformation("Good job guys, keep going!");
            }
        }
    }

Modify the Program.cs file.

    builder.Services.AddHostedService<CheeringBackgroundService>();

Docker Images
-------------
Create the .dockerignore file.

    Dockerfile
    bin
    obj

Create the Dockerfile file.

    FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
    WORKDIR /src
    COPY . .
    RUN dotnet restore
    RUN dotnet publish -c release -o /app

    FROM mcr.microsoft.com/dotnet/aspnet:7.0
    WORKDIR /app
    COPY --from=build /app .
    ENTRYPOINT ["dotnet", "Learning-DotNet7.dll"]

Run the following commands.  
The base image forces dotnet to bind on port 80.

    > docker image build -t learning-dotnet7 .
    > docker container run -p 5000:80 learning-dotnet7

Distributed Services
--------------------
Run the following commands.

    > dotnet new web -o Customers-Api
    > dotnet new web -o Orders-Api

Modify the Customers-Api/Program.cs file.

    app.MapGet("/", () => "Alice Alisson");

Create the Orders-Api/Models.cs file.

    public record Order(string customer, string coffee, int doughnuts, decimal total);

Modify the Orders-Api/Program.cs file.

    var customersApiUrl = app.Configuration.GetValue<string>("customers-api-url");

    app.MapGet("/", async () =>
    {
        using HttpClient httpClient = new HttpClient();
        String customer = await httpClient.GetStringAsync(customersApiUrl);
        return new Order($"{customer}", "Large", 2, 4.99M);
    });

Run the following commands.

    > dotnet run --project Customers-Api --urls http://localhost:5001
    > dotnet run --project Orders-Api --urls http://localhost:5002
        --customers-api-url http://localhost:5001

Composing Services
------------------
Create a Dockerfile for each project.  
Create the docker-compose.yml file.

    services:
      customers-api:
        build: ./Customers-Api
        ports:
          - 5001:80
      orders-api:
        build: ./Orders-Api
        environment:
          customers-api-url: http://customers-api
        ports:
          - 5002:80

Run the following commands.

    > docker compose up

Collecting Telemetry
--------------------
Follow these steps for each project.  
Run the following commands.

    > dotnet add package OpenTelemetry.Extensions.Hosting
    > dotnet add package OpenTelemetry.Instrumentation.AspNetCore
    > dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
    > dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol.Logs

Modify the Program.cs file.

    var resourceBuilder = ResourceBuilder.CreateDefault().AddService([name-of-api]);
    var endpoint = new Uri("http://otel-collector:4317");
    
    builder.Logging.AddOpenTelemetry(configure =>
    {
        configure.SetResourceBuilder(resourceBuilder);
        configure.AddOtlpExporter(opt => { opt.Endpoint = endpoint; });
    });
    
    builder.Services.AddOpenTelemetry().WithMetrics(configure =>
    {
        configure.SetResourceBuilder(resourceBuilder);
        configure.AddAspNetCoreInstrumentation();
        configure.AddOtlpExporter(opt => { opt.Endpoint = endpoint; });
    });
    
    builder.Services.AddOpenTelemetry().WithTracing(configure =>
    {
        configure.SetResourceBuilder(resourceBuilder);
        configure.AddAspNetCoreInstrumentation();
        configure.AddOtlpExporter(opt => { opt.Endpoint = endpoint; });
    });

Modify the docker-compose.yml file.  

    otel-collector:
      image: otel/opentelemetry-collector-contrib:latest
      command: [ --config=/etc/otel-collector-config.yaml ]
      volumes:
        - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml

Create the otel-collector-config.yaml file.

    receivers:
      otlp:
        protocols:
          grpc:
    
    exporters:
      loki:
        endpoint: https://********:********@logs-prod-********.grafana.net/loki/api/v1/push
      prometheusremotewrite:
        endpoint: https://********:********@prometheus-********.grafana.net/api/prom/push
      otlp:
        endpoint: tempo-prod-********.grafana.net:443
        headers:
          authorization: Basic ********
    
    service:
      pipelines:
        logs:
          receivers: [ otlp ]
          exporters: [ loki ]
        metrics:
          receivers: [ otlp ]
          exporters: [ prometheusremotewrite ]
        traces:
          receivers: [ otlp ]
          exporters: [ otlp ]

Register to grafana.com.

    Create an API key
    Name: learning-dotnet7
    Role: MetricsPublisher

    Click on Send Logs under Loki
    Modify the loki exporter endpoint
    Set the username to the User number
    Set the password to the API key
    Complete the server name

    Click on Send Metrics under Prometheus
    Modify the prometheusremotewrite exporter endpoint
    Set the username to the User number
    Set the password to the API key
    Complete the server name

    Click on Send Traces under Tempo
    Modify the otlp exporter endpoint
    Set the authorization to echo -n "User number:API key" | base64
    Complete the server name

Metrics are exported every 60 seconds.  
Add the logging exporter to see exports at the console.

Uncomposing a Service
---------------------
Modify the docker-compose.yml file.  
Remove the service that will run locally.  
Replace its address by host.docker.internal and its local port.

    services:
      orders-api:
        build: ./Orders-Api
        environment:
          customers-api-url: http://host.docker.internal:5001
        ports:
          - 5002:80

Run the following commands.

    > docker compose up
    > dotnet run --project Customers-Api --urls http://localhost:5001

Unit Tests
----------
Run the following commands.

    > dotnet new classlib -o Rebates
    > dotnet new xunit -o Rebates.Tests
    > dotnet add ./Rebates.Tests/Rebates.Tests.csproj
        reference ./Rebates/Rebates.csproj

Rename Rebates/Class1.cs to RebateCalculator.cs.  
Modify the file content.

    public class RebateCalculator
    {
        public decimal GetRebate(decimal orderAmount)
        {
            if (orderAmount >= 1000.00M)
                return orderAmount * 0.05M;

            return 0;
        }
    }

Rename Rebates.Tests/UnitTest1.cs to RebateCalculatorTests.cs.  
Modify the file content.

    public class RebateCalculatorTests
    {
        [Theory]
        [InlineData(250.00)]
        [InlineData(999.99)]
        public void GetRebate_SmallOrder_ReturnsZero(decimal orderAmount)
        {
            RebateCalculator rebateCalculator = new();
            decimal rebate = rebateCalculator.GetRebate(orderAmount);
            Assert.Equal(0.00M, rebate);
        }

        [Theory]
        [InlineData(1_000.00, 50.00)]
        [InlineData(250_000.00, 12_500.00)]
        public void GetRebate_BigOrder_ReturnsCorrectRebate(
            decimal orderAmount, decimal expectedRebate)
        {
            RebateCalculator rebateCalculator = new();
            decimal rebate = rebateCalculator.GetRebate(orderAmount);
            Assert.Equal(expectedRebate, rebate);
        }
    }

Run the following commands.

    > cd Rebates.Tests
    > dotnet test

Code Coverage
-------------
Run the following commands.

    > dotnet test --collect:"XPlat Code Coverage"

    > dotnet tool install -g dotnet-reportgenerator-globaltool

    > reportgenerator
        -reports:"TestResults/00000000-0000-0000-0000-000000000000/coverage.cobertura.xml"
        -targetdir:"CoverageResults"
        -reporttypes:Html

Browse to the following file.

    ./CoverageResults/index.html

Integration Tests
-----------------
Run the following commands.

    > dotnet new web -o Shouter
    > dotnet new xunit -o Shouter.Tests
    > dotnet add ./Shouter.Tests/Shouter.Tests.csproj
        reference ./Shouter/Shouter.csproj

Modify the Shouter/Program.cs file.

    app.MapGet("/{value}", (string value) => value.ToUpper());

    public partial class Program { }

Run the following commands.

    > cd Shouter.Tests
    > dotnet add package Microsoft.AspNetCore.Mvc.Testing

Rename Shouter.Tests/UnitTest1.cs to IntegrationTests.cs.  
Modify the file content.

    public class IntegrationTests : IClassFixture<WebApplicationFactory<Program>>
    {
        private readonly WebApplicationFactory<Program> factory;

        public IntegrationTests(WebApplicationFactory<Program> factory)
        {
            this.factory = factory;
        }

        [Theory]
        [InlineData("/mumble", "MUMBLE")]
        [InlineData("/whisper", "WHISPER")]
        public async Task Get_DoesShout(string url, string expectedResponse)
        {
            HttpClient httpClient = this.factory.CreateClient();
            string result = await httpClient.GetStringAsync(url);
            Assert.Equal(expectedResponse, result);
        }
    }

Run the following commands.

    > dotnet test

Packaging Libraries
-------------------
Run the following commands.

    > dotnet new classlib

Modify the Learning-DotNet7.csproj file.

    <PropertyGroup>
      <PackageId>Learning.DotNet7.Test</PackageId>
      <Version>1.0.0</Version>
      <Authors>Learning-DotNet7</Authors>
      <Company>Learning-DotNet7</Company>
    </PropertyGroup>

Rename Class1.cs to NumberGenerator.cs.  
Modify the file content.

    public static class NumberGenerator
    {
        public static int GetNumber()
        {
            return 100;
        }
    }

Register to nuget.org.

    Create an API key
    Key Name: learning-dotnet7-api-key
    Take note of the key: ********

Run the following commands.

    > dotnet pack
    > dotnet nuget push ./bin/Debug/Learning-DotNet7.1.0.0.nupkg
        --source https://api.nuget.org/v3/index.json
        --api-key ********

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
