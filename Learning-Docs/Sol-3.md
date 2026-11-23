# Entity Framework Core (EF Core)

## Introduction:

### **What is Entity Framework Core (EF Core)?**

**Entity Framework Core (EF Core)** is an ORM (Object-Relational Mapper) provided by Microsoft for .NET applications.

It allows developers to interact with databases using C# objects instead of writing SQL queries for every database operation.

in simple terms:

Without EF Core, we generally write SQL queries manually.

With EF Core, we can perform database operations using C# code.

#### **Example: Without EF Core**

Suppose we want to retrieve all employees from a SQL Server database.

```
SELECT * FROM Employees;
```

We execute this SQL query and manually map the returned records to C# objects.

#### **Example: With EF Core**

First, create an Employee class:

```
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Salary { get; set; }
}
```

Then retrieve employees using EF Core:

```
var employees = await _context.Employees.ToListAsync();
```

EF Core automatically translates this LINQ query into SQL and executes it against the database.

The results are mapped back into C# Employee objects.

### **How EF Core works**

**1. C# Application** -> Business logic

**2. Entity Framework Core** -> Converts LINQ and entity operations into SQL

**3. SQL Server Database** -> Executes SQL queries and returns results

### What is ORM?

ORM stands for Object-Relational Mapping.

It is a technique that maps:

- C# classes to database tables.
- C# properties to database columns.
- C# objects to database records (rows).

For example:

| C#              | SQL Server      |
| --------------- | --------------- |
| Employee class  | Employees table |
| Id property     | Id column       |
| Name property   | Name column     |
| Employee object | Employee row    |

This allows developers to work with databases using object-oriented programming.

### Why do we use EF Core?

Important advantages:

1. Less SQL code: Perform many database operations using C#.
2. LINQ support: Query databases using LINQ.
3. Change Tracking: Automatically tracks changes made to entities.
4. Migrations: Helps manage database schema changes.
5. Database support: Supports SQL Server, SQLite, PostgreSQL, and other providers.
6. CRUD operations: Makes Create, Read, Update, and Delete operations easier.
7. Integration: Works well with ASP.NET Core and Dependency Injection.

### **EF Core vs ADO.NET**

This is a very common interview question.

| EF Core                                  | ADO.NET                                         |
| ---------------------------------------- | ----------------------------------------------- |
| ORM framework                            | Data access technology                          |
| Works primarily with C# objects and LINQ | Works with SQL commands and data readers        |
| Supports change tracking                 | No automatic entity change tracking             |
| Less manual SQL mapping                  | More manual mapping                             |
| Generally faster to develop with         | Provides more direct control over SQL execution |

**Interview point:** EF Core does not completely eliminate SQL. It generates SQL for many operations, but developers can also write raw SQL when needed.

### **Important interview questions**

**Q1. What is Entity Framework Core?**

EF Core is an open-source, cross-platform ORM framework developed by Microsoft. It allows .NET applications to interact with relational databases using C# objects and LINQ instead of writing SQL queries manually for every operation.

**Q2. What is ORM?**

ORM stands for Object-Relational Mapping. It maps object-oriented programming concepts such as classes and properties to relational database concepts such as tables and columns.

**Q3. Is EF Core a replacement for SQL?**

No. EF Core generates SQL queries behind the scenes. Developers still need SQL knowledge for performance optimization, complex queries, joins, and troubleshooting.

**Q4. Is EF Core the same as Entity Framework?**

No. EF Core is a modern, lightweight, cross-platform version of Entity Framework. It is a redesign and is not fully backward-compatible with the older Entity Framework 6.

**Q5. Can EF Core work with databases other than SQL Server?**

Yes. EF Core supports multiple database providers, including SQL Server, SQLite, PostgreSQL, and MySQL.

### Common interview traps

- EF Core is not a database. It is an ORM framework.
- EF Core does not mean you never need SQL.
- EF Core supports LINQ, but LINQ queries are translated into SQL when executed against the database.
- EF Core is not limited to SQL Server.
- EF Core is different from the older Entity Framework 6.

## **DbContext and DbSet**

### What is DbContext?

DbContext is the main class in EF Core that manages communication between your .NET application and the database.

In simple terms, DbContext acts as a bridge between your C# application and the database.

It is responsible for:

- Establishing database communication.
- Executing database queries.
- Tracking changes to entities.
- Saving changes to the database.
- Managing entity relationships.

#### **Practical example**

Suppose we have an Employee class:

```
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Salary { get; set; }
}
```

We create our own DbContext class by inheriting from DbContext.

```
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Employee> Employees { get; set; }
}
```

Here:

- AppDbContext is our custom database context.
- It inherits from EF Core's DbContext.
- DbContextOptions provides configuration such as the database provider and connection string.
- Employees represents the Employee collection in the database.

### What is DbSet?

DbSet represents a collection of entities of a particular type that EF Core can query and manage.

For example:

```
public DbSet<Employee> Employees { get; set; }
```

This represents the Employee entity set, typically mapped to the `Employees` table.

Using DbSet, we can perform CRUD operations.

| Operation | Example                        |
| --------- | ------------------------------ |
| Create    | `Employees.Add(employee)`    |
| Read      | `Employees.ToListAsync()`    |
| Update    | Modify entity properties       |
| Delete    | `Employees.Remove(employee)` |

**Important:** DbSet is not literally the database table. It is an EF Core abstraction through which we work with entities mapped to the database.

### **Difference between DbContext and DbSet**

This is a very common interview question.

| DbContext                              | DbSet                                           |
| -------------------------------------- | ----------------------------------------------- |
| Represents a session with the database | Represents a collection of entities             |
| Manages database operations            | Performs operations on a particular entity type |
| Tracks changes across entities         | Provides access to specific entities            |
| Saves changes using`SaveChanges()`   | Supports Add, Remove, Find, etc.                |

Think of it this way:

**AppDbContext (DbContext)** [Manages database interaction]

**1. DbSet Employee** [Employees table]

**2. DbSet Department** [Departments table]

A single DbContext can contain multiple DbSet properties.

### How do we use DbContext in ASP.NET Core Web API?

In ASP.NET Core, we generally register DbContext using Dependency Injection.

Example in `Program.cs`:

```
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")
    ));
```

Connection string in `appsettings.json`:

```
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

Now we can inject `AppDbContext` into a controller.

```
public class EmployeeController : ControllerBase
{
    private readonly AppDbContext _context;

    public EmployeeController(AppDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public async Task<IActionResult> GetEmployees()
    {
        var employees = await _context.Employees.ToListAsync();

        return Ok(employees);
    }
}
```

Here:

1. ASP.NET Core creates the DbContext through DI.
2. The controller receives it through constructor injection.
3. _context.Employees accesses the employee entity set.
4. ToListAsync() executes the database query.
5. The results are returned as a list.

### **Important interview questions**

**Q1. What is DbContext in EF Core?**

DbContext is the primary class responsible for interacting with the database. It manages queries, change tracking, entity relationships, and saving changes.

**Q2. What is DbSet in EF Core?**

DbSet represents a collection of entities of a particular type. It provides methods to query and perform CRUD operations on those entities.

**Q3. Can one DbContext have multiple DbSet properties?**

Yes. A single DbContext can contain multiple DbSet properties representing different entity types.

**Q4. Is DbContext thread-safe?**

No. DbContext is not thread-safe. We should not use the same DbContext instance for multiple parallel operations.

**Q5. What is the default lifetime of DbContext when registered using AddDbContext?**

Scoped.

It means one DbContext instance is generally created per HTTP request and shared within that request's dependency injection scope.

**Q6. What happens if we don't call SaveChanges()?**

Changes tracked by EF Core, such as Add, Update, or Remove operations, are not persisted to the database until `SaveChanges()` or `SaveChangesAsync()` is called.

### Common interview traps

- DbContext is not the database itself.
- DbSet is not a physical database table; it represents an entity set.
- DbContext is not thread-safe.
- Add() does not immediately insert data into the database. It marks the entity as Added.
- SaveChanges() or SaveChangesAsync() persists tracked changes.
- AddDbContext() registers the context with a scoped lifetime by default.
