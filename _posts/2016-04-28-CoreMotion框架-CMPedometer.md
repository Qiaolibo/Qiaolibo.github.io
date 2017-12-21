---

layout: post
title: "CoreMotion框架-CMPedometer"
author: "乔黎博"
categories: Coding
date: 2016-4-28 9:27:00
tags: [CoreMotion, iOS]

---

比赛作品涉及计步功能，计步属于健康范畴，当前Apple的HealthKit可以很好的提供健康数据，但是由于涉及隐私，在添加HealthKit时必须要付费的开发者账号，而且包含HealthKit的App审核也更严格。
我目前只需实现简单的计步功能，因此关注到了CoreMotion框架的CMPedometer类，今天简单说说：

## CMPedometer
这块国内资料较少，科学上网一阵之后发现 [官方文档](https://developer.apple.com/library/ios/documentation/CoreMotion/Reference/CMPedometer_class/index.html)
已经足够清晰明了。顾名思义，该类为了方便获取行走相关数据而设计，除了可用性检查几个关键方法：

### 生成计步器实时数据
``` swift
//Swift
func startPedometerUpdatesFromDate(_ start: NSDate,
                       withHandler handler: CMPedometerHandler)
```
- **start**：NSDate类型，指明开始获取数据的时间，不能为空
- **handler**：传入一个block，每当新数据可用时，这个block在后台线程（异步后台串行队列）重复执行，不能为空
- App暂停时，计步数据暂停生成，返回前台或后台执行时，计步数据继续生成
- 可用`stopPedometerUpdates`停止更新

### 获取历史计步数据
``` swift
//Swift
func queryPedometerDataFromDate(_ start: NSDate,
                             toDate end: NSDate,
                    withHandler handler: CMPedometerHandler)
```
- **start**：NSDate类型，指明开始获取数据的时间，不能为空
- **end**：NSDate类型，指明结束获取数据的时间，不能为空
- **handler**：传入一个block，这个block在后台线程（异步后台串行队列，和实时更新方法在同一队列）执行一次，不能为空
- 最多只能获取过去7天的计步数据，如果start参数设置比7天前还早，就只返回可用数据，所以要是做长期分析可能会用到后台服务器

## CMPedometerHandler
上面提到的block
``` swift
//Swift
typealias CMPedometerHandler = (CMPedometerData?, NSError?) -> Void
```
步数主要在**CMPedometerData**类的**numberOfSteps**属性里取

## 坑
昨天做了个测试，踩进坑里倒腾了一下午，总结一下：
1. **CMPedometer最好声明为一个属性！**我开始放在viewDidLoad中，声明的临时变量，结果始终取不到数据，报错`Error Domain=CMErrorDomain Code=103 "(null)"`。原因就是该方法在启动执行完毕后，临时变量就释放了。对于这样一个持续获取数据的类来说，自然在启动后获取不到数据，这里利用打印block中的error参数可以帮助找到问题
2. 实时更新计步数据的方法异步运行在后台串行队列里，因此如果需要实现界面也跟着实时变化，需要异步回主线程更新UI（GCD很便捷），不然只有在停止数据更新时界面才会变

最后，stackoverflow要多用！
