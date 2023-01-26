---
title: NodeJS的定时任务
date: 2023-01-26 14:36:48
tags:
---
# NodeJS的定时任务

### 使用node-schedule库

```jsx
npm install node-schedule
```

Node Schedule 中的每个定时任务都由一个`Job`对象表示。可以手动创建，然后执行`schedule()`方法以应用任务，或使用`scheduleJob()`如下。

`Job`对象是`EventEmitter，并发出以下事件：

- `run`每次执行后的事件。
- `scheduled`每次计划运行时的事件。
- 一`canceled`，当它在执行之前调用被取消的事件。
- 一个`error`当被触发调度作业调用抛出或退出事件拒绝`Promise`。

（`scheduled`和`canceled`事件都接收一个 JavaScript 日期对象作为参数）。 注意的是，任务是第一次立即执行的，因此如果使用`scheduleJob()`方法创建任务，将错过第一个`scheduled` 事件触发，但您可以手动查询调用。

### **Cron格式**

```jsx
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    │
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)

每分钟的第30秒触发： '30 * * * * *'

每小时的1分30秒触发 ：'30 1 * * * *'

每天的凌晨1点1分30秒触发 ：'30 1 1 * * *'

每月的1日1点1分30秒触发 ：'30 1 1 1 * *'

2016年的1月1日1点1分30秒触发 ：'30 1 1 1 2016 *'

每周1的1点1分30秒触发 ：'30 1 1 * * 1'
```

🤨**这里是个人对node-schedule封装类来使用增删改查定时任务**

```jsx
const schedule = require('node-schedule');

exports.Interval = class Interval {
  constructor({ unit_name, maintain_time, last_alarm }) {
    this.unit_name = unit_name          // 任务名字
    this.maintain_time = maintain_time  // 定时时间
    this.last_alarm = last_alarm || ""        // 上一次定时任务名字
  }

  // 生成新的定时任务
  async create(callback) {
    // 终止之前的定时任务
    if (this.last_alarm !== "") {
      this.delete(this.last_alarm)
    }
    schedule.scheduleJob(`${this.unit_name}`, `${this.maintain_time}`, callback);
  }

  // 删除定时任务
  delete() {
    if (schedule.scheduledJobs[this.unit_name]) {
      schedule.scheduledJobs[this.unit_name].cancel();
      return true
    }
    return false
  }

  // 找到一个定时任务
  findOne(name) {
    if (schedule.scheduledJobs[name]) {
      return schedule.scheduledJobs[name]
    } else {
      throw new Error("未找到任务名")
    }
  }

  // 查看所有的定时任务
  findAll() {
    return schedule.scheduledJobs
  }
}
```

### **这里是调用时定时任务Interval实例**

```jsx
// 定时任务
new Util.Interval({
  unit_name: '自动分发任务 0 0 12 * * *',
  maintain_time: '0 0 12 * * *',
  last_alarm: '自动分发任务 0 0 12 * * *'
}).create(async () => {
  // 写入你自己想在定时任务触发的时候，想要执行的函数
})
```