# The concept

Service contains application logic. Logic is series of computation that results number of `To<Channel>.%something%` calls that enqueue [[commands]] into commands queue for later dispatching. In the scope of Tecture, business logic makes decisions about which commands to run on external systems. Logic is the the code that takes *user input*, reads *existing data* from channels and computes series of updates to be performed on external systems. 

Example: service responsible for `Orders` has business logic method that takes `orderId` and sends to DB SQL coda that calls stored procedure performing clearing.

```csharp
public class Orders : TectureService<Order>, INoContext
    {
        private Orders() { }

        public void DoClearing(int orderId)
        {
            To<Db>().SqlStroke(()=>$"sp_exec DoClearing {orderId}");
        }
    }
```

It is important to understand that `sp_exec ...` will **not** be executed immediately. `.SqlStroke` does not address database immediately. It just *enqueues the command* that will send this command to database after you call `ITecture.Save()`. See [[integration|Ioc]] section in order to understand where to trigger commands dispatching.

# Writing service

As you have noticed, services are classes that inherit `TectureService<>` class. There are several noticable things in Tecture service:

- **Private constructor**. It is mandatory. Without it you will get runtime exception. Tecture ensures its services not to be instantiated via `new` or any other method. **Warning!** This mechanism may change soon in favor of using internal-only created class;
- **Type parameters**. Service may contain *up to 8* type parameters. It is presumed that you pass there types of your entities. It is needed to explicitly declare what entities are being *changed* by this service. E.g. method `.Add` of ORM feature will not allow you to create entities that are not listed in service's type parameters list. 
- `INoContext` marker interface. **Warning!** This mechanism may change soon in favor of removing it entirely. For now just add it to all your Tecture services.

The limitation of *8 entities per service* is intentional. It is presumed that if you need to change more than 8 entities within one service - you are doing something wrong and your services need de-coupling.

Inheritance of Tecture services considered to be bad practice. Avoid if possible.

# Invoking service

Service can be invoked using `Do<_Service_>` method where `_Service_` is *type* of service that you want to invoke. The `Do<>` method is available from 2 places:
- `ITecture` instance to invoke Tecture service from outer world. See [[integration|Ioc]] section in order to understand how to invoke your Tecture service e.g. from your ASP.NET MVC application
- Also `Do<>` method exists withing each service. Its purpose is to allow inter-communication between services (to invoke one service from another).

Example: invoking one service from another service:

```csharp
public class B : TectureService<Blueprint>, INoContext
{
	private B() { }

	public void DoLogicB() { /*...*/ }
}

public class A : TectureService<Blueprint>, INoContext
{
	private A() { }

	public void DoLogicA()
	{
		Do<B>().DoLogicB();
	}
}
```

Example: calling service from ITecture injected into web application (see [[integration|Ioc]] section for details)

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
		_tecture.Do<Orders>().PerformAction(id);
		_tecture.Save();

		return Ok();
	}
}
```

# Service lifetime



# Defining service

# Primary service methods

## `From<>`

## `To<>`