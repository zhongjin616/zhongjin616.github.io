hi,all:
in this note, i talk about the Task. looking forward to it :-)”

Task Concept
=============
as i talked in note1, task in nothing other than a block code formed function.
but you should also acknowledge that `tasks are the building blocks of Celery applications`.

task performs dual roles: it defines both what happens when a task is called(sends a message),
and what happens when a worker receives that message.

the same as Rabbitmq no_ack attribute, a task message does not disappear untill the message acknowledged by a worker.
Ideally task functions should be `idempotent`, so that i can set `acks_late` option to have the the worker acknowledge the message
after the task returns INSTEAD the default behavior, which is to acknowledge in advance, before it’s executed, so that a task that has been
started is never executed again..

Basics
-------
i can easily create a task from __any callable__ by using the task() decorator:
>  @app.task(serializer=’json’)
>  def add(x, y):
>     return x+y

Names
------
every task must have a unique name.
if not specified, the function name will be used.
>  @app.task(‘sum-of-two-numbers’)
>  def add(x, y):
>     return x+y

but this is not the best practice, we can use the module name as a namespace to avoid confilction
>  @app.task(name=’moudleName.add’)
>  def add(x, y):
>     return x + y

Automatic naming and Relative imports
-------------------------------------
Relative imports and automatic name generation does not go well together, so if you’re using relative imports
you should set the name explicitly just like __above__

for example, if the client imports the module ‘myapp.tasks’ as ‘.tasks’, and the worker imports the module
as ‘myapp.tasks’, then generated names __won’t match__ and an __NotRegistered__ error will be raised by the __worker__
>  from project.myapp.tasks import mytask
>  mytask.name
‘project.myapp.tasks.mytask’

>  from myapp.tasks import mytask
>  mytask.name
‘myapp.tasks.mytask’

so for this reason you must be consistent in __how you import modules__, which is also a __python best practice__

similary, you should not use __old-style__ relative imports:
> from module import foo  # very bad!!

> from proj.module import foo # good practice

__new-style__ relative imports are fine and can be used:(but not the best practice)
> from .module import foo # Good

so if you don’t have time to refactor old code, then you have to specifying the names explicitly instead of
the automatic naming.


List of Options
---------------
the @task decorator can take a number of options that change the way the task behaves. Any keyword argument
passed to the task decorator will actually be set as an attribute of the resulting task class.
* Task.name
* Task.request
* Task.default_retry_delay
* Task.rate_limit
* Task.ignore_result
* Task.serializer
* Task.backend
* Task.__acks_late__

RabbiMQ Result Backend
-----------------------

this backed is special as it does not actually store the states, but rather sends them as message. This is an
important difference as it means that a result can only be retrieved once; if you have two processes waiting
for same result, one of the process will never receive the result!

this may also affect Rabbitmq performs in negative ways as every task will creates a new queue on the server,
with thousands of tasks the broker may be overloaded with queues.

Even with that limitation, it’s excellent choice if you need to receive state changes in real-time. 
Using messagi
ng means the client does not have to poll for new states.


Custom task classes
-------------------
  All tasks inherit from the app.Task class. the run() method becomes the task body.
> @app.task
> def add(x, y):
    return x+ y

celery will do this behind the scense:
> class AddTask(app.Task):

    def run(self, x, y):
        return x+y
add = app.tasks(AddTask.name)

__Instantiation__
  A task is not instantiated for every request, but is registered in the task registry as a global intance.
this is very useful to cache resources, e.g. a base Task that caches a database connection:
> from celery import Task

> class DatabaseTask(Task):

>    abstract = True

>    db = None

> 
>    @property
    def db(self):
        if self._db is None:
            self._db = Database.connect()
        return self._db

Handlers
---------
* after_return(self, status, retval, task_id, args, kwargs, einfo)
* on_failure(self, exc, task_id, args, kwargs, einfo)
* on_retry(self, exc, task_id, args, kwargs, einfo)
* on_success(self, retval, task_id, args, kwargs)

Tips and Best practice
------------------------

* Ignore results you don’t want as storing results wastes time and resources.
> @app.task(ignore_result=True)
> def mytask(...):
    doSomething(...)

Results can even be disabled globally using the CELERY_IGNORE_RESULT setting.

* Disabling rate limits altogether is recommanded if you don’t have any tasks using them.
    this is because the rate limit subsystem introduces quite a lot of comlexity.
set the __ CELERY_DISABLE_RATE_LIMITS=True to disable rate limits globaly.

* Avoid launching synchronous subtasks
    __subtask__ can be seen as a callback function chain after the task.
  having a task wait for the result of another task is really ineffcient, and may cause a deadlock.
  make your design asynchronous instead, for example by using `callbacks`. In celery you can create a chain of
  tasks by linking together differnt subtasks.


Performance and Strategies
----------------------------

* Granularity
    in general, it’s better to split the problem up into many small tasks, than have a few long running tasks.

* Data locality
    the worker processing the task should be as close to the data as possible. the best would be to have a copy
    in memory, the worst would be a full transfer from another continent.

* State
    Since celery is a distributed system, you can’t know in which process, or on what machine the task will be
    executed. You can’t even know if the task wil run in a timely manner.i

reference [celery user guide](http://docs.celeryproject.org/en/latest/userguide/tasks.html)
