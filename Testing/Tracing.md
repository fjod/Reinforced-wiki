# The concept

Trace in Tecture is sequential recording of all made [[queries]]  and all enqueued [[commands]]. Trace is exposed by `Trace` class. It can be obtained by calling `BeginTrace`/`EndTrace` of the `ITecture` instance. Everything that happened between `BeginTrace` and `EndTrace` will be reflected within trace.

Using information from `Trace` you can [[catch test data, serialize it into C# code|Test-Data]] and [[generate code to validate the trace|Generate-Validation]], therefore you can create unit tests of your logic almost automatically.

# `Trace` contains

- `All`/`Commands`/`Queries` collections: sequences of tracing records in a form of commands, allows to proceed programmaticaly
- `OfChannel<>` method that retrieves another `Trace` object containing commands related only to particular channel
- `Begins` method that starts trace validation. This method is not intended to be called manually: see [[how to generate validation|Generate-Validation]] out of trace and to create unit tests
- `ToText`: this method gives trace explanation: human-readable representation of what happened within the trace step by step. See [[describe]] section about how to use and improve textual trace explanations.

# Difference between the `Trace` and logging

Tecture trace is not intended to contain any *technical* information regarding to what is happening. It should not contain records about initialization of something, calling of methods etc. It should contain brief explanation of *what is being _sent to_ and _received from_ external systems*. So trace will give you information about what will go to database, message queue, e-mail and so on at the end. It does reveal information about *what* decisions were taken by your code, but should *not* explain how these decisions were made. 

# Trace text example

```
1. [QRY] Check unit existence: 'False' obtained

2. [ADD] Add create measurement unit 'Kilograms' (kG)

====== Saved =====

3. [QRY] [TEST DATA] ORM Addition PK retrieval

4. [QRY] Get MeasurementUnit by Id #42 (required)

5. Update 

====== Saved =====

6. [DPK] remove measurement unit#42

====== Saved =====

======  END  =====
```

You can [[describe queries and annotate commands|describe]] in order to get more explanatory trace text.