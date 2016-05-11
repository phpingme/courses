---
head:
  title: "Php7 Introduction Course"
---
## Php7 Introduction Course
Hi, this course covers the basics of new features that come with PHP7. It lives on a github repository https://github.com/phpingme/courses, feel free to contribute to it.


## Combined Comparison Operator

A new operator ```<=>``` also known as "spaceship operator" has been added to PHP7, which is similar to the one in [Ruby](http://ruby-doc.org/core-1.9.3/Comparable.html). It compares two expressions and returns -1, 0, +1 depending if the left hand expression is less than, equal to or greater than the right hand expression respectively. For example:
```php
<?php
  // lefthand expression is lexically larger than righthand one
  "Foo" <=> "foo" // return 1

  // lefthand expression is less than righthand one
  1 <=> 2 // return -1

  // lefthand expression is equal to righthand one
  ['foo', 'bar'] <=> ['foo', 'bar'] // return 0

```

> To pass this task, please use PHP REPL below and compare the float value 5.6 with integer 7

```repl
---
match: \s?5\.6\s?\<=\>\s?7
success: null-coalesce-operator
---
```

## Null Coalesce Operator

Another syntactic sugar is the shorthand notation for ```isset``` function which is the null coalesce operator ```??```. Check and example below:

```php
<?php
$foo = ['a'=>1, 'b'=>2];

// Before PHP7
isset($foo['some'])? $foo['some'] : 'another'; // outputs 'another'
```
With PHP7 you can do the same in a much simpler way with the new ```??``` operator:

```php
<?php
$foo = ['a'=>1, 'b'=>2];
$foo['some'] ?? 'another'; // outputs 'another';

// it also works with chaining
$foo['some'] ?? $foo['a'] ?? 'another'; // just try it in the PHP REPL below

```

> As a task, use PHP REPL below and apply ```??``` operator on array element ```$foo['c']``` from above example

```repl
---
fixture: $foo = ['a'=>1, 'b'=>2];
match: \$foo\['c'\]\s?\?\?
success: group-use-declarations
---
```

## Group use Declarations

The ```use``` statement brings an ability to shortcut ```class```, ```function``` and ```const``` namespaces, it saves a lot of typing. Also now it is possible to reduce the amount of ```use``` statements if the classes have the same parent namespace, here are some examples:

```php
<?php
// Before PHP7
use Foo\Bar\Class1;
use Foo\Bar\Class2 as ClassTwo;
use Foo\Bar\Class3 as three;

// With PHP7 you can now group the use statement for classes with same parent namespace
use Foo\Bar\{Class1, Class2 as ClassTwo, Class3 as three};

// For functions
use function some\namespace\{fn_a, fn_b, fn_c};

// For constants
use const some\namespace\{SomeConst, ConstB as B, ConstC as C};
```

> As a task, extend the code below so that the following string appears as the output:

> **"i'm doing php7 intro course on phping.me"**


```editor+repl
<?php

namespace Some\FuncsAndConsts {

  const COURSE = "php7 intro";

  const PROJECT = "phping.me";

  function onProject($arg) {
    return $arg;
  }

  function doCourse($projectName, $courseName){
    return sprintf("i'm doing %s course on %s", $courseName, $projectName);
  }

}

namespace {

  // TASK: Add "use .." statements here, so that the command below, outputs:
  // "i'm doing php7 intro course on phping.me"

  echo doCourse(onProject(P), C) , PHP_EOL;
}

---
match:
 - use\s+const\s+Some\\FuncsAndConsts\\\{
 - use\s+function\s+Some\\FuncsAndConsts\\\{
success: scalar-type-declarations
---

```

## Scalar Type Hinting

PHP7 comes with a new optional mode to hint a scalar type argument explicitly as ```int```, ```float```, ```bool```, ```string```.

All it needs is a ```declare(strict_types=1)``` directive placed at the beginning of the php script, where you need it.

```php
<?php
declare(strict_types=1);

function product(int $a, int $b){
  return $a * $b;
}

product(2, 3); // outputs 6
```

> Try to rewrite above in the editor below to implement and call a function that takes float arguments and returns it's sum.

```editor+repl
<?php
---
match:
  - declare\(strict_types=1\)
  - \(float\s+
success: return-type-declarations
---
```

## Return Type Declarations

Along with scalar types declarations for arguments, it's also possible to declare the return type, that a class method or a standalone function must return. The supported types here are:

- ```string```
- ```int```
- ```float```
- ```bool```
- ```array```
- ```Closure```
- class or interface names
- ```self```, just within the class body
- ```parent```, also just within the class body

For example:

```php
<?php

function sumToStack(array ...$arr): SplStack {

  $stack = new SplStack();
  array_map(function(array $a) use ($stack): bool {
      return $stack->push(array_sum($a));
    }, $arr);

    return $stack;
}

foreach(sumToStack([1,2,3], [4,5,6], [7,8,9]) as $sum) {
    echo $sum . PHP_EOL;
}

// sends to stdout
// 24
// 15
// 6
```

> As the task adjust the above function to return ```SplQueue``` object (instaed of ```SplStack```), if you have not used any of those before, take a look at the PHP documentation http://php.net/manual/en/class.splqueue.php

```editor+repl
<?php
---
 match: \)\s?:\s+SplQueue
 success: return-type-invariance
---
```

## Return Type Invariance

To specify return types on a class level, one should be aware that once a returned type is specified, it can not be overridden by any of it's child classes.

for example:
```php
<?php
 class MyStack extends SplStack {}

 class A
 {
   public function createStack() : SplStack
   {
      return new SplStack();
   }
 }

 class B extends A
 {
   public function createStack() : MyStack // will fail
   {
      return new MyStack();
   }
 }


```
This will throw a ```E_COMPILE_ERROR``` because of broken *invariance*.

> As a task, change the ```class``` A to an ```interface``` and let ```class``` B's createStack method declare and return a correct return type.

```editor+repl
<?php
---
 match:
  - interface A
  - \)\s?:\s+SplStack
 success: anonymous-classes
---
```

## Anonymous Classes

The concept of anonymous classes are already available in quite a range of programming languages from Java to Ruby. Since PHP now a days is mostly used for object oriented programming (We just need to forget about Wordpress here ;)), anonymous classes definitely make sense in for simple *pass and forget* objects. Also use cases like testing and talking to libraries in general can make use of anonymous classes.

They can be handled just like normal classes, means they can:
- implement interfaces
- extend a non-anonymous class
- use constructors
- inject traits

For example:

```php
<?php
use Psr\Log\LoggerInterface;
use Psr\Log\LogLevel;

function stdOutTest(LoggerInterface $logger){
  $logger->log(LogLevel::INFO, "test");
}

stdOutTest(new class implements LoggerInterface {

    use Psr\Log\LoggerTrait;

    public function log($level, $message, array $context = array())
    {
        echo $message;
    }
});

```

> As a Task, rewrite the anonymous class example above in a way that,
> ```stdOutTest``` function works with following body. (Tip: Use a constructor and class properties for the anonymous class).
>```php
>function stdOutTest(LoggerInterface $logger){
>  $logger->log($logger->level, "test");
>}
>```

```editor+repl
<?php
use Psr\Log\LoggerInterface;
use Psr\Log\LogLevel;
use Psr\Log\LoggerTrait;

function stdOutTest(LoggerInterface $logger){
  $logger->log($logger->level, "test");
}

stdOutTest(new class(/*level*/)  implements LoggerInterface {

    /*implementation*/

} )
---
match: class\s?\(\s?LogLevel::\w+\s?\)
success: closure-call-method
---

```

## Closure call() Method

Anyone who has written some serious javascript code, could have overseen how excessive JS community take use of the ```call``` method. Now it's available in PHP7 and has absolutely the same goal and functionality. It enables injection of object instance and arguments into a closure function, It looks like this:

```php
<?php
class MyStorage extends SplStack {}

$closure = function($elem){
  $this->push($elem);
  return $this;
};

$stack = $closure->call(new MyStorage(), 1);
$stack = $closure->call($stack, 2);

echo $stack->pop(); // return 2
```

> As an exercise let ```MyStorage``` class extends from ```SplQueue``` and run the rest of above example as it is.

```editor+repl
<?php






---
match:
 - extends\s+SplQueue
 - ->pop\(
success: expectations
---
```

## Exceptions in the Engine

It was always pretty straightforward to build a ```try/catch``` based error handling in php. The side effect of that simplicity was that PHP engine errors and business domain violations have to be catched with an instance of ``` Exception```. With PHP7 fatal and recoverable fatal errors from now on throw an instance of ```Error```.  The new hierarchy is the following:
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
```

For example:

```php
<?php
try {

 (function($obj) {
    $obj->method();
 })(null);

} catch(Error $e) {

    var_dump($e instanceof Exception); // bool(false)
    var_dump($e instanceof Error);     // bool(true)
    var_dump($e instanceof Throwable); // bool(true)
    echo($e->getMessage());            // Call to a member function method() on null

}
```

Also take a look at how the closure function was called above, it follows the pattern ```(function () {})()``` and it has the IIFE syntax from JS.

> As a task find out a specific ```Error``` child instance (```ParseError```, ```TypeError```, ...) and catch that for the following closure function:
>```php
>(function(callable $obj) {
>   $obj->call();
>})(null);
>```

```editor+repl
<?php

// try and catch it
(function(callable $obj) {
   $obj->call();
})(null);

---
match: catch\s?\(\s?TypeError
success: expectations
---
```

## Expectations

To prove if the some expression is true, you can use ```assert()``` function, and with ```ini_set('assert.exception', 1);``` it enables to throw an instance of ```AssertionError```.

```php
<?php
ini_set('assert.exception', 1);
assert(false);
```

> As a task add a second argument to an ```assert()``` function, the one that have to be an instance of ```AssertionError``` with a custom message as it's constructor argument, just like you would do it with Exception, and make sure to catch the Error.

```editor+repl
<?php

ini_set('assert.exception', 1);

// your assert(false, ...) function within a try/catch block


---
match:
- AssertionError
- catch\s?\(
success: generator-return-expressions
---
```

## Generator Return Expressions

The use case of iterating something is pretty common in php applications, it is less often done with the object oriented way to work with iteration, mostly cause of it bloat up tto do it with the [Iterator](http://php.net/manual/en/class.iterator.php) interface. Generator is a simple way to do it by prepending an ```yield``` operator, but that is actually it. 

For more advanced use cases like multitasking in coroutine context Generator was quite counterintuitive and which made implementing multitasking features quite cumbersome.  With PHP7, now it's possible to declare a final return expression of a Generator instance.

```php
<?php
$generator = (function(int $index){
  yield $index++;
  return $index;
})(1);

foreach($generator as $i){
    echo $i . PHP_EOL;        // return 1
}
echo $generator->getReturn(); // return 2

```

> As an exercise, rewrite the above code and try to add some more ```yield``` operators within the closure body, so that it returns a value **3**.

```editor+repl
<?php

---
match_output: 3
success: generator-delegation
---
```

## Generator Delegation

Another quiet handy feature in terms of making Generator more useful is it's delegation. In other words along with returning an expression, you can also return another Generator by referencing with ```yield from``` expression to it.

```php
<?php
$generator = (function(int $index){
  yield $index++;
  return yield from generatorPow($index);
})(1);

function generatorPow(int $index){
  yield pow($index, $index);
  return $index;
}

foreach($generator as $item){
    echo $item, PHP_EOL;   
}
echo $generator->getReturn();

// output
// 1
// 4
// 2

```
> As an exercise, let the function *generatorPow* from the example above also get delegated generator, so that whole output changes like this:
```
>1
>4
>16
>```


```editor+repl
<?php

$generator = (function(int $index){
  yield $index++;
  return yield from generatorPow($index);
})(1);

// overwrite generatorPow function and add a new one, that gives a generator to generatorPow


// output part
foreach($generator as $item){
    echo $item, PHP_EOL;   
}
echo $generator->getReturn();
---
match_output: 1416
success: reflection-additions
---
```

## Reflection Additions

Reflections in PHP was always a nice tool to inspect classes and when necessary tweak also to their behavior, for example for tests. In PHP7 it comes with a new reflection class for Generator namely ```ReflectionGenerator```, and a ```ReflectionType``` that extends an existing reflection classes (```ReflectionParameter```, ```ReflectionFunctionAbstract```) in order to give the access to newly introduced return type declarations for scalar type hints and return expressions.

```php
<?php
$reflector = new ReflectionGenerator((function(){
  yield 'hello';
})());

// public API
echo $reflector->getExecutingGenerator()->current(); // hello
echo $reflector->getFunction()->name; // string(9) "{closure}"
echo $reflector->getExecutingFile(); //  /var/www/test.php
echo $reflector->getExecutingLine(); // 3
echo $reflector->getTrace(); // Array
var_dump($reflector->getThis()); // NULL
```

> To practice in using ```Generator``` and ```ReflectionGenerator``` let the following generator:
```
>(function(){
  $str = yield;
  echo $str;
})()
```
> be injected in ```ReflectionGenerator``` instance and let resulted instance output any string that you like. To do this you may want to take a look at the ```Generator``` documentation http://php.net/manual/de/class.generator.php and specifically the method ```Generator::send```


```editor+repl
<?php

---
match: getExecutingGenerator\(\s?\)->send
success: congrats
---
```
## Congrats

Cool, now you've covered the most interesting and useful parts of PHP7, It was not hard right? ;). Also, if you have any suggestions to make this better just open an issue or make a pull request on https://github.com/phpingme/courses
