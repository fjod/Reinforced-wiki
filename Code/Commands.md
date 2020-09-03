# The concept

Command is *atomic change to external system*. They are not being performed instantly but instead of that, collected into *commands queue* that Tecture maintains. Dispatching of commands queue happens on [[saving]] stage. Set of commands to be enqueued is being defined by [[features]] that are connected to your channel. Usually feature provides set of extension methods for channel's write end that are responsible for proper instantiation and enqueueing commands. 

Technically command is instance of class inherited from `CommandBase`. Good tradition is that features are hiding command constructor behind `internal` modifier so you cannot instantiate them directly. 

# Commands queue

**Commands are not being run immediately!**. Extension methods provided by features only *enqueue commands* into commands queue that is being dispatched when [[saving]] happens. Commands are being saved in strict order and are not designed to be executed in parallel. They are being executed *asynchronously* if you perform [[saving]] via `.SaveAsync` method in order to save your threads. 

Example: this piece of logic code
```csharp
To<Db>.Add(new Product() { Name = "test1" });
To<Db>.Add(new Product() { Name = "test2" });
```

does not actually run addition. It just tells Tecture to remember these addition and execute them on [[saving]] stage sequentially, one after another.

You don't have access to commands queue from `ITecture` interface. Only from services. So if you want to modify data - you have to create and use service via `Do<>` method.

# Dispatching commands

You don't actually need to know how commands are being run unless you are going to implement your own [[features]] or your [[runtimes]].

Tecture itself only initiates commands queue dispatching, traverses it and picks correct tooling to execute command. It does not runs command by itself. [[Features]] and [[Runtimes]] are responsible of exact execution of commands. Technically it involves Savers and Runners:

- *Runner* is class that inherits `CommandRunner<T>` where `T` is `CommandBase` instance. It contains methods `Run` and `RunAsync` to be implemented by [[Runtimes]] that actually execute the command
- *Saver* is class that inherits `SaverBase` and performs any work that has to be done after all command runners completed their job. Also saver is responsible for command runners instantiation.

E.g.: EF.Core runtime calls `SaveChanges` of `DbContext` in saver whether `AddCommandHandler` manipulates changes tracker in order to recognize your entity as added.

[[Features]] usually define base class for saver and all the necessary abstractions whether [[runtimes]] define how *exactly* command must be executed.