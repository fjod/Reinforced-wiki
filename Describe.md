# The concept

Every query that is being made through Tecture and every command that is enqueued can be explained with few words using `.Describe` and `.Annotate` methods. The text that you write within these methods is being used in textual representation of the [[trace|tracing]]. So textual descriptions of several traces captured on same piece of business logic under different input conditions and different data from channels can give you an understanding what *actually* happens inside your business logic. 

In perfect world, this information can be presented to customer/product owner/other non-IT people for the sake of finding common language and improving the quality of requirements.

Unfortunately, Tecture can not and should not know what are you trying to achieve *semantically*. It is not aware of the purpose of particular command or the meaning of particular query, but you can provide small description by yourself that will be always included into textual representation of the trace.

So your business logic becomes *dynamically self-explanatory* when run under different conditions.

# Command annotations

Every [[feature|features]] method returns an instance of the command. `CommandBase` contains `Annotation` property and fluent method `.Annotate` that allows you to describe the semantic sense of what this command means and why we used the command in the first place.

Example: we reduce tax fee rate if order subtotal is less than $1000

```csharp
decimal total = order.Subtotal;
foreach(var g in goods)
{
	To<Db>.Add(new Good {
		OrderId = order.Id, 
		Price = g.Price,
		Quantity = g.Quantity
	}).Annotate($"attach {g.Quantity} products of ${g.Price} to the order");
	
	total += g.Price * g.Quantity;
}

if (total < 1000)
{
	To<Db>().Update(order)
		.Set(x => x.TaxRate, 0.01)
		.Annotate($"reduce order tax rate (subtotal is less than $1000) because accountant said so");
} else 
{
	Comment($"order tax rate remains unchanged");
}
```

Trace explanation of this logic will look following:

```
...
N. [ADD] attach 2 products of $10 to the order
N. [ADD] attach 4 products of $5 to the order
N. [ADD] attach 8 products of $15 to the order
N. [UPD] reduce order tax rate (subtotal is less than $1000) because accountant said so
...
```

But under different input trace explanation will look different. Like this:

```
...
N. [ADD] attach 2 products of $800 to the order
N. [COMMENT] order tax rate remains unchanged
...
```

# Query descriptions

Queries can be described using `.Describe` method. All the [[features]] must supply their own `.Describe` method implementation. That is how it works in ORM and SqlStroke features:

```csharp
                               // here!
if (From<Db>().All<Resource>().Describe("check resource existence").Any(x => x.Name == name))
{
	throw new Exception($"Cannot add resource '{name}' because it already exists");
}
var desired = 50;
var resources = From<Db>()
	.SqlQuery<Resource>(x => $"SELECT * FROM {x} WHERE {x.Quantity > desired}")
	.Describe($"obtain all resources with quantity more than {desired}") // <- here!
	.As<Resource>();
```

So the trace explanation be like:
```
...
N. [QRY] check resource existence: 'False' obtained
N. [QRY] obtain all resources with quantity more than 50: 10 results obtained
...
```

# `Comment` method

There is `Comment` method available within [[service|services]]. Formally it enqueues empty command into the queue. Empty command holds it's annotation. This is needed to improve trace explanation, like this:

```csharp
if (condition)
{
	Comment("Do this");
} else 
{
	Comment("Do that");
}
```

"Do this" or "Do that" will appear in the trace explanation according to condition.

# `Cycle` method

Cycle explanation is used to shorten trace explanation on repetitive commands. Usage as follows:

```csharp
using (var c = Cycle("adding products"))
{
	foreach (var g in goods)
	{
		To<Db>().Add(new Product() {Name = g.Name});
		c.Iteration("good added");
	}
}
```

And trace explanation be like :

```
N. [CYC] Cycle of adding products
N. [ADD] Add Product 
N. [ITR] --- Cycle iteration ---
N. [CEND] adding products ends in X iterations and produces Y commands
```