---
title: Scheduled Tasks
---

[[toc]]

Skygear supports creating functions that run at specific time intervals using
the method `skygearCloud.every()`. It works similar to the
[cron daemon][cron-wiki] in UNIX.

```javascript
// Cron job that runs every 2 minutes
skygearCloud.every('@every 2m', function() {
	// show 'Meow Meow!' every 2 minutes in Skygear Portal Production Log
	console.log('Meow Meow!');
});
```

The method takes one argument, the `interval`, which specifies the
time at which the function should be invoked and any tasks should be pass with a function in second argument.

Some examples of `interval` values are shown below. For detailed
specifications, you can refer to the [cron library for go][robfig-cron-doc],
which is the parser used in Skygear.

```javascript
skygearCloud.every('1 2 3 4 5 *') # Every 4th May at 03:02:01
skygearCloud.every('0 0 0 * * TUE') # Every Tuesday at 00:00:00
skygearCloud.every('0 15 */2 * * *') # Every 2 hours at 15th minute
skygearCloud.every('0 0 6,18 * * WED,SUN') # On 6am and 6pm of every Wed/Sun
skygearCloud.every('0 16-30 * * * *') # At 16-30 minute of every hour
skygearCloud.every('@daily') # Daily at 0:00:00
skygearCloud.every('@every 1h') # At 1-hour intervals since the server starts
skygearCloud.every('@every 1m30s') # At 90-second intervals since the server starts
```

::: note

**Note:**

1. If there are multiple functions to run at the same second, their execution order
   is not guaranteed.
2. For intervals using `@every` (the last two examples above),
   the interval will be reset upon the re-deployment of cloud code.
   For instance, an existing scheduled task running at 1-hour interval
   `skygearCloud.every('every 1h')` will be run 1 hour after the re-deployment,
   meaning that the actual interval will be longer than 1 hour due to
   the re-deployment.

:::

[cron-wiki]: https://en.wikipedia.org/wiki/Cron
[robfig-cron-doc]: https://github.com/robfig/cron/blob/master/doc.go
