---
title: "Overview of Multi Threading in java"
datePublished: Fri Mar 29 2024 04:38:31 GMT+0000 (Coordinated Universal Time)
cuid: cluc6b7pw000808ia1b6b6ysl
slug: overview-of-multi-threading-in-java
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711697580809/69a93b04-5b5a-4d23-b624-dfa8ca17b927.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1711687071136/5ac2dd0c-64c7-4149-a1b2-302d2e4aaeb9.png
tags: java, multithreading, programming-languages, codenewbies

---

> Multi-threading refers to the concurrent execution of multiple threads within the

## Thread

A thread is the smallest unit of execution within a process, and multithreading allows multiple threads to execute concurrently, sharing the same resources such as memory space, file handles, and other process-wide resources.

### How to create a thread in java ?

We can create our custom thread by two methods

1. By extending Thread class
    
2. By implementing Runnable interface
    

### Creating a thread by extending Thread class

* We define our custom thread class `MyThread` which extends `Thread` Class
    
* In `MyThread` Class we override `run` method ,which contains the code that the thread will execute when started
    
* In the `Main` class, we create an instance of `MyThread` and start it using the `start()` method.
    
* We can execute our task defined in the run method by either calling `start()` or by using `run()` method the main difference is that if we use `run()` then a new thread will not be created.
    

```java
class MyThread extends Thread {
    
    // Override the run() method to define the code that the thread will execute
    @Override
    public void run() {
        // Code inside this method will be executed when the thread starts
        for (int i = 0; i < 5; i++) {
            System.out.println("Thread running: " + i);
            
            try {
                // Introduce a short delay to simulate some work being done
                Thread.sleep(1000); // Sleep for 1 second (1000 milliseconds)
            } catch (InterruptedException e) {
                // Handle the InterruptedException if needed
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // Create an instance of your custom thread class
        MyThread myThread = new MyThread();
        
        // Start the thread
        myThread.start();
        
        // The code below will continue to execute 
        //immediately after starting the thread
        // It will not wait for the thread to finish its execution
        System.out.println("Main thread exiting...");
    }
}
```

### Output

![Output](https://cdn.hashnode.com/res/hashnode/image/upload/v1711684615046/7df69aff-c294-4d74-b17e-4e30deacfcc3.png align="left")

## Creating a thread by implementing Runnable interface

Creating a thread by implementing the `Runnable` interface is a common practice in Java and provides more flexibility compared to extending the `Thread` class

* We define a class `MyRunnable` that implements the `Runnable` interface.
    
* Inside `MyRunnable`, we implement the `run()` method, which contains the code that the thread will execute when started.
    
* In the `Main` class, we create an instance of `MyRunnable`
    
* We then create a `Thread` object, passing the `MyRunnable` instance as a parameter to its constructor
    
* Now `start()` method of `Thread` object begins the execution of the thread, and the `run()` method of `MyRunnable` will be executed concurrently within this `Thread` along with the main thread.
    

```java
// Define a class that implements the Runnable interface
class MyRunnable implements Runnable {
    
    // Implement the run() method defined by the Runnable interface
    @Override
    public void run() {
        // Code inside this method will be executed when the thread starts
        for (int i = 0; i < 5; i++) {
            System.out.println("Thread running: " + i);
            
            try {
                // Introduce a short delay to simulate some work being done
                Thread.sleep(1000); // Sleep for 1 second (1000 milliseconds)
            } catch (InterruptedException e) {
                // Handle the InterruptedException if needed
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // Create an instance of your custom Runnable class
        MyRunnable myRunnable = new MyRunnable();
        // Create a Thread object, passing the Runnable instance as a parameter
        Thread thread = new Thread(myRunnable);
        // Start the thread
        thread.start();
        // It will not wait for the thread to finish its execution
        System.out.println("Main thread exiting...");
    }
}
```

### Output

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711685093182/c35f76bd-86c1-4d6e-b7d3-e9832ad6c32d.png align="center")

# Various Constructors of Thread class

Java provides various contructors of thread class for various usecases

1. ## Thread()
    
    * This constructor creates a new thread object.
        
    * The thread's runnable target is set to null, meaning it does not have a target to execute.
        
    * You would typically subclass (basically extending Thread class) `Thread` and override its `run()` method to define the thread's behavior before starting it.
        
    
2. ## **Thread(Runnable target)**
    
    * This constructor creates a new thread object with the specified `Runnable` target.
        
    * The `Runnable` target represents the task that the thread will execute when started.
        
    * You pass an instance of a class that implements the `Runnable` interface, providing the `run()` method implementation as the target for the thread.
        
    
3. ## **Thread(Runnable target, String name)**
    
    * This constructor creates a new thread object with the specified `Runnable` target and name.
        
    * The `name` parameter allows you to give the thread a meaningful name for identification and debugging purposes.
        
    * This is useful when working with multiple threads to distinguish them in logs or thread dumps.
        
    
4. * ## **Thread (ThreadGroup group, Runnable target)**
        
    * This constructor creates a new thread object with the specified `ThreadGroup` and `Runnable` target.
        
    * The `ThreadGroup` parameter allows you to specify the thread group to which the new thread belongs.
        
    * <details data-node-type="hn-details-summary"><summary>A thread group in Java is a way to group multiple threads together for easier management and control. It provides a mechanism for organizing threads into a hierarchical structure, allowing you to perform operations on multiple threads collectively</summary><div data-type="detailsContent"></div></details>
5. ## **Thread(ThreadGroup group, Runnable target, String name)**
    
    * This constructor creates a new thread object with the specified `ThreadGroup`, `Runnable` target, and name.
        
    * It combines the functionalities of the previous two constructors, allowing you to specify both the thread group and the name of the thread.
        
    * And also giving your own runnable target.
        
    

# Lifecycle of a thread

A thread passes through various stages during it's lifetime from creation to termination . Various stages in a lifecycle of a thread are

## New (or Born)

A thread enters the new state when an instance of the `Thread` class is created but has not yet been started by invoking its `start()` method.

## **Runnable (or Ready)**

* Once the `start()` method is called on a thread, it enters the runnable state.
    
* Now the thread is ready to be executed but it's on thread scheduler to when execute it
    
* In this state, the thread may be executing if the scheduler has allocated CPU time to it, or it may be waiting for CPU time.
    

## **Running State**

* When the scheduler selects the thread from the runnable state, it enters the running state.
    
* The thread's `run()` method is being executed, and it is actively using CPU resources.
    
* Only one thread can be in the running state at any given time on a single CPU core.
    

## **Blocked (or Waiting) State**

* A thread enters the blocked state when it is temporarily unable to proceed with its execution, typically because it's waiting for a resource or condition to become available.
    
* For example, a thread might enter a blocked state when waiting for I/O operations to complete, waiting to acquire a lock, or waiting on a condition variable , or waiting for other thread to complete it's execution.
    
* The thread remains in this state until the condition it's waiting for is satisfied.
    

## **Timed Waiting State**

* Threads can also enter a timed waiting state, which is similar to the blocked state but with a timeout.
    
* This state occurs when a thread invokes methods like `Thread.sleep()`, `Object.wait()`, or `Thread.join()` with a specified timeout.
    
* The thread remains in this state until either the specified time elapses or the condition it's waiting for is satisfied.
    

## **Terminated (or Dead) State**

* A thread enters the terminated state when its `run()` method completes, or when it is explicitly terminated by calling `Thread.stop()` (not recommended) or when an uncaught exception terminates the thread.
    
* Once a thread is terminated, it cannot be restarted.
    
* After termination, a thread can be garbage collected.
    

# Methods for pausing the execution of a thread

* `yield()`
    
* `join()`
    
* `sleep()`
    

This was the overview of multithreading in java , in the next post we will talk more about these methods of a Thread class and synchronization.  
Thank you , Have a nice day 😊

Bye Bye🙋‍♂️ see you all in the next post.