# Multithreaded-Js-Patterns
Lecture and exercises

# CHAPTER 6

## Contents

- Thread Pool
- Mutex 
- Streaming Data
- Actor Model

## Multithreaded Patterns

In previous chapters we have seen how to store raw data and its binary representation, even in the previous chapter primitive methods of modifying the use of bytes at the same time were expropriated.

In this chapter, design patterns will be explored to convert these concepts into functional applications.

### 1. Thread Pool Pattern

---

###### Definition

A thread pool is a collection of homogeneous worker threads that are each capable of carrying out CPU-intensive tasks that the application may depend on. This differs somewhat from the approach you’ve been using so far where usually a single worker thread, or a finite number of workers, has been used. As an example of this, the libuv library that Node.js depends on provides a thread pool, defaulting to four threads, for performing low-level I/O operations.

The first question when creating a thread pool is how many threads should be in the pool?

#### Pool Size

###### Concept Example

There are essentially two types of programs: those that run in the background, like a system daemon process, which ideally shouldn’t consume that many resources, and programs that run in the foreground that any given user is more likely to be aware of, like a desktop application or a web server

##### Number of Cores

Run this comand

// Browser

`cores = navigator.hardwareConcurrency;`

// Node.js

`cores = require('os').cpus().length;`

###### **NOTE:** `Most operating systems there is not a direct correlation between a thread and a CPU core ***`

Balance between the use of cores and the use of threads results in program efficiency.

`4 workers = number of cores used = 5`

`-> 3 threads for the workers`
` -> 1 for the livbuv (node)`
` -> 1 garbage collection and so on`

this affect the performace of the application.

### Dispatch Strategies

---

Because the goal of a thread pool is to maximize the work that can be done in parallel, it stands to reason that no single worker should get too much work to handle and no threads should be sitting there idle without work to do. A naive approach might be to just collect tasks to be done, then pass them in once the number of tasks ready to be performed meets the number of worker threads and continue once they all complete. Here’s a list of the most common strategies:

- ###### Round Robin: Each task is given to the next worker in the pool

- ###### Random: Each task is assigned to a random worker in the pool.

- ###### Least Busy: A count of the number of tasks being performed by each worker is maintained, and when a new task comes along it is given to the least busy worker.

#### Example implementation

---

Navigate to the ch6-thread-pool/ directory

- Example 1 run

`THREADS=3 STRATEGY=leastbusy node main.js`

- Example 2 run

`THREADS=3 STRATEGY=roundrobin node main.js`

- Example 3 run

`THREADS=3 STRATEGY=random node main.js`

### 2. Mutex: A Basic Lock Pattern

---

###### Definition 

Shared data access mechanism, ensures that the tasks use the resources given in time

commonly used when there are atomic operations

#### Example Mutext

---

- Example 1

Result:

Run 1: `node thread-product.js --> works fine`

Run 2: `node thread-product.js`

Shows:

`Expected values to be strictly equal:14 !== 21 `
To solve this, we’ll implement a Mutex class using the primitives we have in Atomics. We’ll be making use of Atomics.wait() to wait until the lock can be acquired, and Atomics.notify() to notify threads that the lock has been released. We’ll use Atomics.compareExchange() to swap the locked/unlocked state and determine whether we need to wait to get the lock.

- Example 1 solution

Result:

`Mutex { shared: Int32Array(5) [ 2, 3, 5, 14, 1 ], index: 4 }`


`Mutex { shared: Int32Array(5) [ 2, 3, 5, 70, 1 ], index: 4 }`


`Mutex { shared: Int32Array(5) [ 2, 3, 5, 210, 1 ], index: 4 }`

You can run this example with Node.js as follows:
`$ node thread-product-mutex.js`

### 3. Streaming Data with Ring Buffers Pattern

---

###### Definition

A ring buffer is an implementation of a first-in-first-out (FIFO) queue, implemented using a pair of indices into an array of data in memory. Crucially, for efficiency, when data is inserted into the queue, it won’t ever move to another spot in memory. Instead, we move the indices around as data gets added to or removed from the queue. The array is treated as if one end is connected to the other, creating a ring of data. This means that if these indices are incremented past the end of the array, they’ll go back to the beginning.

#### Example Ring Buffer

---

You can run this example with Node.js as follows:

`$ node ring-buffer.js`

### 4. Actor Model Pattern

---

###### Definition

The actor model is a programming pattern for performing concurrent computation that was first devised in the 1970s. With this model an actor is a primitive container that allows for executing code. An actor is capable of running logic, creating more actors, sending messages to other actors, and receiving messages.

#### Example actor model

---

You can run this example with Node.js as follows:

###### Terminal 1

`$ node server.js 127.0.0.1:8000 127.0.0.1:9000`
`# web: http://127.0.0.1:8000`
`# actor: tcp://127.0.0.1:9000`

###### Terminal 2

`$ node actor.js 127.0.0.1:9000`

###### Terminal 3

`curl -o - http://localhost:8000/8888888`
`# {"id":8,"value":17667693458.923462,"pid":"server"}`

---

### TIP

###### **_Adding -> `-o -` tell curl explicitly that you want the output to stdout. This feature should be able to drastically reduce the risk for false positives._**

---

##### Result:

`{"id":7,"value":17667693458.923462,"pid":2970}`

were `"id":7` is the new actor
