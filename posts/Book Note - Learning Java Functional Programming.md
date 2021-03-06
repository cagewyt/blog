# Book Note - Learning Java Functional Programming (Richard M Reese)

The problem to solve is how to blend the various programming styles 
(functional programming in this case) available in Java 8 to 
meet an application's need. 

**Disclaimer**: 
All the paragraphs/codes are direct copy from the [book](https://www.oreilly.com/library/view/learning-java-functional/9781783558483/). 
This note captures the key points in the book I found particularly useful to myself. 
Thank the author for creating such a good book on the functional programming in Java. 

## Functional Programming Concept

First-class and high-order functions are associated with functional programming. 

A **first-class function** is a computer science term. 
It refers to functions that can be used anywhere a first-class entity can be used. 
A first-class entity includes elements such as numbers and strings. 
They can be used as an argument to a function, returned from a function, or assigned to a variable. 

**High-order functions** depend upon the existence of first-class functions. 

They are functions that either: 

* Take a function as an argument 
* Return a function 

Java 8 has introduced the concept of **lambda expressions** to the language. 
These are essentially anonymous functions that can be passed to and returned from functions. 
They can also be assigned to a variable.

A **pure function** is a function that has no side effects. 

This means that memory external to the function is not modified, 
IO is not performed, and no exceptions are thrown. 

With a pure function, when it is called repeatedly with the same parameters, it will return the same value. 
This is called **referential transparency**. 
With referential transparency, it is permissible to modify local variables within the function as 
this does not change the state of the program. 
Any changes are not seen outside of the function. 

Advantages of pure function include: 
* The function can be called repeatedly with the same argument and get the same results. 
This enables caching optimization (memorization).
* With no dependencies between multiple pure functions, they can be reordered and performed in parallel. 
They are essentially thread safe.
* Pure function enables lazy evaluation as discussed later in the Strict versus non-strict evaluation section. 
This implies that the execution of the function can be delayed and its results can be cached 
potentially improving the performance of a program.
* If the result of a function is not used, then it can be removed since it does not affect other operations.

**Closure** refers to functions that enclose variable external to the function. 
This permits the function to be passed around and used in different contexts.

**Currying**: Some functions can have multiple arguments. 
It is possible to evaluate these arguments one-by-one. 
This process is called currying and normally involves creating new functions, which have one fewer arguments than the previous one. 
The advantage of this process is the ability to subdivide the execution sequence and work with intermediate results. 
This means that it can be used in a more flexible manner. 

```java
Function<String, Function<String, String>> curryConcat; 
curryConcat = (a) -> (b) -> biFunctionConcat.apply(a, b);
```

Functional programming places more emphasis on how these functions are arranged and combined. 
It is this composition of functions, which typifies a functional style of programming. 
Functions are not only used to organize the execution process, 
but are also passed and returned from functions. 
Often data and the functions acting on the data are passed together promoting more capable and expressive programs.

**The incorporation of these functional programming techniques does not make Java a functional programming language. 
It means that we now have a new set of tools that we can use to solve the programming problems presented to us. 
It behooves us to take advantage of these techniques whenever they are applicable.**

## Java Functional Flavours

### Lambda Expressions
Lambda expressions are essentially anonymous functions. 
Lambda expressions are often converted to a functional interface automatically simplifying many tasks. Lambda expressions can access other variables outside of the expression. The ability to access these types of variables is an improvement over anonymous inner functions, which have problems in this regard.

### Method and Constructor References
A method or constructor reference is a technique that allows a 
Java 8 programmer to use a method or constructor as if it was a lambda expression.

### Functional Interfaces 
A functional interface is an interface that has one and only one abstract method. 
The Computable interface declared in the previous section is a functional interface. 
It has one and only one abstract method: compute. 
If a second abstract method was added, the interface would no longer be a functional interface.

Lambda expressions can **throw exceptions**. However, they must only be those specified by its functional interface's abstract method.

#### Functional Interface in JDK 
We can group these interfaces into five categories: 
* **Function**: These transform their arguments and return a value 
  * Function<T,R> R apply(T t) 
  * BiFunction<T,U,R> R apply(T t, U u) 
* **Predicate**: These are used to perform a test, which returns a Boolean value 
  * Predicate boolean test(T t) 
  * BiPredicate<T,U> boolean test(T t, U u) 
* **Consumer**: These use their arguments, but do not return a value 
  * Consumer void accept(T t) 
  * BiConsumer<T,U> void accept(T t, U u) 
* **Supplier**: These are not passed data, but do return data 
  * Supplier T get() 
* **Operator**: These perform a reduction type operation 
  * UnaryOperator R apply(T t) 
  * BinaryOperator R apply(T t1, T t2)

For example, Function<T, R>
```java
@FunctionalInterface
   public interface Function<T, R> {
       R apply(T t);
       default <V> Function<V, R> compose(
               Function<? super V, ? extends T> before) {
           Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
      }
      default <V> Function<T, V> andThen(
              Function<? super R, ? extends V> after) {
          Objects.requireNonNull(after);
          return (T t) -> after.apply(apply(t));
      }
      static <T> Function<T, T> identity() {
          return t -> t;
      } 
}
```

**Method composition** provides a flexible way of combining two or more functions into a single function. 
This offers flexibility that will otherwise not be present. 
We illustrate how this technique can be implemented using the Function interface. 
Its compose and andThen methods support the execution of functions before or after another function.

**As languages mature, they tend to find better ways of doing things.**

Diamond problem a class that implements several interfaces that have the same default method 
implemented, has to implement this method itself. 
We are going to explain this with an example.

## Stream

Java 8's **stream** concept is often a better design and performance choice 
than many equivalent imperative approaches. 

**It helps separate the "what to do" from "how to do".**

**Intermediate methods** always return a stream and do not actually modify the stream, 
but create a new stream instead. 
The processing of a stream starts when the terminal operation starts and 
stops when the **terminal method** completes.

stream.sorted() .distinct() .filter() .map() .flatMap() be useful when multiple streams need to be combined

```java
Stream flatMap(Function<? super T, ? extends Stream<? extends R>> mapper) 

rectangleLists.stream()
    .flatMap((list) -> list.stream())
    .forEach(System.out::println);

Stream stream3 = Stream.concat(stream1, stream2);
```

**Terminal methods** may produce a result as the sum method did, 
or it can produce a side effect. 
After a terminal method has been executed, 
the stream is said to have been consumed. 
This means the stream **cannot be used again**.

Methods such as findFirst are called **short-circuiting methods**. 
When they are used with an infinite stream, they will return a finite stream.

`anyMatch` Any element matches 

`allMatch` All of the elements match. 

For an infinite stream, a limit-type method will restrict its length 

`noneMatch` None of the elements match 

`findAny` Finds any element that matches 

`limit` Restrict the number of elements 

`subStream` Creates a substream


**parallelStream** There are several factors you need to take into consideration before making a stream parallel. We will not be able to address all of these issues, but will address many of them including:

* Non-inference: 
During the processing of the stream, the stream's data source must not be modified. 
* Stateless operations: A lambda expression whose outcome might vary during 
its execution are called state full. This is potentially a problem, 
because as the stream's operations are executed, 
the results can differ each time it is executed. Instead, 
lambda expressions should be written to not use a state. 
* Side effects: A stream operation can affect other parts of a program. 
They should be avoided if possible. 
* Ordering: The ordering of elements produced by a parallel stream may be important. 
If so, care must be taken to address the ordering issue.

### Testing Exceptions in Stream
Read this [blog](https://blog.codeleak.pl/2014/07/junit-testing-exception-with-java-8-and-lambda-expressions.html?spref=tw)

## Optional
The Optional class is not serializable

The Optional instances cannot be sorted using the Arrays class's sort method. 
If used, it throws a java.lang.ClassCastException. 
This is because an Optional instance cannot be cast to Comparable. 
However, an array of Optional values can be sorted manually

The Optional instances should not be used as constructor or method parameters.

```java
public Optional map(Function<? super T, ? extends U> mapper);

public Optional flatMap(Function<? super T, Optional> mapper);
```

**Monads**: A monad structure can be thought of as a chain of operations 
wrapped around an object. These operations are executed against an object and 
return some value. In this sense, monads support function composition. 
The chaining sequence allows programmers to create pipelines, a sequence of operations, 
to solve their problems.

## Design Pattern in Functional Programming Style
Overusing design patterns can make the system hard to understand when a simpler solution 
is obfuscated by a more complex pattern implementation. 
**Design patterns are a good communication tool, but should not be treated as gospel. 
When the problem doesn't match the proposed pattern, don't use the pattern.** 
Choose the right pattern for the right problem.

Design patterns are often specific to a specific programming paradigm. 
Some design patterns support Java applications very well. 
However, many object-oriented design patterns are irrelevant to 
some functional programming languages. 
For example, the singleton pattern is not present in pure functional programming languages.

### Execute-Around-Method Pattern
(similar to AOP)
```java
private static int withLog(int value) {
      System.out.print("Operation logged for " + value + " - ");
      return value;
}

private static int executeWithLog(Function<Integer, Integer> consumer, int value) {
      System.out.print("Operation logged for " + value + " - ");
      return consumer.apply(value);
}

System.out.println(executeWithLog(x -> x * x, 5));
public int executeBefore(
     Function<Integer, Integer> beforeFunction,
     Function<Integer, Integer> function,
     Integer value) {
         beforeFunction.apply(value);
         return function.apply(value);
}

public int executeAfter(
        Function<Integer, Integer> function,
        Function<Integer, Integer> afterFunction,
        Integer value) {
         int result =  function.apply(value);
         afterFunction.apply(result);
         return result;
}
```

### Factory Pattern
```java
Supplier<DirtVacuumCleaner> dvcSupplier =
           DirtVacuumCleaner::new;
       dvc = dvcSupplier.get();
Command pattern
The command pattern is useful for storing an arbitrary set of operations that can be executed at a later time. It has been used to support GUI action controls such as buttons and menus, recording macros, and supporting undo operations. It is a behavioral design pattern where an object encapsulates the information needed to perform an operation at a later time.

public class FunctionalCommands {
       private final List<Supplier<Boolean>> commands =
           new ArrayList<>();
       public void addCommand(Supplier<Boolean> action) {
           commands.add(action);
      }
      public void executeCommand() {
             commands.forEach(Supplier::get);
      } 
}

Character character = new Character();
FunctionalCommands fc = new FunctionalCommands();
fc.addCommand(() -> character.walk());
fc.addCommand(() -> character.run());
fc.addCommand(() -> character.jump());
fc.executeCommand();
```

### Strategy Pattern
The pattern does not use inheritance, but rather encapsulates the behavior in another class. This composition approach decouples the behavior from the classes that use the behavior. Changing the behavior does not affect the class that uses it.

The SchedulingStrategy interface shown next will be implemented by the various scheduling 
algorithms. Each algorithm will use a different approach to select the next task to be 
performed. The nextTask method will return this task: we will use three different algorithms: 
first-come-first-serve, shortest-task-first, and longest-task-first.

```java
@FunctionalInterface
public interface SchedulingStrategy {
  Task nextTask(List<Task> tasks);
}

Comparator<Task> comparator = (x,y) -> x.getDuration()-y.getDuration();
SchedulingStrategy STFStrategy = t -> t.stream().min(comparator).get();

SchedulingStrategy FCFSStrategy = t -> t.get(0);

SchedulingStrategy LTFStrategy = t -> t.stream().max(comparator).get();
```
The functional programming solution eliminated the need for the three strategy classes 
and facilitated the implementation of simpler strategy algorithms.

### Template Pattern
The template pattern is based around the idea that certain problems have structures 
that are reflected in a core method. This method uses the same set of operations to 
perform a task. This can be seen in a loading task where the basic steps to load a 
container is essentially the same whether the container is a box or a truck.

### Summary
Lambda expressions were used as an alternate means of expression functionality. 
Streams allowed us to combine operations. 
Functional interfaces and default methods allowed us to reduce the amount of 
coding required to implement a solution. 
However, functional interfaces can limit how problems can be addressed 
since it supports a single abstract method.

