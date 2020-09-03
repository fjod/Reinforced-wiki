# The concept

Runtime implements [[features]]. Single runtime may implement one or more features usually using some 3rd-party technology under the hood. Runtimes are being delivered by separate NuGet packages. Currently Tecture offers single runtime for [ORM](https://github.com/reinforced/Reinforced.Tecture/tree/master/Features/Reinforced.Tecture.Features.Orm) and [SqlStroke](https://github.com/reinforced/Reinforced.Tecture/tree/master/Features/Reinforced.Tecture.Features.SqlStroke) features based on [EntityFramework Core](https://docs.microsoft.com/en-us/ef/core/). This runtime is called [Reinforced.Tecture.Runtimes.EFCore](https://github.com/reinforced/Reinforced.Tecture/tree/master/Runtimes/Reinforced.Tecture.Runtimes.EFCore) and available as NuGet package. 

Keep in mind that runtime may contain dependencies on 3rd-party packages whether features usually do not. So in order to use EF.Core runtime, you have to install it:

```sh
> Install-Package Reinforced.Tecture.Runtimes.EfCore
```

# Bind channels to runtimes

Bindind channels to exact runtime is pretty similar to typical IoC registration, but usually is much shorter.

Runtimes are being connected to each channel according to features that they use. All the features used by channel must be fullfilled with runtimes when [[integrating|Ioc]] Tecture into your system. This can be done when you create `TectureBuilder` object as follows:

```csharp
var ld = new LazyDisposable<MyDbContext>(() => new MyDbContext());

var tb = new TectureBuilder();
tb.WithChannel<Db>(c =>
{
	// these extensions are being supplied by runtime
	c.UseEfCoreOrm(ld); 
	c.UseEfCoreDirectSql(ld);
	// of course, if Db does not implement necessary features, 
	// this code produces compile-time error
});

ITecture tecture = tb.Build();
```

`Build` method of `TectureBuilder` implicitly validates that channels are correctly connected to runtimes. Unfortunately, there is no technical way to check that all the channels that are being used in services are correctly initialized, so be careful. Of course, Tecture will throw runtime exception if it encounders runtime-unbound channel usage. That is best that it can.

Also this is affordable:

```csharp
c.UseEfCoreDirectSqlCommand(ld);
c.UseEfCoreOrmCommand(ld);
```

Runtimes supply extension methods for binding only query or only command channel ends. Keep it in mind when your channel extends `CommandChannel<>` and `QueryChannel<>` but not `CommandQueryChannel<,>`.