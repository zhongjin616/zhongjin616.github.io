hi all, a new day. let’s jump into Celery, can’t wait!

today, i will talk about the Routing Task in Celery, which is a Wrap of RabbitMq

Routing Tasks
===============

* Basics: Automatic routing

* AMQP Primer

* Routing Tasks


Basics: Automatic routing
---------------------------
there are several configure option controls how automatic routing works:

    1. CELERY_QUEUES: is a list of Queue instances.
    
    2. CELERY_CREATE_MISSING_QUEUE
    
    3. CELERY_ROUTES = {‘task.add’: {‘queue’: ‘sum_number’}}
        with this route enabled, ‘task_add’ will be routed to the ‘sum_number’ queue, while all other
        tasks will be routed to the default queue (named “celery” for historical reasons).
        So,from the quotation [above](http://docs.celeryproject.org/en/latest/userguide/routing.html), i may infer that
**we can just use ‘pika’ to send a plain message, as the celery needs to verify the stuff arrive at queue, which must be a `task Signature`**
    
    4. specify the worker to process certain queue’s task.
        $ celery -A proj worker -Q sum_number,celery

__Changing the name of the default queue__
    you can change the name of the default queue by using the following configuration.

    from kombu import Exchange, Queue

    CELERY_DEFAULT_QUEUE = ‘default’
    CELERY_QUEUES = (
        Queue(‘default’, Exchange(‘default’), routing_key=’default’)
    )

**How the queues are defined**
Celery want to hide the complex AMQP protocol for users with only basic needs.

A queue named ‘video’ will be created with the following settings:

    {
        
        ‘exchange’: ‘video’,

        ‘exchange_type’: ‘direct’,

        ‘routing_key’: ‘video’

    }
the **exchange** name must the same with **queue** ,that because the non-AMQP backends does not support exchanges. so using this design to ensure
it will work for them as well.


