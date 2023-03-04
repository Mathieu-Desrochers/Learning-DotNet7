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

Configuration using a .env file.  
Run the following commands.

    > dotnet add package DotNetEnv

Modify the Program.cs file.

    builder.Configuration.AddDotNetEnv();

From the .env file.

    Greeting=Hello from .env.

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
    builder.Logging.AddOpenTelemetry(options =>
    {
        options.AddConsoleExporter();
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

    > dotnet add package OpenTelemetry.Exporter.Console
    > dotnet add package OpenTelemetry.Extensions.Hosting

Modify the Program.cs file.

    builder.Services.AddOpenTelemetry().WithMetrics(configure =>
    {
        configure.AddConsoleExporter();
        configure.AddMeter("learning-dotnet7");
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

Distributed Services
--------------------
Run the following commands.

    > dotnet new web -o Customers-Api
    > dotnet new web -o Orders-Api

Create the Customers-Api/Models.cs file.

    public record Customer(string firstName, string lastName);

Modify the Customers-Api/Program.cs file.

    app.MapGet("/", () => new Customer("Alice", "Alisson"));

Create the Customers-Api/Models.cs file.

    public record Customer(string firstName, string lastName);
    public record Order(string customer, string coffee, int doughnuts, decimal total);

Modify the Customers-Api/Program.cs file.

    builder.Services.AddHttpClient();

    var customersApiUrl = app.Configuration.GetValue<string>("customers-api-url");

    app.MapGet("/", async Task<IResult> (IHttpClientFactory httpClientFactory) =>
    {
        using HttpClient httpClient = httpClientFactory.CreateClient();
        using Stream stream = await httpClient.GetStreamAsync(customersApiUrl);
        Customer customer = await JsonSerializer.DeserializeAsync<Customer>(stream)!;

        Order order = new($"{customer.firstName} {customer.lastName}", "Large", 2, 4.99M);
        return Results.Ok(order);
    });

Run the following commands.

    > dotnet run --project Customers-Api --urls http://localhost:5001
    > dotnet run --project Orders-Api --urls http://localhost:5002
        --customer-apis-url http://localhost:5001

    > curl http://localhost:5002

Creating a Docker Image
-----------------------
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
    CMD ["dotnet", "Learning-DotNet7.dll"]

Run the following commands.  
The base image forces dotnet to bind on port 80.

    > docker image build -t learning-dotnet7 .
    > docker container run -p 5000:80 learning-dotnet7

Composing Docker Images
----------------------- 
Create their Dockerfiles.

Create the docker-compose.yml file.

    version: "3.9"
    services:
      customers-api:
        build: "./Customers-Api"
      orders-api:
        build: "./Orders-Api"
        environment:
          customers-api-url: "http://customers-api"
        ports:
          - "5000:80"

Run the following commands.

    > docker compose up
    > curl http://localhost:5000

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
