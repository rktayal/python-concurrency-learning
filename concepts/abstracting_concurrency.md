### Abstracting Concurrency Mechanisms
In both cases (threading and multiprocessing), at a high level, what we are doing is defining a task,
passing it to some executor, and then getting the result back and waiting for it to complete.
- Define Task
- Pass to Executor
- Get Results
For IO bound task, executor is a thread, while for CPU bound task, executor is a process.
We can turn this high level description into an interface.
What this buys us is that instead of directly dealing with underlying threads and processes,
we simply submit tasks to the interface, and let the implementation manage the execution of the 
task. We don't need to worry about the instantiation of processes and threads, starting them and
joining them, all this is taken care of.<br />
Another thing it buys us is that we can easily switch between threads and processes.

### The Executor API
The executor API exposes three methods:
- submit
- map
- shutdown

The `submit(fn, *args, **kwargs)` method schedules the passing function to run on one of the 
executor's workers and returns a future object that represents the execution state of the function.
Call to submit is non-blocking. It immediately returns the future object to the caller, 
so that the caller can continue executing and get the results of the computation at a later time by
calling the results method on the future object.<br /> <br/>

The `map(func, *iterables, timeout=None, chunksize=1)` method uses the executor worker pool to apply 
the passed-in function to every member of the iterable concurrently. <br/><br/>

The `shutdown(wait=True)` method is used to signal to the executor that no more tasks will be
submitted to it, and that it should free up any resource that it is currently using. Once 
execute is shutdown, any subsequent call to submit or map on it will result in runtime error.
The wait parameter specifies whether the call should block or not.
if set then call is blocking else it is non-blocking call. <br/><br/>

Working Example of ThreadPoolExecutor:
```
from concurrent.futures import ThreadPoolExecutor
from urllib.request import urlopen

def load_url(url, timeout):
    with urlopen(url, timeout=timeout) as conn:
        return conn.read()

with ThreadPoolExecutor(max_workers=2) as executor:
    url1 = "http://www.cnn.com/"
    url2 = "http://www.some-made-up-domain.com/"
    f1 = executor.submit(load_url, url1, 60)
    f2 = executor.submit(load_url, url2, 60)
    try:
        data1 = f1.result()
        print("{} page is {} bytes".format(url1, len(data1)))
        data2 = f2.result()
        print("{} page is {} bytes".format(url2, len(data2)))
    except Exception as e:
        print('Exception downloading page" + str(e))
```


Working Example of ProcessPoolExecutor
```
from concurrent.futures import ProcessPoolExecutor
import hashlib

texts = [b"the quick brown fox jumped over the lazy dog",
         b"the big fat panda sat on the hungry snake",
         b"the slick mountain lion ran up the tall tree"]

def generate_hash(text):
    return hashlib.sha384(text).hexdigest()

if __name__ == "__main__":
    with ProcessPoolExecutor() as executor:
        for text, hash_t in zip(texts, executor.map(generate_hash, texts)):
            print("%s hash is %s" % (text, hash_t))
```
