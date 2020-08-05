---
title: Android Kotlin Coroutines  PART 2
date: "2020-08-05"
description: "How to start Coroutines"
---

In this section we try to implement **coroutine** in android.Create a andriod studio project and to start using **Coroutines** first we need to add dependencies. For that go to **app/build.gradle** file and add following two lines in **dependencies** section.

```
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.7'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7'
```

### Starting a Coroutine

Open a **MainActivity.kt** and we start coroutine as

```
//start Coroutine
GlobalScope.launch {

}

```

Inside that launch block we write our instructions which we want coroutine to execute.
Here we are launching a new coroutine in the GlobalScope, meaning that the lifetime of the coroutine is limited only by the lifetime of the whole application.And of course
the coroutine will end after its task is done. But if task not completed and app closed the coroutine will die.

This launch block will start a new thread and asynchronasly executes task to know the thread is different from Main thread we can log the thread name like this

```
GlobalScope.launch {
    Log.d("MainActivity", "Coroutine Thread ${Thread.currentThread().name}")
}
```

Similar to **Sleep()** function of thread which will just block thread for particuler time coroutine also have their own sleep function called **delay()** so that we can stop coroutine in middle of execution by calling **delay()** that take time in miliseconds. The **delay()** will only stop coroutine not whole thread.

**_NOTE : If the main thread finished, then all the other threads and coroutines will be cancelled even if they started in another thread and executed asynchronasly._**

### Suspend Functions

Suspend Function can be called only from coroutine or another suspend function. **delay()** function which we have seen previously is also a suspend function, we cannot use that outside coroutines.Suspend Function can execute a long running operation and wait for it to complete without blocking. The syntax of a suspending function is similar to that of a regular function except for the addition of the suspend keyword.
**e.g.**

```
GlobalScope.launch {
    val networkCallAnswer = doNetworkCall()
    Log.d("MainActivity", networkCallAnswer)
}

suspend fun doNetworkCall(): String {
    delay(3000L) //Simulating network call by delay()
    return "Network call response"
}

```

In the above code we perform dummy network call. Network call can take a time, so when we call doNetworkCall() function it waits and then Logs the response.

### Coroutine Contexts

Coroutines are always started in specific contexts and context will describe in which thread or coroutine will start. Till now we see only **GlobalScope.Launch** to start the new coroutine but that did not give that much control over it. We can also start the coroutine by passing dispatcher to launch function.

To specify where the coroutines should run, Kotlin provides **Dispatchers** you can use for thread dispatch.

1. **Dispatchers.Main** :
   It will start the coroutine in main thread that is useful if you need to do UI operations from within your coroutine because you can only change the UI from main thread.
2. **Dispatchers.IO** :
   It will start the coroutine optimized for disk and network IO off the main thread.
   e.g. Database, Reading/writing files, Networking

3. **Dispatchers.Default** :
   It will start the coroutine optimized for CPU intensive work off the main thread.

**One useful thing about coroutine contexts is that we can switch context within coroutine.**

**Example :**
In this example, we simulate network call and show result in the TextView. As we know we can do UI update in main thread only and we cannot do network call in main thread because if the network call take too long our main thread will block and app will crash.

```
GlobalScope.launch(Dispatchers.IO) {
    val result = doNetworkCall()
    Log.d("MainActivity", "Starting Coroutine ${Thread.currentThread().name}")
    withContext(Dispatchers.Main) {
        tvResult.text = result
        Log.d("MainActivity", "Setting result ${Thread.currentThread().name}")
    }
}

suspend fun doNetworkCall(): String {
    delay(3000L)
    return "Network Response"
}
```

In above example, we launch coroutine in **Dispatchers.IO** context and do network call and when we get results we switch context to **Dispatchers.Main** with the help of **withContext()** and set result to TextView. Log statement used for checking in which context instructions executing.

### Coroutines - runBlocking

When we start coroutine with **GlobalScope.launch** the **main** thread is not blocked, but sometime we need coroutine behavior and also want to block main thread that time we can use **runBlocking**.

In **runBlocking** coroutine **Main** thread is blocked Until runBlock finish all tasks.

```
Log.d("MainActivity","Before RunBlocking")

runBlocking {

    Log.d("MainActivity","Start RunBlocking")
    delay(5000L)
    Log.d("MainActivity","End RunBlocking")
}

Log.d("MainActivity","After RunBlocking")

```

**Output :**

```
Before RunBlocking

Start RunBlocking

End RunBlocking

After RunBlocking
```

From above output we can see that the last Log will not print Until runBlocking coroutine not complete execution.

**We can also launch other coroutine inside runBlocking scope.**

```
Log.d("MainActivity","Before RunBlocking")

runBlocking {
    // starting other coroutine
    launch (Dispatchers.IO){
        delay(3000L)
        Log.d("MainActivity","Finished IO Coroutine 1")
    }

    launch (Dispatchers.IO){
        delay(3000L)
        Log.d("MainActivity","Finished IO Coroutine 2")
    }

    Log.d("MainActivity","Start RunBlocking")
    delay(5000L)
    Log.d("MainActivity","End RunBlocking")
    }
Log.d("MainActivity","After RunBlocking")
```

As we can see while starting other coroutine we don't need to write GlobalScope as we already in coroutine scope. And also this new coroutine will run asynchronasly to the coroutine launched in main thread.

### Coroutine Jobs

When we use launch coroutine it return a **job** which we can save in variable like this

```
var job = GlobalScope.launch(Dispatchers.Default) {}
```

so what we can do with this job?

##### job.join()

We can wait for this job to complete by using **job.join()**. We cannot use this join() function in main thread as it is a suspend function so we use this in runBlocking for this example.

```
val job = GlobalScope.launch(Dispatchers.Default) {
    //code...
}

runBlocking {
    //join() this will wait till job completes
    job.join()
}
```

runBlocking block main thread till job completes.

##### job.cancel()

job.cancel() stop the job. If the running job has some delays while executing we can cancel it, only using cancel() but if the job is complex and coroutine does not have time to check job is cancelled it remains running. To avoid this we need to manually check job cancellation.

**Manual check :**

```
val job = GlobalScope.launch(Dispatchers.Default) {
    Log.d("MainActivity", "Starting Long Running Calculation...")

        //manually checking job is canceled or not
        if(isActive){
            //complex operation...
        }

    Log.d("MainActivity", "Ending Long Running Calculation...")
}

```

##### We can also use withTimeout() function to cancel job instead of manual check .

```
val job = GlobalScope.launch(Dispatchers.Default) {
    Log.d("MainActivity", "Starting Long Running Calculation...")
        //cancel job if take more than 3000 mili seconds
        withTimeout(3000L){
            //complex operation...
        }

    Log.d("MainActivity", "Ending Long Running Calculation...")
}

```

### Async and Await

Suppose we have multiple suspend functions in coroutine, it will execute one after another, but if we want to execute them simultaneously. So for demonstration we simulate two network calls.

```
GlobalScope.launch(Dispatchers.IO) {
    val time = measureTimeMillis {
        val response1 =  networkCallOne()
        val response2 = networkCallTwo()
        Log.d("MainActivity", "Result One is response1")
        Log.d("MainActivity", "Result Two is response2")
    }
    Log.d("MainActivity", "Total time needed $time")
}

suspend fun networkCallOne(): String {
    delay(3000L)
    return "Response One"
}

suspend fun networkCallTwo(): String {
    delay(3000L)
    return "Response Two"
}

```

In above example, we are making two network call and both executing one after another so it will take 6 seconds. To show this we used **measureTimeMillis** to calculate total execution time.

To solve this we can start the new coroutine for each function we call and set result, according to that but it makes the code complex. Instead of that we can use Async.
**Async** coroutine builder is similar to launch in its structure but returns a Deferred<T> instead of a Job. A Deferred<T> is a light-weight non blocking future that represents a promise to deliver a result later. You will need to call the suspending function await on the deferred object to get the eventual result.
We can implement it as follows:

```
GlobalScope.launch(Dispatchers.IO) {
    val time = measureTimeMillis {
        val response1 = async { networkCallOne() }
        val response2 = async { networkCallTwo() }
        Log.d("MainActivity", "Result One is ${response1.await()}")
        Log.d("MainActivity", "Result Two is ${response2.await()}")
    }
    Log.d("MainActivity", "Total time needed $time")
}

suspend fun networkCallOne(): String {
    delay(3000L)
    return "Response One"
}

suspend fun networkCallTwo(): String {
    delay(3000L)
    return "Response Two"
}

```

> In this part we explored coroutines in details.
