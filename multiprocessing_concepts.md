In running Python programs there is one interpreter per process. And since the GIL
is lock on the interpreter, there is one GIL per process. Therefore, if we want to get around 
GIL blocking multiple instructions from running simultaneously on different cores, we can create 
copies of process.

<p align="center">
	<img src="./tmp/gil_per_process.png" width="500"/>
</p> <br />

### Multiprocessing API
