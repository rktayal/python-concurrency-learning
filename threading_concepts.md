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
