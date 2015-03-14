hi all, still me :-)

here i will talk about the most insteresting part, from my personal perispective.

Designing workflow
===================


Signatures
-----------
what exactly is this thing? instead explain this, i will talk about **why we need this thing?**
sometimes, you may want to pass the **signature** of a __task invocation__ to another process
or as an argument to another function. 

So, a signature wraps the __arguments__,__keyword arguments, and execution options__ of a single task **invocation**
in a way such that it can be passed to functions or even serialized and sent across the wire.
<-**this is what app.task.delay() send to worker?-**>

Thus Signatures are often nicknamed **subtasks** because they describe a task to be called within a task.

Partials
---------
With a signature, you can execute the task in a worker:
    >>> add.s(2,2).delay()
    >>> add.s(2,2).apply_async(countdown=1)
or you can call it directly in the current process:
    >>> add.s(2,2)()

specifying addtional args, kwargs or options to **apply_async/delay** create partials:
so __partials are meant to be used with callbacks__ any tasks linked or chord callbacks will be applied with
 the result of the parent task. Sometimes you want to specify a callback that does not take additional arguments
  and in that case you can set the signature to be immutable:
  >>> add.delay((2,2), link=add.subtask((2,2), immutable=True))
  or for short:
  >>> s = add.s(2,2)
  >>> add.delay((2,2), link=s.si())

**To conclude, signature evaluated:  Signature ==> subtask ==> partial  all related to callbacks.**

Callbacks
----------
Callbacks can be added to any task using the `link` argument to `apply_async`:
    >>> add.apply_async((2,2), link=add.subtask(2))

The Primitives
---------------
New in version 3.0

  **Overview**
  * group

  * chain

  * chord

  * map

  * starmap

  * chunks



