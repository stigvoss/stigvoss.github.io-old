---
layout: post
title: TimeoutStream and .NET Standard 2.0
tags: [.NET, C#, Code example, I/O, Toolbox, Github]
categories: [Programming, .NET]
---

Currently, Microsoft is working hard on providing better cross-platform tools for  .NET developers and this is primarily manifested in their effort of  developing the .NET Core and .NET Standard frameworks for cross-platform deployment of e.g. .NET Core MVC applications on Windows, several Linux distributions, and macOS.

One of the open-source frameworks attempting to support the .NET Standard 2.0 is the [Accord.NET framework](http://accord-framework.net/). As the .NET Standard is only an API specification and not a  specification of implementation, certain differences will be found  between the full .NET framwork and e.g. .NET Core 2.0 which honors the  .NET Standard 2.0.

During a experiment with Accord.NET in .NET  Core 2.0, an incompatibility arose. The primary cause of the issue was  related to the implementation of HttpWebResponse in .NET Standard 2.0,  which in .NET Core 2.0, uses the [WinHttpResponseStream](https://github.com/dotnet/corefx/blob/master/src/System.Net.Http.WinHttpHandler/src/System/Net/Http/WinHttpResponseStream.cs) instead of the [ConnectStream](https://github.com/Microsoft/referencesource/blob/master/System/net/System/Net/_ConnectStream.cs), as used by full .NET Framework.

As Accord.NET is a free and open-source framework, I was able to investigate the issue and provide a solution for the framework.

The solution was implemented based on the following idea:

```c#
try
{
    // set timeout cancellation token
    tokenSource.CancelAfter(requestTimeout);
    // read next portion from stream, enforce timeout
    read = stream.ReadAsync(buffer, total, readSize, tokenSource.Token).Result;
}
catch (AggregateException ae)
{
    if(ae.InnerException is TaskCanceledException)
    {
        // reset canceled token source
        tokenSource = new CancellationTokenSource();

        throw new TimeoutException("The operation has timed out.");
    }

    throw ae.InnerException;
}
```

The snippet shown above illustrates the solution from the Accord.NET  source. The target is to implement a timeout on the  WinHttpResponseStream, which does not implement timeout, by using the  asynchronous read method and supplying a cancellation token with a  timeout set and immediately read the result in a synchronous fashion. On expiration of the defined token timeout, the read method throws an  AggregateException allowing the cancelled read to be handled and throw a TimeoutException.

The solution code is a messy, but it made  little to no sense to put greater effort into the niceness of the fix.  As an example it wasn't ensured that the TimeoutException aligned with  the existing exception used by ConnectStream, which throws a  WebException.

The drawback of the fix is the requirement of the  System.Threading namespace and therefore it will not compile in  frameworks older than .NET Framework 4.0, which caused a minor issue for Accord.NET, but the issue was easily fixed using a few compiler  pre-processing conditions.

To re-purpose the fix for a more generic use-case, I implemented a [TimeoutStream wrapper](https://github.com/stigvoss/Toolbox/blob/00188e1c8975677db34050d2a268eb147c84354d/Toolbox/IO/TimeoutStream.cs) for streams which became a part of my [Toolbox](https://github.com/stigvoss/Toolbox/tree/develop) pet project.

The wrapper allows the following code using a base stream which otherwise would not allow timeouts:

```c#
TimeoutStream timeoutStream = new TimeoutStream(stream);
timeoutStream.ReadTimeout = ...;

timeoutStream.Read(buffer, offset, count);
```

The functionality in the wrapper is as simple as primarily just passing through the calls to the base stream such as here:

```c#
public override long Seek(long offset, SeekOrigin origin)
{
    return _baseStream.Seek(offset, origin);
}
```

The is the rule with a couple of exceptions which are related to the  timeout functionality. The read method is implemented as shown here:

```c#
public override int Read(byte[] buffer, int offset, int count)
{
    int result;

    if (_baseStream.CanRead && !_baseStream.CanTimeout)
    {
        try
        {
            _source.CancelAfter(_readTimeout);
            Task<int> task = _baseStream.ReadAsync(buffer, offset, count, 
                _source.Token);
            result = task.Result;
        }
        catch (AggregateException)
        {
            _source = new CancellationTokenSource();
            throw new TimeoutException("The operation timed out.");
        }
    }
    else
    {
        result = _baseStream.Read(buffer, offset, count);
    }
    
    return result;
}
```

This implementation uses the principles as shown in the earlier version  implemented in Accord.NET. The implementation will ensure that the base  stream does not already allow timeouts to avoid possible performance  degradation using this more complex implementation.

Unlike earlier, the implementation has also narrowed its exception handling to focus purely on AggregateException.

Image credits: [Xomiele on Flicker](https://www.flickr.com/photos/xomiele/4824711225)