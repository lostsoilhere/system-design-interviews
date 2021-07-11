<!-- MarkdownTOC -->

- [Multithreading](#multithreading)
	- [JMM](#jmm)
		- [Atomic](#atomic)
		- [Reorder](#reorder)
		- [Visibility](#visibility)
	- [Thread vs process](#thread-vs-process)
	- [Thread lifecycle](#thread-lifecycle)
		- [State conversion](#state-conversion)
		- [Create thread - implementing Runnable vs extending Thread](#create-thread---implementing-runnable-vs-extending-thread)
		- [Start a thread](#start-a-thread)
		- [Stop a thread](#stop-a-thread)
		- [Object methods](#object-methods)
			- [Wait, notify and notifyAll](#wait-notify-and-notifyall)
		- [Thread methods](#thread-methods)
			- [Join](#join)
			- [Sleep](#sleep)
	- [Deadlock](#deadlock)
		- [Def](#def)
		- [Conditions](#conditions)
	- [Java Concurrent Utilities - JCU](#java-concurrent-utilities---jcu)
	- [ThreadLocal](#threadlocal)
	- [Counters](#counters)
	- [Singleton pattern](#singleton-pattern)
	- [BoundedBlockingQueue](#boundedblockingqueue)
	- [Readers/writers lock [To be finished]](#readerswriters-lock-to-be-finished)
	- [Thread-safe producer and consumer](#thread-safe-producer-and-consumer)
	- [Delayed scheduler](#delayed-scheduler)
	- [References](#references)

<!-- /MarkdownTOC -->

# Multithreading
## JMM
### Atomic
### Reorder
### Visibility

## Thread vs process
* Similar goals: Split up workload into multiple parts and partition tasks into different, multiple tasks for these multiple actors. Two common ways of doing this are multi-threaded programs and multi-process systems. 
* Differences

| Criteria  |  Thread |  Process  |
| --------------------- |:-------------:| -----:|
| Def | A thread exists within a process and has less resource consumption | A running instance of a program |
| Resources | Multiple threads within the same process will share the same heap space but each thread still has its own registers and its own stack. | Each process has independent system resources. Inter process mechanism such as pipes, sockets, sockets need to be used to share resources. |
| Overhead for creation/termination/task switching  | Faster due to very little memory copying (just thread stack). Faster because CPU caches and program context can be maintained | Slower because whole process area needs to be copied. Slower because all process area needs to be reloaded |
| Synchronization overhead | Shared data that is modified requires special handling in the form of locks, mutexes and primitives | No synchronization needed |
| Use cases  | Threads are a useful choice when you have a workload that consists of lightweight tasks (in terms of processing effort or memory size) that come in, for example with a web server servicing page requests. There, each request is small in scope and in memory usage. Threads are also useful in situations where multi-part information is being processed – for example, separating a multi-page TIFF image into separate TIFF files for separate pages. In that situation, being able to load the TIFF into memory once and have multiple threads access the same memory buffer leads to performance benefits. | Processes are a useful choice for parallel programming with workloads where tasks take significant computing power, memory or both. For example, rendering or printing complicated file formats (such as PDF) can sometimes take significant amounts of time – many milliseconds per page – and involve significant memory and I/O requirements. In this situation, using a single-threaded process and using one process per file to process allows for better throughput due to increased independence and isolation between the tasks vs. using one process with multiple threads. |

## Thread lifecycle


### State conversion
* When will be a thread blocked?
  * Blocked:
  * Waiting:
  * Timed waiting: 
* New -> Runnable -> terminated is not reversible
* Timed waiting / Waiting / Blocked can only transfer to each other by going through Runnable first. 

![](./images/multithreads-threadstatus.jpeg)

### Create thread - implementing Runnable vs extending Thread
* Internal mechanism
  * There is only one way to create thread - create a Thread instance. And there are two ways to implement the run() method - Override the run() method inside Thread instance vs pass an implementation of Runnable interface into Thread constructor. 
  * Thread and Runnable are complement to each other for multithreading not competitor or replacement. Because we need both of them for multi-threading.
  	- For Multi-threading we need two things:
  		+ Something that can run inside a Thread (Runnable).
  		+ Something That can start a new Thread (Thread).
  	- So technically and theoretically both of them is necessary to start a thread, one will run and one will make it run (Like Wheel and Engine of motor vehicle).

```
@Override
public void run() 
{
    if (target != null) 
	{
        target.run();
    }
}
```

* Best practices - Implement Runnable()
  * Code cleaniness perspective: 
    * Decoupling: Implementing Runnable could separate thread creation from running. 
    * Extensibility: If adopting the approach of extending Thread, then it could not extend another class because Java does not support multiple inheritance.
  * Cost of operation perspective: Thread approach will require creating and destroying a thread object each time; When combined with threadpool, Runnable approach could avoid creating a new thread object and deleting it.

```
// Approach 1: Runnable
public class DemoRunnable implements Runnable 
{
    public void run() 
	{
        //Code
    }
}

// Approach 2: Thread
public class DemoThread extends Thread 
{
    public DemoThread() 
	{
        super("DemoThread");
    }

    public DemoThread(Runnable ) 
	{
        super("DemoThread");
    }

    public void run() 
	{
        //Code
    }
}
```

### Start a thread
* Best practices - Use Start()
  * Start() method responsiblity:
    * Start a new thread
    * Check the thread status
    * Add the thread to thread group
    * Kick off the Run()
  * Run() method responsibility:
    * Kick off the code inside Run()
  * Key difference is that Start() method (approach 1 below) will create a new thread to run. Run() (approach 2 below) will run everything inside main() method. 

```
public static void main(string[] args)
{
	// Approach 1: Create a runnable instance and run it
	// Output: main
	Runnable runnable = () -> 
	{
		System.out.println(Thread.currentThread().GetName());
	};
	runnable.run();

	// Approach 2: Start a new thread
	// Output: thread0
	new Thread(runnable).start();
}
```

### Stop a thread
* Java does not provide a way for one thread to force stop of another thread because if it does so, then the other thread might be in a state of inconsistency. Java provides a collaboration mechanism for one thread to notify another thread that it would better stop. 

* Best practices: Please see this folder for sample code: https://github.com/DreamOfTheRedChamber/system-design-interviews/tree/master/code/multithreads/StopThreads

### Object methods
* Please see https://github.com/DreamOfTheRedChamber/system-design-interviews/tree/master/code/multithreads/ObjectMethods for best practices

#### Wait, notify and notifyAll
* Wait and notify are all based on object's monitor mechanism. Therefore, they are declared as methods on top of Object. 
* They are considered the native way of doing multi-threading. Java JDK has shipped packages such as Condition variable which is easier to use. 


### Thread methods
* Please see https://github.com/DreamOfTheRedChamber/system-design-interviews/tree/master/code/multithreads/ThreadMethods

#### Join
* Join thread will be in the waiting status
* Join is the native way of doing waiting. Java JDK has shipped packages such as CountDownLatch or CyclicBarrier.

* Best pratices:  

#### Sleep
* Wait vs Sleep
  * Similarities:
    * Both wait and sleep method could make the thread come into blocked state. Wait will result in Waiting and sleep will result in Time_Waiting. 
    * Both wait and sleep method could respond to interrupt. 
  * Differences:
    * Wait could only be used in synchronized blocks, while sleep could be used in other scenarios. 
    * Wait is a method on Object, and sleep is a method on Thread. 
    * Wait will release monitor lock, and sleep will not. 
    * Wait could only exit blocked state reactively, and sleep could proactive exit after specific time. 

* Yield vs Sleep: Similar. However yield is non-blocking but sleep is blocking




## Deadlock
### Def
* A deadlock is a situation where a thread is waiting for an object lock that another thread holds, and this second thread is waiting for an object lock that the first thread holds. Since each thread is waiting for the other thread to relinquish a lock, they both remain waiting forever. 

### Conditions
* **Mutal Exclusion**: Only one process can access a resource at a given time. (Or more accurately, there is limited access to a resource. A deadlock could also occur if a resource has limited quantity. )
* **Hold and Wait**: Processes already holding a resource can request additional resources, without relinquishing their current resources. 
* **No Preemption**: One process cannot forcibly remove another process' resource.
* **Circular Wait**: Two or more processes form a circular chain where each process is waiting on another resource in the chain. 

## Java Concurrent Utilities - JCU 
* Thread basics - join, yield, future
* Executor services
* Semaphore/Mutex - locks, synchronized keyword
* Condition variables - wait, notify, condition
* Concurrency collections - CountDownLatch, ConcurrentHashMap, CopyOnWriteArrayList

## ThreadLocal
* [Link to the subpage](https://github.com/DreamOfTheRedChamber/system-design-interviews/blob/master/code/multithreads/DelayedQueue.md)


## Counters
* See src dir for details

## Singleton pattern
* [Link to the subpage](https://github.com/DreamOfTheRedChamber/system-design-interviews/blob/master/code/multithreads/SingletonPattern.md)

## BoundedBlockingQueue
* See src dir for details

## Readers/writers lock [To be finished]

## Thread-safe producer and consumer
* See src dir for details

## Delayed scheduler
* [Link to the subpage](https://github.com/DreamOfTheRedChamber/system-design-interviews/blob/master/code/multithreads/DelayedQueue.md)

## References
* [并发多线程常见面试题](https://docs.qq.com/doc/DSVNyZ2FNWWFkeFpO)