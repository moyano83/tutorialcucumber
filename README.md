# Cucumber

## Introduction
Cucumber reads executable specifications written in plain text and validates that the software does what those 
specifications say. Cucumber uses Gherkin documents which are stored in _.feature_ text files and are typically 
versioned in source control alongside the software. Cucumber also needs a set of step definitions.

## Step Definitions
A Step Definition is a Java method with an expression that links it to one or more Gherkin steps:

```java
package foo;
import cucumber.api.java.en.Given;

public class MyStepDefs {
    @Given("I have (\\d+) cukes in my belly")
    public void i_have_n_cukes_in_my_belly(int cukes) {
        System.out.format("Cukes: %n\n", cukes);
    }
}
```

### Expressions
A step definition’s expression can either be a _Regular Expression_ or a _Cucumber Expression_. If you prefer to use 
Regular Expressions, each capture group from the match will be passed as arguments to the step definition’s method.
A step definition can transfer state to a subsequent step definition by storing state in instance variables. The 
definitions aren’t linked to a particular feature file or scenario. If Cucumber encounters a Gherkin step without a 
matching step definition, it will print a step definition snippet with a matching Cucumber Expression as an example.

## Cucumber Expressions
A cucumber expression is simpler than the regex expressions, instead of the capturing group, you put the type of the 
parameters between bracets:

```text
I have {int} cars
```

Apart of word (single word), string (single-quoted or double-quoted strings), int, float, biginteger, bigdecimal, 
byte, short, long and double, you can define custom parameter types.

### Custom Parameter types
we can define a custom parameter type in Cucumber's configuration:

```text
typeRegistry.defineParameterType(new ParameterType<>(
    "color",           // name:The name the parameter type will be recognised by in output parameters 
    "red|blue|yellow", // regexp: A regexp that will match the parameter. May include capture groups
    Color.class,       // type: The return type of the transformer method
    Color::new         // transformer function: A method that transforms the match from the regexp. Must have arity 
    1 if the regexp doesn't have any capture groups. Otherwise the arity must match the number of capture groups in regexp
))
```

Optional text is noted by parentheses like in `I have {int} car(s)`. You can put alternative text (different words 
matching a sentence) like in `I have {int} car/motorbike`. The escaping character is _\\_.

## Tag Expressions
Tag expressions let you select a subset of scenarios, based on tags. A tag expression is simply an infix boolean 
expression. Parentheses can be used for clarity or changing the operator precedence:

```text
@fast: Scenarios tagged with @fast
@wip and not @slow: Scenarios tagged with @wip that aren't also tagged with @slow
@smoke and @fast: Scenarios tagged with both @smoke and @fast
@gui or @database: Scenarios tagged with either @gui and @database
```

## State
It’s important to prevent state created by one scenario from leaking into others:

    * Avoid using global or static variables
    * Make sure you clean your database (or resources) between scenarios
    
Cucumber creates a new instance of each of your glue code classes (step definitions and hooks)  before each scenario.
The recommended approach to clean a database between scenarios is to use a _Before_ hook to remove all data before a 
scenario starts. The _After_ hook, as it allows you to perform a post-mortem inspection of the database if a 
scenario fails.
If your database supports it, you can wrap a transaction around each scenario, you need to tell Cucumber to start a 
transaction in a _Before_ hook, and later roll it back in an _After_ hook. This is such a common thing to do that  
several Cucumber extensions provide ready-to-use tagged hooks using a tag named _@txn_.

```text
@txn
Feature: Let's write a lot of stuff to the DB

 Scenario: I clean up after myself
   Given I write to the DB

 Scenario: And so do I!
   Given I write to the DB
```

## Cucumber Reference
### Step Arguments
The number of parameters in the method has to match the number of capture group s in the expression. If we need to 
pass a list of strings to a method, we can use a DataTable:

```text
Given the following animals:
  | cow   |
  | horse |
```

Which will be flatten to:

```java
@Given("the following animals:")
public void the_following_animals(List<String> animals) {...}
```

### Matching steps

    * Cucumber matches a step against a step definition’s Regexp
    * Cucumber gathers any capture groups or variables
    * Cucumber passes them to the step definition’s method and executes it

Once execution begins, for each step, Cucumber will look for a registered step definition with a matching Regexp. If it finds one, it will execute it, passing all capture groups and variables from the Regexp as arguments to the method or function.

### Steps results
    
    * Success: If the block in the step definition doesn’t raise an error, the step is marked as successful
    * Undefined: When Cucumber can’t find a matching step definition, all subsequent steps in the scenario are skipped
    * Pending: When a step definition’s method or function invokes the pending method, indicating that you have work
     to do
    * Failed Steps: When an error is raised, the step is marked as failed. Returning null or false will not cause a 
    step definition to fail
    * Skipped: Steps that follow undefined, pending, or failed steps are never executed
    * Ambiguous: Non unique steps
    
### Hooks
Hooks are blocks of code that can run at various points in the Cucumber execution cycle. Typically used for setup and
 teardown of the environment before and after each scenario.

```java
@Before(order = 10) // Specifies the order in which is executed
public void doSomethingBefore() {...}

// Lambda style:
Before(() -> {...});
```

Consider using a background as a more explicit alternative, only use a Before hook for low-level logic such as 
starting a browser or deleting data from a database.

The _After_ hooks run after the last step of each scenario, even when steps are failed, undefined, pending, or skipped.

```java
@After
public void doSomethingAfter(Scenario scenario){
    //The scenario parameter is optional, you can inspect the status of the scenario
    if (scenario.isFailed()) {
        byte[] screenshot = webDriver.getScreenshotAs(OutputType.BYTES);
        scenario.embed(screenshot, "image/png");
    }
}
```

### Tagged hooks
Hooks can be conditionally selected for execution based on the tags of the scenario. 

```java
@After("@browser and not @headless")
public void doSomethingAfter(){...}
```

### Tags
Used to organise your features and scenarios, you can put them in Gherkin on the 'Feature', 'Scenario', 'Scenario 
Outline' and Examples':

```text
@billing
Feature: Verify billing

  @important @missing
  Scenario: Missing product description
    Given hello

  Scenario: Several products
    Given hello
```

You can run a subset of the scenarios with the command line: `mvn test -Dcucumber.options='--tags "@smoke and @fast"'`
 or by adding them to your Junit class:

```java
@Cucumber.Options(tags = "@smoke and @fast")
public class RunCucumberTest {...}
```
