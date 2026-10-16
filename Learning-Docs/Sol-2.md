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
