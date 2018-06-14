# Gherkin Reference
Gherkin uses a set of special keywords to give structure and meaning to executable specifications.

## Feature
Provides a high-level description of a software feature, groups related scenarios.

```text
Feature: Adding Item to shopping cart
This is a description of the feature
splited in different lines
lines can't start with reserved keywords
```

You can place _tags_ above Feature to group related features.

## Example
Concrete example that illustrates a business rule. It consists of a list of steps. Examples follow this same pattern:

    * Describe an initial context (Given steps)
    * Describe an event (When steps)
    * Describe an expected outcome (Then steps)
    
## Steps
Cucumber executes each step in a scenario one at a time, in the sequence you’ve written them in. Steps should have 
different text than other steps, even if the keyword is different

    * Given: Used to describe the initial context of the system (preset). The purpose of it is to put the system in a
     known state before executing the test. There can be more than one 'Given' step, but you should use 'And' or 'But' 
     to connect them.
    * When: Used to describe an event or action.There should be a single 'When' per 'Scenario'
    * Then: Used to describe an expected outcome, or result. It should use an assertion to compare the actual outcome
     with the expected outcome. Verify outcome that is observable for the user or external system.
    * And, But: Connectors for 'Given', 'When' or 'Then' to make the tests more readable
    
## Background
If a set of Steps are repeated in every scenario, consider grouping them into the 'Background' section. 'Background'
 is run before each scenario, but after any _Before hook_. Only one 'Background' is allowed per 'Feature'.
 
    * Put it before the first 'Scenario'
    * Don use it to set complicated states
    * Keep it short

## Scenario Outline
The Scenario Outline keyword can be used to run the same Scenario multiple times, with different combinations of values.
It must contain an Examples section with the values to replace the delimited parameters (using <>)

```text
Scenario Outline: eating
  Given there are <start> cucumbers
  When I eat <eat> cucumbers
  Then I should have <left> cucumbers

  Examples:
    | start | eat | left |
    |    12 |   5 |    7 |
    |    20 |   5 |   15 |
```

## Step Arguments
To pass more data to a step than fits on a single line use:

    * Doc Strings: Delimited by three double-quote marks. It will automatically be passed as the last argument in 
    the step definition
    * Data Tables: Like the example before. It will be passed to the step definition as the last argument
    
## Language
The language you choose for Gherkin should be the same language your users and domain experts use when they talk 
 about the domain. To define the language, put the header `# language: <the two letter language identifier>` on the 
 first line of a feature file. 
 
## Step Organization
All of your step definitions can be in one file, or many. Regardless of the directory structure employed, Cucumber 
flattens the features/ directory tree when running tests. Anything ending in _.java_ under the starting point for a 
Cucumber run is searched for 'Feature' matches. The recommendation is to create a separate _Steps.java_ file for each
domain concept. Have one file for each major  domain object. Only implement step definitions that you actually need.
Don’t write step definitions for steps that are not present in one of your scenarios.