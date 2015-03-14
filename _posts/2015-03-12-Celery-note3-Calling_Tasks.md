
Basics
-------
  there is three method to send a task message to broker:
  
  1. apply_async(args, kwargs[, ...])

  2. delay(**args, **kwargs)

  3. calling(__call__)

  so `delay` is clearly convenient, but if you want to set additional `execution options` you have to use apply_async.

  **Tips** if the task is not registered in the current process, you can use send_task() to call the task by name instead.

Linking(callbacks/errbacks) aka. Chain
---------------------------------------
  add.apply_async((2,2), link=add.s(16))

in practice the **link** execution options is considered an internal primitive, and you will probably not use it directly,
 but use **chains** instead.
