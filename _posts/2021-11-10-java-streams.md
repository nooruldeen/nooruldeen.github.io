---
layout: post
title: Java Streams
date: 2021-11-10 23:00
category: java
author: Nooruldeen Alsaif
tags: [java, java-8, java-streams]
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

```java
List<String> topProductsClassic(List<String> reviews,
                            int N,
                            List<String> products)
{
    Map<String, Integer> productsCount = new HashMap<>();
    for (String review: reviews){
        String[] words = review.split("\\s+");
        for (String word: words){
            if (products.contains(word)){
                if (productsCount.containsKey(word)){
                    int count = productsCount.get(word) + 1;
                    productsCount.put(word, count);
                } else {
                    productsCount.put(word, 1);
                }
            }
        }
    }
    List<Map.Entry<String,Integer>> sortedEntries = new ArrayList<>(productsCount.entrySet());
    sortedEntries.sort(Map.Entry.<String, Integer>comparingByValue().reversed());
    List<String> results = new ArrayList<>();
    sortedEntries.subList(0, N).forEach(e -> {
        results.add(e.getKey());
    });
    return results;
}
```

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

- `parallelStream`: Returns a possibly parallel Stream with this collection as its source. It is allowable for this method to return a sequential stream. 
- `flatMap`: Returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element. 
- `map`: Returns a stream consisting of the results of applying the given function to the elements of this stream.
- `filter`: Returns a stream consisting of the elements of this stream that match the given predicate.
- `collect`: Performs a mutable reduction operation on the elements of this stream using a `Collector`.
- `Collectors.groupingByConcurrent`: Returns a concurrent `Collector` implementing a "group by" operation on input elements of type `T`, grouping elements according to a classification function.
- `entrySet`: Returns a Set view of the mappings contained in this map. The set is backed by the map, so changes to the map are reflected in the set, and vice-versa. 
- `stream`: Returns a sequential `Stream` with this collection as its source.
- `sorted`: Returns a stream consisting of the elements of this stream, sorted according to natural order. 
- `limit`: Returns a stream consisting of the elements of this stream, truncated to be no longer than `maxSize` in length.
- `Collectors.toCollection`: Returns a `Collector` that accumulates the input elements into a new `Collection`, in encounter order. The `Collection` is created by the provided factory.


## Conclusion

Java Streams was introduced in Java 8. It has intermidate and terminal operations to process streams. It increases the readability of code by grouping the operations applied to the same data in one stream set of operations. It supports both serial and parrallel steaming.

Happy coding!