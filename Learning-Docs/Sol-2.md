## ASP.NET Core Fundamentals

### What is ASP.NET Core?

ASP.NET Core is a cross-platform, open-source framework from Microsoft for building web applications, REST APIs, and other HTTP-based services using C# and .NET.

For example, suppose you're building an employee management system. ASP.NET Core can expose endpoints that let a frontend retrieve employee details, create employees, or update their information.

Client [Browser, mobile app, or frontend]

HTTP request / response

ASP.NET Core application [Routing · Middleware · Endpoints · Business logic]

Database [SQL Server, PostgreSQL, etc.]

#### Why ASP.NET Core?

- Cross-platform: Runs on Windows, Linux, and macOS.
- High performance: Designed for efficient web request processing.
- Built-in dependency injection: Supports managing application dependencies.
- Middleware pipeline: Lets you handle requests through configurable components.
- Modern configuration and logging: Integrates with the .NET configuration and logging systems.
- Multiple application types: Supports Web APIs, MVC applications, Razor Pages, and more.

#### ASP.NET Core vs ASP.NET Framework

This is a common interview question.


| Feature              | ASP.NET Framework                           | ASP.NET Core                         |
| ---------------------- | --------------------------------------------- | -------------------------------------- |
| Platform             | Windows only                                | Cross-platform                       |
| Performance          | Generally lower for modern web workloads    | Designed for high performance        |
| Hosting              | Commonly IIS                                | Kestrel, IIS, containers, and others |
| Dependency injection | Not built in as a general framework service | Built in                             |
| Development model    | Older ASP.NET model                         | Modern, modular web framework        |
| Current direction    | Existing applications and maintenance       | Modern web development               |

Interview answer: ASP.NET Core is Microsoft's modern, cross-platform web framework. Compared with the older ASP.NET Framework, it provides built-in dependency injection, a middleware-based request pipeline, and flexible hosting options.

Common trap: ASP.NET Core is not simply ASP.NET Framework renamed. It has a different architecture and hosting model. Also, ASP.NET Core is part of the modern .NET ecosystem. The old .NET Framework and modern .NET are different runtimes/platforms.

#### What happens when an ASP.NET Core application starts?

Look at a typical `Program.cs` file:

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();

app.Run();
```

Let's understand each step.

##### Create the builder

`WebApplication.CreateBuilder(args)`

Creates the application builder and loads the default configuration, logging, hosting setup, and service registration infrastructure.

##### Register services

`builder.Services.AddControllers()`

Registers MVC controller services in the dependency injection container.

##### Build the application

`builder.Build()`

Creates the `WebApplication` using the configured services and application settings.

##### Configure endpoints

`app.MapControllers()`

Maps controller routes so matching HTTP requests can reach controller actions.

##### Start the application

`app.Run()`

Starts the web application and waits for incoming requests.

Interview trap: `builder.Services.AddControllers()` registers services; it does not map controller endpoints. `app.MapControllers()` maps the endpoints.

#### What is Kestrel?

Interview question: What web server does ASP.NET Core use?

Kestrel is the default, cross-platform web server used by ASP.NET Core.

It listens for HTTP requests and passes them into the ASP.NET Core request-processing pipeline.

For example, your application might listen on:

```
https://localhost:7001
```

When you send a request to that address, Kestrel accepts the request and the ASP.NET Core application processes it.

Kestrel can run directly as the web server or behind a reverse proxy such as IIS or Nginx.

Common interview trap: IIS and Kestrel are not mutually exclusive. IIS can act as a reverse proxy in front of Kestrel, depending on the hosting configuration.

### HTTP Request Lifecycle

it explains what happens when a client sends a request to your API and how your application returns a response.

#### What happens when an HTTP request reaches ASP.NET Core?

Suppose your frontend calls this API:

```
GET /api/employees/10
```

Your ASP.NET Core application needs to receive the request, identify the correct endpoint, execute the required logic, and return the result.

The overall flow looks like this:

1. Client - Browser, Postman, or frontend
2. Kestrel - Receives the HTTP request
3. Middleware pipeline - Authentication, logging, error handling, etc.
4. Routing and endpoint selection - Finds the matching route and action
5. Controller / endpoint - Executes application logic and retrieves data
6. HTTP response - Status code, headers, and response body

The response then travels back through the applicable middleware in reverse order before being sent to the client.

Interview answer: An HTTP request enters through the web server, passes through the configured middleware pipeline, is routed to a matching endpoint, and produces an HTTP response that returns through the pipeline.

#### What is routing?

Routing is the mechanism ASP.NET Core uses to match an incoming HTTP request to an endpoint.

For example:

```
[ApiController]
[Route("api/[controller]")]
public class EmployeesController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetEmployee(int id)
    {
        return Ok(new
        {
            Id = id,
            Name = "Arun"
        });
    }
}
```

If the client sends:

```
GET /api/employees/10
```

ASP.NET Core matches the request to `GetEmployee`, binds `10` to the `id` parameter, and executes the action.

The response will contain JSON similar to:

```
{
  "id": 10,
  "name": "Arun"
}
```

The `Ok()` method produces an HTTP `200 OK` response.

Important routing terms

- Route template: Defines a URL pattern, such as api/[controller].
- Route parameter: A variable part of a URL, such as {id}.
- Endpoint: A callable destination, such as a controller action or a minimal API handler.
- Endpoint routing: The system that matches requests to endpoints.

Common interview trap: Routing selects the appropriate endpoint. It does not execute every controller action in the application.

#### What is the difference between middleware and routing?

You will study middleware deeply in its own roadmap section. For now, understand the distinction.


| Middleware                                              | Routing                                         |
| --------------------------------------------------------- | ------------------------------------------------- |
| Processes requests and responses                        | Matches requests to endpoints                   |
| Can perform logging, authentication, and error handling | Determines which endpoint matches a request     |
| Can modify the request or response                      | Provides endpoint selection                     |
| Can stop the request pipeline                           | Can lead to an endpoint or no matching endpoint |

Routing itself is commonly integrated into the ASP.NET Core middleware pipeline.

#### What happens if no route matches?

Suppose the client calls:

```
GET /api/unknown
```

If no endpoint matches the URL, the request normally results in 404 Not Found, provided no middleware or fallback endpoint changes the outcome.

#### Common HTTP status codes:


| Status                      | Meaning                              |
| ----------------------------- | -------------------------------------- |
| `200 OK`                    | Request succeeded                    |
| `201 Created`               | Resource created                     |
| `400 Bad Request`           | Invalid request                      |
| `401 Unauthorized`          | Authentication is missing or invalid |
| `403 Forbidden`             | Caller is not permitted              |
| `404 Not Found`             | Resource or endpoint not found       |
| `500 Internal Server Error` | Unexpected server error              |

Interview trap: `401` and `403` are different. A `401` generally indicates an authentication problem; a `403` means access is forbidden.

### Project Structure and Environments

how an ASP.NET Core project is organized, what the important files do, and how Development, Staging, and Production environments work.

#### ASP.NET Core project structure

A typical ASP.NET Core Web API project looks like this:

the request API called EmployeeApi/ flows like this:

1. Program.cs - Application startup, service registration, middleware, endpoint mapping, and hosting.
2. Controllers/ - Handles incoming HTTP requests in a controller-based API.
3. Models/ - Application data models and related types.
4. appsettings.json - Default application configuration, such as logging settings.
5. appsettings.Development.json - Configuration overrides for the Development environment.
6. EmployeeApi.csproj - Project configuration, target framework, package references, and build settings.
7. Properties/launchSettings.json - Local development launch profiles and environment variables for launching the app.

Interview tip: This is a typical structure, not a mandatory one. ASP.NET Core doesn't require you to create folders named `Models` or `Services`; you organize the project according to your application's needs.

#### What is `Program.cs`?

`Program.cs` is the entry point where a modern ASP.NET Core application is configured and started.

Example:

```
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddControllers();

var app = builder.Build();

// Configure request handling
app.MapControllers();

app.Run();
```

Understand the three main variables and methods:

- builder: Configures the application and its services.
- app: Represents the configured web application.
- app.Run(): Starts the application and blocks while the server handles requests.

##### What is the difference between `builder` and `app`?

`builder` is used to configure the application before building it. `app` is the built application, on which you configure the request pipeline and map endpoints.

#### What is `appsettings.json`?

It stores application configuration as JSON.

For example:

```
{
  "AppSettings": {
    "ApplicationName": "EmployeeApi",
    "MaxEmployees": 100
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

ASP.NET Core's configuration system can load these values and make them available to your application.

You can access a value using `IConfiguration`:

```
public class EmployeeService
{
    private readonly IConfiguration _configuration;

    public EmployeeService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void ShowSettings()
    {
        string? name = _configuration["AppSettings:ApplicationName"];
        Console.WriteLine(name);
    }
}
```

The colon (`:`) separates nested configuration keys.

Here, `AppSettings:ApplicationName` retrieves `"EmployeeApi"`.

##### Can we store database connection strings in `appsettings.json`?

Yes, but be careful: a committed configuration file can expose credentials. For production, use environment variables, a managed secret store, or another secure configuration provider. User Secrets can help keep secrets out of source control during local development.

#### What is launchSettings.json?

It defines local development launch profiles, such as application URLs and environment variables. It is not the production deployment configuration.

#### ASPNETCORE ENVIRONMENT

##### What are Development, Staging, and Production?

ASP.NET Core applications can run in different environments, allowing settings and behavior to change based on where the application runs.

Development: Used while building and debugging the application. Detailed developer error pages may be enabled.

Staging: Used to test a release in an environment resembling production before deployment.

Production: The live environment used by customers. Detailed exception information should not be exposed to clients.

ASP.NET Core commonly reads the environment name from the `ASPNETCORE_ENVIRONMENT` environment variable.

For example:

```
ASPNETCORE_ENVIRONMENT=Development
```

When the environment is `Development`, the framework can load `appsettings.Development.json` in addition to the default `appsettings.json`.

For production, it can load `appsettings.Production.json` instead.

##### How do environment-specific settings work?

Suppose your default `appsettings.json` contains:

```
{
  "AppSettings": {
    "ApplicationName": "EmployeeApi"
  }
}
```

And `appsettings.Development.json` contains:

```
{
  "AppSettings": {
    "ApplicationName": "EmployeeApi-Dev"
  }
}
```

When the application runs in Development, the environment-specific value normally overrides the default value. The application name will be `EmployeeApi-Dev`.

Common interview trap: The environment name doesn't automatically select a different C# codebase. It influences configuration and any environment-specific behavior you explicitly implement.

##### How do you check the current environment?

ASP.NET Core provides `IWebHostEnvironment`.

```
public class EnvironmentService
{
    private readonly IWebHostEnvironment _environment;

    public EnvironmentService(IWebHostEnvironment environment)
    {
        _environment = environment;
    }

    public void ShowEnvironment()
    {
        if (_environment.IsDevelopment())
        {
            Console.WriteLine("Running in Development");
        }
        else if (_environment.IsProduction())
        {
            Console.WriteLine("Running in Production");
        }
    }
}
```

`IWebHostEnvironment` exposes the environment name and web-hosting information, including the web root path.

You may also see `IHostEnvironment` when only the general hosting environment information is required.

### Configuration System

Real applications need to manage database connections, API URLs, feature settings, logging levels, and environment-specific values.

We'll cover configuration providers, configuration precedence, `<span>IConfiguration</span>`, and strongly typed configuration using the Options pattern.

#### What is configuration in ASP.NET Core?

Configuration is the mechanism used to supply settings to an application without hardcoding those settings into the C# code.

For example, instead of writing:

```
string connectionString =
    "Server=localhost;Database=EmployeeDb;";
```

you store the setting in configuration and retrieve it when needed.

This makes it easier to change settings between Development, Staging, and Production without changing the application code.

#### What are configuration providers?

A configuration provider reads settings from a particular source and makes them available to ASP.NET Core.

Common providers include:


| Provider                                | Typical use                                    |
| ----------------------------------------- | ------------------------------------------------ |
| `appsettings.json`                      | Default application settings                   |
| `appsettings.Development.json`          | Development-specific overrides                 |
| Environment variables                   | Deployment-specific settings                   |
| User Secrets                            | Local development secrets                      |
| Command-line arguments                  | Settings passed when launching the application |
| Azure Key Vault or another secret store | Securely managed secrets                       |

ASP.NET Core's default builder configures several standard providers automatically.

Interview trap: `IConfiguration` is not a JSON file. It is an abstraction that lets you read configuration regardless of its source.

#### Configuration precedence - very important

Suppose the same setting exists in multiple configuration sources.

Which value does ASP.NET Core use?

With the default `WebApplication.CreateBuilder(args)` configuration setup, later providers generally override earlier providers when keys match.

For the standard application configuration sources, a simplified order from lower to higher precedence is:

LOWER PRECEDENCE

`appsettings.json`

`appsettings.{Environment}.json`

User Secrets (in Development)

Environment variables

Command-line arguments

HIGHER PRECEDENCE

This is a simplified view of the default setup. Custom providers can change the order, and host configuration has its own details.

##### Example

In `appsettings.json`:

```
{
  "AppSettings": {
    "AppName": "EmployeeApi"
  }
}
```

Suppose an environment variable is set to:

```
AppSettings__AppName=EmployeeApi-Production
```

Then the environment variable overrides the JSON value, so the effective value is `EmployeeApi-Production`.

Important: Double underscores (`__`) represent nested configuration separators in environment variable names. This makes `AppSettings__AppName` correspond to `AppSettings:AppName`.

#### What is `IConfiguration`?

`IConfiguration` is an interface used to read configuration values.

Example:

```
public class EmployeeService
{
    private readonly IConfiguration _configuration;

    public EmployeeService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void DisplayAppName()
    {
        string? appName =
            _configuration["AppSettings:AppName"];

        Console.WriteLine(appName);
    }
}
```

The framework registers configuration services for you when using the standard application builder.

You can also retrieve typed values:

int maxEmployees =
    _configuration.GetValue<int></int>("AppSettings:MaxEmployees");

Interview trap: IConfiguration values are commonly read as strings, even when the underlying setting represents a number or Boolean. Methods such as GetValue<T></t>() can convert values to the requested type.

#### What is the Options pattern?

Imagine your application has many settings:

```
{
  "EmailSettings": {
    "Host": "smtp.example.com",
    "Port": 587,
    "EnableSsl": true
  }
}
```

You could read each value individually using `IConfiguration`. But as settings grow, this becomes harder to maintain.

The Options pattern lets you bind a configuration section to a strongly typed C# class.

##### Step 1: Create a settings class

```
public class EmailSettings
{
    public string Host { get; set; } = "";
    public int Port { get; set; }
    public bool EnableSsl { get; set; }
}
```

##### Step 2: Register the configuration

In `Program.cs`:

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

var app = builder.Build();

app.Run();
```

Configure< T > () registers the options configuration with the dependency injection container.

##### Step 3: Inject IOptions< T >

```
using Microsoft.Extensions.Options;

public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }

    public void DisplaySettings()
    {
        Console.WriteLine(_settings.Host);
        Console.WriteLine(_settings.Port);
    }
}
```

Now your service receives strongly typed settings rather than repeatedly retrieving individual configuration keys.

##### Difference between `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor`


| Feature<T></t>   | IOptions< T >               | IOptionsSnapshot< T >                               | IOptionsMonitor< T >                         |
| ------------------ | ----------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| Main purpose     | Basic options access        | Per-scope options snapshot                          | Current options and change notifications     |
| Scoped lifetime  | No; singleton-friendly      | Yes                                                 | No; singleton-friendly                       |
| Observes changes | Not as a refreshed value    | Can refresh between scopes when reload is supported | Can observe supported configuration changes  |
| Typical use      | Stable application settings | Request-scoped settings                             | Long-lived services needing updated settings |

Remember these interview points

- IOptions<T></t> provides a straightforward way to access options.
- IOptionsSnapshot<T></t> is scoped and is useful when settings should be evaluated per scope, such as per web request.
- IOptionsMonitor<T></t> supports retrieving current options and subscribing to change notifications.

Common trap: IOptionsSnapshot<T></t> is not suitable for injecting into a singleton because it is scoped. IOptionsMonitor<T></t> is suitable for singleton services.
Also, configuration reload behavior depends on the provider and whether reload is enabled. Not every configuration source supports live updates.

##### Configuration vs Options pattern


| IConfiguration                           | Options pattern                                                   |
| ------------------------------------------ | ------------------------------------------------------------------- |
| Reads values using configuration keys    | Maps a configuration section to a C# class                        |
| Useful for individual or dynamic keys    | Useful for related groups of settings                             |
| Values are often accessed by string keys | Provides strongly typed properties                                |
| Example: GetValue< int >(...)            | Example: IOptions< EmailSettings ><EmailSettings></emailsettings> |

Both approaches are useful. The Options pattern is often preferable for a group of related settings because the structure is clearer and easier to maintain.

## Dependency Injection (DI)

Dependency Injection
Starting now

- 1. DI fundamentals and the problem DI solves
- 2. Inversion of Control (IoC)
- 3. Constructor injection
- 4. Service registration and resolution
- 5. Transient, Scoped, and Singleton lifetimes
- 6. Lifetime mismatches and captive dependencies
  7. Other important DI concepts and interview questions

### DI fundamentals

#### What is Dependency Injection (DI)?

##### Simple definition

Dependency Injection is a technique in which a class receives the objects it depends on from outside instead of creating those objects itself.

Let’s understand this with a practical example.

Imagine you are developing an ASP.NET Core application that sends email notifications.

Your `NotificationService` needs an email sender to send emails. That email sender is its dependency.

##### Without Dependency Injection

```
public class EmailSender
{
    public void SendEmail()
    {
        Console.WriteLine("Email sent");
    }
}

public class NotificationService
{
    private readonly EmailSender _emailSender;

    public NotificationService()
    {
        _emailSender = new EmailSender();
    }

    public void Notify()
    {
        _emailSender.SendEmail();
    }
}
```

What is the problem here?

- NotificationService creates its own EmailSender.
- It is tightly coupled to the concrete EmailSender class.
- Replacing it with an SMS sender or a different email provider requires changing the class.
- Testing becomes harder because you cannot easily substitute a fake sender.

The main issue is that `NotificationService` controls both its business responsibility and the creation of its dependency.

##### With Dependency Injection

Instead of creating `EmailSender` inside `NotificationService`, we pass it through the constructor.

```
public interface IMessageSender
{
    void Send();
}

public class EmailSender : IMessageSender
{
    public void Send()
    {
        Console.WriteLine("Email sent");
    }
}

public class NotificationService
{
    private readonly IMessageSender _sender;

    public NotificationService(IMessageSender sender)
    {
        _sender = sender;
    }

    public void Notify()
    {
        _sender.Send();
    }
}
```

Now `NotificationService` doesn't create its dependency. It receives an object that implements `IMessageSender`.

For example, we can create another implementation:

```
public class SmsSender : IMessageSender
{
    public void Send()
    {
        Console.WriteLine("SMS sent");
    }
}
```

We can supply either an `EmailSender` or an `SmsSender` without changing the `NotificationService` class.

##### How it works

EmailSender or SmsSender - Dependency provided from outside

Passed through the constructor

NotificationService - Uses IMessageSender without creating it

Important: Using an interface is not mandatory for Dependency Injection. You can inject a concrete class too. Interfaces simply make implementations easier to replace and test.

#### Where does ASP.NET Core DI come in?

ASP.NET Core provides a built-in DI container. You register services and their implementations in `Program.cs`.

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<IMessageSender, EmailSender>();
builder.Services.AddScoped<NotificationService>();

var app = builder.Build();
```

Here:

- IMessageSender is the service abstraction.
- EmailSender is the implementation.
- AddScoped registers the service with a particular lifetime. We’ll study lifetimes separately.
- ASP.NET Core can create NotificationService and supply its IMessageSender dependency when the service is requested.

For this example, `NotificationService` doesn't need to be registered if you instantiate it manually. Registration allows the DI container to resolve it automatically.

#### Important interview questions

##### Q1. What is Dependency Injection?

Dependency Injection is a design technique in which a class receives its dependencies from an external source rather than creating them itself. It reduces tight coupling and improves testability and maintainability.

##### Q2. Why do we use DI in ASP.NET Core?

- To reduce tight coupling between classes.
- To replace implementations without changing dependent classes.
- To make unit testing easier by injecting mock or fake dependencies.
- To let the DI container manage service creation and lifetimes.

##### Q3. Is an interface mandatory for DI?

No. DI can inject concrete classes, interfaces, or other supported dependency types. Interfaces are useful when we want interchangeable implementations.

##### Q4. Is DI a design pattern or a framework feature?

DI is a technique and is commonly described as a design pattern. ASP.NET Core provides a built-in DI container to support it.

#### Common interview traps

- DI and the DI container are not the same thing. DI is the technique; the container is a tool that can perform dependency resolution and manage registered service lifetimes.
- DI does not automatically mean loose coupling. Injecting a concrete class can still leave a class coupled to that implementation.
- DI does not require interfaces. Constructor injection of a concrete class is still DI.
- DI does not automatically make code testable. Good design and appropriate dependency boundaries still matter.

### Inversion of Control (IoC)

#### What is Inversion of Control?

Inversion of Control (IoC) is a principle in which control over creating objects or managing dependencies is moved from a class to an external component or framework.

Let's understand it using our previous `NotificationService` example.

##### Without IoC

```
public class NotificationService
{
    private readonly EmailSender _sender;

    public NotificationService()
    {
        _sender = new EmailSender();
    }
}
```

Here, `NotificationService` decides which dependency to create. It controls object creation itself.

##### With IoC

```
public class NotificationService
{
    private readonly IMessageSender _sender;

    public NotificationService(IMessageSender sender)
    {
        _sender = sender;
    }
}
```

Now, `NotificationService` receives its dependency from outside. It no longer controls how that dependency is created.

In ASP.NET Core, the built-in DI container can create and supply the required dependencies.

#### How IoC and DI are related

##### IoC — Principle

Control over object creation or dependency management is delegated elsewhere.

##### DI — Technique

Dependencies are supplied to a class instead of being created inside it.

##### ASP.NET Core DI container

Creates registered services and resolves their dependencies according to their configured lifetimes.

Interview distinction: IoC is the broader principle. Dependency Injection is one way to implement IoC.

IoC can also be achieved through other approaches, such as the Template Method pattern or framework-controlled callbacks.

#### A real ASP.NET Core example

Consider this registration in Program.cs:

```
builder.Services.AddScoped<IMessageSender, EmailSender>();
builder.Services.AddScoped<NotificationService>();
```

And this constructor:

```
public NotificationService(IMessageSender sender)
{
    _sender = sender;
}
```

When the DI container resolves NotificationService, it:

1. Checks which dependencies NotificationService requires.
2. Finds the registered implementation of IMessageSender.
3. Creates or obtains an appropriate EmailSender instance.
4. Supplies it to the NotificationService constructor.

The container handles the dependency resolution rather than requiring `NotificationService` to create its own sender.

#### Important interview questions

##### Q1. What is IoC?

IoC is a principle in which control over certain operations, such as object creation or dependency management, is delegated from application classes to an external component or framework.

##### Q2. What is the difference between IoC and DI?

IoC is a principle, whereas DI is a technique used to achieve IoC by supplying dependencies from outside a class.

##### Q3. Is the DI container the same as IoC?

No. IoC is a principle; a DI container is a tool that helps implement it.

##### Q4. Is DI the only way to achieve IoC?

No. Other approaches include framework callbacks, event-driven mechanisms, and the Template Method pattern.

#### Common interview traps

- Saying that IoC and DI are exactly the same.
- Saying IoC is a class or a library. It is a design principle.
- Assuming that IoC only applies to dependency injection.
- Assuming that ASP.NET Core must use a DI container for every object. Objects can also be created manually, although application services are commonly managed through DI.

### Constructor Injection

#### What is Constructor Injection?

Constructor Injection is a technique where a class receives its required dependencies through its constructor.

Instead of creating dependencies using `new`, the class declares what it needs, and the caller or DI container provides those dependencies.

##### Without Constructor Injection

```
public class OrderService
{
    private readonly EmailService _emailService;

    public OrderService()
    {
        _emailService = new EmailService();
    }

    public void PlaceOrder()
    {
        // Place order
        _emailService.SendEmail();
    }
}
```

Problem: `OrderService` creates `EmailService` directly. Replacing it with another implementation or a test double is harder.

##### With Constructor Injection

```
public interface IEmailService
{
    void SendEmail();
}

public class EmailService : IEmailService
{
    public void SendEmail()
    {
        Console.WriteLine("Email sent");
    }
}

public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
  
    public void PlaceOrder()
    {
        // Place order
        _emailService.SendEmail();
    }
}
```

Notice these three things:

- IEmailService defines the dependency contract.
- The constructor accepts IEmailService.
- The readonly field stores the dependency for use by other methods.

`readonly` means the field can be assigned during declaration or in a constructor, but cannot be reassigned elsewhere after construction.

#### How does ASP.NET Core inject the dependency?

Register the service mappings in Program.cs:

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<IEmailService, EmailService>();
builder.Services.AddScoped<OrderService>();

var app = builder.Build();
```

When `OrderService` is requested from the DI container, ASP.NET Core resolves `IEmailService`, creates or reuses an appropriate `EmailService` instance according to its lifetime, and passes it into the constructor.

1. Program.cs - Register service mappings
2. ASP.NET Core DI container - Resolves IEmailService
3. OrderService constructor - Receives IEmailService

Important: Registering a service does not usually create its instance immediately. The built-in container typically creates it when the service is first resolved.

#### Why is Constructor Injection preferred?

1. Required dependencies are explicit: the constructor tells you what the class needs.
2. Better testability: you can pass a fake or mock implementation during unit testing.
3. Supports immutability: dependencies can be stored in readonly fields.
4. Prevents partially initialized objects: construction fails if required dependencies aren't supplied.
5. Works naturally with ASP.NET Core DI: the container resolves constructor parameters automatically.

### Other types of Dependency Injection

There are three commonly discussed forms of injection.


| Type                  | How it works                               | Typical use                                         |
| ----------------------- | -------------------------------------------- | ----------------------------------------------------- |
| Constructor Injection | Dependency is passed through a constructor | Required dependencies; preferred in most cases      |
| Property Injection    | Dependency is assigned through a property  | Optional dependencies in frameworks that support it |
| Method Injection      | Dependency is passed as a method parameter | Needed only for a particular operation              |

#### Property Injection example

```
public class ReportService
{
    public IEmailService? EmailService { get; set; }
}
```

The caller must assign `EmailService` before using it. Otherwise, it remains `null`.

The built-in ASP.NET Core DI container does not automatically inject arbitrary public properties. You would need to assign the property yourself or use a mechanism that explicitly supports property injection.

#### Method Injection example

```
public class ReportService
{
    public void GenerateReport(IEmailService emailService)
    {
        // Generate report
        emailService.SendEmail();
    }
}
```

The dependency is supplied only when `GenerateReport` is called.

ASP.NET Core also supports action method parameters supplied by its parameter-binding system, including service parameters when explicitly configured with `[FromServices]`.

#### Important interview questions

##### Q1. Why is Constructor Injection preferred over Property Injection?

Constructor Injection makes required dependencies explicit and ensures they are supplied during object creation. Property Injection can leave dependencies unassigned, resulting in runtime errors.

##### Q2. Can a class have multiple constructor parameters?

Yes. A class can receive multiple dependencies through its constructor. However, an excessively large constructor may indicate that the class has too many responsibilities.

##### Q3. What happens if a required dependency isn't registered?

If the DI container cannot resolve a required constructor dependency, service resolution typically fails with an exception indicating that the required service cannot be provided.

##### Q4. Can we inject a concrete class instead of an interface?

Yes. For example:

```
public OrderService(EmailService emailService)
{
    // ...
}
```

This is still Constructor Injection. An interface is useful when you need replaceable implementations or an abstraction for testing.

##### Q5. Can a constructor be private for DI?

For the standard Microsoft DI container, services normally need a suitable public constructor. A private constructor is not ordinarily used for standard constructor injection.

#### Common interview traps

- Constructor Injection does not create dependencies by itself; the caller or container supplies them.
- readonly is useful but not mandatory for DI.
- Interfaces are not mandatory for DI.
- Constructor Injection alone does not prevent circular dependencies. For example, if OrderService requires PaymentService and PaymentService requires OrderService, the built-in container cannot resolve that circular dependency.
- Multiple public constructors can create ambiguity if the DI container cannot select an applicable constructor unambiguously.

### Service Registration and Resolution

It explains how the built-in DI container knows which class to create when a dependency is requested.

#### What is service registration?

Service registration is the process of telling the DI container which services your application uses and how to create or provide them.

In ASP.NET Core, we usually register services in `Program.cs` using `builder.Services`.

For example:

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<IEmailService, EmailService>();
builder.Services.AddScoped<OrderService>();

var app = builder.Build();
```

Here:

- IEmailService is the service type (abstraction).
- EmailService is the implementation.
- OrderService is registered so the container can construct it and resolve its dependencies.

Think of registration as giving the container a set of instructions.

#### The three main registration methods

ASP.NET Core provides three commonly used lifetime registration methods.


| Method                                      | Meaning                                                   |
| --------------------------------------------- | ----------------------------------------------------------- |
| `AddTransient<TService, TImplementation>()` | Creates a new instance each time the service is resolved. |
| `AddScoped<TService, TImplementation>()`    | Reuses an instance within the same DI scope.              |
| `AddSingleton<TService, TImplementation>()` | Reuses one instance for the lifetime of the DI container. |

You can also register a concrete service without an interface:

```
builder.Services.AddScoped<OrderService>();
```

And register an existing instance:

```
var emailService = new EmailService();

builder.Services.AddSingleton<IEmailService>(emailService);
```

In the last example, you supply the instance yourself. The container reuses that same instance and does not create it through the registered implementation type.

#### What is service resolution?

Service resolution is the process of obtaining a registered service from the DI container.

Suppose we have this class:

```
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void PlaceOrder()
    {
        _emailService.SendEmail();
    }
}
```

And this registration:

```
builder.Services.AddScoped<IEmailService, EmailService>();
builder.Services.AddScoped<OrderService>();
```

When the container resolves OrderService, it:

1. Examines its constructor.
2. Finds that IEmailService is required.
3. Looks up the registered implementation, EmailService.
4. Creates or obtains an appropriate instance.
5. Passes that instance into OrderService.

This is called resolving a dependency graph: the container resolves the required service and its dependencies recursively.

1. Request `OrderService`

2. Resolve `IEmailService` - Registration points to `EmailService`

3. Construct `OrderService` - Inject the resolved email service

#### What happens if a service is not registered?

Consider this registration:

```
builder.Services.AddScoped<OrderService>();
```

But suppose `IEmailService` has not been registered.

If the container tries to resolve `OrderService`, it cannot supply the required `IEmailService` dependency. Resolution normally fails with an `InvalidOperationException`.

Interview tip: A class can compile successfully but still fail at runtime because its required dependency was not registered.

#### How do we manually resolve a service?

ASP.NET Core normally resolves services for you through constructor injection. However, manual resolution is possible.

```
using var scope = app.Services.CreateScope();

var orderService =
    scope.ServiceProvider.GetRequiredService<OrderService>();
```

GetRequiredService< T >() returns the requested service or throws an exception if it cannot be resolved.

Important: Avoid resolving application services manually throughout your code. Prefer constructor injection for normal application dependencies. Manual resolution is useful in specific scenarios, such as startup tasks and scoped background operations.

#### Important interview questions

##### Q1. Where do we register services in ASP.NET Core?

Usually in Program.cs, using builder.Services before builder.Build().

Q2. What is the difference between registration and resolution?

- Registration tells the container how a service should be provided.
- Resolution obtains the service and resolves its dependencies.

##### Q3. What does `AddScoped<IEmailService, EmailService>()` mean?

It tells the container to provide `EmailService` when `IEmailService` is requested, reusing an instance within the same scope.

##### Q4. What is `IServiceProvider`?

It is the abstraction used to retrieve services from a service container. ASP.NET Core uses it internally to resolve dependencies.

##### Q5. What is the difference between GetService< T >() and GetRequiredService< T >()?

- GetService< T >() returns null if the service is unavailable.
- GetRequiredService< T >() throws an exception if the service is unavailable.

#### Common interview traps

- Registering a service does not necessarily instantiate it immediately.
- Registering an interface-to-implementation mapping does not mean the interface itself is instantiated.
- The DI container can resolve nested dependencies, not just the first constructor parameter.
- Manual service resolution is supported, but excessive use can hide dependencies and make testing harder.
- AddSingleton does not automatically make your service thread-safe.

### Service Lifetimes

Difference between Transient, Scoped, and Singleton, including when each instance is created and reused.

#### What are service lifetimes?

A service lifetime defines how long a service instance is retained and when the DI container reuses or creates instances.

ASP.NET Core has three main service lifetimes:

1. Transient — AddTransient
   New instance per resolution
   The container creates a new instance each time the service is resolved.
2. Scoped — AddScoped
   One instance per scope
   The same instance is reused for resolutions within a particular scope. In a typical ASP.NET Core web app, a scope is created for each HTTP request.
3. Singleton — AddSingleton
   One instance per container
   The same instance is reused for the lifetime of the root DI container.

#### Practical C# example

Let's create a service that generates a unique ID whenever its instance is constructed.

```
public class MyService
{
    public Guid Id { get; } = Guid.NewGuid();
}
```

Register it in `Program.cs`:

```
builder.Services.AddTransient<MyService>();
// Or use AddScoped<MyService>();
// Or use AddSingleton<MyService>();
```

Now inject the service twice into another class:

```
public class TestService
{
    public Guid FirstId { get; }
    public Guid SecondId { get; }

    public TestService(MyService first, MyService second)
    {
        FirstId = first.Id;
        SecondId = second.Id;
    }
}
```

What happens?

- Transient: FirstId and SecondId will normally differ because the two constructor parameters resolve to separate instances.
- Scoped: They will match because both resolutions use the same scope.
- Singleton: They will match because both resolutions use the same singleton instance.

These results assume the service is resolved twice in the same scope, as shown.

#### Compare the three lifetimes


| Behavior                 | Transient                                    | Scoped                                                  | Singleton                                               |
| -------------------------- | ---------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------- |
| Instance reuse           | No reuse by the container across resolutions | Within the same scope                                   | Across scopes within the same container                 |
| Across two HTTP requests | Usually new instances                        | Different instances                                     | Same instance                                           |
| Typical use              | Lightweight, stateless services              | Request-related business services and EFCore`DbContext` | Shared, thread-safe services and caches                 |
| Thread safety            | Depends on usage                             | Depends on usage                                        | Especially important because many requests may share it |

Important distinction: Transient doesn't mean a new instance for every method call. It means a new instance each time the DI container resolves that service. If the service is injected once into a longer-lived object, that object keeps its injected instance.

#### Understand it through HTTP requests

Imagine two HTTP requests arrive at your ASP.NET Core API.

##### HTTP Request 1

Transient  - New

Scoped - Instance A

Singleton - Shared S

##### HTTP Request 2

Transient - New

Scoped - Instance B

Singleton - Shared S

Illustration: assuming each request resolves the services, and the singleton belongs to the same root container.

The scoped instance changes between requests, while the singleton remains shared.

When a request scope ends, the container disposes scoped services it owns that implement `IDisposable` or `IAsyncDisposable`. Disposable transient services resolved from a scope are also disposed with that scope. Singleton services managed by the container are disposed when the container shuts down.

#### When should you use each lifetime?

##### Transient

Use for lightweight services that don't need to preserve instance state between resolutions.

Example: a stateless formatter or simple message-building service.

##### Scoped

Use for services that should share state or resources within a single operation or web request.

Example: EF Core's `DbContext`, which is commonly registered as scoped by `AddDbContext`.

##### Singleton

Use when one shared instance is appropriate for the application's lifetime and the implementation is designed for concurrent access.

Example: an in-memory cache service or immutable application-wide metadata provider.

Avoid storing request-specific data in a singleton field, because multiple requests can access the same object.

#### Important interview questions

##### Q1. What is the difference between Scoped and Singleton?

Scoped services are reused within one scope and typically get a new instance for each HTTP request. Singleton services are shared across scopes for the lifetime of the root container.

##### Q2. Is a Singleton service thread-safe automatically?

No. The DI container reuses the instance, but it does not make the object's fields or operations thread-safe. You must design concurrent access appropriately.

##### Q3. Is `DbContext` usually Transient, Scoped, or Singleton?

Scoped. This normally provides one `DbContext` per HTTP request, enabling related operations in that request to share the same context. A `DbContext` is not thread-safe.

##### Q4. Does Transient mean a new instance every time a method is called?

No. It means a new instance each time the DI container resolves the service. An already-injected instance can be reused by its consumer.

##### Q5. Can a Singleton depend on a Scoped service?

Not directly in the normal constructor-injection pattern. Doing so can cause a scoped service to be retained beyond its intended lifetime. We'll cover this in the next topic.

#### Common interview traps

- A Singleton is one instance per DI container, not necessarily one instance across every process or server.
- A Scoped service isn't automatically one instance per user; its lifetime is tied to a DI scope.
- A Transient service can still hold mutable state. Transient registration doesn't guarantee statelessness.
- A Singleton is not automatically thread-safe.
- Registering a service with the correct lifetime is important, but it doesn't replace proper resource management.

### Lifetime Mismatches and Captive Dependencies

what happens when a Singleton depends on a Scoped service, especially when `DbContext` is involved.

#### What is a lifetime mismatch?

A lifetime mismatch happens when a service depends on another service whose lifetime is not appropriate for the way the dependent service is used.

The most common example is: A Singleton service directly depends on a Scoped service.

Let's understand why this causes problems.

 Example: Incorrect DI configuration

```
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddSingleton<OrderService>();
```

Suppose `OrderService` depends on `IOrderRepository`:

```
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

Here:

- OrderService is Singleton.
- OrderRepository is Scoped.
- The Singleton needs the Scoped repository when it is constructed.

The Singleton is supposed to live for the entire lifetime of the root container, but the repository is supposed to live only within a scope.

This creates a lifetime conflict.

#### What is a captive dependency?

A captive dependency occurs when a longer-lived service holds onto a dependency that was intended to have a shorter lifetime.

For example:

Singleton: `OrderService` - Lives for the application container's lifetime

Scoped: `OrderRepository` - Intended to live within one scope

The longer-lived service retains the shorter-lived dependency.

If the container allows this dependency to be resolved from the root provider, the scoped dependency may effectively be retained for the root container's lifetime rather than being recreated per request.

That defeats the intended scoped lifetime and can cause stale state, resource retention, or incorrect behavior across requests.

In the default ASP.NET Core development environment, scope validation typically detects this invalid dependency when the service provider is built or the service is resolved.

#### Why is Singleton + Scoped dangerous?

Consider OrderRepository using EF Core's DbContext.

If a Singleton retains that repository:

- Different HTTP requests may share the same repository and DbContext.
- DbContext is not thread-safe.
- One request's tracked entities or state may affect another request.
- Concurrent database operations can fail or behave incorrectly.

Interview rule: Do not directly inject a Scoped service into a Singleton through its constructor.

#### How do we fix a lifetime mismatch?

There are two common approaches.

##### Solution 1: Change the Singleton to Scoped

If `OrderService` is a request-oriented business service, make it Scoped too.

```
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<OrderService>();
```

Now both services live within the same request scope. This is usually the simplest solution for typical web application services.

##### Solution 2: Use `IServiceScopeFactory`

Sometimes a service genuinely needs to be Singleton, such as a background worker that periodically processes orders.

In that case, create a new scope whenever you need to use scoped dependencies.

```
public class OrderProcessor
{
    private readonly IServiceScopeFactory _scopeFactory;

    public OrderProcessor(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public async Task ProcessAsync()
    {
        await using var scope = _scopeFactory.CreateAsyncScope();

        var repository = scope.ServiceProvider
            .GetRequiredService<IOrderRepository>();

        // Use repository within this scope.
        // Perform the required async operations here.
    }
}
```

Register the services:

```
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddSingleton<OrderProcessor>();
```

Each call to `ProcessAsync` creates a scope. The scoped repository is resolved inside that scope, and the scope is disposed when processing finishes.

This example demonstrates the lifetime pattern; a real processor would also need to implement its actual database or business operation.

Important: Avoid injecting `IServiceProvider` into every class as a general workaround. Constructor injection should remain the normal approach. `IServiceScopeFactory` is useful when a longer-lived component legitimately needs to create short-lived scopes.

#### What is scope validation?

ASP.NET Core's DI container can validate service lifetimes to catch common mistakes.

For example, in development, it can detect a Singleton that directly depends on a Scoped service.

Scope validation can be configured through ServiceProviderOptions, including:

- ValidateScopes: detects inappropriate resolution of scoped services from the root provider.
- ValidateOnBuild: checks whether registered services can be constructed when the provider is built, where supported by the validation process.

ASP.NET Core commonly enables these validations in development. Do not rely on validation being enabled in every custom configuration or environment.

#### Important interview questions

##### Q1. What is a captive dependency?

A captive dependency occurs when a longer-lived service holds a shorter-lived dependency beyond its intended lifetime.

##### Q2. Can a Singleton depend on a Transient service?

Yes, the container generally permits this. However, if the Singleton stores the Transient instance in a field, that particular instance remains referenced for as long as the Singleton retains it. It is no longer effectively used as a fresh instance for each resolution by that Singleton.

##### Q3. Can a Scoped service depend on a Singleton?

Yes. A Singleton can safely be shared with Scoped services if it is designed for concurrent access and doesn't retain inappropriate request-specific state.

##### Q4. How do you use a Scoped service inside a background service?

Inject `IServiceScopeFactory`, create a scope for each unit of work, resolve the Scoped service from that scope, and dispose of the scope when finished.

##### Q5. Why should `DbContext` not be registered as Singleton?

`DbContext` is not thread-safe and maintains tracking state. Sharing one instance across requests can lead to concurrency errors and unwanted state sharing. It is normally registered as Scoped.

#### Common interview traps

- Saying that every lifetime mismatch causes an exception. Some invalid lifetime combinations are detected by validation, while others can be allowed but cause behavioral problems.
- Assuming a Transient dependency injected into a Singleton is recreated on every method call.
- Creating a scope but retaining its scoped services after disposing that scope.
- Using IServiceProvider everywhere instead of proper constructor injection.
- Assuming a Singleton is safe for concurrent requests merely because the DI container creates only one instance.

### Other Important DI Concepts

#### Multiple implementations of the same interface

Interview question: What if multiple classes implement the same interface? How does ASP.NET Core DI choose which one to inject?

Example:

```
public interface IMessageSender
{
    void Send();
}

public class EmailSender : IMessageSender
{
    public void Send() => Console.WriteLine("Email");
}

public class SmsSender : IMessageSender
{
    public void Send() => Console.WriteLine("SMS");
}
```

Register both:

```
builder.Services.AddScoped<IMessageSender, EmailSender>();
builder.Services.AddScoped<IMessageSender, SmsSender>();
```

What happens when a class requests a single IMessageSender?

- With the built-in Microsoft DI container, the last registered implementation is returned when resolving a single IMessageSender.
- If you request IEnumerable<IMessageSender></imessagesender>, both registered implementations are supplied in registration order.

Example:

```
public class NotificationService
{
    public NotificationService(
        IEnumerable<IMessageSender> senders)
    {
        foreach (var sender in senders)
        {
            sender.Send();
        }
    }
}
```

This allows the service to use both implementations.

Interview trap: Don't assume registering two implementations automatically makes the container choose one based on business requirements. If you need a particular implementation, use a suitable selection mechanism.

#### Circular dependencies

A circular dependency occurs when two or more services depend on each other, directly or indirectly.

For example:

```
public class OrderService
{
    public OrderService(PaymentService paymentService) { }
}

public class PaymentService
{
    public PaymentService(OrderService orderService) { }
}
```

The dependency chain becomes:

1. OrderService
2. PaymentService
3. OrderService again

Neither service can be constructed first.

The built-in DI container cannot resolve this constructor dependency cycle and throws an exception.

How to fix it:

- Review whether both services really need to depend on each other.
- Extract shared logic into a third service.
- Use events or another suitable design to decouple the services.

Interview trap: Switching one of the services from Scoped to Transient or Singleton does not solve the underlying circular dependency.

#### Optional dependencies

Sometimes a dependency is optional rather than essential.

For example, an application might send an email when an email service is available, but continue processing if email notifications are disabled.

You can use an optional constructor parameter with a default value:

```
public class NotificationService
{
    private readonly IMessageSender? _sender;

    public NotificationService(IMessageSender? sender = null)
    {
        _sender = sender;
    }
}
```

However, nullable annotations (?) alone do not make a dependency optional in the built-in DI container. The default parameter value and the container's constructor-resolution behavior matter.

For optional services, another approach is to explicitly check for a registered service using IServiceProvider.GetService< T >(). Use that sparingly because it hides the dependency compared with normal constructor injection.

Interview takeaway: Required dependencies should normally use Constructor Injection without a default value. Make dependencies optional only when the application genuinely supports their absence.

#### What is IEnumerable< T > in DI?

IEnumerable< T > allows a class to receive all registered implementations of a particular service type.

```
builder.Services.AddTransient<IMessageSender, EmailSender>();
builder.Services.AddTransient<IMessageSender, SmsSender>();
```

Then:

```
public class NotificationService
{
    private readonly IEnumerable<IMessageSender> _senders;

    public NotificationService(IEnumerable<IMessageSender> senders)
    {
        _senders = senders;
    }
}
```

The container supplies both implementations.

Typical uses include:

- Sending notifications through multiple channels.
- Running multiple validators.
- Applying a collection of registered processing strategies.

Interview trap: IEnumerable< T > doesn't mean the implementations are singletons. Each implementation follows its own registration lifetime.

#### What is the difference between `AddSingleton` overloads?

ASP.NET Core supports different ways to register a singleton.

```
builder.Services.AddSingleton<IMessageSender, EmailSender>();
```

The container creates and manages the implementation when required.

```
builder.Services.AddSingleton<IMessageSender>(
    new EmailSender());
```

You provide an existing instance, which the container reuses. The container does not create that instance itself.

```
builder.Services.AddSingleton<IMessageSender>(
    serviceProvider => new EmailSender());
```

You provide a factory that the container calls when it needs to create the singleton.

Interview trap: When you supply an existing instance, you are responsible for its creation and any cleanup that belongs to you. The container does not dispose of instances registered through the existing-instance overload.

#### How does DI handle `IDisposable` services?

The DI container manages disposal for services it creates and owns.

For example:

```
builder.Services.AddScoped<MyDisposableService>();
```

If MyDisposableService implements IDisposable, the container disposes of it when its scope ends.

For scoped services in a typical web application, this usually means disposal at the end of the HTTP request.

Important points:

- Don't manually dispose a container-managed service while it is still in use.
- Dispose scopes that you create manually.
- For services implementing IAsyncDisposable, use asynchronous scope disposal when appropriate.
- The container does not dispose an existing instance registered using the AddSingleton(instance) overload.

This connects directly to the `IDisposable`, `using`, and resource-management topics you already completed.

#### What is the Service Locator anti-pattern?

The Service Locator approach retrieves dependencies from `IServiceProvider` inside a class rather than declaring those dependencies in its constructor.

Example:

```
public class OrderService
{
    private readonly IServiceProvider _provider;

    public OrderService(IServiceProvider provider)
    {
        _provider = provider;
    }

    public void PlaceOrder()
    {
        var sender =
            _provider.GetRequiredService<IMessageSender>();

        sender.Send();
    }
}
```

This works technically, but the class's actual dependency is hidden inside the method.

A more explicit design is:

```
public class OrderService
{
    private readonly IMessageSender _sender;

    public OrderService(IMessageSender sender)
    {
        _sender = sender;
    }
}
```

Interview answer: Constructor Injection is generally preferred because dependencies are explicit and easier to test. `IServiceProvider` is appropriate in specific infrastructure scenarios, but should not become the default way to obtain dependencies.

#### Important DI interview questions

What is DI?	Supplying dependencies from outside a class.
What is IoC?	A principle that delegates control to an external component or framework.
What is Constructor Injection?	Supplying dependencies through constructor parameters.
What are the three lifetimes?	Transient, Scoped, Singleton.
What is a captive dependency?	A longer-lived service retaining a shorter-lived dependency beyond its intended lifetime.
What happens with multiple registrations?	Single-service resolution uses the last registration; IEnumerable< T > supplies all registrations.
What is a circular dependency?	A dependency cycle that prevents normal construction.
What is Service Locator?	Retrieving dependencies from a provider rather than declaring them explicitly.
Does DI make Singleton services thread-safe?	No. Thread safety must be designed into the service.
Who disposes container-created Scoped services?	The DI container, when their scope ends.

### Final Revision

#### Tricky interview questions

 Q1. What happens if we register the same service multiple times?

```
builder.Services.AddScoped<IMessageSender, EmailSender>();
builder.Services.AddScoped<IMessageSender, SmsSender>();
```

With ASP.NET Core's built-in DI container:

- Requesting a single IMessageSender returns the last registered implementation, SmsSender.
- Requesting IEnumerable< IMessageSender > returns both implementations, in registration order.

##### Q2. Can we inject a Scoped service into a Singleton?

Not directly through normal constructor injection. This creates a lifetime mismatch.

Solution: Change the consuming service to Scoped if appropriate, or use `IServiceScopeFactory` to create a scope when a Singleton genuinely needs a Scoped dependency.

##### Q3. Does the DI container create every registered service at startup?

No. Most services are created when first resolved. Registration describes how to provide the service; it does not generally instantiate it immediately.

##### Q4. What happens if a dependency is not registered?

If a required constructor dependency cannot be resolved, the container throws an exception, typically an `InvalidOperationException`.

##### Q5. Does registering a service as Singleton guarantee thread safety?

No. Multiple requests or threads can access the same Singleton instance concurrently. Its implementation must handle concurrent access safely.

##### Q6. What is the difference between DI and the Service Locator pattern?

- DI: Dependencies are supplied explicitly, usually through constructors.
- Service Locator: A class retrieves dependencies from a service provider when needed.

Constructor Injection is generally preferred because dependencies are visible and easier to test.

#### One final concept: Open generic registration

This is useful when you work with generic repositories or generic services.

Suppose you have:

```
public interface IRepository<T>
{
    T? GetById(int id);
}

public class Repository<T> : IRepository<T>
{
    public T? GetById(int id)
    {
        return default;
    }
}
```

You can register the generic mapping once:

```
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

Now the DI container can resolve services such as IRepository< Product > and IRepository< Customer > using the corresponding closed generic implementation types.

Interview takeaway: Open generic registration lets you register one generic service mapping instead of registering each concrete type separately.
