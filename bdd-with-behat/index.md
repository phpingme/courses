---
 head:
  title: "BDD with Behat"
---
## Inroduction

To develop a software is always about managing a lot of moving parts. And pretty often it gets messy, in a way that even a developer who wrote the code, cannot recover what is going on after 2-3 weeks the code was shipped.  It is an old problem and had a range of mature concepts around that helps to deal with in a sufficient way. Here we will try a BDD, and show how we can control a software's feature set and move quickly forwards using behat.



## Gherkin

Gherkin is the first class citizen of a BDD we want to learn here. It's just an english language, pressed in **Given**, **When**, **Then** steps.
Let us try to write our first Scenario.
for that case after we installed behat, let us:
1. create behat's bdd boilerplate structure
2. Create a feature file
3. Generate tests File.

### create a boilerplate


```projecteditor+terminal
---
filetree: true
id_suffix: ide
commands:
   - vendor/bin/behat
---
```

## CRUD Routine

when we want to test some persistence mechanism, then it's better to provide all arguments we want to test already in a **Given** part, and **When** will just trigger the process.

## Domain language
**When* and **Then** has often a problem to refer to some abstract "it" like:
  When i execute it
  Then i get it

it is the first sign that you loose a focus on your domain.

Another edge case is refer to "this" in a Scenario, like
  Given there is a product "car"
  And this has a "red" color

It inevitably leads to introduce state in your test context file.

## Where to build a state what state in a context file.



## Consptional problem

With **Given** you have to provide some state, that you want to run against **When** statement. And it implies that you have to expose a public method in order to make **When** call possible. And this a situation that can make someone pretty uncomfortable about this and that is a good start to make a refactoring.

As less state you have as better.

## Work with context files
 A new Context file is needed when you need to touch your Context constuctor.
