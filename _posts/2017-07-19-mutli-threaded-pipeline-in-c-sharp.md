---
layout: post
title: Mutli-threaded pipeline in C#
tags: [.NET, C#, Threading, Code example, Toolbox, Github]
categories: [Programming, .NET]
image: '/images/posts/assembly-line.jpg'
redirect_from:
  - /blog/2017/7/7/net-threaded-pipeline
---

As the processors of today's computers are getting increasingly more powerful and with the raise of multi-core processors, multi-threading is becoming increasingly more important for CPU intensive tasks.

Although not all tasks are suitable for execution in complete parallel, as the input-output order of data is crucial and it may consist of a series of operations which need to be executed sequentially, most tasks can be split into smaller isolated chunks of operations.

This is where a pipeline pattern, or pipes and filters pattern, is useful.

A pipeline is similar to a real world assembly line, you have a series of highly specialized processors which applies its operation and hands it to the next processor. For example in a vehicle assembly line, one cuts the metal, one shapes the metal, one welds the metal and one paints the metal.

In the illustration below, it is shown how different stages are connected using blocking buffers to perform single specific tasks with multiple operations in parallel where each task runs in its own thread.

![Pipeline](/images/posts/pipeline.png)

Source: [Microsoft](https://msdn.microsoft.com/en-us/library/ff963548.aspx)

The .NET framework provides very useful tools and building blocks for performing these kinds of producer-consumer patterns using multiple threads in its System.Threading namespace and you can read more about pipelines in C# [here](https://msdn.microsoft.com/en-us/library/ff963548.aspx).

The problem implementing pipelines using the elements provided in the System.Threading namespace is that you can easily end up with lots of nasty boilerplate code for BlockingCollection buffers, Task initialization and stage logic.

To combat the boilerplate code and to encourage myself to implement the pipeline pattern more often in different contexts without much work overhead, I implemented a small framework to provide a fluent-like API for making pipeline patterns.

The basic idea looks like this:

```c#
Pipeline pipeline = Pipeline.First<Read>()
    .Then<Parse>()
    .Then<Add>()
    .Then<Multiply>()
    .Finally<Write>();
```

The code basically speaks for itself. The pipeline is defined as five operations, first the pipeline reads, then it parses, then performs addition, then it multiplies and finally it writes.

The pipeline framework has three abstract classes as building blocks, or stages, and these are as follows:

- InitialBlock<TOutput>
- IntermediateBlock<TInput, TOutput>
- FinalBlock<TInput>

These three block types define different behavior and positions in the pipeline.

An initial block is purely a producer and the abstraction allows slightly more liberty in its implementations to accommodate greater freedom for supplying data input to the pipeline. Being an IProducer block, the InitialBlock exposes the Then<>() and Finally<>() methods, which allow you to connect the next block or stage in the pipeline.

The initial block implementation could look like this:

```c#
public class Read : InitialBlock<string>
{
    private const string FILE_NAME = "Numbers.txt";

    public override void Process()
    {
        using (Stream stream = File.OpenRead(FILE_NAME))
        using (StreamReader reader = new StreamReader(stream))
        {
            string line;
            while((line = reader.ReadLine()) != null)
            {
                Output.Add(line);
            }
        }
    }
}
```

When implementing the initial block, all required is to implement the Process method. In the Process, you need to define whatever loop will be responsible for adding your inputs into the output collection. This could be a text file containing multiple gigabytes of numbers on separate lines being read as a stream.

The intermediate block is both a consumer and a producer and acts, as the name says, an intermediate block in the chain of stages, either after an initial block, other intermediate block, or before a final block. As with the initial block, it exposes Then and Finally methods.

An example of the Parse intermediate block:

```c#
public class Parse : IntermediateBlock<string, int?>
{
    protected override int? Process(string item)
    {
        if (int.TryParse(item, out int result))
        {
            return result;
        }
        else
        {
            return null;
        }
    }
}
```

The intermediate block is relative simple to implement. The process is simply to implement to Process method which will receive a single item of the desired input type and return a single item of the desired output type.

In here, you can do whatever is needed, some examples are: convert data, filter data, cache data, access I/O devices, process database access, process images and much more.

There is one important thing to note here, I am using a *nullable* int. This is because the framework is designed to discard nulls as a way to filter along the process. In the future, I believe this should be a configurable behavior in case, as it may not be desirable behavior in all cases.

The last block type is the FinalBlock. The purpose here is to output the processed data, e.g. to the network, to a database, to a user interface or to files on the disk. This type of block is required to populate the Finally<>() method exposed by an InitialBlock or an IntermediateBlock.

This will illustrate the implementation of the Write final block:

```c#
public class Write : FinalBlock<int>
{
    protected override void Process(int item)
    {
        Repository.Numbers.Add(item);
    }
}
```

The final thing is to implement the Process method which receives an item of the desired type and handle it however is wanted. In this example, it is stored to a database.

These are the basic steps for using the framework. It does have more advanced features, but those will not be covered here.

As the last notes, behind the scenes, when the final block is cast to a Pipeline object, the framework will start all blocks in individual threads and the pipeline will start processing. As soon as the Process of the InitialBlock class exits, the Pipeline will start closing down by marking the buffers as completed as they go empty and eventually all tasks will have shutdown.

And finally, as shown below, a Pipeline class implements an implicit operator to allow an implicit cast from a Pipeline to a collection of Task objects and allows the pipeline to be waited using the Task WaitAll method:

```c#
Pipeline pipeline = Pipeline.First<Read>()
    .Then<Parse>()
    .Then<Add>()
    .Then<Multiply>()
    .Finally<Write>();

Task.WaitAll(pipeline);
```