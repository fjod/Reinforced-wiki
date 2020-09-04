# The concept

Service contains application logic. Logic is a sequence of computation that produce number of `To<Channel>.%something%` calls that enqueue [[commands]] into commands queue for later dispatching. In the scope of Tecture, business logic makes decisions about which commands to run on external systems. Logic is the the code that takes *user input*, reads *existing data* from channels and computes series of updates to be executed on external systems. 

Example: service responsible for `Orders` has business logic method that takes `orderId` and sends to DB SQL code that calls stored procedure performing clearing.

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

# Defining service

As you might have noticed, services are classes that inherit `TectureService<>` class. There are several noticable things in Tecture service:

- **Private constructor**. It is mandatory. Without it you will get runtime exception. Tecture ensures its services not to be instantiated via `new` or any other method. **Warning!** This mechanism may change soon in favor of using internal-only created class;
- **Type parameters**. Service may contain *up to 8* type parameters. It is presumed that you pass there types of your entities. It is needed to explicitly declare what entities are being *changed* by this service. E.g. method `.Add` of ORM feature will not allow you to create entities that are not listed in service's type parameters list. 
- `INoContext` marker interface. **Warning!** This mechanism may change soon in favor of removing it entirely. For now just add it to all your Tecture services.

The limitation of *8 entities per service* is intentional. It is presumed that if you need to change more than 8 entities within one service - you are doing something wrong and your services need de-coupling.

Inheritance of Tecture services considered to be a bad practice. Avoid if possible.

# Invoking service

Service can be invoked using `Do<_Service_>` method where `_Service_` is a *type* of service that you want to invoke. The `Do<>` method is available from 2 places:
- `ITecture` instance to invoke Tecture service from outer world. See [[integration|Ioc]] section in order to understand how to invoke your Tecture service, e.g. from your ASP.NET MVC application
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

- Tecture service instance is being created on demand when `Do<>` from this service as type parameter is invoked for the first time. 
- Tecture service is disposed when entire `ITecture` instance is disposed. So if you define `ITecture` within per-request lifetime scope then all tecture services will exist within this scope. Not more, not less. 

Keep this information in mind when defining private variables within service.

# What is available inside service?

Service contains primary methods to work with channels

## `From<>` method

This method is invoked with `TChannel` as type parameter. It obtains `Read<TChannel>` object that reveals reading end of the channel. Extension methods for reading from channel will be automatically provided by corresponding [[features]].

## `To<>` method

This method is invoked with `TChannel` as type parameter. It obtains `Write<TChannel>` object that reveals writing (commands) end of the channel. Extension methods for reading from channel will be automatically provided by corresponding [[features]].

Methods `From<>` and `To<>` are lightweight, so can be safely called as much as needed. 

# Lifecycle methods

- `Init` and `Dispose` methods are being called when service is going to be created and destroyed correspondingly.
- `OnSave`/`OnSaveAsync` methods are being called when [[saving]]/async saving is initiated
- `OnFinally`/`OnFinallyAsync`  methods are called after clearing the entire commands queue and all `Save` iterations passed

`OnSave*` and `OnFinally*` can be called several times because of the nature of post-save actions.

# Post-save actions

`Save` and `Finally` are properties that allows you to perform actions AFTER [[saving]] actually happened. You may turn your business logic method into async one and than use `await Save` and `await Finally` to enqueue actions that might take place after [[saving]] happened or all commands from queue were dispatched (`Finally`). Please keep in mind that if you enqueue more and more commands after save/finally - you are triggering them to happen and happen again. Be careful, it may be tricky. 