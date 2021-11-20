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

- Create a hash map to count the number of possible `products`
- Loop through the list of `reviews`
- Split the comment by non-word characters
- Increment the count in the products map every time we see a word that matches a product in `products`
- Sort the `products` by count in decending order
- Return the top `N` products


## Java Streams

Now let's solve the same problem using Java Streams.


``` java
List<String> topProducts(int N,
                        List<String> products,
                        List<String> reviews)
{
    Set<String> productsIgnoreCase = products.stream()
        .map(String::toLowerCase)
        .collect(toSet());

    return reviews.parallelStream()
        .flatMap(request -> Arrays.stream(request.trim().split("\\s+")))
        .map(String::toLowerCase)
        .filter(productsIgnoreCase::contains)
        .collect(Collectors.groupingByConcurrent(Function.identity(), Collectors.counting()))
        .entrySet()
        .stream()
        .sorted(((o1, o2) -> o2.getValue().compareTo(o1.getValue())))
        .map(Map.Entry::getKey)
        .limit(N)
        .collect(Collectors.toCollection(ArrayList::new));
}

```


