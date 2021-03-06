# The concept

Features define [[channel's|channels]] contract that Tecture forces you to conform with. They do declare *possible channel capabilities*. Technically they define what methods are available after calling `From<Channel>` and `To<Channel>`. Features are pure abstract. Ideal feature does not declare any executable code. But it is still good place to perform syntax sugar around channel's read and write ends (inputs and outputs). Features are delivered as separate NuGet packages and created from scratch. Usually system architect creates new features.

Currently there are 2 features available for Tecture: [ORM](https://github.com/reinforced/Reinforced.Tecture/tree/master/Features/Reinforced.Tecture.Features.Orm) and [SqlStroke](https://github.com/reinforced/Reinforced.Tecture/tree/master/Features/Reinforced.Tecture.Features.SqlStroke).

# Using features with channels

Features do supply `Command` and `Query` classes from their root. These classes are intended to implement common feature's functionality that is usually hidden from feature users by `protected` and `internal` interfaces. See [example of feature code](https://github.com/reinforced/Reinforced.Tecture/tree/master/Features/Reinforced.Tecture.Features.Orm) (O/RM feature). If you act as feature consumer (means: you want to just use feature) then *you don't have* to care about feature internals. Just connect it to your channel by extending channel's interface with either:
- `CommandChannel<_Command_>` - where `_Command_` is a feature of `Command` class. This allows you to use *write* capabilities of the channel
- `QueryChannel<_Query_>` - where `_Query_` is  a feature `Query` class. This allows you to use *read* capabilities of the channel
- `CommandQueryChannel<_Command_,_Query_>` - just both of above combined

You can connect as much features to your channel as you want. Examples:

```csharp

// Database supporting reading and writing through ORM
public interface Db1 :
        CommandQueryChannel<Reinforced.Tecture.Features.Orm.Command, Reinforced.Tecture.Features.Orm.Query>
    { }

// Database supporting writing by ORM but querying by direct SQL
public interface Db2 :
        CommandChannel<Reinforced.Tecture.Features.Orm.Command>,
        QueryChannel<Reinforced.Tecture.Features.SqlStroke.Query>
    { }

// Database that is impossible to write to - only query by SQL or ORM
public interface Db3 :
        QueryChannel<Reinforced.Tecture.Features.Orm.Command>,
        QueryChannel<Reinforced.Tecture.Features.SqlStroke.Query>
    { }

// Database supporting everything
public interface Db4 :
        CommandQueryChannel<Reinforced.Tecture.Features.Orm.Command, Reinforced.Tecture.Features.Orm.Query>,
        CommandQueryChannel<Reinforced.Tecture.Features.SqlStroke.Command, Reinforced.Tecture.Features.SqlStroke.Query>
    { }
```

Notice `Db3` channel. Comment says "it is impossible to write to it". It means that you *technically* will not have tools to write into this DB. If you try - you will get *compile-time* error.