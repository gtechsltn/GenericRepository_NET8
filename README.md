# Understanding GenericRepository in C# and .NET 8

Understanding GenericRepository in C# and .NET 8

https://medium.com/@valentin.osidach/understanding-genericrepository-in-c-and-net-8-b44db3b0b7db

Implementing the Repository and Unit of Work Patterns in an ASP.NET MVC Application (9 of 10)

https://learn.microsoft.com/en-us/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application?source=post_page-----b44db3b0b7db--------------------------------

## Benefits of Using Generic Repository

+ **Code Reusability**: By encapsulating common data access logic, the GenericRepository allows for code reuse across different types of entities.
+ **Maintainability**: With data access logic centralized, any changes required can be made in a single place, simplifying maintenance.
+ **Testability**: The abstraction provided by repositories makes unit testing easier, as you can mock repository methods without dealing with the actual data source.
+ **Consistency**: Ensures a consistent approach to data access across the application.

## Implement
```
dotnet new webapi -n GenericRepositoryDemo
```

## Entity
```
public interface IEntity
{
    int Id { get; set; }
}

public class Product : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

## Generic Repository Interface
```
public interface IRepository<TEntity> where TEntity : class, IEntity
{
    Task<IEnumerable<TEntity>> GetAllAsync();
    Task<TEntity> GetByIdAsync(int id);
    Task AddAsync(TEntity entity);
    Task SaveAsync();
    void Update(TEntity entity);
    void Delete(TEntity entity);
}
```

## Generic Repository
```
public class Repository<TEntity> : IRepository<TEntity> 
       where TEntity : class, IEntity
{
    private readonly DbContext _context;
    private readonly DbSet<TEntity> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<TEntity>();
    }

    public async Task<IEnumerable<TEntity>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task<TEntity> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

   public async Task AddAsync(TEntity entity)
   {
       await _dbSet.AddAsync(entity);
   }

    public void Update(TEntity entity)
    {
        _dbSet.Update(entity);
    }

    public void Delete(TEntity entity)
    {
        _dbSet.Remove(entity);
    }

    public async Task SaveAsync()
    {
        await _context.SaveChangesAsync();
    }
}
```

## DbContext
```
public class Context : DbContext
{
    public Context(DbContextOptions<Context> options) : base(options) {}

    public DbSet<Product> Products { get; set; }
}
```

## Registering the Repository with Dependency Injection
```
var builder = WebApplication.CreateBuilder(args);
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

builder.Services.AddDbContext<Context>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

## Using the Generic Repository
```
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IRepository<Product> _productRepository;

    public ProductsController(IRepository<Product> productRepository)
    {
        _productRepository = productRepository;
    }

    [HttpGet]
    public async Task<IActionResult> GetAllProducts()
    {
        var products = await _productRepository.GetAllAsync();
        return Ok(products);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetProductById(int id)
    {
        var product = await _productRepository.GetByIdAsync(id);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }

    [HttpPost]
    public async Task<IActionResult> AddProduct(Product product)
    {
        await _productRepository.AddAsync(product);
        await _productRepository.SaveAsync();
        return CreatedAtAction(nameof(GetProductById), new { id = product.Id }, product);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(int id, Product product)
    {
        if (id != product.Id)
        {
            return BadRequest();
        }

        _productRepository.Update(product);
        await _productRepository.SaveAsync();
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        var product = await _productRepository.GetByIdAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        _productRepository.Delete(product);
        await _productRepository.SaveAsync();
        return NoContent();
    }
}
```

# Conclusion
The GenericRepository pattern in C# and .NET 8 is a powerful tool for developers, enabling efficient and maintainable data access layers. Abstracting common CRUD operations promotes code reuse, improves maintainability, and enhances testability. With .NET 8, implementing and using the GenericRepository pattern has become more streamlined, allowing developers to focus on building robust and scalable applications.
