---
layout: post
title: Seekable FTP file stream in C#
tags: [C#, I/O, Toolbox, FTP, Stream, Code example, Compression, Github]
categories: [Programming, .NET]
image: '/images/posts/ftp-file-list.jpg'
redirect_from:
  - /blog/2017/7/9/seekable-ftp-file-stream-in-c-sharp
---

## The challenge

Certain types of files are dependent on metadata stored within the file itself  to be read before the file can be used for anything. One of these file  types is the ZIP archive format.

A ZIP-file stores a data  directory at the end of the file specifying metadata for the contained  files and therefore the data directory is required to be read before  attempting to read or extract any of the contained files.

A  benefit of this structure is that it allows the file format to be  self-extracting by prepending the archive data with the application  needed to extract the archive and still let the application seek out the data dictionary as normally.

In .NET, it is fairly straight  forward to read a ZIP-file from the disk. The following example presents a simple way to achieve this:

```c#
// Read number of entries in ZIP-file from disk
string zipFile = @"C:\LargeArchive.zip";
using (Stream stream = File.OpenRead(zipFile))
using (ZipArchive archive = new ZipArchive(stream))
{
    int fileCount = archive.Entries.Count;
    Console.WriteLine($"Count: {fileCount}");
}
```

The [ZipArchive](https://msdn.microsoft.com/da-dk/library/system.io.compression.ziparchive(v=vs.110).aspx) class, which is built into the .NET framework, simply consumes a stream as seen in the example. Allowing the use of a stream creates great  flexibility, but it has its limitations.

As mentioned earlier, a  ZIP-file has the data dictionary located in the of the file and when  processing smaller ZIP-files, the type of stream one uses may not impact the product in any noticeable way, but when talking compressed files of several gigabytes, the type of stream can be rather important as  certain streams do not offer the ability to seek.

Opening a  ZIP-file from such a stream, which does not offer the ability to seek,  will result in the framework buffering the complete file in memory  before processing it. This can have unwanted consequences when  processing large files and it will either cause you to run out of memory while loading or you wait for an extended amount of time for a full  download. This behavior can be rather unfortunate if you are looking for a single smaller file in a large archive of many gigabytes.

A  case where this is true is if you attempt to open a ZIP-file from the  response stream from an FTP server. The response stream is not seekable  and may result in extreme buffering. This is illustrated in this  example:

```c#
// Read number of entries in ZIP-file from FTP server
WebRequest request = WebRequest.Create("ftp://localhost/LargeArchive.zip");
using (WebResponse response = request.GetResponse())
using (Stream stream = response.GetResponseStream())
using (ZipArchive archive = new ZipArchive(stream))
{
    int fileCount = archive.Entries.Count;
    Console.WriteLine($"Number of entries: {fileCount}");
}
```

If the file is adequately large, this operation may last hours or you may  exceed physical memory limits. Just retrieving the number of entries in  the file.

## [A solution](https://github.com/stigvoss/Toolbox/blob/9b4b4c5db1ecbb9015ba498207102a4ce12922e4/Toolbox/IO/SeekableFtpFileStream.cs)

Most current FTP servers allow requests to define an offset in the content  which is requested, this will be the basis for the solution proposed  here.

For solving the challenge, I wrapped an FtpWebRequest  instance in an implementation inheriting from Stream, as a  SeekableFtpFileStream, and thereby allowing the implementation to close  and reopen connections with an offset to skip to file offsets of  interest on FTP.

When seeking for a position with a lower index in the stream than the current position, the implementation will close and reopen a new connection with an offset equating to the byte position  desired by the request to seek.

The source code for the solution can be seen following [this link](https://github.com/stigvoss/Toolbox/blob/9b4b4c5db1ecbb9015ba498207102a4ce12922e4/Toolbox/IO/SeekableFtpFileStream.cs).

Combined with a handy extension method on the [FtpWebRequest](https://msdn.microsoft.com/en-us/library/system.net.ftpwebrequest(v=vs.110).aspx) class, the result will look like this:

```c#
// Read number of entries in ZIP-file from FTP server using seekable FTP file stream
FtpWebRequest request = (FtpWebRequest)WebRequest.Create("ftp://localhost/LargeArchive.zip");
using (Stream stream = request.GetSeekableFileStream())
using (ZipArchive archive = new ZipArchive(stream))
{
    int fileCount = archive.Entries.Count;
    Console.WriteLine($"Number of entries: {fileCount}");
}
```

Unlike the earlier FTP example, this will execute in a matter of seconds depending on the server's response time.

The SeekableFtpFileStream class is a bit atypical, compared to the usual  response stream, by requiring the request and not the response. This is  because the implementation will need to control and clone the request  for seeking as WebRequests are not able to be reissued by default.

### Limitations

Currently, to avoid excessive re-connections, the implementation will always  sequentially read, and drop, data to access a requested position further ahead in the stream. If attempting random access in a stream where  entry offsets are scattered, the implementation may suffer from the same issue as the non-seekable solution present earlier and cause long  loading times, but the exceeding memory limits should have been  mitigated by not buffering large amounts of stream data in memory.

Another limitation may be that this implementation could trigger a sort of  connection flood protection on the FTP server. Excessive seeking may  result in heavy and frequent re-connecting to the server and depending  on the server's implementation and configuration, the client may be  temporarily banned or blocked from the server.

As the  implementation requires a FtpWebRequest, one cannot directly access the  response, including response headers, in the current version as the  response is not exposed as a property. Exposing the response may also be problematic as it is subject to change throughout the processing of  accessing a file.

## The targeted use case

The use case for  and reason why the SeekableFtpFileStream wrapper was implemented was to  consume large amounts of large XML-files ZIP archived on an FTP server.  The ZIP files are of 2.7 to 2.8 GB in size and steadily increasing over  time.

The XML-file contained in each ZIP-file is, at writing time, slowly reaching 70 GB of raw text. The file consists of data from the  danish vehicle registry and a new revision is published slightly before  noon (CET) every Sunday. You can access the data publicly [here](ftp://5.44.137.84/ESStatistikListeModtag/) via FTP.

In my case, the entries of the XML-file needed to be read, parsed,  processed, normalized, filtered and stored in a database. Due to the  inherent nature of humans manually inserting data into governmental  systems, the data is filled with different variations of the same items, missing fields and not all types of data is interesting for my purpose. This causes the process of importing items to the database to be  relatively CPU performing all kinds of checks and normalization, and RAM intensive due to caching, which results in a relative long running task even when applying multiple threads.

The processing of XML items  is not the majority of the time consumption though. Right after  publishing to the FTP, a large number of systems are starting to  download the newest file for all kinds of purposes, the data is actually rather interesting and can be used for a wide array of purposes. This  brings the FTP server to its knees concerning bandwidth and causes the  actual download process to extend to between 6 and 9 hours on the first  day.

By applying the method of streaming the ZIP-file directly  from the FTP server, my application can start processing XML data from  the first moment the download is initiated and the processing of XML  data will be done simultaneously with the download finishing potentially saving many hours of processing and allowing lower powered systems to  handle the task without loss of time as the limited bandwidth naturally  also limited the required CPU resources consumed.