Welcome to **Reinforced.Tecture** documentation. This is guide through main Tecture concepts and features. 

It will be best if you'd follow it step by step *from the top to the bottom*.

Enjoy!

## Basics
Core concepts of Tecture getting explained
 - [[Channels]]: abstract type-bound external system delimiters
 - [[Features]]: defining the channel's functionality
 - [[Entities]]: common game rules regarding entities
## Writing code
How to write code with Tecture and implement required functionality.
 - [[Services]]: the place for your business logic
 - [[Queries]]: how to properly query data from external sources
 - [[Commands]]: a way to perform exact action on external system
## Integration
How to use Tecture with your application
 - [[Tecture and Ioc|Ioc]]: how to propagate tecture access points through the IoC
 - [[Saving]]: how to apply changes on external systems
## Testing with Tecture
Testing, workflow and informational capabilitites of Tecture are being explained here
 - [[Tracing]]: how to collect information about what it happening
 - [[Capture test data|Test-Data]]: extract test data from the trace
 - [[Generate validation|Generate-Validation]]: generate command flow validation from the trace
 - [[Create unit test|Unit-Test]]: combine test data with validation and get meaningful and lightweight unit tests that do not require infrastructure
# Customization and add-ons
Writing extensions for Tecture in order to best-fit your workflow
 - [[How to create feature|How-to-create-feature]]: protocol of creating feature for channel
 - [[How to implement runtime|How-to-implement-runtime]]: protocol of implementing channel features
