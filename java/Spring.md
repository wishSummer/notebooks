# Spring
## @Scheduled
	- fixedDelay 表示在上一次任务完成后，延迟一段时间再执行下一次任务。这个延迟时间可以通过 initialDelay 属性设置。如果未设置 initialDelay，任务将在应用启动后立即执行，并且之后根据 fixedDelay 的设置周期性执行。
	- fixedRate 表示在上一次任务开始执行后，一段时间间隔就执行下一次任务。类似于 fixedDelay，initialDelay 属性可以用来在应用启动后延迟执行第一次任务。应用启动后不会立即执行，需搭配initialDelay。
	- cron 表达式是一种更加复杂的调度方式，它是基于日历的，可以指定非常灵活的定时规则。应用启动后不会立即执行。
	- initialDelay 表示在应用启动后延迟多少时间才执行第一次任务。该属性可以与 fixedDelay 和 fixedRate 一起使用。但不能够和cron一起使用。