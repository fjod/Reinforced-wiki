# The concept

Channel is intrinsic Tecture abstraction. Technically it wont be instantiated. In fact, Tecture needs channel just for its type metadata. 

The concept of channels can be understood as *reference to external system*. Logic in tecture is separated to [[commands]] and [[queries]], but they all have to be addressed to some channel. E.g.: `Db` channel will be used to address relational database for query or command purposes. 

# Defining channel

Basically, channel is an interface:

```csharp
public interface Db : { }
```

Do not start its name from letter `I`, cause channel is non-common usage of interfaces and they are bit off conventions.

The behavior of a channel is being defined by its [[features]] set. Every assembly with Tecture feature supplies necessary `Command` and `Query` aspects of feature. So in order to declare channel that supports O/RM feature (means: allows to work with it using O/RM practices) you have to extend your channel with `CommandQueryChannel<,>` interface, supplying both `Command` and `Query` classes as its type parameters:

```csharp
public interface Db :
        CommandQueryChannel<Reinforced.Tecture.Features.Orm.Command, Reinforced.Tecture.Features.Orm.Query>
    { }
```

Also you can use `Command` and `Query` channel capabilities separately. E.g. channel below supports command capabilities from O/RM feature and query capablities of SqlStroke feature:

```csharp
public interface Db :
        CommandChannel<Reinforced.Tecture.Features.Orm.Command>,
        QueryChannel<Reinforced.Tecture.Features.SqlStroke.Query>
    { }
```

Practically it means that you will have `Db` channel that you can Add/Remove/Update entities using O/RM but query only using direct SQL.

# Channel usage

Channels are being used from [[services]]. There are protected `From<>` and `To<>` methods in service:
- `From<>` gives you *reading* capabilities of the channel (aka "read end of channel")
- `To<>` gives you *writing* capabilities of the channel (aka "write end of channel")

Both of these methods consume channel as type parameter. Consider following example of accessing channel within service of 2 entities (`Product` and `Order`):

```csharp
public class SeriousService : TectureService<Product /*, Order */>, INoContext
{
	private SeriousService() { }

	public void DoBusinessLogic()
	{
		var order = From<Db>().Get<Order>().ById(10);

		To<Db>().Add(new Product() {Name = "New one"});

		To<Db>().Add(new Order() {Name = "New one"}); // <- Compile-time error: 
		// ^-- cannot add Order because this service does not modify Orders
	}
}
```

Read usage of the channel can be used directly. `From<>` method is available on `ITecture` inteface. See [[Tecture and Ioc|Ioc]].

# Relation between channels and features

Simple. By example: if `Db` channel does not implement `CommandChannel<Orm.Command>` - there is no extension method `.Add`, which would produce compile-time error. It forbids you from adding entities into message queue and forces  to keep channel contract.