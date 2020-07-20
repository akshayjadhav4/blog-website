---
title: Android Kotlin Coroutines  PART 1
date: "2020-07-20"
description: "Coroutines is used for asynchronous programming on Android."
---

Everywhere you hear about **kotlin**, you will also hear about **coroutines**.What they actually are?and why people like them so much?

To understand coroutines first we need to understand the **functions** and **threads** otherwise you won't be able to understand coroutines.

#### What is functions?

> **_A function is a group of statements that together perform a task._**

#### What is threads?

> **_A thread describes in which context sequence of instructions should be executed._**

Generally without multi-threading all instructions in a program will be executed one after another that is because they are all executed in the same single thread.

```
println("Hello World")
var x = 3
x += x
println("Result : $x")
```

In above given block of code no two instructions executed at some time they execute one by one.

```
Thread 1                             Thread 2

println("Hello World")                  println("Hello Kotlin")
var x = 3                               var y = 3
x += x                                  var z = 5
println("Result : $x")                  var result = y + z
                                        println("Result : $result")
```

But suppose if we start the new thread and both threads will run in parallel so each thread will execute its own instructions in word ,but we cannot predict that in which order all instructions in total executed. So it may happen when **Thread 1** finish line 3 **Thread 2** executing line 2.

## So why threading is important for Android apps ?
When an application is launched, the system creates a thread of execution for the application, called "main." This thread is very important because it is in charge of dispatching events to the appropriate user interface widgets, including drawing events. Also for other system calls system cannot create other thread. so in case your app performs intensive work specifically, if everything is happening in the UI thread, performing long operations such as network access or database queries will block the whole UI. When the thread is blocked, no events can be dispatched, including drawing events. From the user's perspective, the application appears to hang. Even worse, if the UI thread is blocked for more than a few seconds the user is presented with the  **application not responding** dialog.  

***so it's important to the responsiveness of your application that you do not block the UI thread. If you have operations to perform that take long time or extensive, you should make sure to do them in separate threads.***

## What distinguishes coroutines from threads?
1. Executed within thread.
2. Coroutines are suspendable.
3. They can switch their context.


>In the next part we will see coroutines in details.