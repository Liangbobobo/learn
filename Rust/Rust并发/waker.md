# Task Wakeups with Waker
It's common that futures aren't able to complete the first time they are polled. When this happens, the future needs to ensure that it is polled again once it is ready to make more progress. This is done with the Waker type.

Each time a future is polled, it is polled as part of a "task". Tasks are the top-level futures that have been submitted to an executor.?  

Waker provides a wake() method that can be used to tell the executor that the associated task should be awoken. When wake() is called, the executor knows that the task associated with the Waker is ready to make progress, and its future should be polled again.

Waker also implements clone() so that it can be copied around and stored(存储).  

