# The concept

Saving is a process when Tecture dispatches [[commands]] queue. It is executed by picking commands from queue one-by-one, locating corresponding runner and passing command to it. Then it runs `Save`/`SaveAsync` methods of corresponding savers. After that it sorts out all the [[post-save|Services]] deferred commands. This process happens again and again until commands queue is empty.

# Queue dispatching process in pseudo-code

```
void Dispatch() {
	while(queue.HasCommands)
	{
	   var chunk = queue.Clone();
	   queue.Purge();
	   while(chunk.HasCommands)
	   {
		  var command = chunk.Dequeue();
		  var runner = LocateRunner(command);
		  runner.Run(command);
	   }
	   Savers.Each(x => x.Save());
	   RunActions(AfterSave);
	}
}

Dispatch();
DisableAddition(Finally);
RunActions(Finally);
Dispatch();
```

So savers and runners can be invoked multiple times. Dispatching strategy may change in the future. 

# Async save

Using `ITecture` interface you can call `Save` or `SaveAsync`. The last one does exactly the same, but uses `RunAsync` of runners and `SaveAsync` for savers providing `await` for each one that can help to save threads of main application.

# Transactions

Before saving data, Tecture uses `ITransactionManager` instance supplied to `TectureBuilder` in order to open transaction for saving. If runners and savers do not fail then Tecture commits the transaction. `ITransactionManager` must return suitable `IOuterTransaction` implementation that will open and commit/reject the transaction if necessary.

**Warning!** Transactions mechanism might be changed soon.