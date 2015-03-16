hello, there.

today, i will talk about the ‘apps’ in celery.

What app?
------------
the app, here, i mean the celery instance.
there are several attribute i can use in Celery:

* [celery.current_app](http://celery.readthedocs.org/en/latest/reference/celery.html#celery.current_app)
    but this is not the recommended method. not a best practice

* [celery.app.default_app](http://celery.readthedocs.org/en/latest/reference/celery.app.html)

* celery.app.app_or_default(app=None)   this is the could be a chooice, if you don’t want to create a new celery instance and just use default application

reference celery userguide application [app chain”][1]

[1]:(http://celery.readthedocs.org/en/latest/userguide/application.html)
