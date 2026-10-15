## Exception Handling:

### Basics:

#### What is an Exception?

An **exception** is an unexpected runtime event that interrupts the normal flow of program execution.

For example:

```
inta=10;
intb=0;

intresult=a/b;
```

This causes:

```
System.DivideByZeroException
```

Instead of allowing the application to crash, we can handle the exception using  **exception handling** .

#### Why do we need Exception Handling?

In a backend application, many things can go wrong:

- Database connection failure
- Invalid user input
- File not found
- Network/API failure
- Null reference
- Invalid conversion
- Timeout
- Unauthorized access
- Unexpected application errors

Exception handling allows us to:

1. Prevent unexpected application termination
2. Handle errors gracefully
3. Log errors
4. Return meaningful responses
5. Perform cleanup
6. Maintain application stability

Interview answer: **Exception handling is a mechanism in C# used to handle runtime errors gracefully without terminating the normal execution of the application. It uses constructs such as** `try`**,** `catch`**,** `finally`**, and** `throw`**.**

#### `try`, `catch`, `finally`

These are the core keywords.

##### `try`

Contains code that might throw an exception.

##### `catch`

Handles the exception.

##### `finally`

Contains cleanup code that should generally execute regardless of whether an exception occurred.

Basic structure:

```
try
{
    // Risky code
}
catch (Exceptionex)
{
    // Handle exception
}
finally
{
    // Cleanup
}
```

Conceptually:

```
          try
           │
           ▼
    Exception occurs?
       /          \
     Yes           No
      │             │
      ▼             │
    catch           │
      │             │
      └──────┬──────┘
             ▼
          finally
             │
             ▼
         Continue
```

#### Multiple `catch` blocks

You can have multiple catch blocks for different exception types.

```
try
{
    // code
}
catch (DivideByZeroExceptionex)
{
    // handle division error
}
catch (FormatExceptionex)
{
    // handle format error
}
catch (Exceptionex)
{
    // handle other exceptions
}
```

Important interview point

The  **most specific exception should come first** , and the more general exception should come later.

Correct:

```
catch (DivideByZeroExceptionex)
{
}
catch (Exceptionex)
{
}
```

Incorrect:

```
catch (Exceptionex)
{
}
catch (DivideByZeroExceptionex)
{
}
```

Why? Because `Exception` can catch `DivideByZeroException` already, making the later catch unreachable.

#### Exception hierarchy

This is important for interviews.

Most exceptions ultimately derive from:

```
System.Object
    ↓
System.Exception
    ↓
System.SystemException
    ↓
Specific Exception
```

For example:

```
Exception
   ↓
SystemException
   ↓
ArithmeticException
   ↓
DivideByZeroException
```

Another example:

```
Exception
   ↓
SystemException
   ↓
NullReferenceException
```

Therefore:

```
catch (Exceptionex)
```

can catch many different exception types.

#### `throw` keyword

`throw` is used to explicitly throw an exception.

```
thrownewException("Something went wrong");
```

More commonly, you should throw a meaningful specific exception:

```
thrownewArgumentException("Invalid customer ID");
```

or:

```
thrownewInvalidOperationException("Customer cannot be processed.");
```

Interview question

**What is the difference between** `throw` **and** `throw ex`**?**

This is a  **very important interview question** .

Suppose:

```
try
{
    // code
}
catch (Exceptionex)
{
    throw;
}
```

`throw;`  **preserves the original stack trace** .

But:

```
catch (Exceptionex)
{
    throwex;
}
```

resets the stack-trace information from the point where it is rethrown, making debugging harder.

Interview answer

> Use `throw;` when rethrowing an existing exception because it preserves the original stack trace. `throw ex;` resets the stack trace and should generally be avoided.

#### `finally`

`finally` is typically used for cleanup.

For example:

```
try
{
    // Database/file operation
}
catch (Exceptionex)
{
    // Handle error
}
finally
{
    // Cleanup
}
```

Typical cleanup includes:

- Closing resources
- Releasing unmanaged resources
- Cleaning temporary state

However, in modern C#, resources are often better handled using:

```
using
```

or:

```
usingvarresource= ...;
```

which we'll cover later when we reach  **IDisposable / resource management** .

#### Can `finally` execute if there is a `return`?

**Yes.**

For example:

```
try
{
    return;
}
finally
{
    // Usually executes
}
```

The `finally` block normally executes before control leaves the method.

Interview question

**Does finally always execute?**

The safe interview answer is:

> `finally` normally executes regardless of whether an exception occurs or a return statement is encountered. However, there are abnormal situations such as process termination or environment failure where it may not execute.

#### Exception handling best practices

For interviews, remember these:

##### Don't catch everything unnecessarily

Avoid:

```
try
{
    // everything
}
catch (Exception)
{
}
```

Especially an empty catch:

```
catch
{
}
```

This hides problems.

##### Catch exceptions you can actually handle

For example:

```
catch (FileNotFoundExceptionex)
{
    // Provide fallback behavior
}
```

##### Don't use exceptions for normal program flow

Exceptions should represent  **exceptional situations** , not ordinary business logic.

##### Preserve stack trace

Use:

```
throw;
```

instead of:

```
throwex;
```

##### Log useful information

In real .NET applications, exceptions should generally be logged with appropriate context.

#### Can we have `try` without `catch`?

**Yes.** In C#, a `try` block can exist without a `catch` block if it has a `finally` block.

For example:

```
try
{
    Console.WriteLine("Executing try block");
}
finally
{
    Console.WriteLine("Cleanup");
}
```

Here, the `finally` block executes after the `try` block completes.

**Interview point:** A `try` block cannot stand alone. It must be followed by at least one `catch` block, a `finally` block, or both.

#### Can we have `try` without `catch` and with only `finally`?

**Yes.** This is a valid pattern, commonly used when cleanup must happen even if an exception occurs.

```
try
{
    Console.WriteLine("Opening resource");

    thrownewException("Something went wrong");
}
finally
{
    Console.WriteLine("Performing cleanup");
}
```

The `finally` block executes, and the exception continues propagating to the caller because there is no `catch` block to handle it.

**Remember:** `finally` performs cleanup; it does not handle or suppress the exception.

#### What happens if an exception isn't handled?

If an exception isn't handled in the current method, it propagates up the call stack to its caller.

Consider this flow:

**Method C**

Exception occurs

**Method B**

No matching handler → propagates

**Method A**

No matching handler → propagates

**Application/runtime boundary**

May terminate the affected execution or process

If no suitable handler is found, the exception becomes unhandled. Depending on the application type and execution context, this can terminate a thread, request, or process.

In ASP.NET Core, exception-handling middleware can catch unhandled exceptions that propagate through the request pipeline and return an appropriate HTTP response.

**Interview answer:** An unhandled exception propagates up the call stack. If no handler catches it, it may terminate the affected execution or application, depending on the context.

#### Should we catch `Exception` directly? When?

**Yes, but only when there's a good reason to handle a broad range of exceptions.**

Prefer catching specific exception types when you know how to recover.

For example:

```
try
{
    // Read a file
}
catch (FileNotFoundExceptionex)
{
    // Handle a missing file
}
```

A broad catch can be appropriate at an application boundary, such as ASP.NET Core exception-handling middleware, where you log unexpected errors and return a generic error response.

```
catch (Exceptionex)
{
    logger.LogError(ex, "Unexpected error occurred");
    throw;
}
```

This example logs the exception and rethrows it, so it is **not** swallowed.

Avoid empty catches or catching `Exception` merely to hide failures. Also, don't assume every exception is recoverable.

**Interview answer:** Catch specific exceptions whenever possible. Catch `Exception` at appropriate boundaries for logging, consistent error handling, or controlled cleanup, and avoid silently swallowing exceptions.

#### When should you create a custom exception?

Create a custom exception when your application needs to represent a meaningful, specific error that built-in exceptions don't adequately describe.

For example, in an e-commerce application, you might define an `InsufficientStockException` when an order requests more items than are available.

Typical reasons include:

- Representing a specific business or domain failure.
- Allowing callers to handle a particular failure differently.
- Making error handling clearer in a large application.

Example:

```
publicclassInsufficientStockException : Exception
{
    publicInsufficientStockException(stringmessage)
        : base(message)
    {
    }
}
```

You can then throw it when the relevant business rule fails.

However, don't create custom exceptions for every error. Use built-in exceptions when they already describe the problem well.

**Interview answer:** Create a custom exception when a distinct application or domain error needs its own type so that callers can identify and handle it specifically.


| Exception                     | When it occurs                                 |
| ------------------------------- | ------------------------------------------------ |
| `NullReferenceException`      | Accessing a member through a null reference    |
| `ArgumentNullException`       | A method receives`null`where it isn't allowed  |
| `ArgumentException`           | An argument is invalid                         |
| `ArgumentOutOfRangeException` | An argument is outside the permitted range     |
| `InvalidOperationException`   | An operation isn't valid for the current state |
| `FormatException`             | A string or value has an invalid format        |
| `DivideByZeroException`       | Integer division by zero                       |
| `IndexOutOfRangeException`    | An array index is outside its bounds           |
| `KeyNotFoundException`        | A requested key isn't present in a dictionary  |
| `IOException`                 | An input/output operation fails                |
| `UnauthorizedAccessException` | Access to a resource is denied                 |
| `TimeoutException`            | An operation times out                         |

#### How do you create a custom exception?

A custom exception generally derives from `<span data-d-text-flow="word" data-d-stream-word="">Exception</span>` and provides constructors that pass messages and inner exceptions to the base class.

```
publicclassInsufficientStockException : Exception
{
    publicInsufficientStockException()
    {
    }

    publicInsufficientStockException(stringmessage)
        : base(message)
    {
    }

    publicInsufficientStockException(
        stringmessage,
        ExceptioninnerException)
        : base(message, innerException)
    {
    }
}
```

Now your application can throw this exception when an order requests more stock than is available.

```
thrownewInsufficientStockException(
    "Requested quantity exceeds available stock.");
```

The main advantage is that callers can catch `InsufficientStockException` specifically instead of catching every exception.

Interview question: Why derive a custom exception from `Exception`?

Because it integrates with .NET's exception-handling mechanism. It can be thrown, caught, propagated, and logged like other exceptions.

#### What is an inner exception?

An inner exception is the original exception stored inside another exception.

Suppose your application calls a database service and the operation fails. You want to add context explaining what the application was trying to do without losing the original failure.

```
try
{
    // Database operation
}
catch (Exceptionex)
{
    thrownewInvalidOperationException(
        "Unable to process the order.",
        ex);
}
```

Here:

- `InvalidOperationException` provides additional application context.
- `ex` is the original exception.
- `InnerException` lets you inspect the original failure.

#### What is the difference between `Message` and `InnerException`?


| Property         | Purpose                                                            |
| ------------------ | -------------------------------------------------------------------- |
| `Message`        | Describes the current exception                                    |
| `InnerException` | Provides the exception that caused or explains the current failure |
| `StackTrace`     | Shows the call path associated with the exception                  |

These details help developers diagnose problems in production.

Also remember that exception messages and stack traces may contain sensitive information. Log them securely, and avoid exposing internal details in public API responses.

### Exception Filters (`when`)

Exception filters allow you to handle an exception **only when a specified condition is true**.

They use the `when` keyword in a `catch` block.

#### Why do we need exception filters?

Normally, we catch an exception based on its type.

For example, you might catch `ArgumentException` whenever it occurs. But sometimes, you want to handle that exception only under a particular condition.

Exception filters let you specify that condition directly in the `catch` block.

Syntax

```
try
{
    // Code that might throw an exception
}
catch (ExceptionTypeex) when (condition)
{
    // Handle the exception if the condition is true
}
```

The catch block executes only if:

1. The exception matches the specified type.
2. The `when` condition evaluates to `true`.

#### Example: Handling exceptions conditionally

Imagine a backend application that processes customer requests. You want a particular handler to run only when the exception message contains a specific value.

```
try
{
    // Some operation
}
catch (InvalidOperationExceptionex)
    when (ex.Message.Contains("Order"))
{
    // Handle order-related failure
}
catch (InvalidOperationExceptionex)
{
    // Handle other InvalidOperationException cases
}
```

Here:

- If the exception message contains `"Order"`, the first handler is selected.
- Otherwise, the second handler can handle the exception.

In real applications, it's generally better to use meaningful exception types or structured error information rather than relying on exception message text.

#### Can we use multiple conditions with `when`?

Yes. You can combine conditions using logical operators such as `&&` and `||`.

```
catch (Exceptionex)
    when (exisTimeoutException||exisIOException)
{
    // Handle selected exceptions
}
```

This handler applies to either a timeout or an I/O exception.

Note that a `when` filter doesn't change the exception's type or suppress the exception by itself. If the filter evaluates to `false`, the runtime continues looking for another matching handler.

#### What happens if a `when` condition is false?

Consider:

```
try
{
    // Operation
}
catch (InvalidOperationExceptionex)
    when (false)
{
    // This block won't execute
}
```

Because the filter is false, this handler is skipped. The exception continues through normal exception-handler selection and propagation.

**Interview point:** A false filter doesn't mean the exception has been handled. Another handler must catch it; otherwise, it propagates to the caller.

#### Why use exception filters instead of an `if` inside `catch`?

Compare the approaches:

\*\*Using \*\*`if`:

```
catch (InvalidOperationExceptionex)
{
    if (ShouldHandle(ex))
    {
        // Handle the exception
    }
    else
    {
        throw;
    }
}
```

\*\*Using \*\*`when`:

```
catch (InvalidOperationExceptionex)
    when (ShouldHandle(ex))
{
    // Handle the exception
}
```

The filter version is more concise and allows the runtime to decide whether the handler matches before entering the catch block. This is especially useful when you want exceptions that don't meet the condition to continue propagating naturally.

Avoid putting complicated logic or side effects inside a filter. Keep the condition simple and predictable.

### Exception Propagation and Stack Traces

#### What is exception propagation?

Exception propagation is the process by which an exception travels up the call stack until a matching `<span data-d-text-flow="word" data-d-stream-word="">catch</span>` block handles it.

Consider this flow:

Controller -&gt; Calls the service

Service -&gt; Calls the repository

Repository -&gt; Exception occurs here

Exception propagates upward until a matching handler is found. For example, if a database operation throws an exception inside the repository and neither the repository nor service catches it, the exception propagates to the controller or to a higher-level exception handler.

If no suitable handler exists, it continues propagating outward. Interview answer: Exception propagation is the process of passing an unhandled exception from the method where it occurred to its callers until an appropriate handler catches it.

#### What is a stack trace?

A stack trace shows the sequence of method calls associated with an exception, helping developers identify where the error occurred and how execution reached that point.

For example, a simplified stack trace might look like this:

```
System.InvalidOperationException: Order processing failed
   at MyApp.Repository.SaveOrder()
   at MyApp.Service.ProcessOrder()
   at MyApp.Controllers.OrderController.Create()
```

This tells us that:

- `<span data-d-text-flow="word" data-d-stream-word="">SaveOrder()</span>` is where the exception originated or was thrown.
- `<span data-d-text-flow="word" data-d-stream-word="">ProcessOrder()</span>` called the repository method.
- `<span data-d-text-flow="word" data-d-stream-word="">Create()</span>` called the service method.

The exact stack trace depends on where the exception was thrown and rethrown.

Why is it useful?

- Debugging production errors
- Identifying the source of a failure
- Understanding the method call sequence
- Investigating failures across application layers

In C#, you can inspect the stack trace using:

```
catch (Exceptionex)
{
    Console.WriteLine(ex.StackTrace);
}
```

In real applications, prefer structured logging rather than writing directly to the console.

#### Difference between `throw` and `throw ex`

##### `throw;` — rethrow the current exception

```
try
{
    ProcessOrder();
}
catch (Exceptionex)
{
    logger.LogError(ex, "Order processing failed");
    throw;
}
```

`throw;` rethrows the current exception while preserving its original stack-trace information.

##### `throw ex;` — rethrow using the exception variable

```
try
{
    ProcessOrder();
}
catch (Exceptionex)
{
    throwex;
}
```

`throw ex;` resets the stack trace to the current rethrow location, obscuring the original throw location in the resulting stack trace.


| `throw;` | `throw ex;` |
| ---------- | ------------- |


| Preserves the original stack trace | Resets the stack trace at the rethrow location |
| ------------------------------------ | ------------------------------------------------ |


| Preferred for rethrowing in a`catch`block | Generally avoid |
| ------------------------------------------- | ----------------- |


| Helps preserve debugging context | Can make the original failure harder to locate |
| ---------------------------------- | ------------------------------------------------ |

**Interview answer:** Use `throw;` when rethrowing the current exception from a catch block because it preserves the original stack trace. Avoid `throw ex;` when you want to preserve the original throw location.

#### What is the difference between `throw;` and `throw new Exception()`?

This is another useful interview distinction.

- `throw;` rethrows the current exception.
- `throw new Exception("...")` creates and throws a new exception, with a new throw location and a new exception type.
- `throw new Exception("...", ex)` creates a new exception while retaining the original as its inner exception.

### Global Exception Handling in ASP.NET Core

#### What is global exception handling?

Global exception handling is a centralized way to catch unhandled exceptions across an application instead of writing separate `try-catch` blocks in every controller or service.

Imagine your ASP.NET Core application has this structure:

**Client**

Sends HTTP request

**Exception-handling middleware**

Centralized error handling

**Controller → Service → Repository**

Application business logic and data access

If an unhandled exception propagates back through the middleware, it can be logged and converted into an appropriate HTTP error response.

For example, suppose a database operation fails inside your repository. The exception travels up through the service and controller. If no lower-level handler catches it, the global exception handler can process it.

Without global handling, developers may end up duplicating error-handling logic across multiple controllers.

#### Why do we need global exception handling?

In a production Web API, it helps you:

- **Centralize error handling:** avoid repetitive `try-catch` blocks.
- **Log unexpected failures:** capture diagnostic information for troubleshooting.
- **Return consistent responses:** clients receive a predictable error format.
- **Protect sensitive information:** avoid exposing stack traces or database details.
- **Separate concerns:** keep business logic focused on application behavior.

Interview answer: Global exception handling is a centralized mechanism for handling unhandled exceptions across an application. In ASP.NET Core, exception-handling middleware can log errors and return consistent HTTP responses without duplicating exception-handling logic in every controller.

#### `UseExceptionHandler()` middleware

ASP.NET Core provides built-in exception-handling middleware through `UseExceptionHandler()`.

In a typical application, it is configured in `Program.cs`.

```
varbuilder=WebApplication.CreateBuilder(args);

varapp=builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
}

app.Map("/error", errorApp=>errorApp.Run(asynccontext=>
{
    context.Response.StatusCode=500;
    awaitcontext.Response.WriteAsync(
        "An unexpected error occurred.");
}));

app.MapControllers();

app.Run();
```

This is an illustrative configuration: the `/error` path is re-executed by the exception handler, and the mapped endpoint returns a generic error response. In a production API, you would typically return a structured JSON error response instead.

How it works

1. A request enters the middleware pipeline.
2. A controller or service throws an unhandled exception.
3. The exception propagates back to the exception-handling middleware.
4. The middleware invokes the configured error handler.
5. The client receives a controlled error response.

**Important:** Register exception-handling middleware early enough in the pipeline to catch exceptions from the middleware that follows it.

#### Development vs Production


| Development                                            | Production                                       |
| -------------------------------------------------------- | -------------------------------------------------- |
| Detailed diagnostic information helps developers debug | Generic error responses protect internal details |
| Developer Exception Page is useful                     | `UseExceptionHandler()`is commonly used          |
| Stack traces may be shown locally                      | Stack traces should not be exposed to clients    |

ASP.NET Core templates commonly enable the Developer Exception Page in Development and use exception-handling middleware in non-development environments.

#### Which HTTP status code should you return?

Global exception handling should return a suitable status code rather than returning `200 OK` for every outcome.


| Status code                 | Meaning                                                 | Example                                  |
| ----------------------------- | --------------------------------------------------------- | ------------------------------------------ |
| `400 Bad Request`           | The request is invalid                                  | Invalid request data                     |
| `401 Unauthorized`          | Authentication is required or failed                    | Missing or invalid authentication        |
| `403 Forbidden`             | The caller isn't allowed to access the resource         | Insufficient permissions                 |
| `404 Not Found`             | The requested resource doesn't exist                    | Customer not found                       |
| `409 Conflict`              | The request conflicts with current resource state       | Duplicate resource or conflicting update |
| `500 Internal Server Error` | An unexpected server failure occurred                   | Unhandled application exception          |
| `503 Service Unavailable`   | The service is temporarily unable to handle the request | Temporary service unavailability         |

These are common mappings, not universal rules. For example, validation errors are often returned as `400`, and a missing resource may be handled directly in the application rather than thrown as an exception.

#### What is `ProblemDetails`?

`ProblemDetails` is a standardized format for describing HTTP API errors. ASP.NET Core supports it through its built-in error-handling features.

A response might look like this:

```
{
  "title": "An unexpected error occurred.",
  "status": 500
}
```

This gives the client a predictable response structure without exposing implementation details.

`ProblemDetails` is a standardized format for representing HTTP API errors, based on RFC 9457.

It can include fields such as:

- `<span data-d-text-flow="word" data-d-stream-word="">type</span>` — URI identifying the problem type.
- `<span data-d-text-flow="word" data-d-stream-word="">title</span>` — short summary.
- `<span data-d-text-flow="word" data-d-stream-word="">status</span>` — HTTP status code.
- `<span data-d-text-flow="word" data-d-stream-word="">detail</span>` — explanation of the specific occurrence.
- `<span data-d-text-flow="word" data-d-stream-word="">instance</span>` — URI identifying the particular occurrence.

ASP.NET Core supports returning `<span data-d-text-flow="word" data-d-stream-word="">ProblemDetails</span>` responses to make API error handling more consistent.

#### Why is middleware order important?

Answer: ASP.NET Core middleware executes in the order in which it is registered for requests. Exception-handling middleware must generally be placed before components whose exceptions it should handle.

For example:

```
app.UseExceptionHandler("/error");

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

An exception thrown by downstream middleware or a controller can propagate back to the exception handler. An exception thrown by middleware registered before it won't normally be caught by it.

### Custom Middleware vs `IExceptionHandler`

#### What are the three approaches?

ASP.NET Core offers several ways to handle exceptions centrally.

1. UseExceptionHandler()

Built-in exception-handling middleware. Often the simplest choice for standard application-wide error handling.

2. Custom exception middleware

Your own middleware class that catches exceptions and creates responses according to application-specific requirements.

3. IExceptionHandler

A dedicated exception-handler abstraction introduced in ASP.NET Core 8, useful for implementing and registering centralized exception-handling logic.

#### What is IExceptionHandler?

IExceptionHandler is an interface in ASP.NET Core that lets you implement centralized exception-handling logic.

It was introduced in ASP.NET Core 8.0.

Its main method is:

ValueTask< bool > TryHandleAsync(
HttpContext httpContext,
Exception exception,
CancellationToken cancellationToken);

The handler returns:

true — the exception has been handled.

false — this handler hasn't handled the exception.

#### How is it configured?

In Program.cs, register the handler and exception-handling middleware:

builder.Services.AddExceptionHandler< MyExceptionHandlerClass >();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();

app.MapControllers();

app.Run();

Here, MyExceptionHandlerClass is your implementation of IExceptionHandler. The registration enables ASP.NET Core to use it through the exception-handling middleware.

AddProblemDetails() registers services for generating standardized error responses. The actual response depends on the exception handler and configured error-handling services.

#### Which approach should you choose?


| Approach                | Best suited for                                                        |
| ------------------------- | ------------------------------------------------------------------------ |
| `UseExceptionHandler()` | Standard centralized handling with minimal custom infrastructure       |
| Custom middleware       | Special handling that requires control over the entire middleware flow |
| `IExceptionHandler`     | Dedicated, testable exception-handling logic in modern ASP.NET Core    |

**Interview answer:** For a modern ASP.NET Core application, `IExceptionHandler` provides a clean abstraction for centralized exception handling. Custom middleware is useful when you need broader control over request-pipeline behavior. `UseExceptionHandler()` is the built-in middleware mechanism that can host centralized exception-handling logic.

One important distinction: these approaches aren't necessarily alternatives. `IExceptionHandler` is commonly used together with `UseExceptionHandler()`.

### Exception Logging with ILogger

#### What is logging?

Logging is the process of recording information about an application's execution, including errors, warnings, and important events.

In .NET, `ILogger` is the standard logging abstraction used by application code. For example, suppose your Web API fails while saving an order. You want to record the exception details for troubleshooting rather than simply returning an error response.

 Common logging levels


| Level         | Purpose                                                            | Example                                   |
| --------------- | -------------------------------------------------------------------- | ------------------------------------------- |
| `Trace`       | Very detailed diagnostic information                               | Detailed execution steps                  |
| `Debug`       | Information useful during debugging                                | Intermediate processing details           |
| `Information` | Normal application events                                          | Order successfully created                |
| `Warning`     | An unexpected situation that doesn't necessarily prevent operation | Retry after a temporary failure           |
| `Error`       | An operation failed                                                | Failed to save an order                   |
| `Critical`    | A severe failure affecting application operation                   | A critical service component cannot start |

#### How do you use ILogger?

ASP.NET Core provides dependency injection for ILogger< T >. The generic type identifies the category, usually the class where the logger is used.
For example, in a service:
public class OrderService
{
    private readonly ILogger< OrderService > _logger;

    public OrderService(ILogger< OrderService > logger)
    {
        _logger = logger;
    }
}

You can log an error like this:

```
_logger.LogError(
    exception,
    "Failed to process order {OrderId}",
    orderId);
```

Notice that the exception is passed as an argument. This lets the logging provider capture useful exception details, including its stack trace.

The `{OrderId}` placeholder is structured logging: the order ID is recorded as a property rather than merely concatenated into a string.

#### Should you log an exception and rethrow it?

Sometimes, but be careful not to log the same exception redundantly at every application layer.

For example:

```
try
{
    await repository.SaveOrderAsync(order);
}
catch (Exception ex)
{
    _logger.LogError(ex, "Failed to save order");
    throw;
}
```

This preserves the original stack trace, but if global exception-handling middleware also logs the same exception, you may end up with duplicate error logs.
A good general approach is:

- Handle an exception where you can genuinely recover from it.
- Add contextual logging when it provides useful information.
- Let centralized exception handling log unexpected failures that remain unhandled.
- Avoid logging and swallowing exceptions silently.

#### How does logging work with global exception handling?

A common production flow looks like this:

**Controller / Service / Repository [Unexpected exception occurs]**

**Global exception handler [Logs exception details]**

**Application logs For diagnosis OR API response for safe error details**

The client receives a controlled HTTP error response, while authorized developers can investigate the detailed logs.

Remember: don't expose stack traces, connection strings, passwords, or other sensitive details in API responses or logs.

#### What is the difference between logging and exception handling?


| Exception handling                                    | Logging                                        |
| ------------------------------------------------------- | ------------------------------------------------ |
| Determines how an exception is handled                | Records information about an event or failure  |
| May recover, transform, or propagate an exception     | Helps developers diagnose and monitor problems |
| Uses`try`,`catch`, middleware, and exception handlers | Uses`ILogger`and configured logging providers  |

Interview answer: Exception handling controls the application's response to a failure, whereas logging records diagnostic information about that failure. They complement each other but serve different purposes.

### Business Exceptions vs Unexpected Exceptions

#### What are business exceptions?

Business exceptions represent situations where an operation cannot proceed because of a business rule or domain condition.
Examples in an e-commerce application:

- A customer tries to order more items than are available.
- A customer tries to cancel an already-shipped order.
- A user attempts to withdraw more money than their available balance.
  These situations may be expected outcomes of business operations, even though the requested operation cannot be completed.
  For example, insufficient stock doesn't necessarily mean the application is broken. It means the order cannot be fulfilled under the current business rules.

#### What are unexpected technical exceptions?

Unexpected exceptions arise from technical failures or conditions the application isn't prepared to handle.
Examples:

- A database connection fails unexpectedly.
- An external service times out.
- A programming error causes a NullReferenceException.
- An unexpected file or I/O operation fails.
  These errors often require investigation, monitoring, or recovery mechanisms.

#### Compare the two


| Businessexception                              | Unexpectedtechnicalexception                        |
| ------------------------------------------------ | ----------------------------------------------------- |
| Related to a business rule or domain condition | Related to an unexpected technical failure          |
| May be a normal outcome of a request           | Usually indicates a failure requiring investigation |
| Often maps to a meaningful client response     | Often maps to a generic server error                |
| Example: insufficient stock                    | Example: database unavailable                       |

Important: A business exception is not automatically a custom exception. You can represent business failures with a result type, a domain-specific exception, or another suitable design. Choose exceptions when they fit the flow and semantics of your application.

#### How should a Web API handle them?

Imagine an order API.

Expected business outcome: Insufficient stock

The application identifies the condition and returns a meaningful response, such as `<span>409 </span><span>Conflict</span>`, if that matches the API's contract.

Unexpected technical failure: Database connection failure

The application logs diagnostic details and returns a safe server error response, typically `<span>500 </span><span>Internal </span><span>Server </span><span>Error</span>` unless a more appropriate status applies.

The precise status code depends on the API contract and the nature of the failure. Don't expose database errors or stack traces to the client.

#### Should every business failure throw an exception?

No. This is a valuable interview point.

If a failure is an expected result of normal business processing, returning a result object may be clearer than throwing an exception.

For example, a service might return a result containing:

- Whether the operation succeeded
- A business error code
- A user-safe message

Exceptions are more appropriate when using exception-based control flow makes sense, such as when a domain operation must abort or when the failure needs to propagate to a centralized handler.

Interview answer:

> Business exceptions represent domain or business-rule failures, whereas unexpected exceptions usually indicate technical failures or unanticipated conditions. Expected business outcomes can often be handled with result objects, while unexpected exceptions are commonly logged and handled centrally.
>

### Exception Handling Best Practices:

##### Catch exceptions only when you can handle them

Don't add `try-catch` blocks to every method automatically. Catch an exception when you can recover, translate it meaningfully, or add useful handling.

##### Never silently swallow exceptions

Avoid empty catch blocks:

```
catch (Exception)
{
    // Nothing happens
}
```

This hides failures and makes troubleshooting difficult.

##### Preserve the original stack trace

Use `throw;` when rethrowing the current exception from a catch block.

Avoid `throw ex;` when you need to preserve the original throw location in the stack trace.

##### Use global exception handling

In ASP.NET Core, use exception-handling middleware or a registered `IExceptionHandler` to centralize unexpected error handling.

##### Return safe and meaningful HTTP responses

Use appropriate status codes and consistent error formats, such as `ProblemDetails`. Never return internal stack traces or sensitive technical information to clients in production.

##### Log exceptions appropriately

Use `ILogger` and structured logging. Avoid redundant logs at every layer, and protect sensitive information in logs

##### Don't use exceptions for ordinary control flow

Prefer normal conditions or result types for expected outcomes, such as checking whether an item exists or whether a validation rule passes.


## `IDisposable`, `using`, and Resource Management

### Why do we need resource management?

 When an application runs, it uses resources such as:

- Memory for objects.
- Database connections.
- Files and file streams.
- Network connections.
- Operating system handles.

.NET's Garbage Collector (GC) automatically manages managed memory, but it does not guarantee that external resources are released immediately when an object is no longer needed.

For example, a file may remain open longer than necessary if its stream is not disposed.

### What is `IDisposable`?

`IDisposable` is a built-in .NET interface that provides a standard way to release resources explicitly.

It contains one method:

```
public interface IDisposable
{
    void Dispose();
}
```

A class implements `IDisposable` when it needs to clean up resources or delegate cleanup to other disposable objects.

#### Practical example

```
using System;
using System.IO;

public class FileService : IDisposable
{
    private readonly FileStream _stream;

    public FileService(string path)
    {
        _stream = new FileStream(path, FileMode.Open);
    }

    public void ReadFile()
    {
        // Read from the stream
    }

    public void Dispose()
    {
        _stream.Dispose();
    }
}
```

Here, `FileService` owns a `FileStream`. Calling `FileService.Dispose()` releases the stream.

Important distinction: `Dispose()` is about deterministic cleanup. The GC is responsible for reclaiming managed memory; `Dispose()` is not a command to run the GC.

### What is the `using` statement?

The `using` statement ensures that an object implementing `IDisposable` is disposed when execution leaves the scope, including when an exception occurs.

#### Without `using`:

```
var stream = new FileStream("data.txt", FileMode.Open);

try
{
    // Work with the file
}
finally
{
    stream.Dispose();
}
```

#### With `using`:

```
using (var stream =
       new FileStream("data.txt", FileMode.Open))
{
    // Work with the file
}
```

The `using` statement provides automatic cleanup without requiring you to write the `finally` block yourself.

#### What if an exception occurs?

```
using (var stream =
       new FileStream("data.txt", FileMode.Open))
{
    throw new Exception("Something went wrong");
}

// Dispose() is still called as control leaves the scope.
```

Even if an exception occurs inside the block, the compiler-generated cleanup ensures that `Dispose()` is called.

#### What is a using declaration?

Modern C# also supports a using declaration, introduced in C# 8.0.

```
using var stream =
    new FileStream("data.txt", FileMode.Open);

// Work with the stream
```

The object is disposed at the end of the enclosing scope, rather than at the end of an explicitly written `using` block.

#### Compare:


| `using`statement                      | `using`declaration                     |
| --------------------------------------- | ---------------------------------------- |
| Has an explicit block                 | Uses the enclosing scope               |
| Disposes when the block ends          | Disposes when the enclosing scope ends |
| Useful for limiting resource lifetime | Useful for cleaner, flatter code       |

Interview trap: A using declaration does not dispose the object immediately after the declaration. Its lifetime extends to the end of its enclosing scope.

#### What happens internally when we use `using`?

Conceptually, the compiler transforms a using statement into a `try/finally` structure.

```
var stream = new FileStream("data.txt", FileMode.Open);

try
{
    // Work with the stream
}
finally
{
    if (stream != null)
    {
        stream.Dispose();
    }
}
```

This is a simplified illustration of the generated behavior, not an exact representation of every compiler transformation.

You generally do not need to write this boilerplate yourself.

### What is the difference between `Dispose()` and a finalizer?

Dispose()

- Called explicitly, commonly through using.
- Gives predictable cleanup timing.
- Should release resources the object owns.

Finalizer

- Runs when the GC determines that an object is eligible for finalization.
- Timing is nondeterministic.
- Is primarily a fallback for types that directly own unmanaged resources and need finalization as a safety net.

#### Example finalizer syntax:

```
public class MyResource
{
    ~MyResource()
    {
        // Finalizer cleanup
    }
}
```

Most application-level classes do not need a finalizer. If a class only owns other disposable managed objects, it normally implements `IDisposable` and disposes those objects.

#### A useful rule

- Use using for objects you own and need to dispose.
- Implement IDisposable when your class owns disposable resources and must expose cleanup.
- Use a finalizer only when there is a specific unmanaged-resource cleanup requirement; the SafeHandle pattern is usually preferable for unmanaged handles.

### What is the difference between managed and unmanaged resources?


| Managed resources                           | Unmanaged resources                                        |
| --------------------------------------------- | ------------------------------------------------------------ |
| Managed by the .NET runtime                 | Acquired from external systems or operating system APIs    |
| Examples: ordinary objects, strings, arrays | Examples: native handles, raw unmanaged memory, OS handles |
| Memory is reclaimed by the GC               | Often require explicit release or a safe-handle mechanism  |

One important nuance: a `FileStream` is a managed .NET object, but it uses an operating system file handle internally. Disposing the stream releases that handle.

### Does the Garbage Collector call `Dispose()` automatically?

No, not in general.

The GC automatically reclaims managed memory when objects are no longer reachable. It does not ordinarily call `Dispose()` on every object implementing `IDisposable`.

For example:

```
var connection = new SqlConnection(connectionString);

// Use the connection
connection.Dispose();
```

Prefer:

```
using var connection =
    new SqlConnection(connectionString);

// Use the connection
```

The second version ensures cleanup when the scope ends.

Interview trap: An object becoming eligible for garbage collection does not mean its external resources have been immediately released.

### What is `IAsyncDisposable`?

Some resources need asynchronous cleanup. .NET provides `IAsyncDisposable`, which exposes `ValueTask DisposeAsync()`.

It is commonly used with asynchronous resource types.

```
await using var resource =
    new SomeAsyncDisposableResource();

// Use the resource asynchronously
```

The resource is asynchronously disposed when the enclosing scope exits.


#### Remember:


- IDisposable → Dispose()
- IAsyncDisposable → DisposeAsync()
- using → synchronous disposal
- await using → asynchronous disposal

Use `await using` when the resource supports `IAsyncDisposable` and asynchronous cleanup is appropriate.

### Common interview traps

#### Trap 1: Does `using` destroy an object?

No. It ensures disposal at the end of the relevant scope. The object may remain in memory until the GC reclaims it.

#### Trap 2: Does `Dispose()` force garbage collection?

No. It performs cleanup; it does not force the GC to reclaim memory.

#### Trap 3: Can `Dispose()` be called more than once?

It can be called more than once, but the class should be designed to handle repeated disposal safely where appropriate. Do not assume every custom implementation is safe to call repeatedly.

#### Trap 4: Should every class implement `IDisposable`?

No. Implement it when the class needs to manage cleanup of owned resources or disposable dependencies.

#### Trap 5: Should you dispose an injected dependency?

Usually, no. If a dependency is created and owned by a DI container, the container generally manages its lifetime and disposal. Avoid disposing a dependency you do not own.

#### Q1. Why do we need `IDisposable`?

To release resources deterministically instead of relying on garbage collection timing.

#### Q2. What is the purpose of the `using` statement?

It ensures that a disposable resource is cleaned up when execution leaves its scope, even if an exception occurs.

#### Q3. What is the difference between `Dispose()` and the GC?

`Dispose()` releases resources explicitly; the GC reclaims managed memory.

#### Q4. What is the difference between a using statement and a using declaration?

The statement disposes at the end of its block; the declaration disposes at the end of its enclosing scope.

#### Q5. What is the difference between `IDisposable` and `IAsyncDisposable`?

`IDisposable` provides synchronous cleanup; `IAsyncDisposable` provides asynchronous cleanup.

#### Q6. When should a class have a finalizer?

Only when it has a specific unmanaged-resource cleanup requirement. Prefer `SafeHandle` for operating system handles and avoid unnecessary finalizers.

#### Q7. Does `using` work when an exception occurs?

Yes. The compiler ensures disposal through cleanup logic equivalent in purpose to a `finally` block.


## Garbage Collection (GC) and Memory Management

### What is Garbage Collection?

Garbage Collection (GC) is a feature of the .NET runtime that automatically reclaims memory occupied by managed objects that are no longer reachable by the application.

Example:

```
public class Employee
{
    public string Name { get; set; }
}

void Test()
{
    Employee emp = new Employee();
    emp.Name = "Arun";

    emp = null;
}
```

After `emp = null`, the `Employee` object may become eligible for garbage collection, provided no other references point to it.

Important: Eligible for collection does not mean collected immediately. The GC decides when to reclaim the memory.

#### Why do we need GC?

- Reduces manual memory-management work.
- Reclaims memory from unreachable managed objects.
- Helps prevent errors associated with manually freeing managed memory.
- Can compact parts of the managed heap to reduce fragmentation.

GC does not automatically release every external resource, such as a database connection or file handle. That is why `IDisposable` and `using` are important.

#### Where are objects stored?

For interview purposes, remember the managed heap.

```
Employee emp = new Employee();
```

Conceptually:

Local variable: `emp` Holds a reference to the object

Managed heap: Employee object and its instance data

This is a conceptual illustration. The actual placement of variables and objects depends on runtime implementation and context; not every local variable necessarily resides on the stack.

Interview trap: Don't claim that all value types are always stored on the stack and all reference types are always stored on the heap. Value types can be fields inside objects, and JIT optimizations can change where values are represented.

### What are GC generations?

.NET uses generations to optimize garbage collection. The idea is that many objects become unused quickly, while objects that survive tend to live longer.

#### There are three main generations:

##### Gen  0: Short-lived objects

Newly allocated objects generally start here. This generation is collected frequently.

Example: temporary objects created while formatting a response.

##### Gen 1: Intermediate objects

Objects that survive a Gen 0 collection can be promoted here. It acts as a buffer between short- and long-lived objects.

##### Gen 2: Long-lived objects

Objects that survive further collections can be promoted here. Collecting Gen 2 generally requires more work.

Example: long-lived application state that remains reachable.

#### How promotion works

Imagine an object is allocated in Gen 0:

1. It survives a Gen 0 collection because it is still reachable.
2. It may be promoted to Gen 1.
3. If it survives further collections, it may be promoted to Gen 2.

Promotion is based on survival, not simply on how many seconds the object has existed.

#### Why does .NET have generations?

To improve performance by collecting short-lived objects frequently without repeatedly scanning all long-lived objects.

### What is a GC collection?

When a generation is collected, the GC identifies live objects, reclaims memory occupied by unreachable objects, and may compact the heap.
A simplified process is:

1. Identify GC roots, such as active stack references and static references.
2. Trace reachable objects.
3. Identify unreachable objects.
4. Reclaim their managed memory.
5. Potentially compact the heap and update references.

The exact process varies depending on the GC mode, generation and object types.

### What are GC roots?

GC roots are references from which the GC begins determining which objects are reachable.
Examples include:

- References held by active method calls.
- Static references.
- References maintained by the runtime.

An object that is reachable from a GC root is generally considered live, subject to runtime details such as finalization.

### What is the Large Object Heap (LOH)?

The Large Object Heap stores large managed allocations. In the standard .NET GC, objects around 85,000 bytes or larger are generally allocated on the LOH.

Example:

```
byte[] buffer = new byte[100_000];
```

This array is typically allocated on the LOH.

Important points:

- LOH objects are collected as part of Gen 2 collections.
- Large allocations can increase memory pressure.
- LOH fragmentation can affect performance.
- Large Object Heap compaction is possible, but it is not generally performed on every collection.

Interview trap: The LOH is not a fourth generation. It is a separate heap area associated with Gen 2 collection behavior.

### What is a memory leak in C#?

A memory leak can occur even in a garbage-collected language.

It happens when objects are no longer logically needed but remain reachable, so the GC cannot reclaim them.

Example:

```
public class CacheService
{
    private readonly List<byte[]> _cache = new();

    public void StoreData()
    {
        _cache.Add(new byte[1_000_000]);
    }
}
```

If this method repeatedly adds arrays and the list continues growing without removing entries, memory usage can grow significantly.

The GC cannot simply remove those arrays because the list still references them.

Common causes include:

- Unbounded caches or collections.
- Event subscriptions that keep objects alive.
- Static collections holding references indefinitely.
- Long-lived references to objects that are no longer needed.

Key distinction: A memory leak is not necessarily an object that is unreachable. In managed applications, leaked objects are often still reachable but are retained unnecessarily.

### What is the difference between `Dispose()` and GC?

This is one of the most important interview questions from your previous topic.


| `Dispose()`                                         | Garbage Collector                                       |
| ----------------------------------------------------- | --------------------------------------------------------- |
| Explicitly releases resources owned by an object    | Reclaims memory occupied by unreachable managed objects |
| Cleanup timing is predictable when called correctly | Collection timing is nondeterministic                   |
| Does not automatically free all managed memory      | Does not ordinarily call`Dispose()`on every object      |
| Often used for files, streams and connections       | Manages the managed heap                                |

Example:

```
using var reader = new StreamReader("data.txt");

string content = reader.ReadToEnd();
```

When the scope ends, the reader is disposed. The GC later reclaims its managed memory when appropriate.

### What is `GC.Collect()`?

`GC.Collect()` requests that the GC perform a collection.

```
GC.Collect();
```

You might see it in legacy code, but you should generally avoid calling it manually.

#### Why?

- The GC already uses runtime heuristics to decide when collection is appropriate.
- Forcing collections can increase CPU overhead.
- It can interfere with normal GC optimization and latency.
- It does not fix objects that are still unnecessarily referenced.

Interview answer: “I generally let the .NET GC manage collections automatically. I would consider a forced collection only in a specific, measured scenario where there is a clear justification.”

### Workstation GC vs. Server GC

.NET provides different GC modes for different application workloads.


| Workstation GC                                           | Server GC                                          |
| ---------------------------------------------------------- | ---------------------------------------------------- |
| Commonly suited to desktop and client applications       | Commonly suited to server workloads                |
| Optimized for responsiveness and resource considerations | Designed for throughput on server workloads        |
| Uses workstation-oriented collection behavior            | Can use multiple GC heaps and dedicated GC threads |

The actual configuration depends on the application type and runtime settings. Server GC is not automatically the best choice for every application.

For ASP.NET Core, understanding Server GC is useful because backend services often prioritize throughput under concurrent workloads.
