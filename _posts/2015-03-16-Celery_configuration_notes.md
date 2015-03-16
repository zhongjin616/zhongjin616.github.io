
## we cannot use:

    celery worker -A addTask subTask -l info

    try to start up two worker.



## To specify task name is the best practice in celery.

    from celery.app import app_or_default

    app = app_or_default()

    @app.task(name=`name_to_reference_globally`)

    def add(x, y):

        return x + y



## by default, all worker will listen to:

     queue:celery     exchange=celery(direct) key=celery

    also, if i want to change this, i have two ways:
    
        * set the route of task by CELERY_ROUTES in celeryconfig.py config file
        
            this is the best practice

        * or override this when you define a task:
        
            @app.task(queue=’queue_to_delivery’,exchange=’’,serializer=’json’)  //no routing_key argument
            
                queue must exists or CELERY_CREATING_MISSING_QUEUE enable

                this is not recommanded, as it use hard code. very difficult to maintain

        * when you assign a new task, use **async _ apply(routing_key=’’,queue=’’,exchange=’’)**
        reference [celery.app.tasks](http://celery.readthedocs.org/en/latest/reference/celery.app.task.html)
            if the task fail to delivery by the arg setting, the route set in celeryconfig.py CELERY_ROUTES will use to deliver it

## if we want a worker just listen one specific queue, we need a **-Q** to exclude other queue explicitly.
