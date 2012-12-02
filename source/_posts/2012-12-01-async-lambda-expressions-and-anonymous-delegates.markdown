---
layout: post
title: "Async lambda expressions and anonymous delegates"
date: 2012-12-01 22:34
comments: true
categories:
 - .net
 - C#
 - Delegates
 - Microsoft
 - New technology
 - Threading
 - Visual studio
---

I've known about the new `async` and `await` keywords in C# 5 for a while now,
but I saw a use of it I hadn't anticipated. I saw Jon Skeet create an
asyncronous lambda expression and was caught a bit off guard. After thinking
about it, it really isn't that amazing, but I thought I would spread the word
anyway because I think it could be a really handy feature.

Anonymous functions are declared without a method name and can be passed around
and invoked at will. Lambda expression syntax creates anonymous functions that
can be used as delegates or expression trees. In C# 5, you can adorn functions
with the `async` keyword which signifies that the function will return a `Task`
object from the Task Parallel Library as a return value. You will also find
somewhere in the method body the keyword `await` next to a statement that could
take a while to process and be a blocking operation. Now your method can resume
operation after the asyncronous operation has completed and has its result.

Here is a simple example:

<pre class="brush: csharp">
    public void ButtonClick()
    {
        ProcessFiles();
    }

    private async void ProcessFiles()
    {
        var files = new string[] { "file1.txt", "file2.txt", "file3.txt" };

        foreach (var file in files)
        {
            var fileContents = await ReadTextAsync(file);
            // likely do something with file contents
        }
    }

    private async Task&lt;string&gt; ReadTextAsync(string filePath)
    {
        // Some text reading implementation that supports the TPL
    }
</pre>

The button click handler method would return at the first sign of a blocking
operation and callback delegates are set up under the covers by the compiler to
finish executing the rest of the code after the blocking process finishes.

Here is the same example but using an async lambda expression:

<pre class="brush: csharp">
    public void ButtonClick()
    {
        var files = new string[] { "file1.txt", "file2.txt", "file3.txt" };

        files.ForEach(async file => await ReadTextAsync(file));
    }

    private async Task&lt;string&gt; ReadTextAsync(string filePath)
    {
        // Some text reading implementation that supports the TPL
    }
</pre>

Please overlook my disregard of single responsibility and encapsulation, and so
on so forth. This is an overly simplified example intended solely to show what
the syntax looks like. Either way, I could see this lighter weight syntax coming
in handy in the future.

In the same example where I saw async lambda syntax used, I also saw Jon Skeet
show off a really clever extension method that took an enumerable collection of
`Task` objects and reordered them so they could be enumerated and handled in the
order that they complete their asynchronous operations. Perhaps I will show that
piece of code next time.
