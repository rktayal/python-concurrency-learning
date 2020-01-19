A process can be defined as execution context of a running program. Or simply defined as running instance of a computer program.
Every execution process has system resources and a section of memory that is assigned to it. A process is composed of 
one or more threads of execution.<br />

<p align="center">
        <img src="./tmp/process_vs_thread.png" height="250"/>
</p> <br />

A thread is the smallest sequence of instructions that can be managed, that is scheduled 
and executed by the operating system.<br />
The use of multiple threads allows the process to perform multiple tasks at once.<br />
```
For example in a media player, one thread can be running the current song, 
while the other thread is figuring out the next song to play while the other
thread is responding to user clicks and navigation. 
```

### Creating threads in Python
Simplest way to create a thread is to instantiate the object of the thread class, passing in
the thread function and any function arguments.
```
import threading

def do_some_work(val):
    print('doing some work in thread')
    print('echo: {}'.format(val))
    return

val = "text"
t = threading.Thread(target=do_some_work, args=(val,))
t.start()
t.join()
```

The full thread constructor is:
```
threading.Thread(group=None,
                 target=None,
                 name=None,
                 args=(),
                 kwargs={},
                 daemon=None)
```

### Working of Threads
Image below shows the execution flow of multi-threaded program
<p align="center">
        <img src="./tmp/thread_working.png"/>
</p> <br />

A Thread can be in one of the following states:
<p align="center">
        <img src="./tmp/thread_lifecycle.png" />
</p> <br />

Threads within a process share code, common memory space and other OS resources such as open files
or network sockets. Therefore any memory owned by process can be modified by any of the thread.
Memory sharing combined with indeterminate scheduling can lead to suitation known as thread interference.
Below is an example for thread interference.
<p align="center">
        <img src="./tmp/thread_interference.png"/>
</p> <br />

### Thread Synchronization
Python has various mechanism to synchronize access to shared resources.
### threading.Lock
The most fundamental of these is a lock.<br />
It can be in one of the two states: locked or unlocked.<br />
When locked by one thread, it can only be unlocked by the same thread.<br />
When a thread tries to acquire a lock that is already acquired, that thread goes into a blocking state,
which means its execution is paused and can't continue until the lock is released by the owning thread.<br />
We can use locking mechanism to synchronize access to shared resource. 
```
lock = threading.Lock()
lock.acquire()
try:
    ... access the shared resource ...
finally:
    lock.release()
```
By putting the `release` in finally block, we ensure that it is executed even if an exception 
is thrown, else the lock may never be released and cause the waiting threads to wait forever.
Same can be done using below syntax:
```
lock = threading.Lock()
with lock:
    ... access shared resource ...
```
the lock is automatically acquired and released when leaving.
Other lock APIs are
```
if lock.acquire(False):
    ... Lock acquired, do stuff with lock
else:
    ... could not acquire lock, do other stuff
```
The `lock.locked()` method allows the called to determine whether the lock has been acquired or not
```
if lock.locked():
    ... do other stuff
else:
    lock.acquire()
```

### threading.Semaphore
Another widely used thread synchronization mechanism is the semaphore.
```
semaphore = threading.Semaphore()

semaphore.acquire() # decrements the counter
... access the shared resource
semaphore.release() # increments the counter
```
The internal counter can never go below 0, so if a thread makes a call
to acquire, and the value of counter is 0, the thread gets blocked and 
must wait for another thread to call release. <br />
A common analogy is that the semaphore maintains a set of permits.
```
num_permits = 3
semaphore = threading.BoundedSemaphore(num_permits)

semaphore.acquire() # decrements the counter
... upto 3 threads can access the shared resource at a time ...
semaphore.release() # increments the counter
```
Standard semaphore allows us to call `release` unlimited number of times,
more times than we called `acquire`. <br />
With `BoundedSemaphore`, if we inadvertenly try to call `release` more times
than we have called acquire, an error will be thrown. <br />

### threading.Event
One thread signals the event and the other thread waits for it. <br />
Event object has an internal flag. <br />
the thread or threads that need to wait for the event to occur do so
by calling the wait method on the event. As long as event's internal flag 
is `False`, the threads that call `wait` will block. A manager or server thread
can then call the set method on the event, which sets the internal flag to `True`
and releases the block thread or threads.
```
event = threading.Event()

# a client thread can wait for the thread to be set
event.wait()  # blocks if flag is false

# a server thread can set or reset it
event.set()    # sets the flag to true
event.clear()  # resets the flag to false
```

### threading.Condition
Condition combines the property of a lock and an event.
Like a lock it has `acquire()` and `release()` method to synchronize
access to some shared state, and like an event it has `wait()`, `notify()`,
and `notify_all()` methods so that threads interested in knowing when the
state changes can call the wait method and can be notified when the condition changes.<br />
Typical use case is Producer-Consumer Pattern. <br />
```
cond = Condition()

# Consume one item
cond.acquire()
while not is_item_available():
    cond.wait()
get_an_available_item()
cond.release()

# Produce one item
cond.acquire()
make_an_item_available()
cond.notify()
cond.release()
```

### Inter Thread communication using Queues
Queue makes it easier to exchange messages b/w multiple threads. It 
handles all the locking semantics for us and allows us to focus on 4 methods:
- put(): Put an item in the queue
- get(): Removes an item from queue and returns it
- task_done(): Marks an item that was gotten from the queue as completed/processed.
- join(): blocks until all the items in the queue have been processed.
```
from threading import Thread
from queue import Queue

def producer():
    for i in range(10):
        item = make_an_item_available(i)
        queue.put(item)

def consumer():
    while True:
        item = queue.get()
        # do something with item
        queue.task_done() # mark item as done

queue = Queue()
t1 = Thread(target=producer, args=(queue,))
t2 = Thread(target=consumer, args=(queue,))
t1.start()
t2.start()
```
