How to blend the various programming styles available in Java 8 to meet an application's need?

First-class and high-order functions are associated with functional programming. A first-class function is a computer science term. It refers to functions that can be used anywhere a first-class entity can be used. A first-class entity includes elements such as numbers and strings. They can be used as an argument to a function, returned from a function, or assigned to a variable.
High-order functions depend upon the existence of first-class functions. They are functions that either:
• Take a function as an argument
• Return a function
Java 8 has introduced the concept of lambda expressions to the language. These are essentially anonymous functions that can be passed to and returned from functions. They can also be assigned to a variable. 

A pure function is a function that has no side effects. This means that memory external to the function is not modified, IO is not performed, and no exceptions are thrown. With a pure function, when it is called repeatedly with the same parameters, it will return the same value. This is called referential transparency.
With referential transparency, it is permissible to modify local variables within the function as this does not change the state of the program. Any changes are not seen outside of the function.
Advantages of pure function include:
• The function can be called repeatedly with the same argument and get the same results. This enables caching optimization (memorization).
• With no dependencies between multiple pure functions, they can be reordered and performed in parallel. They are essentially thread safe.
• Pure function enables lazy evaluation as discussed later in the Strict versus non-strict evaluation section. This implies that the execution of the function can be delayed and its results can be cached potentially improving the performance of a program.
• If the result of a function is not used, then it can be removed since it does not affect other operations.

Closure
Closure refers to functions that enclose variable external to the function. This permits the function to be passed around and used in different contexts.

Currying
Some functions can have multiple arguments. It is possible to evaluate these arguments one-by-one. This process is called currying and normally involves creating new functions, which have one fewer arguments than the previous one.
The advantage of this process is the ability to subdivide the execution sequence and work with intermediate results. This means that it can be used in a more flexible manner.
Function<String, Function<String, String>> curryConcat;
curryConcat = (a) -> (b) -> biFunctionConcat.apply(a, b);


Functional programming places more emphasis on how these functions are arranged and combined. It is this composition of functions, which typifies a functional style of programming. Functions are not only used to organize the execution process, but are also passed and returned from functions. Often data and the functions acting on the data are passed together promoting more capable and expressive programs.

The incorporation of these functional programming techniques does not make Java a functional programming language. It means that we now have a new set of tools that we can use to solve the programming problems presented to us. It behooves us to take advantage of these techniques whenever they are applicable.


• Lambda expressions: Lambda expressions are essentially anonymous functions. 
• Functional interfaces
• Method and constructor references

A functional interface is an interface that has one and only one abstract method. The Computable interface declared in the previous section is a functional interface. It has one and only one abstract method: compute. If a second abstract method was added, the interface would no longer be a functional interface.

Lambda expressions are often converted to a functional interface automatically simplifying many tasks. Lambda expressions can access other variables outside of the expression. The ability to access these types of variables is an improvement over anonymous inner functions, which have problems in this regard.

A method or constructor reference is a technique that allows a Java 8 programmer to use a method or constructor as if it was a lambda expression. 



Lambda expressions can throw exceptions. However, they must only be those specified by its functional interface's abstract method.


Functional Interface in Java
We can group these interfaces into five categories:
• Function: These transform their arguments and return a value
     Function<T,R>      R apply(T t)
     BiFunction<T,U,R>  R apply(T t, U u)
• Predicate: These are used to perform a test, which returns a Boolean value
      Predicate<T>      boolean test(T t)
      BiPredicate<T,U>  boolean test(T t, U u)
• Consumer: These use their arguments, but do not return a value
      Consumer<T>       void accept(T t)
      BiConsumer<T,U>   void accept(T t, U u)
• Supplier: These are not passed data, but do return data
      Supplier<T>       T get()
• Operator: These perform a reduction type operation
      UnaryOperator<T>  R apply(T t)
      BinaryOperator<T> R apply(T t1, T t2)
      
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
Method composition provides a flexible way of combining two or more functions into a single function. This offers flexibility that will otherwise not be present. We illustrate how this technique can be implemented using the Function interface. Its compose and andThen methods support the execution of functions before or after another function. 

As languages mature, they tend to find better ways of doing things. 


Diamond problem
a class that implements several interfaces that have the same default method implemented, has to implement this method itself. We are going to explain this with an example.
