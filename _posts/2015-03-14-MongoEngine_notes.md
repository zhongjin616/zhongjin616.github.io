hi, all. today i will talk about MongoEngine

Tutorial
===========

if you have a class like this:

`
    class PeriodicTask(Document):
        meta = {
            collection= “recipes”,
            allow_inheritance= True
        }

        class CronTab(EmbeddedDocument):
            meta = { allow_inheritance=True}
            ...

`

then in the mongodb recipes collection, the documents will looks like this:

{
    “_id” : ObjectId(“5507e64253e95b34b4ce166b”),
    “_cls” : “PeriodicTask”,
    “name” : “task1”,
    “task” : “celerybeatmongo.add”,
    “crontab” : {
        “_cls” : “Crontab”,
        “minute” : “*/1”,
        “hour” : “*”,
        “day_of_week” : “*”,
        “day_of_month” : “*”,
        “month_of_year” : “*”
        },
    “args” : [ 1, 0 ],
    “kwargs” : {  },
    “exchange” : “celery”,
    “routing_key” : “celery”,
    “enabled” : true,
    “run_immediately” : false,
    “total_run_count” : 5,
    “last_run_at” : ISODate(“2015-03-17T08:35:00.015Z”)
}

it will contain a `_cls` field to relate the specific class.


