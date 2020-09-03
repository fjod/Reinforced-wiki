# The concept

Tecture gives you ability to [[capture test data|Test-Data]] and to [[generate validation|Generate-Validation]]. The idea is to extract piece of business logic into unit test project, set up infrastructure using `TectureBuilder`, do several "code-build-debug" cycles until feature will not be implemented and after that, capture final test data, generate validation and leave test method for further regression checks.

# Set up infrastructure

In playground I use typical [unit test project](https://github.com/reinforced/Reinforced.Tecture/tree/master/Playground/Reinforced.Samples.ToyFactory.Tests) with xUnit connected. Also I've made several infrastructure classes that invoke test data capturing, validation generation and write resulting files under the certain folder of tests project. [Feel free to borrow them](https://github.com/reinforced/Reinforced.Tecture/tree/master/Playground/Reinforced.Samples.ToyFactory.Tests/Infrastructure), but do not hesitate to create yours if you feel not comfortable with my setup.

## How I do that
- Create unit test class that inherits `TectureTestBase`
- Create test method that validates particular case
- Put the following line into it:
```csharp
using var c = Case(out ITecture ctx);
```
- Call my services, logic or queries using `ctx` variable
- Run/debug unit test on local database
- If it succeeds - I receive `%TestName%_Validation.cs` and `%TestName_TestData.cs` in the folder `%TestName%` near the test class
- I turn my initial using into
```csharp
using var c = Case<%TestName%_TestData>(out ITecture ctx);
```
and append validation at the end:
```csharp
c.Validate<%TestName%_Validation>();
```

As result, I get test like [this one](https://github.com/reinforced/Reinforced.Tecture/blob/master/Playground/Reinforced.Samples.ToyFactory.Tests/WarehouseTests/ManageTests.cs#L39) with the following [test data](https://github.com/reinforced/Reinforced.Tecture/blob/master/Playground/Reinforced.Samples.ToyFactory.Tests/WarehouseTests/RenameMeasurementUnit/RenameMeasurementUnit_TestData.cs) and following [validation](https://github.com/reinforced/Reinforced.Tecture/blob/master/Playground/Reinforced.Samples.ToyFactory.Tests/WarehouseTests/RenameMeasurementUnit/RenameMeasurementUnit_Validation.cs).

Finally, I run test method again to ensure that it works fast enough and does not use database.

# Why this approach is trustful?

This approach validates several aspects of your business logic at once and reports if something has been changed. To be exact, Tecture validates:

## Query order

Such test data class reproduces all the responses *sequentially*, step by step. So test data retrieval will fail if query order is violated. This circumstance is used as part of business logic validation during tests. Keep in mind that test data relies on circumstance that *particular logic* do produce *particular queries* in *particular order*.

*Example*: you have changed the method `DoBusiness` of service `A` that is being used from method `CreateOrder` in service `B`. If there are properly composed Tecture unit tests validating logic of `B.CreateOrder` and your modifications of method `DoBusiness` break query order - then unit test for `B.CreateOrder` will fail and rise the red flag of affected functionality. It allows you to keep track of functionality consistency across all of your application.

## Query hashes

Test data entries contains hash of the query. It is being computed by feature that was used to produce this query. Hash is needed to check query consistency itself (even if query order seems to be maintained). 

*Example:* the same setup with services `A` and `B` but you changed some `.Where` criterias. ORM feature will compute the hash of expression and if it is different - test fails again because we cannot be sure anymore that logic works correctly.

## Commands order

Basically Tecture ensures that commands in trace follows one after another and if validation says that after command of type `Add` there must be command of type `Save` (synthetic command that denotes that `Save`/`SaveAsync` call happened at this point of the trace). So if order is violated - Tecture throws meaningful exception. 

## Commands consistence

Basic Tecture checks validates command annotation to correspond to initial value. Can be disabled by removing `.Basics` call from validation generator's configuration ([here](https://github.com/reinforced/Reinforced.Tecture/blob/master/Playground/Reinforced.Samples.ToyFactory.Tests/Infrastructure/TectureCase.cs#L49)).

Default checks for `Add` and `Delete`/`DeletePk` commands validate the state of entity that is being added or deleted. Default checks for SqlStroke validates SQL command checks and parameters... et cetera, et cetera.

So therefore we have plenty of check places that are basically enough to catch critical or ruining regression changes in business logic. The more tests you create - the more cases you cover - the more stability you have.

# What I do if unit test fails?

Pay attention.

- You can ignore particular test for a while. Not recommended.
- If you are sure that your modifications comply with requirements, you can remove old test data, re-run test and capture new test data. Recommended.
- You can rewrite this unit test or completely remove it if you consider requirements for underlying functionality completely changed.

