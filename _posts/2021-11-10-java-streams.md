---
layout: post
title: Java Streams
date: 2021-11-10 23:00
category: java
author: Nooruldeen Alsaif
tags: [java, java-11, java-streams]
summary: An advanced Java Streams example
---

Java Streams was introduced in Java 8. In this article we will see how we can use Java Streams to write a function in a more elegant and fancy way. 

If you like to learn how to write few lines that can solve complex problems then you should give Java Streams a try.

## The Problem

Let's start with a problem statement, think about how we can solve it using loops and if-else statements, then we will see how to solve it using Java Streams.

You have been asked to write a function to return a list of the top `N`  products from a list of `products` that appear in customer comments or `reviews`.


## Solving Problems The Old Way

Before Java Streams, this problem can solved as follows:

- Create a hash map `productsCount` to count the number of possible `products`
- Loop through the list of `reviews`
- Split the comment by non-word characters
- Increment the count in `productsCount` every time we see a word that matches a product in `products`
- Sort the `productsCount` by count in decending order
- Return the top `N` products

You can notice something here. There is a data flow from one step to another, like a pipe-and-filter stream. This is your indicator that this problem can be solved using functional programming, or Java Streams.

## Java Streams

Now let's solve the same problem using Java Streams.


```java
List<String> topProducts(List<String> reviews,
                        int N,
                        Set<String> products)
{
    return reviews.parallelStream()
        .flatMap(review -> Arrays.stream(review.trim().split("\\s+")))
        .map(String::toLowerCase)
        .filter(products::contains)
        .collect(Collectors.groupingByConcurrent(Function.identity(), Collectors.counting()))
        .entrySet()
        .stream()
        .sorted(((o1, o2) -> o2.getValue().compareTo(o1.getValue())))
        .map(Map.Entry::getKey)
        .limit(N)
        .collect(Collectors.toCollection(ArrayList::new));
}

```

Java Streams supports multi-threding out of the box. In the above example, we used `parallelStream` to stream reviews which will run the next steps in multiple threads under the hood. 

Let's hava a look at each function we used above to see how they work (Reference: [Java 8 Docs](https://devdocs.io/openjdk~8/java/)):

- `parallelStream`: Returns a possibly parallel Stream with this collection as its source. It is allowable for this method to return a sequential stream. For serial streams, use `stream` instead.
- `flatMap`: Returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element. Each mapped stream is closed after its contents have been placed into this stream. (If a mapped stream is `null` an empty stream is used, instead.)
- `map`: Returns a stream consisting of the results of applying the given function to the elements of this stream.
- `filter`: Returns a stream consisting of the elements of this stream that match the given predicate.
- `collect`: Performs a mutable reduction operation on the elements of this stream using a `Collector`. A `Collector` encapsulates the functions used as arguments to collect(Supplier, BiConsumer, BiConsumer), allowing for reuse of collection strategies and composition of collect operations such as multiple-level grouping or partitioning.
- `Collectors.groupingByConcurrent`: Returns a concurrent `Collector` implementing a "group by" operation on input elements of type `T`, grouping elements according to a classification function. This is a concurrent and unordered `Collector`. The classification function maps elements to some key type `K`. The collector produces a `ConcurrentMap<K, List<T>>` whose keys are the values resulting from applying the classification function to the input elements, and whose corresponding values are `List`s containing the input elements which map to the associated key under the classification function. There are no guarantees on the type, mutability, or serializability of the `Map` or `List` objects returned, or of the thread-safety of the List objects returned.
- `entrySet`: Returns a Set view of the mappings contained in this map. The set is backed by the map, so changes to the map are reflected in the set, and vice-versa. If the map is modified while an iteration over the set is in progress (except through the iterator's own `remove` operation, or through the `setValue` operation on a map entry returned by the iterator) the results of the iteration are undefined. The set supports element removal, which removes the corresponding mapping from the map, via the `Iterator.remove`, `Set.remove`, `removeAll`, `retainAll` and `clear` operations. It does not support the `add` or `addAll` operations.
- `stream`: Returns a sequential `Stream` with this collection as its source.
- `sorted`: Returns a stream consisting of the elements of this stream, sorted according to natural order. If the elements of this stream are not Comparable, a `java.lang.ClassCastException` may be thrown when the terminal operation is executed.
- `limit`: Returns a stream consisting of the elements of this stream, truncated to be no longer than `maxSize` in length.
- `Collectors.toCollection`: Returns a `Collector` that accumulates the input elements into a new `Collection`, in encounter order. The `Collection` is created by the provided factory.


## Conclusion

Java Streams was introduced in Java 8. It has intermidate and terminal operations to process streams. It increases the readability of code by grouping the operations applied to the same data in one stream set of operations. It supports both serial and parrallel steaming.

Happy coding!