


Entries
--------
Like with cron, the tasks may overlap if the first task does not complete **before** the next.
You have to ensure only a single scheduler is running for a schedule at a time, otherwise you would end up
with duplicate tasks.
if that is a concern, you should use a locking strategy to ensure only one instance can run at a time

Using a centralized approach means the schedule does not have to be synchronized, and the service can operate
without using locks



Starting the Schedular
-----------------------
Beat needs to store the last run times of the task in a local database file (named celerybeat-schedule by default)
so make sure i has access to write in the current directory.
