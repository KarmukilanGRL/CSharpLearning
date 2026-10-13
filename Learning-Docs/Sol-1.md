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

| Exception                       | When it occurs                                  |
| ------------------------------- | ----------------------------------------------- |
| `NullReferenceException`      | Accessing a member through a null reference     |
| `ArgumentNullException`       | A method receives`null`where it isn't allowed |
| `ArgumentException`           | An argument is invalid                          |
| `ArgumentOutOfRangeException` | An argument is outside the permitted range      |
| `InvalidOperationException`   | An operation isn't valid for the current state  |
| `FormatException`             | A string or value has an invalid format         |
| `DivideByZeroException`       | Integer division by zero                        |
| `IndexOutOfRangeException`    | An array index is outside its bounds            |
| `KeyNotFoundException`        | A requested key isn't present in a dictionary   |
| `IOException`                 | An input/output operation fails                 |
| `UnauthorizedAccessException` | Access to a resource is denied                  |
| `TimeoutException`            | An operation times out                          |

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

| Property           | Purpose                                                            |
| ------------------ | ------------------------------------------------------------------ |
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
| ---------------------------------- | ---------------------------------------------- |

| Preferred for rethrowing in a`catch`block | Generally avoid |
| ------------------------------------------- | --------------- |

| Helps preserve debugging context | Can make the original failure harder to locate |
| -------------------------------- | ---------------------------------------------- |

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
| ------------------------------------------------------ | ------------------------------------------------ |
| Detailed diagnostic information helps developers debug | Generic error responses protect internal details |
| Developer Exception Page is useful                     | `UseExceptionHandler()`is commonly used        |
| Stack traces may be shown locally                      | Stack traces should not be exposed to clients    |

ASP.NET Core templates commonly enable the Developer Exception Page in Development and use exception-handling middleware in non-development environments.

#### Which HTTP status code should you return?

Global exception handling should return a suitable status code rather than returning `200 OK` for every outcome.

| Status code                   | Meaning                                                 | Example                                  |
| ----------------------------- | ------------------------------------------------------- | ---------------------------------------- |
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

ValueTask TryHandleAsync(
HttpContext httpContext,
Exception exception,
CancellationToken cancellationToken);

The handler returns:

true — the exception has been handled.

false — this handler hasn't handled the exception.

#### How is it configured?

In Program.cs, register the handler and exception-handling middleware:

builder.Services.AddExceptionHandler();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();

app.MapControllers();

app.Run();

Here, MyExceptionHandler is your implementation of IExceptionHandler. The registration enables ASP.NET Core to use it through the exception-handling middleware.

AddProblemDetails() registers services for generating standardized error responses. The actual response depends on the exception handler and configured error-handling services.

#### Which approach should you choose?

| Approach                  | Best suited for                                                        |
| ------------------------- | ---------------------------------------------------------------------- |
| `UseExceptionHandler()` | Standard centralized handling with minimal custom infrastructure       |
| Custom middleware         | Special handling that requires control over the entire middleware flow |
| `IExceptionHandler`     | Dedicated, testable exception-handling logic in modern ASP.NET Core    |

**Interview answer:** For a modern ASP.NET Core application, `IExceptionHandler` provides a clean abstraction for centralized exception handling. Custom middleware is useful when you need broader control over request-pipeline behavior. `UseExceptionHandler()` is the built-in middleware mechanism that can host centralized exception-handling logic.

One important distinction: these approaches aren't necessarily alternatives. `IExceptionHandler` is commonly used together with `UseExceptionHandler()`.
