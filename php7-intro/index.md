---
head:
  title: "Php7 Introduction Course"
template: editor+stdout

---
## Introduction
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

### Solution
```php
5.6 <=> 7
```

```task+data
---
match_input:
  match:
   - \s?5\.6\s?\<=\>\s?7
---
```

## Null Coalesce Operator

Another syntactic sugar is the shorthand notation for ```isset``` function which is the null coalesce operator ```??```. Check and example below:

```php
<?php
$foo = ['a'=>1, 'b'=>2];

// Before PHP7
isset($foo['a'])? $foo['a'] : 'another'; // outputs '1'
isset($foo['b'])? $foo['b'] : 'another'; // outputs '2'
isset($foo['some'])? $foo['some'] : 'another'; // outputs 'another'

// With PHP7
$foo['a'] ?? 'another'; // outputs '1'
$foo['b'] ?? 'another'; // outputs '2'
$foo['some'] ?? 'another'; // outputs 'another';

// it also works with chaining
$foo['some'] ?? $foo['a'] ?? 'another'; // just try to echo it in the Editor on place

```

> As a task, use PHP REPL below and apply ```??``` operator on array element ```$foo['c']``` from above example

### Solution
```php
<?php
...
echo $foo['c'] ?? 'key "c" has no value';
```

```task+data
<?php

$foo = ['a'=>1, 'b'=>2];
---
match_input:
  match:
   - \$foo\['c'\]\s?\?\?
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

> As a task, extend the code in a worksheet in a way it works. All you need is two define 2 ```use``` expressions.
One for functions and one for constants.

### Solution
```php
<?php

  ...
  use function Some\FuncsAndConsts\{doCourse, onProject};
  use const Some\FuncsAndConsts\{COURSE as C, PROJECT as P};
  ...

```

```task+data
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
match_input:
  match:
   - use\s+const\s+Some\\FuncsAndConsts\\\{
   - use\s+function\s+Some\\FuncsAndConsts\\\{
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

echo product(2, 3); // outputs 6
```

> Try to rewrite above in a worksheet to implement and call a function that takes float arguments and returns it's sum.

### Solution
```php
<?php

declare(strict_types=1);

function sum(float $a, float $b){
  return $a + $b;
}

echo sum(2.5, 3.1); // outputs 5.6
```

```task+data
---
match_input:
  match:
    - declare\(strict_types=1\)
    - \(float\s+
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

### Solution
```php
<?php
function sumToStack(array ...$arr): SplQueue{

  $stack = new SplQueue();
  array_map(function(array $a) use ($stack): bool {
      return $stack->push(array_sum($a));
    }, $arr);

    return $stack;
}

foreach(sumToStack([1,2,3], [4,5,6], [7,8,9]) as $sum) {
    echo $sum . PHP_EOL;
}

```

```task+data
---
match_input:
 match:
  - \)\s?:\s+SplQueue
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

### Solution
```php
<?php
 class MyStack extends SplStack {}

 interface A
 {
   public function createStack() : SplStack;
 }

 class B implements A
 {
   public function createStack() : SplStack
   {
      return new MyStack();
   }
 }
```


```task+data
---
match_input:
  match:
   - interface A
   - \)\s?:\s+SplStack
---
```

## Anonymous Classes

The concept of anonymous class is already available in quite a range of programming languages from Java to Ruby. Since PHP nowadays is mostly used for object oriented programming (We just need to forget about Wordpress here ;)), anonymous classes definitely make sense for simple *pass and forget* objects. Also use cases like testing and talking to libraries in general can make use of anonymous classes.

They can be handled just like normal classes, means they can:
- implement interfaces
- extend a non-anonymous class
- use constructors
- inject traits

For example:

```php
stdOutTest(new class(LogLevel::INFO) implements LoggerInterface {
  public $level;
  use LoggerTrait;

  public function __construct($level)
  {
    $this->level = $level;
  }

  public function log($level, $message, array $context = [])
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

### Solution
```php
<?php
...
stdOutTest(new class(LogLevel::INFO) implements LoggerInterface {
  public $level;

  public function __construct($level)
  {
    $this->level = $level;
  }

  public function log($level, $message, array $context = [])
  {
    echo $message;
  }
});

```

```task+data
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
match_input:
 match:
  - class\s?\(\s?LogLevel::\w+\s?\)
---

```

## Closure call() Method

Anyone who has written some serious plain javascript code, have definitely not overseen how excessive JS community take use of the ```call``` method. Now it's available in PHP7 and has absolutely the same goal and functionality. It enables injection of object instance and arguments into a closure function. It looks like this:

```php
<?php
class MyStorage extends SplStack {}

$closure = function($elem){
  $this->push($elem);
  return $this;
};

$stack = $closure->call(new MyStorage(), 1);
$stack = $closure->call($stack, 2);
$stack->rewind();

echo $stack->current(); // return 2
```

> As an exercise let ```MyStorage``` class extends from ```SplQueue``` and run the rest of above example as it is. You have to see that output is now have to be **1** instead of **2** in SplStack case

### Solution
```php
<?php
  class MyStorage extends SplQueue {}
  // the rest is the same as in example
  // ...
```

```task+data
---
match_input:
  match:
   - extends\s+SplQueue
   - ->current\(
---
```

## Exceptions in the Engine

It was always pretty straightforward to build a ```try/catch``` based error handling in php. The side effect of that simplicity was that PHP engine errors and business domain violations have to be catched with an instance of ``` Exception```. With PHP7 fatal and recoverable fatal errors from now on will be thrown as an instance of ```Error```.  The new hierarchy is the following:
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
As you see all about implementation errors get ```Error``` instance. The Domain Logic can be from now own be handled by ```Exception```

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

### Solution
```php
<?php

try{
  (function(callable $obj) {
    $obj->call();
  })(null);
} catch (TypeError $e){
  echo $e->getMessage();
}
```

```task+data
<?php

// try and catch it
(function(callable $obj) {
   $obj->call();
})(null);

---
match_input:
  match:
    - catch\s?\(\s?TypeError
---
```

## Expectations

Everyone have to be familiar with [PHPUnit assertion methods](https://phpunit.de/manual/current/en/appendixes.assertions.html), but some times it can be an overkill, or we may be we just need a well defined assertions in a production environment. For that case - to prove if the some expression is true - you can use ```assert()``` function, and with ```ini_set('assert.exception', 1);``` it enables to throw an instance of ```AssertionError```.

```php
<?php
ini_set('assert.exception', 1);
assert(false);
```

> As a task add a second argument to an ```assert()``` function, the one that have to be an instance of ```AssertionError``` with a custom message as it's constructor argument, just like you would do it with Exception, and make sure to catch the Error.

### Solution
```php
<?php

ini_set('assert.exception', 1);

try{
  assert(false, new AssertionError("false have to be true"));
} catch (AssertionError $e) {
  echo $e->getMessage();
}


```

```task+data
<?php

ini_set('assert.exception', 1);

// your assert(false, ...) function within a try/catch block


---
match_input:
  match:
  - AssertionError
  - catch\s?\(
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

### Solution
```php
<?php
$generator = (function(int $index){
  yield $index++;
  yield $index++;
  return $index;
})(1);
...
```

```task+data
---
match_output:
  match:
   - 3
---
```

## Generator Delegation

Another quiet handy feature in terms of making Generator more useful is it's delegation. In other words along with returning an expression, you can also return another Generator, Traversable Object or just an array by referencing to it by ```yield from``` keyword.

```php
<?php
$generator = (function(int $index){
  yield $index++;
  yield from [$index++];
  yield from new ArrayIterator([$index++]);
  yield $index++;
  return $index;
})(1);


// output part
foreach($generator as $item){
    echo $item, PHP_EOL;   
}
echo $generator->getReturn();
// output
// 1
// 2
// 3
// 4
// 5

```
> You can see in the Example above, that ```yield from``` keyword gets everything but Generator. As an exercise, move ```yield from [$index++];``` and  ```yield from new ArrayIterator([$index++])``` into separate functions and let the closure's ```yield from``` keyword accept just Generators. The output have to be the same.

### Solution
```php
<?php

$generator = (function(int $index){
  yield $index++;
  yield from incArray($index);
  yield from incIterator($index);
  yield $index++;
  return $index;
})(1);

function incArray(&$index) {
	yield from [$index++];
}

function incIterator(&$index) {
	yield from new ArrayIterator([$index++]);
}
```

```task+data
<?php

$generator = (function(int $index){
  yield $index++;
  yield from /* Generator */
  yield from /* Another Generator */
  yield $index++;
  return $index;
})(1);

// put here your two functions.

// output part
foreach($generator as $item){
    echo $item, PHP_EOL;   
}
echo $generator->getReturn();
---
match_input:
  match:
   - function\s+\w.[\s\S]*yield\s+from\s+\[.*?\]
   - function\s+\w.[\s\S]*yield\s+from\s+new\s+ArrayIterator\s?\(
match_output:
  match:
    - 1
    - 2
    - 3
    - 4
    - 5
---
```

## Reflection Additions

Reflections in PHP was always a nice tool to inspect classes and when necessary also to tweak their behavior, for example for tests. And it quite obvious, that with new features for Generator there have to be new Reflection api to handle this. In PHP7 it comes with a new reflection class for Generator, namely ```ReflectionGenerator```, and a ```ReflectionType``` that extends an existing reflection classes (```ReflectionParameter```, ```ReflectionFunctionAbstract```) in order to give the access to newly introduced return type declarations for scalar type hints and return expressions.

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
> be injected in ```ReflectionGenerator``` instance and let the resulted instance output any string that you like. To do this you may want to take a look at the ```Generator``` documentation http://php.net/manual/de/class.generator.php and specifically the method ```Generator::send```

### Solution
```php
<?php
 $reflector = new ReflectionGenerator((function(){
  $str = yield;
  echo $str;
})());

echo $reflector->getExecutingGenerator()->send('hello');
```

```task+data
<?php
 $reflector = new ReflectionGenerator(
   // put in task defined closure
);


echo $reflector->...

---
match_input:
  match:
    - getExecutingGenerator\(\s?\)->send
---
```
## Conclusion

Cool, now you've covered the most interesting and useful parts of PHP7, It was not hard right? ;). Also, if you have any suggestions to make this better just open an issue or make a pull request on https://github.com/phpingme/courses
