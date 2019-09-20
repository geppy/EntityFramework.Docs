---
title: New features in EF Core 3.0 - EF Core
author: divega
ms.date: 02/19/2019
ms.assetid: 2EBE2CCC-E52D-483F-834C-8877F5EB0C0C
uid: core/what-is-new/ef-core-3.0/features
---

# New features included in EF Core 3.0

The following list includes the major new features planned for EF Core 3.0.

EF Core 3.0 is a major release and also contains numerous [breaking changes](xref:core/what-is-new/ef-core-3.0/breaking-changes), which are API improvements that may have negative impact on existing applications.  

## LINQ improvements 

LINQ enables you to write database queries without leaving your language of choice, taking advantage of rich type information to offer IntelliSense and compile-time type checking.
But LINQ also enables you to write an unlimited number of complicated queries containing arbitrary expressions (method calls or operations).
Handling all those combinations has always been a significant challenge for LINQ providers.
In EF Core 3.0, we've rearchitected our LINQ implementation to enable translating more expressions into SQL, to generate efficient queries in more cases, to prevent inefficient queries from going undetected, and to make it easier for us to gradually introduce new query capabilities and performance improvementswithout breaking existing applications and data providers.

### Client evaluation

The most important design change in EF Core 3.0 has to do with how it handles LINQ expressions that it cannot translate to SQL or to parameters:

In the first few versions, EF Core simply figured out what portions of a query could be translated to SQL, and executed the rest of the query on the client.
This type of client-side execution can be desirable in some situations, but in many other cases it can result in inefficient queries.
For example, if EF Core 2.2 couldn't translate a predicate in a `Where()` call, it executed a SQL statement without a filter, read all all the rows from the database, and then filtered them in-memory.
That may be acceptable if the database contains a small number of rows, but can result in significant performance issues or even application failure if the database contains a large number or rows.
In EF Core 3.0 we have restricted client evaluation to only happen on the top-level projection (the last call to `Select()`).
When EF Core 3.0 detects expressions that cannot be translated anywhere else in the query, it throws a runtime exception.

See the [breaking changes documentation](xref:core/what-is-new/ef-core-3.0/breaking-changes#linq-queries-are-no-longer-evaluated-on-the-client) for more details about how this can affect existing applications.

## Cosmos DB support 

The Cosmos DB provider for EF Core enables developers familiar with the EF programing model to easily target Azure Cosmos DB as an application database.
The goal is to make some of the advantages of Cosmos DB, like global distribution, "always on" availability, elastic scalability, and low latency, even more accessible to .NET developers.
The provider will enable most EF Core features, like automatic change tracking, LINQ, and value conversions, against the SQL API in Cosmos DB.

See the [Cosmos DB provider documentation](xref:core/providers/cosmos/index) for more details.

## C# 8.0 support

EF Core 3.0 takes advantage of some of the [new features in C# 8.0](https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8):

### Asynchronous streams

Asynchronous query results are now exposed using the new standard `IAsyncEnumerable<T>` interface and can be consumed using `await foreach`.

``` csharp
var orders = 
  from o in context.Orders
  where o.Status == OrderStatus.Pending
  select o;

await foreach(var o in orders)
{
  Process(o);
} 
```

See the [asynchronous streams in the C# documentation](https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams) for more details.

### Nullable reference types 

When this new feature is enabled in your code, EF Core can reason about the nullability of properties of refrence types (either of primitive types like string or navigation properties) to decide the nullability of columns and relationships in the database.
EF Core 3.0 treats C# 8.0 reference type nullability in the same it treats the `[Required]` data annotation atttribute.  
For example, in the following class, properties marked as of type `string?` will be configured as optional, whereas `string` will be configured as required:

``` csharp
public class Customer
{
  public int Id { get; set; }
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public string? MiddleName { get; set; }
}
```

See [nullable reference types in the C# documentation](https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#nullable-reference-types) for more details.

## Interception

The new interception API in EF Core 3.0 allows programatically observing and modifying the outcome of low-level database operations that occur as part of the normal operation of EF Core, such as opening connections, initating transactions, and executing commands. 

## Reverse engineering of database views

Entity types without keys (previously known as [query types](xref:core/modeling/keyless-entity-types)) represent data that can be read from the database, but cannot be updated.
This characteristic makes them an excellent fit for mapping database views in most scenarios, so we automated the creation of entity types without keys when reverse engineering database views.

## Dependent entities sharing the table with the principal are now optional

Starting with EF Core 3.0, if `OrderDetails` is owned by `Order` or explicitly mapped to the same table, it will be possible to add an `Order` without an `OrderDetails` and all of the `OrderDetails` properties, except the primary key will be mapped to nullable columns.

When querying, EF Core will set `OrderDetails` to `null` if any of its required properties doesn't have a value, or if it has no required properties besides the primary key and all properties are `null`.

``` csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public OrderDetails Details { get; set; }
}

[Owned]
public class OrderDetails
{
    public int Id { get; set; }
    public string ShippingAddress { get; set; }
}
```

## EF 6.3 on .NET Core

We understand that many existing applications use previous versions of EF, and that porting them to EF Core only to take advantage of .NET Core can sometimes require a significant effort.
For that reason, we have enabled the newewst version of EF 6 to run on .NET Core 3.0.
There are some limitations, for example:
- New providers are required to work on .NET Core
- Spatial support with SQL Server won't be enabled

## Postponed features

Some features originally planned for EF Core 3.0 were postponed to future releases: 

- Ability to ingore parts of a model in migrations, tracked as [#2725](https://github.com/aspnet/EntityFrameworkCore/issues/2725).
- Property bag entities, tracked as two separate issues: [#9914](https://github.com/aspnet/EntityFrameworkCore/issues/9914) about shared-type entities and [#13610](https://github.com/aspnet/EntityFrameworkCore/issues/13610) about indexed property mapping support.
