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


## Expectations





## Generator Return Expressions

## Generator Delegation

## Integer Division with intdiv()

## session_start() Options

## preg_replace_callback_array() Function

## CSPRNG Functions

## Support for Array Constants in define()

## Reflection Additions

Reflections in php was always a nice tool to inspect classes and sometimes tweak their behavior, for example for tests. In php7 it is now also possible to apply Reflections to Generators or in other words to ```yield``` constructs.
