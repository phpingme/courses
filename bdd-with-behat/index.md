---
 head:
  title: "BDD with Behat"
 commands:
   - vendor/bin/behat
 template: task+data
---

## Introduction

To develop a software is always about managing a lot of moving parts. And pretty often it gets messy, in a way that even a developer who wrote the code, cannot recover what is going on after 2-3 weeks the code was shipped.  It is an old problem and had a range of mature concepts around that helps to deal with in a sufficient way. Here we will try a BDD, and show how we can control a software's feature set and move quickly forwards using behat. Course data itself is living on [https://github.com/phpingme/courses](https://github.com/phpingme/courses/blob/drafts/bdd-with-behat/index.md), feel free to contribute, it's open source.



## Write Feature and Scenarios
The concept of **Feature** in BDD let us focus on a specific problem from a perspective of certain customer.  To describe this problem in a Feature form we need to provide 2 things:
 - Feature inroduction
 - Scenario

---

### Feature introduction
To introduce a Feature we use quite the same pattern we apply to describe a user story, namely:
```html
   In order to <achieve a goal>
   As a <stakeholder>
   I want to <do something>
```
For a developer the 3'd line is most likely always clear, it describes an outcome of implementation, like *Product could be ordered*, *Post comments can be blocked*... But it's not always clear about first two lines, especially if stuff have to be implemented in a microservice scope, where a customer could be less personalizable.
Here is important to work with PO and apply a [Ubiquitous Language](http://martinfowler.com/bliki/UbiquitousLanguage.html) of a domain you are implementing.

---

### Scenario
**Scenario** is an essential part of the whole BDD process, this is a part that describes a behaviour of our software. Each Scenario follows the following *Given-When-Then* schema:
```html
  Given <initial state>
  When <apply a change to the state>
  Then <verify a changed state>
```

---

### Initiate BDD environment.

To start to write Features, let's first initiate environment where we want to write it.
```bash
$> vendor/bin/behat --init
```
be default it creates a following tree structure:
```bash
.
+-- features
|   +-- bootstrap
|       +-- FeatureContext.php   

```

---

### Create a first feature

In this course we will try to provide a BDD spec for some small but pretty calculator. And we start with our first feature. Let's create a .feature file with the following file path ```features/calculator.feature``` (you can do it with a write click on **features** folder) with a first Scenario:
```
Feature: Calculator
  In order to know the results of calculation
  As a user
  I want to get numbers calculated

  Scenario: Calculate a factorial
    Given i need to calculate a factorial of "5"
    When i execute a factorial calculation
    Then i get "120" back
```

As you see we defined our Feature in a user story form, and then the first Scenario that describes a way a calculation of factorial have to be calculated from a user perspective. The one problem here, we describe here just a "happy path", what if we want to calculate a factorial of 6, should it be still 120?

---

> An actual Task, write the second scenario which checks that factorial of 6 doesn't have to give 120 back.

### Solution
```
Scenario: Calculate a right factorial
  Given i need to calculate a factorial of "6"
  When i execute a factorial calculation
  Then i don't get the "120" back
```



```task+data
---
match_file:
  path: /features/calculator.feature
  match:
   - Then.*(don\'t|not|doesn\'t|didn\'t).*back
---
```

## Implementing test first

Now, since we have our feature described, we can implement a test for it and take an advantage of behat as well.
First of all, we need to convert our feature in a PHP form. For that, let's run:
```sh
$> vendor/bin/behat --append-snippets
```
---

### Tests boilerplate

Try to run behat again, just execute ```vendor/bin/behat```, and see on STDOUT, it has to look smth. like this:
```
2 scenarios (2 pending)
6 steps (2 pending, 4 skipped)
```

Out of 6 Scenario steps we will get 4 methods in **features/bootstrap/FeatureContext.php**, that throws a **Pending** exception, where each method is responsible for a concrete Scenario step.
We have to have the same *Given* and *Then* descriptions in both of our Scenarios, that's why behat apply the same methods in **FeatureContext** class for them, the only thing, that use to be variable - are values provided within method arguments. For example in case of factorial of "5" and factorial of "6".

And we have two different *When*, for them we get another two methods that have to separately check whether calculation is ok or not.

---

### Creating Calculation app

To start implement the test, let's create an empty **Factorial** class in an application root. And make it look like this:
```php
<?php
// path: /Factorial.php

class Factorial {

	public function __invoke() {

	}

}

```


First we want to do is resolve pending steps. To achieve it, we need to overwrite ```throw new PendingException();``` in our autogenerated ```features/bootstrap/FeatureContext.php``` with a real implementation. Let's start with the first **Given** step and put a code like this:
```php
<?php
...
  /**
   * @Given i need to calculate a Factorial of :arg1
   */
  public function iNeedToCalculateAFactorialOf($arg1)
  {
      $this->factorialInput = $arg1;
  }
```

Then we go to the following **When** step, controlled by ```iExecuteAFactorialCalculation``` method, that can be implemented like this:
```php
<?php
...
  /**
   * @When i execute a factorial calculation
   */
  public function iExecuteAFactorialCalculation()
  {
      $this->result = (new Factorial())($this->factrorialInput);
  }

```

And now the the most nice part, checking whether the app we build run, the way we expect it. We will do it in a php7 way a take usage of ```assert()``` function(it needs an extra ```ini_set('assert.exception', 1)```).
```php
<?php  
...
  /**
   * @Then i get :arg1 back.
   */
  public function iGetBack($arg1)
  {
    ini_set('assert.exception', 1);
    assert($this->result === (int)$arg1, new AssertionError('Factorial return a wrong result'));
  }
```
and for negotiation case:
```php
<?php
...
  /**
   * @Then i don't get the :arg1 back.
   */
  public function iDonTGetTheBack($arg1)
  {
    ini_set('assert.exception', 1);
    assert($this->result !== (int)$arg1, new AssertionError('Factorial have to return a proper result'));
  }
```

let's try to run it: ```vendor/bin/behat```, the output now, have to look like this:
```
2 scenarios (1 passed, 1 failed)
6 steps (5 passed, 1 failed)
```

And as you see from 2 Scenarios, we have 1 that already pass. It feels freaky that smth. working without we implement anything, but from outside perspective it just right, that factorial ships a wrong results if there is no proper factorial implementation.

---

> It's quite obvious step to do, namely implementation of a factorial calculation. To pass this course step, implement a ```Factorial``` class and run behat in order to check that both from out 2 scenarios are passing.

### Solution
```php
<?php
declare(strict_types=1);

class Factorial
{
  public function __invoke(int $number, int $factorial = 1) : int
  {
    while ($number > 0) {
        return $this->__invoke($number - 1, $factorial * $number);
    }
    return $factorial;
  }
}
```

```task+data
---
match_output:
  match:
   - 2 scenarios \(2 passed
---
```

## Applying Scenario Outline

At that point you have to understand basics of BDD with behat. The rest of the course we will try to make our tests more suitable for a daily work.
You may noticed that our feature scenarios are a bit duplicated, even a autogenerated snipptes for a ```FeatureContext.php``` resolved it in a way that for 6 steps we got 4 methods, we can simply reduce it to 3.

---

### Refactoring a specification

To achieve it, we will use **Scenario Outline**. The general it could look like  following:
```html
  Scenario Outline: <Scenario Name>
    Given a step with <variable1>
    When add <variable2>
    Then we get <variableN>

    Examples:
    | variable1 | variable2 | ... | variableN  |
    | value11   | value12   | ... | variable1N |
    | value21   | value22   | ... | variable2N |
```

You just have to declare **Scenario Outline** instead of **Scenario** and provide and **Examples** block
in a table format.

Let's take a look how we can overwrite course feature.

```html
Scenario Outline: Calculate a Factorial
  Given i need to calculate a factorial of <input>
  When i execute a factorial calculation
  Then i get <result> back

  Examples:
    | input | result |
    | 5     | 120    |
    | 6     | 720    |
```
As you can see we can even extend with any other input/result combination and be flexible to edit this. We also get rid of negotiation scenario, cause we can cover in generic way each possible input, without need to copy paste our scenario block.
Each **Examples** row will be handled as separate Scenario, so the the whole test context will be reinitiated for each row respectively.

---

> To pass this course step, extend Scenario Outline Examples with the case of factorial of 0, that has to be equal 1. The resulted test run, have to output:  **3 scenarios (3 passed)**

### Solution
```
Examples:
  | input | result |
  | 5     | 120    |
  | 6     | 720    |
  | 0     | 1      |
```

```task+data
---
match_output:
  match:
   - 3 scenarios \(3 passed
match_file:
  path: /features/calculator.feature
  match:
   - Scenario [O|o]utline
---
```


## Role driven behaviour

After we learned how to write compact and at the same time powerful scenarios, let's take a look at how we can apply different use cases to a different php test context files.

---

### Applying another Persona

Let's create another feature for calculator admin who has to be able to activate/deactivate some specific calculations for what ever reason. For the sake of simplicity we will refer to the same Factorial implementation. Create ```features/admin.feature``` file and put the following content there:
```
Feature: Administration
  In order to control a calculation appearance
  As an admin
  I want to be able to deactivate/activate it

 Scenario Outline: Activate/Deactivate Calculation
   Given there is a calculation of <calculation> of any random number
   When i apply <state> to it
   Then i get <outcome> for a calculation

   Examples:
   | calculation  | state         | outcome |
   | 'Factorial'  | 'deactivated' | 'error' |
   | 'Factorial'  | 'activated'   | 'value' |
   | 'Factorial'  | 'default'     | 'value' |

```

Here we would definitely want to apply a different php test context file as for calculation feature, because it has pretty much another perspective and needs another logic for populating a context state.

---
### Extend Behat configuration

For that case we create in an application root a ```behat.yml``` file and pass the default execution flow of behat in a way it suites our feature definition.

```
default:
    suites:
        feature_features:
            paths:    [ %paths.base%/features ]
            filters:  { role: user }
            contexts: [ FeatureContext ]
        admin_features:
            paths:    [ %paths.base%/features ]
            filters:  { role: admin }
            contexts: [ AdminContext ]
```
You definitely have noticed a ```{ role: <role> }``` pattern applied to **filters**. The certain **role** correlates directly to role identifications in a feature file definition:

```
Feature: ...
  In order to ...
  As an admin
  I want ...
```
An **admin** role in a ```As an admin``` line of a Feature head is a meaningful statement for our application specification, that can be used to apply a certain php test context to it.

---
### Preparing tests

To get our tests for admin role on place try to run:
```bash
$> vendor/bin/behat --init --role admin
```
and after ```AdminContext```used to be created, we just go and append a test snippets to it, as we already did it for Factorial calculation spec, but in this case we specify a role we want to append it to:
```bash
$> vendor/bin/behat --append-snippets --role admin
```

We get three predifined methods:
```php
<?php
  ...
  /**
   * @Given there is a calculation of :arg1 of any random number
   */
  public function thereIsACalculationOfOfAnyRandomNumber($arg1)
  ...
  /**
   * @When i apply :arg1 to it
   */
  public function iApplyToIt($arg1)
  ...
  /**
   * @Then i get :arg1 for a calculation
   */
  public function iGetForACalculation($arg1)
  ...
```

---

### Implementing tests

Start with a **Given** step. It implies initialisation of 2 variables, that we can provide like this:
```php
<?php

  public function thereIsACalculationOfOfAnyRandomNumber($arg1)
  {
     $this->calculationName = $arg1;
     $this->calculationInput = rand(0, 10);
  }
```

To implement a **When** step we need to map a human readable state spec into some more handy from a developer perspective, and then initiated a state that we will check in a **Then** step. This could be done in a following way:
```php
<?php

  public function iApplyToIt($arg1)
  {
    $argument = null;
    switch($arg1){
      case 'deactivated':
        $argument = false;
        break;
      case 'activated':
      default:
        $argument = true;
        break;
    }
    $this->calculationInstance = new $this->calculationName($argument);
  }
```

And a **Then** step. Before assert an expected outcome we also need to map a calculation result back to spec. We will define it like this:
```php
<?php

public function iGetForACalculation($arg1)
{
  $outcome = null;
  try {
    $outcome = is_int(call_user_func($this->calculationInstance, $this->calculationInput))?'value':null;
  } catch (Error $e) {
    $outcome = 'error';
  }

  assert($arg1 === $outcome);
}

```
You may wonder, why don't we provide a "ready to run" values and do all that mapping before/after. It's important to understand, that the spec, that we defined in a feature file, have to be also a reference for a non-tech persons like stakeholder, who may not be really comfortable with **boolens** and **Exceptions** and it will make a discussion about implementing a feature most probably less sufficient.

Further, all that mapping have to be a part of application core(and move away from test context), since definitions we met in a spec are applied as a part of Domain's Ubiquitous Language. But before that, we just place it our tests.

now run the tests:
```bash
$>  vendor/bin/behat --role admin
```

and the output have to be most likely like this:
```
...
3 scenarios (2 passed, 1 failed)
9 steps (8 passed, 1 failed)
```
---

> As Task extend our Factorial class Implementation so that it pass the tests. And run ```vendor/bin/behat --role admin``` again

### Solution
```php
<?php
declare(strict_types=1);

class Factorial
{
  public function __construct(bool $activated = true)
  {
      $this->activated = $activated;
  }

  public function __invoke(int $number, int $factorial = 1) : int
  {
    if (!$this->activated) {
      throw new Error('Factorial is deactivated');
    }
    return $this->calculate($number, $factorial);
  }

  private function calculate(int $number, int $factorial) : int
  {
    while ($number > 0) {
      return $this->calculate($number - 1, $factorial * $number);
    }
    return $factorial;
  }
}
```

```task+data
---
match_output:
  match:
   - 3 scenarios \(3 passed
---
```

## Conclusion

It was a first filling of BDD in php, if you noticed some inconsistency in this course you a free to make a PR to this course on github.com/phpingme/courses
