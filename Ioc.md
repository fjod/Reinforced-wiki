# `ITecture` interface

Instance of `ITecture` interface is main access point to all Tecture functionality for the rest of the application. 

Wherever you want to access your services via `Do<>` or your queries via `From<>` - you need to obtain `ITecture` instance. Implementation of this instance is internal, so the one and only way to create it is to use `TectureBuilder`. 

# Tecture and IoC

Instance of `ITecture` is presumed to be obtained from IoC container of your application where needed. So therefore it must be registered inside IoC. 

Example: registering Tecture within ASP.NET MVC IoC container

```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddControllers();
		services.AddTransient<MyDbContext>();
		services.AddTransient(sp =>
		{
			var ld = new LazyDisposable<MyDbContext>(() => sp.GetService<MyDbContext>());

			var tb = new TectureBuilder();
			tb.WithChannel<Db>(c =>
			{
				c.UseEfCoreDirectSqlCommand(ld);
				c.UseEfCoreOrmCommand(ld);
			});

			return tb.Build();
		});
	}
}
```

Example: using Tecture from ASP.NET MVC application:
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

# `ITecture` methods

- `Do<>`: this method grants access to your [[services]]. Usage: `_tecture.Do<Orders>().MarkAsDraft(id)`
- `From<>`: this method grants read access from specified channel. Usage: `return _tecture.From<Db>().Get<User>().ById(10)`
- `Save`/`SaveAsync`: initiates [[saving]] and triggers commands queue dispatch. Consume type of transaction to start before saving and to commit after saving successfully finishes. **Warning!** This functionality may change soon in favor of achieving more flexible transactional mechanism.
- `BeginTrace`/`EndTrace`: access to [[tracing]] capabilities. Avoid using these methods in production.

# `TectureBuilder` class

It is a builder for `ITecture` instance. It is used to bind channels and configure some other stuff. `TectureBuilder` contains following methods:

- `WithChannel<>`: reveals channel type and gives access to channel runtime configuration. Allows to call `.Use*` methods provided by runtime
- `WithExceptionHandler`: specifies delegate to be called when exception takes place during command dispatch
- `WithTransactions`: allows to specify transactions manager that will be used during [[saving]]
- `WithTestData`: provides class with test data. Read [[testing|Capture-test-data]] section about it

# Methods of channel configurator

This information will be helpful if you are going to implement new feature along with runtime.

`WithChannel<>` reveals following methods:

- `ForQuery<TFeature>`: registers query-part for particular feature. Consumes query feature instance
- `ForCommand<TFeature, TCommand1...TCommandN>(TFeature feature, Saver<TCommand1,..., TCommandN> saver)`: registers command feature instance along with corresponding saver instance. Сoherence of particular command feature and it's saver is controlled in compile-time by `Produces<...>` interface. So your command feature must implement `Produces<Command1,...,CommandN>` and your saver must inherit `SaverBase<Command1,...,CommandN>` of the same commands in exactly the same order to be registered.


