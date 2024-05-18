---
title: Stream 流式编程：并行流
date: 2023/11/01
categories:
  - coding
tags:
  - 技术文章
  - stream
  - 并行流
abbrlink: 52392
---
# 1、基本概念

Stream是工作中经常用到的集合操作工具，Stream将要处理的元素集合看作一种流，在流的过程中，借助Stream API对流中的元素进行操作，比如：筛选、排序、聚合等。同时它提供的parallelStream方法能够充分利用多核CPU的优势，使用多线程加快对集合数据的处理速度

相对于顺序流而言，并行流在执行某些中间操作时，会自动将数据分成若干个小块，并在多个线程中进行处理，最终将结果合并起来。可以通过调用`parallel()`方法将顺序流转换为并行流

例如，我们可以使用以下代码使用并行流对一个整数列表进行求和：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10); 
int sum = numbers.parallelStream() .mapToInt(Integer::intValue) .sum();
```


这里，parallelStream()方法创建一个并行流，mapToInt()方法将Stream中的元素转换为int类型，sum()方法对所有元素求和。

需要注意的是，并行流并不是适用于所有情况的，如果数据量较小或者处理操作复杂度较低，使用并行流反而会使程序变慢。此外，使用并行流时，需要考虑并发安全问题，确保多个并行操作之间不会发生冲突。所以，在开发中需要根据具体的数据量和操作复杂度来决定是否使用并行流。


# 2、使用并行流提高性能

1、创建并行流：要创建一个并行流，只需在普通流上调用 parallel() 方法。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5); 
Stream<Integer> parallelStream = numbers.parallelStream();
```

2、利用任务并行性：并行流会将数据分成多个小块，并在多个线程上并行处理这些小块。这样可以充分利用多核处理器的优势。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5); 
numbers.parallelStream() .map(n -> compute(n)) // 在多个线程上并行处理计算 
.forEach(System.out::println);
```

在这个示例中，使用 map 方法对流中的每个元素进行计算。由于并行流的特性，计算操作会在多个线程上并行执行，提高了计算的效率。

3、避免共享可变状态：在并行流中，多个线程会同时操作数据。如果共享可变状态（如全局变量）可能导致数据竞争和不确定的结果。因此，避免在并行流中使用共享可变状态，或者采取适当的同步措施来确保线程安全。

4、使用合适的操作：一些操作在并行流中的性能表现更好，而另一些操作则可能导致性能下降。一般来说，在并行流中使用基于聚合的操作（如 reduce、collect）和无状态转换操作（如 map、filter）的性能较好，而有状态转换操作（如 sorted）可能会导致性能下降。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
 
// good performance
int sum = numbers.parallelStream()
                 .reduce(0, Integer::sum);
 
// good performance
List<Integer> evenNumbers = numbers.parallelStream()
                                   .filter(n -> n % 2 == 0)
                                   .collect(Collectors.toList());
 
// potential performance degradation
List<Integer> sortedNumbers = numbers.parallelStream()
                                     .sorted()
                                     .collect(Collectors.toList());
```

 

 
在这个示例中，reduce 和 filter 的操作在并行流中具有良好的性能，而 sorted 操作可能导致性能下降。

除了上述方法，还应根据具体情况进行评估和测试，并行流是否能够提高性能。有时候，并行流的开销（如线程的创建和销毁、数据切割和合并等）可能超过了其带来的性能提升。因此，在选择使用并行流时，应该根据数据量和操作复杂度等因素进行综合考虑，以确保获得最佳的性能提升。


# 3、并行流的适用场景

1、大规模数据集：当需要处理大规模数据集时，使用并行流可以充分利用多核处理器的优势，提高程序的执行效率。并行流将数据切分成多个小块，并在多个线程上并行处理这些小块，从而缩短了处理时间。

2、复杂的计算操作：对于复杂的计算操作，使用并行流可以加速计算过程。由于并行流能够将计算操作分配到多个线程上并行执行，因此可以有效地利用多核处理器的计算能力，提高计算的速度。

3、无状态转换操作：并行流在执行无状态转换操作（如 map、filter）时表现较好。这类操作不依赖于其他元素的状态，每个元素的处理是相互独立的，可以很容易地进行并行处理。

