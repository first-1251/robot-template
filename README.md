# Robot Template

A work-in-progress project which can be cloned as a base for all future robots. This project provides a consistent
foundation for the Team 1251 projects along with documentation on related practices and patterns.

There are `README.md` files in each of the packages which talk more about how the package is to be used.

## Dependency Injection vs. Static Access

The WPILib, Command-Based programming documentation relies heavily on making  `Subsystems`, `Commands`, and other 
components statically accessible. 

By contrast, Team 1251 has chosen to use more [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
because of the specific benefits it offers:

- More obvious dependency graph

  Static access has the benefit of simplifying class interfaces because instances do not need to be passed around as much.
  However, it makes dependencies hard to track in very much the same way global variables do in other languages because
  it creates _hidden dependencies_ -- dependencies which are not visible from the interface of the class(es).

  Here is a stack overflow question/answer that talks more about `Hidden Dependencies`:

  https://stackoverflow.com/questions/46616222/what-are-hidden-dependencies

- More predictable creation order

  Using dependency injection forces you to think about the order in which objects are created. All too often, we've run
  into problems where we have had to tweak the order of our static properties to try to get the classes created in a
  sequence that fulfills the dependency graph.  With the hidden dependencies created by static access (see 
  first bullet point), this often became a frustrating guessing game of trial and error. 

Using dependency injection is not without its challenges. One of the more interesting obstacles was a 
["chicken or egg"](https://en.wikipedia.org/wiki/Chicken_or_the_egg) problem relating to default commands...

The WPILib documentation says that default commands are assigned by implementing `Subsystem.initDefaultCommand()`:

```java
class MySubsystem extends Subsystem {
    // ...
    
    @Override
    protected void initDefaultCommand() {
        setDefaultCommand(new MyDefaultCommand());
    }
}
```

Ideally, we would be _injecting_ that default command instead of creating it from within the subsystem. That seems easy
enough, but there is another rule that says that a command must `require()` any subsystem that it is a default command 
for. This creates a circular dependency between the `Subsystem` and its default`Command`.

Thankfully, this ended up being easier to solve that we initially thought. After digging into the code we found that you
can simply call `setDefaultCommand()` at any time after creating the `Subsystem` to assign a default command. So this
becomes possible:

```java
class Robot {
    //...
    
    public void robotInit() {
        DriveTrain driveTrain = new DriveTrain();
        TeleopDrive teleopDrive = new TeleopDrive(driveTrain);
        driveTrain.setDefaultCommand(teleopDrive);
    }
}

```