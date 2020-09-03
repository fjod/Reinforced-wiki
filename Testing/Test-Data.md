# The concept

Trace embraces copies (deep clones) of all the results of the [[queries]] made through Tecture infrastructure. This circumstance allows us to persist query results and reproduce them without external infrastructure. The most efficient way to store query results is... C# code. It allows to maintain compile-time checks of the test data and brings unit testing experience on the new level. 

Well, actually automated capturing of query results sets you free from typing test data manually and/or maintain test infrastructure setup. It is intended to reduce test maintenance costs.

Automated test data capture is core concept that is needed to support [data-driven testing](https://en.wikipedia.org/wiki/Data-driven_testing) within Tecture.

# How?

Data itself is being already captured by `Trace`. In order to serialize it into the C# code you have to install `Reinforced.Tecture.Testing` NuGet package. Test data serialization capabilities are not included into core because they use [Roslyn](https://github.com/dotnet/roslyn) for code generation whether Tecture core maintains 0-dependency.

```bash
> Install-Package Reinforced.Tecture.Testing
```

This package supplies 2 extension methods for `Trace`:
- `GenerateTestData`: serializes data from captured queries into C# code. Explained below.
- `GenerateUnitTest`: serializes commands captured within `Trace` into validation code. Explained [[here|Generate-Validation]].

# Generate test data

Use following code to generate test data:
```csharp
Trace t = _tecture.EndTrace();
var output = t.GenerateTestData("ClassWithTestData", "Namespace.Of.My.Tests");
output.ToFile(@"Full\Path\To\Code\File.cs");
```
It will generate `File.cs` within `Full\Path\To\Code\` folder. File with test data will contain something like this:

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using Reinforced.Tecture.Testing.Data;
using Reinforced.Samples.ToyFactory.Logic.Warehouse.Entities;

namespace Reinforced.Samples.ToyFactory.Tests.WarehouseTests.RenameMeasurementUnit
{
		class RenameMeasurementUnit_TestData : CSharpTestData
		{
			private Boolean GetEntry_1()
			{ 
				return false;
			}

			private Int32 GetEntry_2()
			{ 
				return 42;
			}

			private MeasurementUnit GetEntry_3()
			{ 
				var v1 = New<MeasurementUnit>();
				Set(v1, x=>x.Id, 42);
				Set(v1, x=>x.ShortName, @"kG");
				Set(v1, x=>x.Name, @"Kilograms");
				return v1;
			}

			public override IEnumerable<ITestDataRecord> GetRecords()
			{ 
				yield return new TestDataRecord<Boolean>(GetEntry_1()) { 
					Hash = @"OrmQuery_AB78B1DAADDAD273C885641045DBA4575D5FF2948319913F64C58E6FC40F215",
					Description = @"check unit existence"
				};
				yield return new TestDataRecord<Int32>(GetEntry_2()) { 
					Hash = @"ORM_AdditionPK_1",
					Description = @"ORM Addition PK retrieval"
				};
				yield return new TestDataRecord<MeasurementUnit>(GetEntry_3()) { 
					Hash = @"OrmQuery_733639199A5697167B201F9165F5D2487D9588D52EC594686EF5C98F2D619B",
					Description = @"Get MeasurementUnit by Id #42 (required)"
				};
			}

		}
}
```

# Test generation customization

You can substitute values of particular query result that test generator will embed into resulting code. Like this:
```csharp
Trace t = _tecture.EndTrace();

var output = t.GenerateTestData("ClassWithTestData", "Namespace.Of.My.Tests",
                x => x.WithHijack(d =>
                  {
                      d.For<Product>().Hijack(x => x.Name, x => "Fake product name");
                  }));
				  
output.ToFile(@"Full\Path\To\Code\File.cs");
```
This will make test generator to embed `"Fake product name"` instead of actual data in `Name` property of every `Product` that appears in query results.