# The concept

Since we have queue of the commands appearing in particular order in context of particular user input and particular data in channels and *we consider this behavior correct and complying with current business requirements* - we can automatically generate code that will validate this case and wrap it into regression data-driven unit test.

In other words, as soon as we've got business logic scenario that satisfies business requirements, we can [[capture test data|Test-Data]] for it and produce code that validates the fact that this business logic under certain conditions *produces exactly desired set of commands*. This formal check is intended to be left in the code, can run without any infrastructure and will make sure that the process of order creation works exactly the same as 10 years ago and we did not ruin it by new features, bugfixes and other code modifications.

# How to generate validation code

Just like in case of [[test data capturing|Test-Data]] you need to install `Reinforced.Tecture.Testing` NuGet package.

```bash
> Install-Package Reinforced.Tecture.Testing
```

Then you have to capture the [[trace|tracing]] and use method `GenerateUnitTest` of it like that:
```csharp
Trace t = _tecture.EndTrace();

var output = t.GenerateUnitTest("MyBusinessCaseValidation", "Namespace.Of.My.Tests", g =>
            {
                g.CheckOrm(); // <- generate checks for ORM commands supplied by ORM feature
                g.CheckSql(); // <- generate checks for SqlStroke commands supplied by SqlStroke feature
                g.Basics(); // <- generate basic checks supplied by Tecture core
            });
			
output.ToFile(@"Full\Path\To\Code\File.cs");
```

`GenerateUnitTest` method consumes following parameters:
- `className`: name of class that will contain validation code
- `ns`: namespace that will contain the class with validation code
- `config`: which checks will be embedded into resulting validation

Here is the sample of validation class generated for some random business logic:

```csharp
using System;
using Reinforced.Tecture.Testing.Validation;
using Reinforced.Tecture.Tracing;
using Reinforced.Samples.ToyFactory.Logic.Warehouse.Entities;
using Reinforced.Tecture.Features.Orm.Commands.Add;
using Reinforced.Tecture.Tracing.Commands;
using Reinforced.Tecture.Features.Orm.Commands.Update;
using Reinforced.Tecture.Features.Orm.Commands.DeletePk;
using static Reinforced.Tecture.Features.Orm.Testing.Checks.Add.AddChecks;
using static Reinforced.Tecture.Testing.BuiltInChecks.CommonChecks;
using static Reinforced.Tecture.Features.Orm.Testing.Checks.Update.UpdateChecks;
using static Reinforced.Tecture.Features.Orm.Testing.Checks.DeletePk.DeletePKChecks;

namespace Namespace.Of.My.Tests
{
		class MyBusinessCaseValidation: ValidationBase
		{
			protected override void Validate(TraceValidator flow)
			{ 
				flow.Then<Add>
				(
					Add<MeasurementUnit>(x=>
					{ 
						if (x.Name != @"Kilograms") return false;
						if (x.ShortName != @"kG") return false;
						return true;
					}, @"create measurement unit 'Kilograms' (kG)"), 
					Annotated(@"create measurement unit 'Kilograms' (kG)")
				);
				flow.Then<Save>();
				flow.Then<Update>
				(
					Update<MeasurementUnit>(x=>
					{ 
						if (x.Name != @"Kilo") return false;
						if (x.ShortName != @"kg") return false;
						if (x.Id != 42) return false;
						return true;
					}, @"")
				);
				flow.Then<Save>();
				flow.Then<DeletePk>
				(
					DeleteByPK<MeasurementUnit>(@"remove measurement unit#42", 42), 
					Annotated(@"remove measurement unit#42")
				);
				flow.Then<Save>();
				flow.TheEnd();
			}

		}
}

```
The captured trace can be validated with it as follows:
```csharp
Trace t = _tecture.EndTrace();
var validation = new MyBusinessCaseValidation();
validation.Validate(t);
```
Validation class ensures that commands appear in described order and each command satisfy number of checks.

# Checks

Each command that is captured within the trace can be validated against number of *checks*. [[Features]] that supply available commands also supply checks and their descriptions for testing purposes. 

Check is a class that inherits `CommandCheck<TCommand>`. It supplies command that will be validated as type parameter. Typical check overrides following `CommandCheck<>` methods:
- `IsActuallyValid`: determines whether command actually passes check or not
- `GetMessage`: obtains meaningful error message in case if check fails

[Here is example of check](https://github.com/reinforced/Reinforced.Tecture/blob/master/Features/Reinforced.Tecture.Features.Orm/Testing/Checks/Add/AddPredicateCheck.cs) supplied by ORM feature that validates added entity with specified predicate (i.e. function that tell whether added entity is valid or not).

# Checks descriptions

By convention, checks are being instantiated by invoking static method of checks container class. Like that:
```csharp
public static class AddChecks
{
	public static AddPredicateCheck<T> Add<T>(Func<T, bool> predicate, string explanation) => new AddPredicateCheck<T>(predicate, explanation);
}
```
Check descriptions are intended to tell validation generator how to instantiate particular checks. [Here is example](https://github.com/reinforced/Reinforced.Tecture/blob/master/Features/Reinforced.Tecture.Features.Orm/Testing/Checks/Add/Descriptions.cs) description for `Add` command check from example above.

Finally, all the checks supplied by feature are [packed into single extension method](https://github.com/reinforced/Reinforced.Tecture/blob/master/Features/Reinforced.Tecture.Features.Orm/Testing/Checks/Extensions.cs) for `UnitTestGenerator` class, so it can be disassembled back. Therefore, you can select only particular checks that you need.

