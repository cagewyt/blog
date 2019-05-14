

## Stream
Similarly to iterators, a stream can be traversed only once.

Streams and collections philosophically
For readers who like philosophical viewpoints, you can see a stream as a set of val- ues spread out in time. 
In contrast, a collection is a set of values spread out in space (here, computer memory), 
which all exist at a single point in time—and which you access using an iterator to access members inside a for-each loop.


Using the Collection interface requires iteration to be done by the user (for exam- ple, using for-each); 
this is called **external iteration**. The Streams library, by contrast, uses **internal iteration**—it does the iteration 
for you and takes care of storing the result- ing stream value somewhere; 
you merely provide a function saying what’s to be done.

intermediate methods
`filter` keeps the predicate true
Java 9 added two new methods that are useful for efficiently selecting elements in a stream: 
`takeWhile` it stops once it has found an element that fails to match
`dropWhile` It throws away the ele- ments at the start where the predicate is false. Once the predicate evaluates to true it stops and returns all the remaining elements, 
`limit` keeps the first n
`skip`  skips the first n

`map`   map to R from Function<T, R>
`flatMap` replaces each value of a stream with another stream and then concatenates all the generated streams into a single stream. 

```java
List<String> uniqueCharacters =
  words.stream()
       .map(word -> word.split(""))
       .flatMap(Arrays::stream)
       .distinct()
       .collect(toList());
```

Find and matching terminal methods (Short-circuiting)
`allMatch`   boolean  match all,    return true for empty stream, read [this](https://stackoverflow.com/questions/30223079/why-does-stream-allmatch-return-true-for-an-empty-stream)

`noneMath`   boolean  no match, the opposite of allMatch       return true for empty stream

`anyMath`    boolean  match at least one element,    return false for empty stream

`findAny`    Optional find an arbitrary element, good for parallel stream

`findFirst`  Optional find the first encountered element

Reducing terminal methods

`reduce`     initial value, and BinaryOperator<T> to combine two elements and produce a new value. 
  
`int sum = numbers.stream().reduce(0, (a, b) -> a + b);`

or

`int sum = numbers.stream().reduce(0, Integer::sum);`


There’s also an overloaded variant of reduce that doesn’t take an initial value, but it returns an Optional object:
`Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));`

Why does it return an Optional<Integer>? Consider the case when the stream con- tains no elements. The reduce operation can’t return a sum because it doesn’t have an initial value. This is why the result is wrapped in an Optional object to indicate that the sum may be absent. 

`Optional<Integer> max = numbers.stream().reduce(Integer::max);`

`count`
`collect`
`forEach`

