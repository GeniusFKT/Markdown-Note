# Foundation - Processes and Threads

Manage your app's interaction with the host operating system and other processes, and implement low-level concurrency features.

## Runloop Scheduling

### Runloop

```swift
class RunLoop : NSObject
```

Runloop对象可以处理来自视窗的鼠标、键盘事件，同时其可以处理Timer事件。

在应用中不需要创建或直接声明一个Runloop对象。对于每一个线程，包括应用的主线程在需要时都会自动创建一个RunLoop对象。如果需要访问当前线程的RunLoop只需要调用类方法current。

从RunLoop的角度上说，Timer对象并非一种输入，当Timer启动时并不会使RunLoop返回。

> Warning
>
> The `RunLoop` class is generally not considered to be thread-safe and its methods should only be called within the context of the current thread. You should never try to call the methods of an `RunLoop` object running in a different thread, as doing so might cause unexpected results.

### Timer

```swift
class Timer : NSObject
```

Timer与RunLoop结合共同应用。

To use a timer effectively, you should be aware of how run loops operate. See [Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i) for more information.

定时器并非一个实时的机制。通常来说，定时器的启动时间会显著延后。See also [Timer Tolerance](apple-reference-documentation://hsaODPrPmP#1667624).

#### Comparing Repeating and Nonrepeating Timers

我们在声明时就应当确定定时器是重复的还是非重复的。非重复定时器只会被触发一次，之后便会自动失效；重复定时器被触发后会将其自身重新调度至相同的RunLoop中。重复计时器总会基于调度启动时间而非真实启动时间去进行调度（真实调度时间可能会被延迟）

#### Timer Tolerance

定时器的启动时间总会在调度时间至调度时间加上容忍度之间。我们可以设置tolerance属性对该项进行调整。

#### Scheduling Timers in Run Loops

一个定时器只可以在一个RunLoop中，下面是三种创建Timer的方式：

- Use the [`scheduledTimer(timeInterval:invocation:repeats:)`](apple-reference-documentation://hsB6vmtKkW) or [`scheduledTimer(timeInterval:target:selector:userInfo:repeats:)`](apple-reference-documentation://hs022AtRDR) class method to create the timer and schedule it on the current run loop in the default mode.
- Use the [`init(timeInterval:invocation:repeats:)`](apple-reference-documentation://hsO41AMaTf) or [`init(timeInterval:target:selector:userInfo:repeats:)`](apple-reference-documentation://hs_98i3I9i) class method to create the timer object without scheduling it on a run loop. (After creating it, you must add the timer to a run loop manually by calling the [`add(_:forMode:)`](apple-reference-documentation://hsocJkO-uk)method of the corresponding `RunLoop` object.)
- Allocate the timer and initialize it using the [`init(fireAt:interval:target:selector:userInfo:repeats:)`](apple-reference-documentation://hsU-g26SI2) method. (After creating it, you must add the timer to a run loop manually by calling the [`add(_:forMode:)`](apple-reference-documentation://hsocJkO-uk) method of the corresponding `RunLoop` object.)

对于重复定时器我们应当调用invalidate()方法手动失效（在相同的线程内），当调用invalidate之后这个计时器就无法再次使用。

#### Subclassing Notes

Do not subclass [`Timer`](apple-reference-documentation://hsaODPrPmP).

## Operations

### Operation Queue

A queue that regulates the execution of operations.

```swift
class OperationQueue : NSObject
```

#### Overview

操作队列会根据Operation对象的优先级以及意愿去对其进行执行。在将操作加入队列之后，其将保留在队列中直到执行完毕。在加入之后你不能直接把它从队列中移除。

> Note
>
> Operation queues retain operations until they're finished, and queues themselves are retained until all operations are finished. Suspending an operation queue with operations that aren't finished can result in a memory leak. 

#### Determining Execution Order

Operations within a queue are organized according to their readiness, priority level, and interoperation dependencies, and are executed accordingly. If all of the queued operations have the same [`queuePriority`](apple-reference-documentation://hsGcR_rqa1) and are ready to execute when they are put in the queue—that is, their [`isReady`](apple-reference-documentation://hspCMv4pon) property returns `true`—they’re executed in the order in which they were submitted to the queue. Otherwise, the operation queue always executes the one with the highest priority relative to the other ready operations. 

However, you should never rely on queue semantics to ensure a specific execution order of operations, because changes in the readiness of an operation can change the resulting execution order. Interoperation dependencies provide an absolute execution order for operations, even if those operations are located in different operation queues. An operation object is not considered ready to execute until all of its dependent operations have finished executing. 

For details on how to set priority levels and dependencies, see [Managing Dependencies](apple-reference-documentation://hsHeJtH0Tx) in [`Operation`](apple-reference-documentation://hsHeJtH0Tx).

### Operation

An abstract class that represents the code and data associated with a single task.

```swift
class Operation : NSObject
```

#### Overview

Operation为抽象类，不能直接使用这个类而应该使用子类或系统定义的子类去执行实际的任务。Operation对象只能够被执行一次，通常将它加入队列中去进行执行操作。你也可以通过调用start方法手动执行Operation，但若该operation未准备好可能会触发异常。

#### Operation Dependencies

操作的依赖可以让各个操作以特定的顺序进行执行。默认情况下，如果操作有依赖，当依赖的操作执行完毕后（正常结束或者直接取消）这个操作才可以进入准备状态。

#### Subclassing Notes

The `NSOperation` class provides the basic logic to track the execution state of your operation but otherwise must be subclassed to do any real work. How you create your subclass depends on whether your operation is designed to execute concurrently or non-concurrently. 

##### Methods to Override

对于非并行操作只需重载下面一个方法：

- [`main()`](apple-reference-documentation://hsuy4NRNTq)

在这个方法中，你应该放置具体任务的代码。你也应该定义一个初始化方法以便于对自定义类的实例化，也可以定义getter和setter对操作里的数据进行访问。如果定义了getter和setter，你应当确保这个方法在多线程环境下是安全的。

对于并行操作应当至少重载以下操作：

- [`start()`](apple-reference-documentation://hsvmIrqBO3)
- [`isAsynchronous`](apple-reference-documentation://hs_Tb_O7Jv)
- [`isExecuting`](apple-reference-documentation://hs8WKmsidQ)
- [`isFinished`](apple-reference-documentation://hsnGGlxiKg)

In a concurrent operation, your `start()` method is responsible for starting the operation in an asynchronous manner. Whether you spawn a thread or call an asynchronous function, you do it from this method. Upon starting the operation, your `start()` method should also update the execution state of the operation as reported by the `isExecuting` property. You do this by sending out KVO notifications for the `isExecuting` key path, which lets interested clients know that the operation is now running. Your `isExecuting` property must also provide the status in a thread-safe manner.

Upon completion or cancellation of its task, your concurrent operation object must generate KVO notifications for both the `isExecuting` and `isFinished` key paths to mark the final change of state for your operation. (In the case of cancellation, it is still important to update the `isFinished` key path, even if the operation did not completely finish its task. Queued operations must report that they are finished before they can be removed from a queue.) In addition to generating KVO notifications, your overrides of the `isExecuting` and `isFinished`properties should also continue to report accurate values based on the state of your operation.

For additional information and guidance on how to define concurrent operations, see [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091).























