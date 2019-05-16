

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
The methods takeWhile and dropWhile are more efficient than a filter when you know that the source is sorted. 

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

`Arrays.stream()`
`Stream.generate(Math::random)`
`IntStream.iterate(0, n -> n < 100, n -> n + 4)`
`IntStream.rangeClosed(1, 100).boxed()`


Java 9, `Stream.ofNullable`

Some operations such as filter and map are stateless: they don’t store any state. Some operations such as reduce store state to calculate a value. Some opera- tions such as sorted and distinct also store state because they need to buffer all the elements of a stream before returning a new stream. Such operations are called stateful operations.

There are three primitive specializations of streams: IntStream, DoubleStream, and LongStream. Their operations are also specialized accordingly.

Streams can be created not only from a collection but also from values, arrays, files, and specific methods such as iterate and generate.

## Collectors
`import static java.util.stream.Collectors.*;`

`long howManyDishes = menu.stream().collect(Collectors.counting());` is the same as 
`long howManyDishes = menu.stream().count();`

Max
```
Comparator<Dish> dishCaloriesComparator =
    Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish =
    menu.stream()
        .collect(maxBy(dishCaloriesComparator));
```

Sum
```int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

Average
``` double avgCalories =
            menu.stream().collect(averagingInt(Dish::getCalories));
```

Summary Stats
``` IntSummaryStatistics menuStatistics =
                menu.stream().collect(summarizingInt(Dish::getCalories));
    
    IntSummaryStatistics{count=9, sum=4300, min=120,
                     average=477.777778, max=800}
```

Join Strings
```
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

All the collectors we’ve discussed so far are, in reality, only convenient specializations of a reduction process that can be defined using the reducing factory method. For example, 
sum 
```
int totalCalories = menu.stream().collect(reducing(
                                           0, Dish::getCalories, (i, j) -> i + j));
```
max
```
Optional<Dish> mostCalorieDish =
    menu.stream().collect(reducing(
        (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

Multiple way to join string
```String shortMenu = menu.stream().map(Dish::getName)
.collect( reducing( (s1, s2) -> s1 + s2 ) ).get()
```

```
String shortMenu = menu.stream()
.collect( reducing( "", Dish::getName, (s1, s2) -> s1 + s2 ) );
```

groupingBy
```
Map<Dish.Type, List<Dish>> dishesByType =
                      menu.stream().collect(groupingBy(Dish::getType));
```
note that the regular one-argument groupingBy(f), where f is the classification function is, in reality, shorthand for groupingBy(f, toList()).

```
public enum CaloricLevel { DIET, NORMAL, FAT }
Map<CaloricLevel, Set<Dish>> dishesByCaloricLevel = menu.stream().collect(
groupingBy(dish -> {
  if (dish.getCalories() <= 400) return CaloricLevel.DIET;
  else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL; 
  else return CaloricLevel.FAT;
  },
  toCollection(HashSet::new)
));
```

```
Map<Dish.Type, List<String>> dishNamesByType =
      menu.stream()
          .collect(groupingBy(Dish::getType,
                   mapping(Dish::getName, toList())));
```

```
Map<Dish.Type, Set<String>> dishNamesByType =
   menu.stream()
      .collect(groupingBy(Dish::getType,
               flatMapping(dish -> dishTags.get( dish.getName() ).stream(),
toSet())));
```

Multilevel grouping
```
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
menu.stream().collect( 
  groupingBy(Dish::getType,
      groupingBy(dish -> {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL; 
        else return CaloricLevel.FAT;
      })
);
```

Collecting data in subgroups
```
Map<Dish.Type, Long> typesCount = menu.stream().collect(
                    groupingBy(Dish::getType, counting()));
```
```
Map<Dish.Type, Integer> totalCaloriesByType =
               menu.stream().collect(groupingBy(Dish::getType,
                        summingInt(Dish::getCalories)));
```
```
Map<Dish.Type, Optional<Dish>> mostCaloricByType =
    menu.stream()
        .collect(groupingBy(Dish::getType,
                            maxBy(comparingInt(Dish::getCalories))));
```
```
Map<Dish.Type, Dish> mostCaloricByType =
    menu.stream()
    .collect(groupingBy(Dish::getType,
        collectingAndThen(
          maxBy(comparingInt(Dish::getCalories)),
        Optional::get)));
```

partitioningBy
```
Map<Boolean, List<Dish>> partitionedMenu =
             menu.stream().collect(partitioningBy(Dish::isVegetarian));

{false=[pork, beef, chicken, prawns, salmon],
 true=[french fries, rice, season fruit, pizza]}
```
Similar to 
```
List<Dish> vegetarianDishes =
              menu.stream().filter(Dish::isVegetarian).collect(toList());
```
Partitioning has the advantage of keeping both lists of the stream elements, for which the application of the partitioning function returns true or false.
```
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType =
        menu.stream().collect(
          partitioningBy(Dish::isVegetarian,
                       groupingBy(Dish::getType)));
                       
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},
 true={OTHER=[french fries, rice, season fruit, pizza]}
```

```
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian =
menu.stream().collect(
    partitioningBy(Dish::isVegetarian,
         collectingAndThen(maxBy(comparingInt(Dish::getCalories)),
                           Optional::get)));

{false=pork, true=pizza}
```








