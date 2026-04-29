# Dependency Injection in ASP.NET Core

ASP.NET Core has a built-in dependency injection (DI) framework that provides dependencies to classes rather than having the classes create their own dependencies.

## Service Registration

Services are registered in `Program.cs` using the `IServiceCollection` interface.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddScoped<IMyService, MyService>();
builder.Services.AddTransient<IEmailSender, EmailSender>();
builder.Services.AddSingleton<IConfiguration>(builder.Configuration);
```

## Service Lifetimes

| Lifetime | Description | Use When |
|---|---|---|
| `Singleton` | One instance for the application lifetime | Stateless services, caches |
| `Scoped` | One instance per HTTP request | Database contexts, per-request state |
| `Transient` | New instance every time requested | Lightweight, stateless services |

## Constructor Injection

The most common pattern — declare dependencies as constructor parameters:

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger<OrderService> _logger;

    public OrderService(
        IOrderRepository repository,
        ILogger<OrderService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task<Order> GetOrderAsync(int id)
    {
        _logger.LogInformation("Fetching order {OrderId}", id);
        return await _repository.GetByIdAsync(id);
    }
}
```

## Injecting into Minimal API Endpoints

Services can be injected directly into endpoint handler parameters:

```csharp
app.MapGet("/orders/{id}", async (int id, IOrderRepository repo) =>
{
    var order = await repo.GetByIdAsync(id);
    return order is null ? Results.NotFound() : Results.Ok(order);
});
```

## Injecting into Controllers

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderRepository _repository;

    public OrdersController(IOrderRepository repository)
    {
        _repository = repository;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _repository.GetByIdAsync(id);
        return order is null ? NotFound() : Ok(order);
    }
}
```

## Registering Options

Use the Options pattern to bind configuration sections to strongly-typed classes:

```csharp
// appsettings.json
// { "SmtpSettings": { "Host": "smtp.example.com", "Port": 587 } }

public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
}

// Program.cs
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("SmtpSettings"));

// Usage via injection
public class EmailSender
{
    private readonly SmtpSettings _settings;

    public EmailSender(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }
}
```

## See Also

- [APIs — Minimal and Controllers](./apis-minimal-and-controllers.md)
- [Microsoft Docs: Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
