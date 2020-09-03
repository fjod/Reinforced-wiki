# The concept

Tecture relies on *anemic entities* containing 0 logic. As soon as [C# 9 records](https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/#records) will be released - Tecture will move on them because right now there is no honest reason to keep mutable entities.

Tecture design borrows some concepts of EntityFramework regarding entities, but actually it uses them a lot in order to obtain type information, not data itself. So treat entity type as metadata for your project domain.

# Do not put logic into entities

Keep entity plain and stupid as possible, like that:

```csharp
public class Blueprint
{
	public int Id { get; set; }
	public Product Product { get; set; }
	public int ProductId { get; set; }	
	public string Name { get; set; }	
	public DateTime CreatedAt { get; set; }	
	public DateTime? UpdatedAt { get; set; }	
}
```

If you want to add some entity-centric logic - write *extension method* rather placing logic inside entity. Maintain the [SRP](https://en.wikipedia.org/wiki/Single-responsibility_principle) for entities - let them just carry the data. Leave business operations to services.

# Relations

For many-to-many relations create link entities:

```csharp
public class BlueprintResources
{
	public Resource Resource { get; set; }
	public Blueprint Blueprint { get; set; }
	public int ResourceId { get; set; }
	public int BlueprintId { get; set; }
}
```
For one-to-many relations try to operate singular end of relation:

## Better
```csharp
public class User
{
	public int Id { get; set; }
	public int OrderId { get; set; }
	public Order Order { get; set; }
}

// ...
To<Db>().Add(new User() { OrderId = 10 });
// ...
```
## Worse
```csharp
public class Order
{
	public int Id { get; set; }
	public HashSet<User> Users { get; set; }
}
// ...
var user = new User();
To<Db>().Add(user);
var order = From<Db>().Get<Order>().ById(10);
order.Users.Add(user);
To<Db>().Update(order, x=>x.Users);
// ...
```

It will comply with ORM feature and trigger proper processes in EF Runtime, but in common case that does not work.

# Restrict write access

Try to achieve the state when only *particular services in partucilar assemblies* have write access to the entity. 

E.g. you can move your entities related to some business functionality into separate assembly and then close setters and constructor witn `internal` modifier. `internal` modifier is not a problem for majority of ORMs, so by restricting entity of unwanted modifications and opening it for reading you significantly reduce number of suspicios things happening in your system.

```csharp
public class GoodEntity
{
	internal GoodEntity() { }
	public int Id { get; internal set; }
	public string Name { get; internal set; }
	public Order Order { get; internal set; }
}
```

# DTOs

DTOs are good. Use them. 

The thing is that DTOs are not entities per se. They are containers for parameters of business logic methods:

## Bad
```csharp
public void CreateOrder(string name, DateTime issuedDat, string command, Status status, bool isActive, int ownerId)
{
	//...
}
```

## Better
```csharp
class CreateOrderDto
{
	public string Name { get; set; }
	public DateTime IssueDate { get; set; }
	public string Command { get; set; }
	public Status Status { get; set; }
	public bool IsActive { get; set; }
	public int OwnerId { get; set; }
}

public void CreateOrder(CreateOrderDto orderDto)
{
	//...
}
```

Follow the simple rule: if business logic method has *more than 5 arguments* - it is valid place to create DTO.
