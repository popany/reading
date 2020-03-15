# Managed Threading

- [Managed Threading](#managed-threading)
  - [Managed threading basics](#managed-threading-basics)
    - [Threads and threading](#threads-and-threading)
      - [Processes and threads](#processes-and-threads)
      - [When to use multiple threads](#when-to-use-multiple-threads)
      - [How to use multithreading in .NET](#how-to-use-multithreading-in-net)
    - [Exceptions in Managed Threads](#exceptions-in-managed-threads)
      - [Exposing Threading Problems During Development](#exposing-threading-problems-during-development)
    - [Synchronizing data for multithreading](#synchronizing-data-for-multithreading)
    - [Foreground and Background Threads](#foreground-and-background-threads)
    - [Managed and unmanaged threading in Windows](#managed-and-unmanaged-threading-in-windows)
      - [Mapping from Win32 threading to managed threading](#mapping-from-win32-threading-to-managed-threading)
      - [Managed threads and COM apartments](#managed-threads-and-com-apartments)
      - [Blocking issues](#blocking-issues)
      - [Threads and fibers](#threads-and-fibers)
    - [Thread Local Storage: Thread-Relative Static Fields and Data Slots](#thread-local-storage-thread-relative-static-fields-and-data-slots)
      - [Uniqueness of Data in Managed TLS](#uniqueness-of-data-in-managed-tls)
  - [Using threads and threading](#using-threads-and-threading)
    - [Creating threads and passing data at start time](#creating-threads-and-passing-data-at-start-time)
      - [Creating a thread](#creating-a-thread)
      - [Passing data to threads](#passing-data-to-threads)
      - [Retrieving data from threads with callback methods](#retrieving-data-from-threads-with-callback-methods)
    - [Pausing and interrupting threads](#pausing-and-interrupting-threads)
      - [The Thread.Sleep method](#the-threadsleep-method)
      - [Interrupting threads](#interrupting-threads)
    - [Destroying threads](#destroying-threads)
      - [Handling ThreadAbortException](#handling-threadabortexception)
    - [Scheduling threads](#scheduling-threads)
    - [Cancellation in Managed Threads](#cancellation-in-managed-threads)
      - [Cancellation in Managed Threads](#cancellation-in-managed-threads-1)
        - [Cancellation Types](#cancellation-types)
        - [Code Example](#code-example)
        - [Operation Cancellation Versus Object Cancellation](#operation-cancellation-versus-object-cancellation)
        - [Listening and Responding to Cancellation Requests](#listening-and-responding-to-cancellation-requests)
          - [Listening by Polling](#listening-by-polling)
          - [Listening by Registering a Callback](#listening-by-registering-a-callback)
          - [Listening by Using a Wait Handle](#listening-by-using-a-wait-handle)
          - [Listening to Multiple Tokens Simultaneously](#listening-to-multiple-tokens-simultaneously)
        - [Cooperation Between Library Code and User Code](#cooperation-between-library-code-and-user-code)
      - [Canceling threads cooperatively](#canceling-threads-cooperatively)
      - [How to: Listen for Cancellation Requests by Polling](#how-to-listen-for-cancellation-requests-by-polling)
      - [How to: Register Callbacks for Cancellation Requests](#how-to-register-callbacks-for-cancellation-requests)
      - [How to: Listen for Cancellation Requests That Have Wait Handles](#how-to-listen-for-cancellation-requests-that-have-wait-handles)
      - [How to: Listen for Multiple Cancellation Requests](#how-to-listen-for-multiple-cancellation-requests)
  - [Managed threading best practices](#managed-threading-best-practices)
  - [Threading objects and features](#threading-objects-and-features)
    - [Threading objects and features](#threading-objects-and-features-1)

## Managed threading basics

### [Threads and threading](https://docs.microsoft.com/en-us/dotnet/standard/threading/threads-and-threading)

#### Processes and threads

**A process is an executing program**. An operating system uses processes to separate the applications that are being executed. **A thread is the basic unit to which an operating system allocates processor time**. Each thread has a scheduling priority and maintains a set of structures the system uses to save the **thread context** when the thread's execution is paused. The thread context includes all the information the thread needs to seamlessly resume execution, including the thread's set of CPU registers and stack. Multiple threads can run in the context of a process. All threads of a process **share its virtual address space**. A thread can execute any part of the program code, including parts currently being executed by another thread.

By default, a .NET program is started with a single thread, often called the **primary thread**. However, it can create additional threads to execute code in parallel or concurrently with the primary thread. These threads are often called **worker threads**.

#### When to use multiple threads

#### How to use multithreading in .NET

Starting with the .NET Framework 4, the recommended way to utilize multithreading is to use [Task Parallel Library (TPL)](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl) and [Parallel LINQ (PLINQ)](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/parallel-linq-plinq). For more information, see [Parallel programming](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/).

**Both TPL and PLINQ rely on the ThreadPool threads**. The System.Threading.ThreadPool class provides a .NET application with a pool of worker threads. You also can use thread pool threads.

At last, you can use the System.Threading.Thread class that represents a managed thread. For more information, see Using threads and threading.

Multiple threads might need to access a shared resource. To keep the resource in a uncorrupted state and avoid race conditions, you must synchronize the thread access to it. You also might want to coordinate the interaction of multiple threads. .NET provides a range of types that you can use to synchronize access to a shared resource or coordinate thread interaction. For more information, see [Overview of synchronization primitives](https://docs.microsoft.com/en-us/dotnet/standard/threading/overview-of-synchronization-primitives).

Do handle exceptions in threads. Unhandled exceptions in threads generally terminate the process. For more information, see [Exceptions in managed threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/exceptions-in-managed-threads).

### [Exceptions in Managed Threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/exceptions-in-managed-threads)

Starting with the .NET Framework version 2.0, the common language runtime allows most unhandled exceptions in threads to proceed naturally. In most cases this means that the **unhandled exception causes the application to terminate**.

The common language runtime provides a backstop for certain unhandled exceptions that are used for controlling program flow:

- A ThreadAbortException is thrown in a thread because Abort was called.

- An AppDomainUnloadedException is thrown in a thread because the application domain in which the thread is executing is being unloaded.

- The common language runtime or a host process terminates the thread by throwing an internal exception.

If any of these exceptions are unhandled in threads created by the common language runtime, the exception terminates the thread, but **the common language runtime does not allow the exception to proceed further**.

If these exceptions are unhandled in the main thread, or in threads that entered the runtime from unmanaged code, they proceed normally, resulting in termination of the application.

It is possible for the runtime to throw an unhandled exception before any managed code has had a chance to install an exception handler. Even though managed code had no chance to handle such an exception, the exception is allowed to proceed naturally.

#### Exposing Threading Problems During Development

When threads are allowed to fail silently, without terminating the application, serious programming problems can go undetected. This is a particular problem for services and other applications which run for extended periods. As threads fail, program state gradually becomes corrupted. Application performance may degrade, or the application might become unresponsive.

Allowing unhandled exceptions in threads to proceed naturally, until the operating system terminates the program, exposes such problems during development and testing. Error reports on program terminations support debugging.

### [Synchronizing data for multithreading](https://docs.microsoft.com/en-us/dotnet/standard/threading/synchronizing-data-for-multithreading)

When multiple threads can make calls to the properties and methods of a single object, it is critical that those calls be synchronized. Otherwise one thread might interrupt what another thread is doing, and the object could be left in an invalid state. **A class whose members are protected from such interruptions is called thread-safe**.

.NET provides several strategies to synchronize access to instance and static members:

- Synchronized code regions. You can use the [Monitor](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor) class or compiler support for this class to synchronize only the code block that needs it, improving performance.

- Manual synchronization. You can use the synchronization objects provided by the .NET class library. See [Overview of Synchronization Primitives](https://docs.microsoft.com/en-us/dotnet/standard/threading/overview-of-synchronization-primitives), which includes a discussion of the [Monitor](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor) class.

- Synchronized contexts. For .NET Framework and Xamarin applications, you can use the [SynchronizationAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.remoting.contexts.synchronizationattribute) to enable simple, automatic synchronization for [ContextBoundObject](https://docs.microsoft.com/en-us/dotnet/api/system.contextboundobject) objects.

- Collection classes in the [System.Collections.Concurrent](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent) namespace. These classes provide built-in synchronized add and remove operations. For more information, see [Thread-Safe Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/).

### [Foreground and Background Threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/foreground-and-background-threads)

A managed thread is either a background thread or a foreground thread. Background threads are identical to foreground threads with one exception: **a background thread does not keep the managed execution environment running**. Once all foreground threads have been stopped in a managed process (where the .exe file is a managed assembly), the system stops all background threads and shuts down.

**When the runtime stops a background thread because the process is shutting down, no exception is thrown in the thread**. However, when threads are stopped because the [AppDomain.Unload](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain.unload) method unloads the application domain, a [ThreadAbortException](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadabortexception) is thrown in both foreground and background threads.

Threads that belong to the managed thread pool (that is, threads whose [IsThreadPoolThread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.isthreadpoolthread) property is true) are **background threads**. All threads that enter the managed execution environment from unmanaged code are marked as background threads. All threads generated by creating and starting a new [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread) object are by **default foreground threads**.

If you use a thread to monitor an activity, such as a socket connection, set its [IsBackground](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.isbackground) property to true so that the thread does not prevent your process from terminating.

### [Managed and unmanaged threading in Windows](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-and-unmanaged-threading-in-windows)

Management of all threads is done through the [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread) class, including threads **created by the common language runtime** and those **created outside the runtime that enter the managed environment** to execute code. The runtime monitors all the threads in its process that have ever executed code within the managed execution environment. It does not track any other threads. Threads can enter the managed execution environment through COM interop (because the runtime exposes managed objects as COM objects to the unmanaged world), the COM [DllGetClassObject](https://docs.microsoft.com/en-us/windows/desktop/api/combaseapi/nf-combaseapi-dllgetclassobject) function, and platform invoke.

When an unmanaged thread enters the runtime through, for example, a COM callable wrapper, the system checks the thread-local store of that thread to look for an **internal managed Thread object**. If one is found, the runtime is already aware of this thread. If it cannot find one, however, the runtime builds a new Thread object and installs it in the thread-local store of that thread.

In managed threading, [Thread.GetHashCode](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.gethashcode) is the stable managed thread identification. For the lifetime of your thread, it will not collide with the value from any other thread, regardless of the application domain from which you obtain this value.

An operating-system **ThreadId has no fixed relationship to a managed thread**, because an unmanaged host can control the relationship between managed and unmanaged threads. Specifically, a sophisticated host can use the **Fiber API** to **schedule many managed threads against the same operating system thread**, or to **move a managed thread among different operating system threads**.

#### Mapping from Win32 threading to managed threading

#### Managed threads and COM apartments

A managed thread can be marked to indicate that it will host a [single-threaded](https://docs.microsoft.com/en-us/windows/desktop/com/single-threaded-apartments) or [multithreaded](https://docs.microsoft.com/en-us/windows/desktop/com/multithreaded-apartments) apartment. (For more information on the COM threading architecture, see [Processes, Threads, and Apartments.](https://docs.microsoft.com/en-us/windows/desktop/com/processes--threads--and-apartments)) The [GetApartmentState](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.getapartmentstate), [SetApartmentState](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.setapartmentstate), and [TrySetApartmentState](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.trysetapartmentstate) methods of the [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread) class return and assign the apartment state of a thread. If the state has not been set, [GetApartmentState](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.getapartmentstate) returns [ApartmentState.Unknown](https://docs.microsoft.com/en-us/dotnet/api/system.threading.apartmentstate#System_Threading_ApartmentState_Unknown).

The property can be set only when the thread is in the [ThreadState.Unstarted](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadstate#System_Threading_ThreadState_Unstarted) state; it can be set only once for a thread.

If the apartment state is not set before the thread is started, the thread is initialized as a multithreaded apartment (MTA). The finalizer thread and all threads controlled by [ThreadPool](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadpool) are MTA.

#### Blocking issues

If a thread makes an unmanaged call into the operating system that has blocked the thread in unmanaged code, the runtime will not take control of it for [Thread.Interrupt](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.interrupt) or [Thread.Abort](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.abort). In the case of Thread.Abort, the runtime marks the thread for Abort and takes control of it when it re-enters managed code. It is preferable for you to use managed blocking rather than unmanaged blocking. [WaitHandle.WaitOne](https://docs.microsoft.com/en-us/dotnet/api/system.threading.waithandle.waitone),[WaitHandle.WaitAny](https://docs.microsoft.com/en-us/dotnet/api/system.threading.waithandle.waitany), [WaitHandle.WaitAll](https://docs.microsoft.com/en-us/dotnet/api/system.threading.waithandle.waitall), [Monitor.Enter](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.enter), [Monitor.TryEnter](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor.tryenter), [Thread.Join](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.join), [GC.WaitForPendingFinalizers](https://docs.microsoft.com/en-us/dotnet/api/system.gc.waitforpendingfinalizers), and so on are all responsive to [Thread.Interrupt](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.interrupt) and to [Thread.Abort](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.abort). Also, if your thread is in a single-threaded apartment, all these managed blocking operations will correctly pump messages in your apartment while your thread is blocked.

#### Threads and fibers

The .NET threading model does not support [fibers](https://docs.microsoft.com/en-us/windows/win32/procthread/fibers
). You should not call into any unmanaged function that is implemented by using fibers. Such calls may result in a crash of the .NET runtime.

### [Thread Local Storage: Thread-Relative Static Fields and Data Slots](https://docs.microsoft.com/en-us/dotnet/standard/threading/thread-local-storage-thread-relative-static-fields-and-data-slots)

You can use **managed thread local storage (TLS)** to store data that is unique to a thread and application domain. The .NET Framework provides two ways to use managed TLS: thread-relative static fields and data slots.

- Use thread-relative static fields (thread-relative Shared fields in Visual Basic) if you can anticipate your exact needs at compile time. Thread-relative static fields provide the best performance. They also give you the benefits of compile-time type checking.

- Use data slots when your actual requirements might be discovered only at run time. Data slots are slower and more awkward to use than thread-relative static fields, and data is stored as type [Object](https://docs.microsoft.com/en-us/dotnet/api/system.object), so you must cast it to the correct type before you use it.

#### Uniqueness of Data in Managed TLS

Whether you use thread-relative static fields or data slots, data in managed TLS is **unique to the combination of thread and application domain**.

- Within an application domain, one thread cannot modify data from another thread, even when both threads use the same field or slot.

- When a thread accesses the same field or slot from multiple application domains, a separate value is maintained in each application domain.

## [Using threads and threading](https://docs.microsoft.com/en-us/dotnet/standard/threading/using-threads-and-threading)

### [Creating threads and passing data at start time](https://docs.microsoft.com/en-us/dotnet/standard/threading/creating-threads-and-passing-data-at-start-time)

When an operating-system process is created, the operating system injects a thread to execute code in that process, including any original application domain. From that point on, application domains can be created and destroyed without any operating system threads necessarily being created or destroyed. If the code being executed is managed code, then a [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread) object for the thread executing in the current application domain can be obtained by retrieving the static [CurrentThread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.currentthread) property of type [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread). This topic describes thread creation and discusses alternatives for passing data to the thread procedure.

#### Creating a thread

Creating a new [Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread) object creates a new managed thread. The Thread class has constructors that take a [ThreadStart](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadstart) delegate or a ParameterizedThreadStart delegate; the delegate wraps the method that is invoked by the new thread when you call the [Start](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.start) method. Calling Start more than once causes a [ThreadStateException](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadstateexception) to be thrown.

The Start method returns immediately, often before the new thread has actually started. You can use the [ThreadState](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.threadstate) and [IsAlive](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.isalive) properties to determine the state of the thread at any one moment, **but these properties should never be used for synchronizing the activities of threads**.

Once a thread is started, it is **not necessary to retain a reference to the Thread object**. The thread continues to execute until the thread procedure ends.

#### Passing data to threads

In the .NET Framework version 2.0, the [ParameterizedThreadStart](https://docs.microsoft.com/en-us/dotnet/api/system.threading.parameterizedthreadstart) delegate provides an easy way to pass an object containing data to a thread when you call the [Thread.Start](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.start) method overload. See ParameterizedThreadStart for a code example.

Using the ParameterizedThreadStart delegate is not a type-safe way to pass data, because the Thread.Start method overload accepts any object. An alternative is to **encapsulate the thread procedure and the data in a helper class** and use the ThreadStart delegate to execute the thread procedure.

Neither ThreadStart nor ParameterizedThreadStart delegate has a return value, because there is no place to return the data from an asynchronous call.

#### Retrieving data from threads with callback methods

### [Pausing and interrupting threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/pausing-and-resuming-threads)

The most common ways to synchronize the activities of threads are to block and release threads, or to lock objects or regions of code. For more information on these locking and blocking mechanisms, see [Overview of Synchronization Primitives](https://docs.microsoft.com/en-us/dotnet/standard/threading/overview-of-synchronization-primitives).

You can also have threads put themselves to sleep. When threads are blocked or sleeping, you can use a [ThreadInterruptedException](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadinterruptedexception) to break them out of their wait states.

#### The Thread.Sleep method

Calling the Thread.Sleep method causes the current thread to immediately block for the number of milliseconds or the time interval you pass to the method, and **yields the remainder of its time slice** to another thread. Once that interval elapses, the sleeping thread resumes execution.

One thread cannot call Thread.Sleep on another thread. Thread.Sleep is a static method that always causes the current thread to sleep.

Calling Thread.Sleep with a value of Timeout.Infinite causes a thread to sleep until it is interrupted by another thread that calls the [Thread.Interrupt](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.interrupt) method on the sleeping thread, or until it is terminated by a call to its [Thread.Abort](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.abort) method.

#### Interrupting threads

You can interrupt a waiting thread by calling the Thread.Interrupt method on the blocked thread to throw a ThreadInterruptedException, which breaks the thread out of the blocking call. The **thread should catch the ThreadInterruptedException** and do whatever is appropriate to continue working. **If the thread ignores the exception, the runtime catches the exception and stops the thread**.

If the target thread is not blocked when Thread.Interrupt is called, the **thread is not interrupted until it blocks**. If the thread never blocks, it could complete without ever being interrupted.

If a wait is a managed wait, then Thread.Interrupt and Thread.Abort both wake the thread immediately. If a wait is an unmanaged wait (for example, a platform invoke call to the Win32 WaitForSingleObject function), neither Thread.Interrupt nor Thread.Abort can take control of the thread **until it returns to or calls into managed code**. In managed code, the behavior is as follows:

- Thread.Interrupt wakes a thread out of any wait it might be in and causes a ThreadInterruptedException to be thrown in the destination thread.

- Thread.Abort wakes a thread out of any wait it might be in and causes a ThreadAbortException to be thrown on the thread. For details, see [Destroying Threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/destroying-threads).

### [Destroying threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/destroying-threads)

To terminate the execution of the thread, you usually use the [cooperative cancellation model](https://docs.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads). **Sometimes it is not possible to stop a thread cooperatively**, because it runs third-party code not designed for cooperative cancellation. The Thread.Abort method in .NET Framework can be used to terminate a managed thread forcibly. When you call Abort, the Common Language Runtime throws a ThreadAbortException in the target thread, which the target thread can catch. For more information, see Thread.Abort. **The Thread.Abort method is not supported in .NET Core**. If you need to terminate the execution of third-party code forcibly in .NET Core, run it in the separate process and use Process.Kill.

If a thread is executing unmanaged code when its Abort method is called, the runtime **marks it ThreadState.AbortRequested**. **The exception is thrown when the thread returns to managed code**.

**Once a thread is aborted, it cannot be restarted**.

**The Abort method does not cause the thread to abort immediately**, because the target thread can catch the ThreadAbortException and execute **arbitrary amounts of code in a finally block**. You can **call Thread.Join if you need to wait** until the thread has ended. Thread.Join is a blocking call that does not return until the thread has actually stopped executing or an optional timeout interval has elapsed. The aborted thread could call the ResetAbort method or perform unbounded processing in a finally block, so if you do not specify a timeout, the wait is **not guaranteed to end**.

Threads that are waiting on a call to the **Thread.Join method can be interrupted by other threads that call Thread.Interrupt**.

#### Handling ThreadAbortException

If you expect your thread to be aborted, either as a result of **calling Abort from your own code** or as a result of **unloading an application domain** in which the thread is running (AppDomain.Unload uses Thread.Abort to terminate threads), your thread must handle the ThreadAbortException and perform any final processing in a finally clause, as shown in the following code.

    try
    {  
        // Code that is executing when the thread is aborted.  
    }
    catch (ThreadAbortException ex)
    {  
        // Clean-up code can go here.  
        // If there is no Finally clause, ThreadAbortException is  
        // re-thrown by the system at the end of the Catch clause.
    }  
    // Do not put clean-up code here, because the exception
    // is rethrown at the end of the Finally clause.

Your clean-up code must be in the catch clause or the finally clause, because a **ThreadAbortException is rethrown by the system at the end of the finally clause, or at the end of the catch clause if there is no finally clause**.

You can **prevent the system from rethrowing the exception by calling the Thread.ResetAbort method**. However, **you should do this only if your own code caused the ThreadAbortException**.

### [Scheduling threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/scheduling-threads)

Every thread has a thread priority assigned to it. Threads created within the common language runtime are initially assigned the priority of ThreadPriority.Normal. Threads created outside the runtime **retain the priority** they had before they entered the managed environment. You can get or set the priority of any thread with the Thread.Priority property.

Threads are scheduled for execution based on their **priority**. Even though threads are executing within the runtime, all threads are assigned **processor time slices** by the operating system. The details of the **scheduling algorithm** used to **determine the order** in which threads are executed varies with each operating system. Under some operating systems, the thread with the **highest priority** (of those threads that can be executed) is **always scheduled to run first**. If multiple threads with the same priority are all available, the scheduler **cycles** through the threads at that priority, giving each thread a **fixed time slice** in which to execute. **As long as a thread with a higher priority is available to run, lower priority threads do not get to execute**. When there are no more runnable threads at a given priority, the scheduler moves to the next lower priority and schedules the threads at that priority for execution. If a higher priority thread becomes runnable, the lower priority thread is **preempted** and the higher priority thread is allowed to execute once again. On top of all that, the operating system can also **adjust thread priorities dynamically** as an application's user interface is moved between foreground and background. Other operating systems might choose to use a different scheduling algorithm.

### [Cancellation in Managed Threads](https://docs.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)

#### Cancellation in Managed Threads

Starting with the .NET Framework 4, the .NET Framework uses a unified model for **cooperative cancellation of asynchronous** or **long-running synchronous operations**. This model is based on a lightweight object called a **cancellation token**. The object that invokes one or more cancelable operations, for example by creating new threads or tasks, passes the token to each operation. Individual operations can in turn pass copies of the token to other operations. At some later time, the object that created the token can use it to request that the operations stop what they are doing. Only the requesting object can issue the cancellation request, and each listener is responsible for noticing the request and responding to it in an appropriate and timely manner.

The general pattern for implementing the cooperative cancellation model is:

- Instantiate a **[CancellationTokenSource](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource) object**, which **manages and sends cancellation notification** to the individual **cancellation tokens**.

- Pass the token returned by the [CancellationTokenSource.Token](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.token) property to each task or thread that listens for cancellation.

- Provide a mechanism for each task or thread to respond to cancellation.

- Call the [CancellationTokenSource.Cancel](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.cancel) method to provide notification of cancellation.

The CancellationTokenSource class implements the IDisposable interface. You should **be sure to call the [CancellationTokenSource.Dispose](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.dispose) method** when you have finished using the cancellation token source to free any unmanaged resources it holds.

The new cancellation model makes it easier to create cancellation-aware applications and libraries, and it supports the following features:

- Cancellation is **cooperative** and is **not forced on the listener**. The listener determines how to gracefully terminate in response to a cancellation request.

- Requesting is distinct from listening. An object that invokes a cancelable operation can control when (if ever) cancellation is requested.

- The requesting object issues the cancellation request to all copies of the token by using just one method call.

- A listener can listen to multiple tokens simultaneously by joining them into one linked token.

- User code can notice and respond to cancellation requests from library code, and library code can notice and respond to cancellation requests from user code.

- Listeners can be notified of cancellation requests by polling, callback registration, or waiting on wait handles.

##### Cancellation Types

The cancellation framework is implemented as a set of related types, which are listed in the following table.

|Type name | Description|
----|----
[CancellationTokenSource](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource) | Object that creates a cancellation token, and also issues the cancellation request for all copies of that token.
[CancellationToken](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken) | Lightweight value type passed to one or more listeners, typically as a method parameter. Listeners monitor the value of the IsCancellationRequested property of the token by polling, callback, or wait handle.
[OperationCanceledException](https://docs.microsoft.com/en-us/dotnet/api/system.operationcanceledexception) | Overloads of this exception's constructor accept a CancellationToken as a parameter. Listeners can optionally throw this exception to verify the source of the cancellation and notify others that it has responded to a cancellation request.

The new cancellation model is integrated into the .NET Framework in several types. The most important ones are [System.Threading.Tasks.Parallel](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel), [System.Threading.Tasks.Task](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task), [System.Threading.Tasks.Task<TResult>](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1) and [System.Linq.ParallelEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable). We recommend that you use this new cancellation model for all new library and application code.

##### Code Example

In the following example, the requesting object creates a [CancellationTokenSource](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource) object, and then passes its [Token](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.token) property to the **cancelable operation**. The operation that receives the request monitors the value of the [IsCancellationRequested](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.iscancellationrequested) property of the token by polling. When the value becomes true, the listener can terminate in whatever manner is appropriate. In this example, the method just exits, which is all that is required in many cases.

The example uses the [QueueUserWorkItem](https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadpool.queueuserworkitem) method to demonstrate that the new cancellation framework is compatible with legacy APIs. For an example that uses the new, preferred [System.Threading.Tasks.Task](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task) type, see [How to: Cancel a Task and Its Children](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/how-to-cancel-a-task-and-its-children).

    using System;
    using System.Threading;

    public class Example
    {
    public static void Main()
    {
        // Create the token source.
        CancellationTokenSource cts = new CancellationTokenSource();

        // Pass the token to the cancelable operation.
        ThreadPool.QueueUserWorkItem(new WaitCallback(DoSomeWork), cts.Token);
        Thread.Sleep(2500);

        // Request cancellation.
        cts.Cancel();
        Console.WriteLine("Cancellation set in token source...");
        Thread.Sleep(2500);
        // Cancellation should have happened, so call Dispose.
        cts.Dispose();
    }

    // Thread 2: The listener
    static void DoSomeWork(object obj)
    {
        CancellationToken token = (CancellationToken)obj;

        for (int i = 0; i < 100000; i++) {
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("In iteration {0}, cancellation has been requested...",
                                i + 1);
                // Perform cleanup if necessary.
                //...
                // Terminate the operation.
                break;
            }
            // Simulate some work.
            Thread.SpinWait(500000);
        }
    }
    }
    // The example displays output like the following:
    //       Cancellation set in token source...
    //       In iteration 1430, cancellation has been requested...

##### Operation Cancellation Versus Object Cancellation

In the new cancellation framework, cancellation refers to **operations**, not **objects**. The cancellation request means that the operation should stop as soon as possible after any required cleanup is performed. **One cancellation token should refer to one "cancelable operation"**, however that operation may be implemented in your program. After the [IsCancellationRequested](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.iscancellationrequested) property of the token has been set to true, it cannot be reset to false. Therefore, **cancellation tokens cannot be reused** after they have been canceled.

If you require an object cancellation mechanism, you can base it on the operation cancellation mechanism by calling the [CancellationToken.Register](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.register) method, as shown in the following example.

    using System;
    using System.Threading;

    class CancelableObject
    {
        public string id;
        
        public CancelableObject(string id)
        {
            this.id = id;
        }
        
        public void Cancel() 
        { 
            Console.WriteLine("Object {0} Cancel callback", id);
            // Perform object cancellation here.
        }
    }

    public class Example
    {
        public static void Main()
        {
            CancellationTokenSource cts = new CancellationTokenSource();
            CancellationToken token = cts.Token;

            // User defined Class with its own method for cancellation
            var obj1 = new CancelableObject("1");
            var obj2 = new CancelableObject("2");
            var obj3 = new CancelableObject("3");

            // Register the object's cancel method with the token's
            // cancellation request.
            token.Register(() => obj1.Cancel());
            token.Register(() => obj2.Cancel());
            token.Register(() => obj3.Cancel());

            // Request cancellation on the token.
            cts.Cancel();
            // Call Dispose when we're done with the CancellationTokenSource.
            cts.Dispose();
        }
    }
    // The example displays the following output:
    //       Object 3 Cancel callback
    //       Object 2 Cancel callback
    //       Object 1 Cancel callback

If an object supports more than one concurrent cancelable operation, pass a separate token as input to each distinct cancelable operation. That way, one operation can be cancelled without affecting the others.

##### Listening and Responding to Cancellation Requests

In the user delegate, the implementer of a cancelable operation determines how to terminate the operation in response to a cancellation request. In many cases, the user delegate can just perform any required cleanup and then return immediately.

However, in more complex cases, it might be necessary for the user delegate to notify library code that cancellation has occurred. In such cases, **the correct way to terminate the operation** is for the delegate to call the [ThrowIfCancellationRequested](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.throwifcancellationrequested), method, which will cause an [OperationCanceledException](https://docs.microsoft.com/en-us/dotnet/api/system.operationcanceledexception) to be thrown. Library code can catch this exception on the user delegate thread and examine the exception's token to determine whether the exception indicates **cooperative cancellation** or some **other exceptional situation**.

The Task class handles OperationCanceledException in this way. For more information, see [Task Cancellation](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-cancellation).

###### Listening by Polling

For long-running computations that loop or recurse, you can listen for a cancellation request by periodically polling the value of the CancellationToken.IsCancellationRequested property. If its value is true, the method should clean up and terminate as quickly as possible. The **optimal frequency of polling depends on the type of application**. It is up to the developer to determine the best polling frequency for any given program. **Polling itself does not significantly impact performance**.

###### Listening by Registering a Callback

Some operations can become blocked in such a way that they **cannot check the value of the cancellation token in a timely manner**. For these cases, you can register a callback method that unblocks the metho when a cancellation request is received.

The [Register](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.register) method returns a [CancellationTokenRegistration](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokenregistration) object that is used specifically for this purpose. The following example shows how to use the Register method to cancel an asynchronous Web request.

    using System;
    using System.Net;
    using System.Threading;

    class Example
    {
        static void Main()
        {
            CancellationTokenSource cts = new CancellationTokenSource();

            StartWebRequest(cts.Token);

            // cancellation will cause the web 
            // request to be cancelled
            cts.Cancel();
        }

        static void StartWebRequest(CancellationToken token)
        {
            WebClient wc = new WebClient();
            wc.DownloadStringCompleted += (s, e) => Console.WriteLine("Request completed.");

            // Cancellation on the token will 
            // call CancelAsync on the WebClient.
            token.Register(() =>
            {
                wc.CancelAsync();
                Console.WriteLine("Request cancelled!");
            });

            Console.WriteLine("Starting request.");
            wc.DownloadStringAsync(new Uri("http://www.contoso.com"));
        }
    }

The CancellationTokenRegistration object manages thread synchronization and ensures that the callback will stop executing at a precise point in time.

In order to ensure system responsiveness and to avoid deadlocks, the following guidelines must be followed when registering callbacks:

- The callback method should be fast because it is called synchronously and therefore the call to [Cancel](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource.cancel) does not return until the callback returns.

- If you call [Dispose](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokenregistration.dispose) while the callback is running, and you hold a lock that the callback is waiting on, your program can deadlock. After Dispose returns, you can free any resources required by the callback.

- Callbacks should not perform any manual thread or [SynchronizationContext](https://docs.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext) usage in a callback. If a callback must run on a particular thread, use the [System.Threading.CancellationTokenRegistration](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokenregistration) constructor that enables you to specify that the target syncContext is the active [SynchronizationContext.Current](https://docs.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext.current). Performing manual threading in a callback can cause deadlock.

###### Listening by Using a Wait Handle

When a cancelable operation can block while it waits on a synchronization primitive such as a [System.Threading.ManualResetEvent](https://docs.microsoft.com/en-us/dotnet/api/system.threading.manualresetevent) or [System.Threading.Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore), you can use the [CancellationToken.WaitHandle](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken.waithandle) property to **enable the operation to wait on both the event and the cancellation request**. The wait handle of the cancellation token will become signaled in response to a cancellation request, and the method can use the return value of the WaitAny method to determine whether it was the cancellation token that signaled. The operation can then just exit, or throw a OperationCanceledException, as appropriate.

    // Wait on the event if it is not signaled.
    int eventThatSignaledIndex =
        WaitHandle.WaitAny(new WaitHandle[] { mre, token.WaitHandle }, new TimeSpan(0, 0, 20));

In new code that targets the .NET Framework 4, [System.Threading.ManualResetEventSlim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.manualreseteventslim) and [System.Threading.SemaphoreSlim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim) both support the new cancellation framework in their Wait methods. You can pass the CancellationToken to the method, and when the cancellation is requested, the **event wakes up and throws an OperationCanceledException**.

    try
    {
        // mres is a ManualResetEventSlim
        mres.Wait(token);
    }
    catch (OperationCanceledException)
    {
        // Throw immediately to be responsive. The
        // alternative is to do one more item of work,
        // and throw on next iteration, because
        // IsCancellationRequested will be true.
        Console.WriteLine("The wait operation was canceled.");
        throw;
    }

    Console.Write("Working...");
    // Simulating work.
    Thread.SpinWait(500000);

###### Listening to Multiple Tokens Simultaneously

In some cases, a listener may have to listen to multiple cancellation tokens simultaneously. For example, a cancelable operation may have to monitor an internal cancellation token in addition to a token passed in externally as an argument to a method parameter. To accomplish this, create a **linked token source** that can join two or more tokens into one token, as shown in the following example.

    public void DoWork(CancellationToken externalToken)
    {
        // Create a new token that combines the internal and external tokens.
        this.internalToken = internalTokenSource.Token;
        this.externalToken = externalToken;

        using (CancellationTokenSource linkedCts =
                CancellationTokenSource.CreateLinkedTokenSource(internalToken, externalToken))
        {
            try {
                DoWorkInternal(linkedCts.Token);
            }
            catch (OperationCanceledException) {
                if (internalToken.IsCancellationRequested) {
                    Console.WriteLine("Operation timed out.");
                }
                else if (externalToken.IsCancellationRequested) {
                    Console.WriteLine("Cancelling per user request.");
                    externalToken.ThrowIfCancellationRequested();
                }
            }
        }
    }

Notice that you must call **Dispose** on the linked token source when you are done with it.

##### Cooperation Between Library Code and User Code

The unified cancellation framework makes it possible for library code to cancel user code, and for user code to cancel library code in a cooperative manner. Smooth cooperation depends on each side following these guidelines:

- If library code provides cancelable operations, it should also provide public methods that accept an external cancellation token so that user code can request cancellation.

- If library code calls into user code, the library code should interpret an OperationCanceledException(externalToken) as cooperative cancellation, and not necessarily as a failure exception.

- User-delegates should attempt to respond to cancellation requests from library code in a timely manner.

[System.Threading.Tasks.Task](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task) and [System.Linq.ParallelEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable) are examples of classes that follow these guidelines. For more information, see [Task Cancellation](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-cancellation) and [How to: Cancel a PLINQ Query](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/how-to-cancel-a-plinq-query).

#### [Canceling threads cooperatively](https://docs.microsoft.com/en-us/dotnet/standard/threading/canceling-threads-cooperatively)

#### [How to: Listen for Cancellation Requests by Polling](https://docs.microsoft.com/en-us/dotnet/standard/threading/how-to-listen-for-cancellation-requests-by-polling)

#### [How to: Register Callbacks for Cancellation Requests](https://docs.microsoft.com/en-us/dotnet/standard/threading/how-to-register-callbacks-for-cancellation-requests)

#### [How to: Listen for Cancellation Requests That Have Wait Handles](https://docs.microsoft.com/en-us/dotnet/standard/threading/how-to-listen-for-cancellation-requests-that-have-wait-handles)

#### [How to: Listen for Multiple Cancellation Requests](https://docs.microsoft.com/en-us/dotnet/standard/threading/how-to-listen-for-multiple-cancellation-requests)

## [Managed threading best practices](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)

## Threading objects and features

### [Threading objects and features](https://docs.microsoft.com/en-us/dotnet/standard/threading/threading-objects-and-features)


























