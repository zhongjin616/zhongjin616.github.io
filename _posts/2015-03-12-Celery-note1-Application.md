
hi all:
In this note, i will talk about Celery Application concept

Application
===========
in order to gain the power of Celery library, i need to instantiate one:

>>> from celery import Celery
>>> app = Celery()

now i got an instance of Celery called ‘app’. it’s thread-safe, so...

Main Name
---------
what is a ‘task’ in the Celery world? it’s nothing but a block of code to execute AKA. a function.
But when you send a task in Celery, the task message does not contain any source code. it’s worker’s duty to mapping
task names to their actual functions,which called `task registry`.

# define a task in celery instance.
>>> @app.task
... def add(x, y):
...    return x+y

now i have a task named ‘add’ can be assing to workers. see its full name:
>>> add
<@task: __main__.add>
>>> add.name
__main__.add
>>> app.tasks[`__main__.add`]
<@task: __main__.add>

whenever Celery is not able to detect which module the task aka. function belongs to , it uses main module name to generate the beginning of the task name.

i can also start worker with task define:
 task_with_worker.py:
from celery import Celery
app = Celery()

@app.task
def add(x, y): return x+y

if __name__ == ‘__main_____”:
    app.worker_main() #start a worker to do task.

when this module is imported by another process, the task defined in will be named starting with the module name
>>> from task_with_worker import add
>>> add.name
<task_with_worker.add>

you can also specify another name for the main module:
>>> app = Celery(‘myTasks’)
>>> app.main
‘myTasks’


Configuration
--------------
in case i want to control how Celery works against the default behavior, i can use ‘app.conf.SOME_OPTION’ set directly on the app instance,
or use a dedicated configuration module by call “app.config_from_boject(‘name_of_conf_module’)”
:NOTE  the Changes made at runtime wins the configuration module(if any), if none of this exists, the default configuration will be used aka. celery.app.defaults.

* app.config_from_boject() can have three mode to call: 
    1. using the name of a module
    2. using a configuration module
    3. using a configuration class/object

* app.config_from_envvar(‘variable_name’) allow i to configure through environment variable

Laziness
--------
the Celery instance is lazy. when you create a celery instance say app, and then define a task use app.task():
the task will not be evaluated immediatly untill it’s used or call app.finalize().
>>> @app.task
....def add(x, y): return x+y

>>> type( add )
<class ‘celery.local.PromiseProxy>

>>> add.__evaluated__()
False

>>> add   # this will casuse repr(add) to happen
>>> add.__evaluated__()
True

Insisting the ‘app chain’
-------------------------
it is a good proctice to put the `app` as an argument, i call this the ‘app chain’
* bad practice
from celery import current_app

class Schelduler(object):
    def run(self):  #must have this func to call  .delay()
        app = current_app 

instead, i use this 
from celery.app import app_or_default

class Scheduler(object):
    def __init__(self, app=None):
        self.app = app_or_default(app)

API Evalution
---------------
in Celery2.0, to define a task:
from celery.task import Task
from celery.registy import tasks

class Hello(Task):
    send_error_emails = True

    def run(self, to):  #define a block of code as task
        return ‘hello {0}’.format(to)
tasks.register(Hello) #make class Hello to be a task

>>> Hello.delay(‘world!’)

in Celery3.0, to define a task:
from celery.task import task

@task(send_error_emails=True)
def hello(to):  #not need a class.run(),just a func immediatly
    return ‘hello {0}’.format(to)

once a task is bound to an app, it will read configuration to set default values and so on.
it is also possible to change the default base class for an application by changing its app.Task attribute.

referenc:[Celery user guide Application] (http://docs.celeryproject.org/en/latest/userguide/application.html)
