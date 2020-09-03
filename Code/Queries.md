# The concept

Queries are requests to external system *that does not change their data state*. In ideal world queries are *pure functions* (idempotent - can be called without different outcome), but in realty this is not possible. Two consequent queries *may* return different data. It depends on outer system's behavior. 

Thing that we actually have to ensure - that our logic *does not perform any updated on outer systems*. Tecture tries to ensure it when possible, but often it is not. E.g.: you *may* perform direct SQL queries to the database that actuall do `UPDATE`. But you better to do not. Tecture cannot ensure it technically, so unfortunately it relies on you.

# Writing queries in Tecture

Queries in tecture are being performed as extension methods to channel's read end:

```csharp
var user = From<Db>().All<User>().FirstOrDefault(x=>x.Id == 10);
var order = From<Db>().SqlQuery<Order>(x=>$"SELECT * FROM {x}").As<Order>().First();
```

Both `.All<>` and `.SqlQuery<>` are extension (static) methods provided by the feature. They return their own abstractions that you also may use and create extension methods to. 

Example: generic `ById` method performed using ORM feature. My favorite one:

```csharp

public interface IEntity { Id {get;} }

public static class Extensions
{
	///<summary>
	/// This method will stick to all .Get<T> invokations when T implements IEntity
	///</summary>
	public static T ById<T>(this IQueryFor<T> q, int id) where T : IEntity
	{
		return q.All.FirstOrDefault(x => x.Id == id);
	}
}
```

Query extensions can be placed everywhere. Their intention is to replace read abstractions (Repositories) in software design. Try to avoid creating [[services]] with `Get*` methods and put query functionality into extensions.

# Using queries

In order to perform query you need to obtain reading channel end. You can do that using `From<>` method. It can be located within [[service|services]] or [[`ITecture` instance|Ioc]].

Example: query from the service

```csharp
public class Orders : TectureService<Order>, INoContext
{
	private Orders() { }

	public void MarkAsDraft(int orderId)
	{
		var order = From<Db>.Get<Order>().ById(orderId);
		To<Db>.Update(order)
				.Set(x=>x.Name, order.Name + " (draft)")
	}
}
```

Example: query from MVC controller

```csharp
public class OrdersController : ApiController
{
	private readonly ITecture _tecture;

	public OrdersController(ITecture tecture)
	{
		_tecture = tecture;
	}

	public ActionResult PerformActionWithOrder(int id)
	{
		var orders = _tecture.From<Db>()
			.All<Orders>()
			.Where(x => x.Status == Status.Active)
			.Select(x => new {x.Id, x.Name})
			.ToArray();
		
		return Json(orders);
	}
}
```