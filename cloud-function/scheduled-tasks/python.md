---
title: Scheduled Tasks
---

Skygear supports creating functions that run at specific time intervals using
the `@skygear.every` decorator. It works similar to the
[cron daemon][cron-wiki] in UNIX.

```python
# cron job that runs every 2 minutes
@skygear.every('@every 2m')
def meow_for_food():
    # Skygear Portal Console Log will show 'Meow Meow!' every 2 minutes
    log.info('Meow Meow!')
```

The decorator takes one argument, the `interval`, which specifies the
time at which the function should be invoked.

Some examples of `interval` values are shown below. For detailed
specifications, you can refer to the [cron library for go][robfig-cron-doc],
which is the parser used in Skygear.

```python
@skygear.every('1 2 3 4 5 *') # Every 4th May at 03:02:01
@skygear.every('0 0 0 * * TUE') # Every Tuesday at 00:00:00
@skygear.every('0 15 */2 * * *') # Every 2 hours at 15th minute
@skygear.every('0 0 6,18 * * WED,SUN') # On 6am and 6pm of every Wed/Sun
@skygear.every('0 16-30 * * * *') # At 16-30 minute of every hour
@skygear.every('@daily') # Daily at 0:00:00
@skygear.every('@every 1h') # At 1-hour intervals since the server starts
@skygear.every('@every 1m30s') # At 90-second intervals since the server starts
```

<div class="note">

**Note:**

1. If there are multiple functions to run at the same second, their execution order
   is not guaranteed.
2. For intervals using `@every` (the last two examples above),
   the interval will be reset upon the re-deployment of cloud code.
   For instance, an existing scheduled task running at 1-hour interval
   `@skygear.every('every 1h')` will be run 1 hour after the re-deployment,
   meaning that the actual interval will be longer than 1 hour due to
   the re-deployment.

</div>

[cron-wiki]: https://en.wikipedia.org/wiki/Cron
[robfig-cron-doc]: https://github.com/robfig/cron/blob/master/doc.go
