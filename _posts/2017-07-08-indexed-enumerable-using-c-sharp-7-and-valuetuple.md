---
layout: post
title: Indexed enumerator using C# 7 and ValueTuple
tags: [C#, ValueTuple, Toolbox, .NET, Code example, Github]
categories: [Programming, .NET]
image: '/images/posts/indexedenumerable.png'
redirect_from:
  - /blog/2017/7/8/indexed-enumerable-using-c-sharp-7-and-valuetuple
---

One of the new exciting features of C# 7 is the introduction of the struct- and instance field-based ValueTuple. Due to changed inner workings of a ValueTuple compared to the classic Tuple, deconstructors and syntactic  sugar in Visual Studio, tuples have become a delight to use without  compromising the readability of code.

Combining ValueTuples with  extension methods is a great way to solve a long-standing and relatively hideous looking piece of code. More precisely, keeping count of an  index while enumerating a collection.

Classically, one would probably solve this using one of the two following solutions:

```c#
// Using a for-loop to enumerate and assigning an item to a variable
int[] collection = new int[] { 100, 200, 300 };
for (int index = 0; index < collection.Length; index++)
{
    int item = collection[index];
    Console.WriteLine($"{index}: {item}");
}
// Using a foreach-loop to enumerate and increment an index counter
int[] collection = new int[] { 100, 200, 300 };
int index = 0;
foreach (int item in collection)
{
    Console.WriteLine($"{index}: {item}");
    index++;
}
```

For different types of collections, different approaches may be preferred.  For example, for obvious reasons, enumerating a LinkedList using a  for-loop and attempting to retrieve by indices (e.g. using the ElementAt method from LINQ) in an iterative manner is a extremely bad idea.

Implementing an indexed enumerator wrapper around an ordinary enumerator, as shown [here](https://github.com/stigvoss/Toolbox/tree/9b4b4c5db1ecbb9015ba498207102a4ce12922e4/Toolbox/Collections), offers the ability to generate and access an index along side enumeration of an enumerable data type.

The following example will illustrate the use of the wrapper in practice:

```c#
// Using a foreach-loop to enumerate an indexed Enumerable
int[] collection = new int[] { 100, 200, 300 };
foreach ((int item, int index) in collection.AsIndexedEnumerable())
{
    Console.WriteLine($"{index}: {item}");
}
```

As shown above, before enumerating the collection, the AsIndexedEnumerable extension method is called to create an instance of the  IndexedEnumerator wrapper around the collection's enumerator.

Following, the ValueTuple created in the IndexedEnumerator is deconstructed into  an item and an index resulting in a compact, neat and easy-to-read  standard looking foreach-loop.

The two core pieces of this  IndexedEnumerator wrapper are a simple counter and a simple construction of a ValueTuple as shown here:

```c#
public bool MoveNext()
{
    bool result = _enumerator.MoveNext();
    if (result)
    {
        _index++;
    }
    return result;
}
public (T element, int index) Current => (_enumerator.Current, _index);
```

Upon call of the MoveNext method of the enumerator, the IndexedEnumerator  will first call the wrapped enumerator and increment an index counter.  Thereafter, when calling the Current property, a ValueTuple will be  constructed using the index and the wrapped enumerator's Current  property.

The implementation is not pure good news though. A major drawback which has been proven in tests, is a relative large  performance penalty. Compared to a call of the AsEnumerable method, the  tests showed an almost threefold increase in execution time. Compared to executing a for each loop on a plain integer array, the  IndexedEnumerator showed an eight times speed penalty.

I have not yet attempted any sort of speed optimizations, but it is not unlikely that such would possible.

A major benefit of the implementation is that it allows any existing  enumerator to be used and no re-implementation is needed. At the same  time, it creates more clean and neat looking code with no counter  variable and visible increments. If one is not working with huge  collections or frequent iterations, using this solution may be viable,  but as always, when performance and speed is a concern, other  optimizations may need to be considered.