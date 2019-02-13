# Asynchronicity

## Synchronous vs asynchronous execution

* Having a program being a sequence of operations, they may be executed synchronously or asynchronously
* Asynchronous execution lets us do other things while waiting until an operation finishes
* Therefore, asynchronous execution introduces *concurrency*
* This concurrency may be implemented either as parallelism or as non-blocking waiting
* Old style Java `Future` example - run something, do other things, then `get`
* This way we can batch and "parallelize" non-blocking waiting
* This asynchronicity is not usually what we want

## Non-blocking operations

* Some operations in our program may wait for long or possibly indefinite amount of time
* This program will be run multiple times, concurrently. The longer it is, the more executions running simultaneously
  at given time.
* If the waiting operation is blocking the thread, we may need a lot of threads. This also means a lot of context switches
  which may degrade performance.
* We want the long-waiting operation to be non-blocking on the thread.
* Currently, in Java and Scala, the only way to achieve this is by making it *asynchronous*. There are languages
  which can express non-blocking programs (with underlying asynchronous execution) retaining the desired synchronous
  syntax: Erlang, Haskell.
* We don't care that much about "forking" capabilities of asynchronicity, we want non-blocking execution and an ability
  to specify *continuations*. Therefore, we may write our program in *continuation passing style*.
* There's another name for continuation passing style: *callback hell*.
