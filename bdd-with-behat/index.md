---
 head:
  title: "BDD with Behat"
---

## Introduction

To develop a software is always about managing a lot of moving parts. And pretty often it gets messy, in a way that even a developer who wrote the code, cannot recover what is going on after 2-3 weeks the code was shipped.  It is an old problem and had a range of mature concepts around that helps to deal with in a sufficient way. Here we will try a BDD, and show how we can control a software's feature set and move quickly forwards using behat.



## Feature
The concept of **Feature** in BDD let us focus on a specific problem from a perspective of certain customer.  To describe this problem in a Feature form we need to provide 2 things:
 - Feature inroduction
 - Scenario

### Feature introduction
To introduce a Feature we use quite the same pattern we apply to describe a user story, namely:
```xml
   In order to <achieve a goal>
   As <the stakeholder who wants the goal>
   I want <something>
```
For a developer the 3'd line is most likely always clear, it handles an outcome of implementation, like *Product could be ordered*, *Post has hidden comments*... But it's not always clear about first two lines, especially if stuff have to be implemented in a microservice scope, where a customer could be less personalizable.
Here is important to work with PO and apply a [Ubiquitous Language](http://martinfowler.com/bliki/UbiquitousLanguage.html) of a domain you are implementing.

### Scenario
**Scenario** is an essential part of the whole BDD process, they are the part that describe a behaviour of our software. Each Scenario follows the following *Given-When-Then* schema:
```xml
  Given <initial state>
  When <apply a change to the state>
  Then <verify a changed state>
```

To start to write Features, let's first initiate environment where we want to write it.

```sh
    vendor/bin/behat --init
```

it creates a following tree structure:
```
```

Now let's create a first feature ```features/course.feature```(you can do it with a write click on **features** folder) with a first Scenario:
```
Feature: Course
  In order to learn about programming
  As a trainee
  I want pass a programming course

 Scenario: Pass the Task
   Given there is a task to find factorial of "5"
   When the solution is "120"
   Then i passed the task
```



> An actual Task, write the second scenario that checks the use case when the factorial task is not passed.


```projecteditor+terminal
---
filetree: true
id_suffix: ide
match:
  - not\s+passed
commands:
   - vendor/bin/behat
---
```

## Test

After we create our Feature that tests a passing of a factorial task. Lets implement a test for it and actually take an advantage of behat as well.
First of all, we need to convert our feature in a PHP form. For that, let's run:
```sh
  vendor/bin/behat --append-snippets
```
We have to get 4 methods, that throws a Pending exception.




## Scenario Outline



## Config


## Conclusion

It was a first filling of BDD in php, if you noticed some inconsistency in this course you a free to make a PR to this course on github.com/phpingme/courses
