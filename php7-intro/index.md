---
head:
  title: "Php7 Introduction Course"
---
## Combined Comparison Operator

A new operator ```<=>``` that is also know as "spaceship operator" and is also exists
for example in [Ruby](http://ruby-doc.org/core-1.9.3/Comparable.html). All it does it compares to expressions like this:
```php

  // lefthand expression is lexically larger than righthand one
  "Foo" <=> "foo" // return 1

  // lefthand expression is less than righthand one
  1 <=> 2 // return -1

  // lefthand expression is equal to righthand one
  ['foo', 'bar'] <=> ['foo', 'bar'] // return 0

```

use PHP Repl below and compare float value 5.7 with integer 7

```repl
---
match: \s?5\.7\s?\<=\>\s?7
success: null-coalesce-operator
---
```

## Null Coalesce Operator

Another syntax sugar is to shorthand notation for ```isset``` function with coalesce operator ```??```
For example an array offset check like this one:

```php
$foo = ['a'=>1, 'b'=>2];

$bar = isset($foo['some'])? $foo['some'] : 'another';
```
you can know write with a *coalesce* :
```php
$bar = $foo['some'] ?? 'another';
// also with chaining
$bar_a = $foo['some'] ?? $foo['a'] ?? 'another';

```


try PHP Repl and try check offset key 'c' of array $foo from above

```repl
---
fixture: $foo = ['a'=>1, 'b'=>2];
match: =\s?\$foo\['c'\]\s?\?\?
success: group-use-declarations
---
```

## Group use Declarations

The ```use``` statement brings an ability to shortcut ```class```, ```function``` and ```const``` namespaces, at was always a lot of typing, but lets the code besides look better. Now it is possible to reduce an amount of ```use``` statements if the classes have the same parent namespace, and it looks like that:

```php
use Foo\Bar\Class1;
use Foo\Bar\Class2 as ClassTwo;
use Foo\Bar\Class3 as three;

// or for php7+ just
use Foo\Bar\{Class1, Class2 as ClassTwo, Class3 as three};

```

> as a task take a look at classes in https://github.com/php-fig/log/blob/master/Psr/Log and provide a use group statement for a few of them, and var_dump one of them

```editor+repl
<?php

use // ... declare use group

var_dump(/* classname */)

---
match:
 - use\s+Psr\\Log\\\{
 - var_dump\?
success: scalar-type-declarations
---

```

## Scalar Type Declarations
PHP7 comes with a new optional mode to declare a scalar type arguments explicitly by its type: ```int```, ```float```, ```bool```, ```string```.
All it needs is a ```declare(strict_types=1)``` directive placed at the beginning of a php script file, where you want to use it.

```php
declare(strict_types=1)
function product(int $a, int $b){
  return $a*$b
}

product(2, 3) // returns 6
```

try in PHP editor below implement and call a function that takes float arguments and returns it's sum:
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
Along with scalar types declarations for arguments, it's also possible to define the type, that a class method or a standalone function have to return. Supported types here are:
- ```string```
- ```int```
- ```float```
- ```bool```
- ```array```
- ```Closure```
- class or interface names
- ```self```, but just within a class body
- ```parent```, also just within a class body

for example:
```php
<?php

function sumToStack(array ...$arr): SplStack {

  $stack = new SplStack();
  array_map(function(array $a) use ($stack): bool{
      return $stack->push(array_sum($a));
    }, $arr);

    return $stack;
}

foreach(sumToStack([1,2,3], [4,5,6], [7,8,9]) as $sum){
    echo $sum . PHP_EOL;
}

// sends to stdout
// 24
// 15
// 6
```

> as a task write the same function as before but let it return ```SplQueue``` object(instaed of ```SplStack```)

```editor+repl
<?php
---
 match: \)\s?:\s+SplQueue
 success: return-type-invariance
---
```

## Return Type Invariance
To specify return types on a class level, one should be aware, that once returned type is specifed, it can not be overwritten by any child classes.

for example:
```php
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
Here we will get an ```E_COMPILE_ERROR``` couse of breaked *invariance*.

> as a Task, overwrite class A to interface and let class B declare and return the right type.

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

A concept of anonymous class is already applied in a quite a range of programming languages from Java to Ruby. Since PHP is mostly used for an object oriented programming(we just forget about Wordpress here) anonymous classes definitely make sense to apply it for simple *pass and forget* objects in use cases like testing and talking to libraries in general.

They can be handled just like normal classes, means they can:
- implement interface
- extend non-anonymous class
- use constructors
- inject traits


for example
```php

use Psr\Log\LoggerInterface;
use Psr\Log\LogLevel;

function stdOutTest(LoggerInterface $logger){
  $logger->log(LogLevel::INFO, "test");
}

stdOutTest(new class  implements LoggerInterface {

    use Psr\Log\LoggerTrait;

    public function log($level, $message, array $context = array())
    {
        echo $message;
    }
});

```

> as a Task, overwrite an anonymous class in example in a way, that
> ```stdOutTest``` function works with following body. Use of constructor and class properties.
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

stdOutTest( // anonymous class
  )
---
match: class\s?\(\s?LogLevel::\w+\s?\)
success: closure-call-method
---

```


## Closure call() Method
Everyone how have to write some mature javascript code, couldn't oversee how excessive js community take use of this construct. In PHP it has absolutely the same goal and functionality, it provides a method to inject an object instance and arguments into closure function and look like this:

```php

class MyStorage extends SplStack{}

$closure = function($elem){
  $this->push($elem);
  return $this;
};

$stack = $closure->call(new MyStorage(), 1);
$stack = $closure->call($stack, 2);

echo $stack->pop(); // return 2
```

> as an exercise let ```MyStorage``` class exentds from ```SplQueue``` and run the rest of above example as it is.

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
It was always pretty straightforward  to build up an ```try/catch``` based error handling in php. The side effect of that simplicity was, that php engine errors and business domain violations have to be catched with an instance of ``` Exception```. With PHP 7 fatal and recoverable fatal errors will from now on throw an instance of ```Error```.  The new hierarchy is the following:
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

for example:
```php

try{

 (function($obj) {
    $obj->method();
 })(null);

}catch(Error $e){

    var_dump($e instanceof Exception); // bool(false)
    var_dump($e instanceof Error);     // bool(true)
    var_dump($e instanceof Throwable); // bool(true)
    echo($e->getMessage());            // Call to a member function method() on null

}

```
Also take a look at how owe closure function was called, it follows the following pattern ```(function () {})()``` and it follows IIFE syntax from JS.


> as a task catch a specific ```Error``` child instance (```ParseError```, ```TypeError```, ...) of the following closure function:
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

To prove if the the some expression is true, you can use ```assert()``` function, and with ```ini_set('assert.exception', 1);``` it enables to throwing an instance of ```AssertionError```.

```php

ini_set('assert.exception', 1);
assert(false);
```

> as a task extend an ```assert()``` function with a second argument as an instance of ```AssertionError``` that takes a custom message as it's constructor argument, just like you would do it with Exception, and surely catch an Error.

```editor+repl
<?php

ini_set('assert.exception', 1);
// your assert function within try/catch block

---
match:
- AssertionError
- catch\s?\(
success: generator-return-expressions
---
```

## Generator Return Expressions
The use case of iterating smth. is pretty common in php applications, less often is applying an object oriented way to work with iteration. Mostly cause of it bloat up [Iterator](http://php.net/manual/en/class.iterator.php) interface. Generator is a simple way to do it by prepending an ```yield``` operator. But that is actually it. For more advanced use cases like multitasking in coroutine context Generator was quite counterintuitive and made implementing multitasking features quite cumbersome.  With PHP 7+ it's possible to declare a final return expression of a Generator instance.

```php
$generator = (function(int $index){
  yield $index++;
  return $index;
})(1);

foreach($generator as $i){
    echo $i . PHP_EOL;        // return 1
}
echo $generator->getReturn(); // return 2

```

> as an exercise add some more ```yield``` operators within a closure body from example above and just take a look so that returned value gets to **3**.

```editor+repl
<?php

---
match_output: 3
success: generator-delegation
---
```


## Generator Delegation
Another quiet handy feature in terms of making Generator more handy is it's delegation. In other words along with returning an expression, you can also return another Generator by referencing with ```yield from``` expression to it.

```php
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
> as an exercise let the function *generatorPow* from the example above also get delegeted generator, the one that enables the the whole output like:
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

// overwrite generatorPow function and write a new one that gets delegated to generatorPow


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
Reflections in php was always a nice tool to inspect classes and when necessary tweak their behavior, for example for tests. In PHP 7 comes with a new reflection classes for Generator namely ```ReflectionGenerator```, and a ```ReflectionType``` that extends an existing reflection classes (```ReflectionParameter```, ```ReflectionFunctionAbstract```) on order to give an access to newly introduced Type Declarations for scalar values and return expressions.

```php
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

> to practice in using ```Generator``` and ```ReflectionGenerator``` let a following generator:
```
>(function(){
  $str = yield;
  echo $str;
})()
```
> be injected in ```ReflectionGenerator``` instance and let resulted instance output any string you like. For that you may be want to recover a ```Generator``` api http://php.net/manual/de/class.generator.php


```editor+repl
<?php

---
match: getExecutingGenerator\(\s?\)->send
success: congrats
---
```
## Congrats

Cool, you've covered the most interesting and useful parts of PHP7. It was not hard.
