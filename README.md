using InventarioProductosAPI.Models;
using InventarioProductosAPI.Dtos;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Lista de productos en memoria
var products = new List<Product>();

// Página inicial
app.MapGet("/", () => "API de Inventario de Productos funcionando correctamente");


app.MapPost("/recurso", (CreateProductDto dto) =>
{
    if (string.IsNullOrWhiteSpace(dto.Name) ||
        dto.Price <= 0 ||
        dto.Stock < 0)
    {
        return Results.BadRequest("Los datos del producto son inválidos.");
    }

    var product = new Product
    {
        Id = products.Count == 0 ? 1 : products.Max(p => p.Id) + 1,
        Name = dto.Name.Trim(),
        Price = dto.Price,
        Stock = dto.Stock
    };

    products.Add(product);

    return Results.Created($"/recurso/{product.Id}", product);
});


app.MapGet("/recursos", () =>
{
    return Results.Ok(products);
});


app.MapGet("/recurso/{id:int}", (int id) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);

    if (product is null)
    {
        return Results.NotFound("Producto no encontrado.");
    }

    return Results.Ok(product);
});


app.MapGet("/recurso", (string? name) =>
{
    if (string.IsNullOrWhiteSpace(name))
    {
        return Results.Ok(products);
    }

    var filteredProducts = products
        .Where(p => p.Name.Contains(name, StringComparison.OrdinalIgnoreCase))
        .ToList();

    if (filteredProducts.Count == 0)
    {
        return Results.NotFound("No se encontraron productos con ese nombre.");
    }

    return Results.Ok(filteredProducts);
});


app.MapDelete("/recurso/{id:int}", (int id) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);

    if (product is null)
    {
        return Results.NotFound("Producto no encontrado.");
    }

    products.Remove(product);

    return Results.Ok(new
    {
        message = "Producto eliminado correctamente.",
        product
    });
});


app.MapPatch("/recurso/{id:int}", (int id, PatchProductDto dto) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);

    if (product is null)
    {
        return Results.NotFound("Producto no encontrado.");
    }

    if (dto.Name is null && dto.Price is null && dto.Stock is null)
    {
        return Results.BadRequest("Debes enviar al menos un campo para actualizar.");
    }

    if (dto.Name is not null)
    {
        if (string.IsNullOrWhiteSpace(dto.Name))
        {
            return Results.BadRequest("El nombre no puede estar vacío.");
        }

        product.Name = dto.Name.Trim();
    }

    if (dto.Price.HasValue)
    {
        if (dto.Price.Value <= 0)
        {
            return Results.BadRequest("El precio debe ser mayor que cero.");
        }

        product.Price = dto.Price.Value;
    }

    if (dto.Stock.HasValue)
    {
        if (dto.Stock.Value < 0)
        {
            return Results.BadRequest("El inventario no puede ser negativo.");
        }

        product.Stock = dto.Stock.Value;
    }

    return Results.Ok(product);
});


app.MapPut("/recurso/{id:int}", (int id, UpdateProductDto dto) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);

    if (product is null)
    {
        return Results.NotFound("Producto no encontrado.");
    }

    if (string.IsNullOrWhiteSpace(dto.Name) ||
        dto.Price <= 0 ||
        dto.Stock < 0)
    {
        return Results.BadRequest("Los datos del producto son inválidos.");
    }

    product.Name = dto.Name.Trim();
    product.Price = dto.Price;
    product.Stock = dto.Stock;

    return Results.Ok(product);
});

app.Run();
# inventario-de-productos
