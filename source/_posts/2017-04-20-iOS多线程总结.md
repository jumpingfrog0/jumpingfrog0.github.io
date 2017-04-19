iOS 多线程总结
=========

## GCD

GCD是纯C语言的。

串行队列（`DISPATCH_QUEUE_SERIAL`）：一次只能执行一个任务，队列中的任务按顺序执行，一个接一个，像排队跑步一样，保持队形
并行队列（`DISPATCH_QUEUE_CONCURRENT`）：可以同时执行多个任务，像并排跑，类似于赛跑

同步执行（sync）：不具备开启新线程的能力，任务创建后就要执行完才能继续往下走
异步执行（async）：具备开启新线程的能力，任务创建后可以先绕过，回头再执行

```
//创建串行队列
dispatch_queue_t serial_queue = dispatch_queue_create("my_serial_queue", DISPATCH_QUEUE_SERIAL);
NSLog(@"%@",serial_queue.description);
//添加任务
dispatch_async(serial_queue, ^{
    NSLog(@"%@",[NSThread currentThread]);
    for (int i=0; i<3; i++) {
        NSLog(@"E:%d",i);
        sleep(1);
    }
});
```

```
//创建一个并发队列
dispatch_queue_t concurrent_queue = dispatch_queue_create("my_concurrent_queue", DISPATCH_QUEUE_CONCURRENT);
//添加并发任务
dispatch_async(concurrent_queue, ^{
    NSLog(@"%@",[NSThread currentThread]);
    for (int i=0; i<3; i++) {
        NSLog(@"G:%d",i);
        sleep(1);
    }
});
```

### 队列和任务组合

| | 并行队列 | 串行队列 | 主队列 |
|:--: | :--: | :--: | :--: |
| 异步执行 | 开启多个新线程，任务同时执行 | 开启一个新线程，任务顺序执行 | 不开启新线程，任务顺序执行 |
| 同步执行 | 不开启新线程，任务顺序执行 | 不开启新线程，任务顺序执行 | 死锁 |

### 同步任务嵌套

| | 并行队列 | 串行队列 |
|:--: | :--: | :--: | :--: |
| 同步嵌套异步 | 开启新线程，不阻塞 | 开启新线程，不阻塞 |
| 同步嵌套同步 | 不开启新线程，不阻塞 | 阻塞 |

### GCD常见用法和应用场景

#### Main Dispatch Queue

主队列，是串行队列，一般用于更新界面，跟NSObject类的performSeletorOnMainThread: 方法有相同的作用

```
//获取主队列
dispatch_queue_t main_queue = dispatch_get_main_queue();
```

#### Global Dispatch Queue

全局队列，是并行队列，所有的应用程序都可以使用该并行队列，没必要自已创建并行队列，只需要获取该队列即可

```
//获取全局队列
dispatch_queue_t global_queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

#### dispatch\_set\_target\_queue

设置一个队列的优先级。

手动创建的队列，无论是串行队列还是并发队列，都跟默认优先级的全局并发队列具有相同的优先级。

* `DISPATCH_QUEUE_PRIORITY_HIGH`：高
* `DISPATCH_QUEUE_PRIORITY_DEFAULT`： 默认
* `DISPATCH_QUEUE_PRIORITY_LOW`：低
* `DISPATCH_QUEUE_PRIORITY_BACKGROUND`：后台

#### dispatch_after

延迟执行一个任务，在x秒后把任务追加到队列中，并不是在x秒后执行。

```
//延迟多少秒
double delaySeconds=5;
//创建时间
dispatch_time_t delay_time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delaySeconds * NSEC_PER_SEC));
//执行延迟任务
dispatch_after(delay_time, dispatch_get_main_queue(), ^{
    NSLog(@"hello world");
});
```

#### dispatch\_group & dispatch\_group_notify

调度组，可以把多个队列、多个任务都放到同一个group里面，当group的所有任务都完成了，Dispatch可以异步或者同步通知你。

有些时候，我们会有这样子的需求，执行多种操作，当这些操作都完成的时候，再更新UI，比如多图片下载。如果没有group，你就需要自已去统计哪个操作完成了，但有了 `dispatch_group` 一切将简化。

```
//获取并发队列
    dispatch_queue_t global_queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    //创建调度组
    dispatch_group_t one_group = dispatch_group_create();
    //通过调度组发起并发任务
    dispatch_group_async(one_group, global_queue, ^{
        for (int i=0; i<3; i++) {
            NSLog(@"I:%d",i);
            sleep(1);
        }
    });
    dispatch_group_async(one_group, global_queue, ^{
        for (int i=0; i<4; i++) {
            NSLog(@"J:%d",i);
            sleep(1);
        }
    });
    //调度组完成任务通知
    dispatch_group_notify(one_group, dispatch_get_main_queue(), ^{
        NSLog(@"one_group完成任务");
    });
}
```

#### dispatch\_group\_enter & dispatch\_group\_leave

进入和离开调度组。

#### dispatch\_group\_wait

等待调度组完成所有任务。

如果group的所有任务都执行完毕，它会返回 0 ，如果超时，则会返回一个不为 0 的long类型的值。

`dispatch_group_wait` 第一个参数为需要等待的目标调度组，第二个参数则是等待的时间，`DISPATCH_TIME_FOREVER`，表示永远等待。

```
- (void)gcdWait {
    //获取并发队列
    dispatch_queue_t global_queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    //创建调度组
    dispatch_group_t one_group = dispatch_group_create();
    //进入调度组
    dispatch_group_enter(one_group);
    //发起并发任务
    dispatch_async(global_queue, ^{
        NSLog(@"block 1");
    });
    
    dispatch_async(global_queue, ^{
        NSLog(@"block 2");
    });
    dispatch_async(global_queue, ^{
        [NSThread sleepForTimeInterval:5];
        NSLog(@"block 3");
        //离开调度组
        dispatch_group_leave(one_group);
    });
    
    //调度组等待:等待过程中，下面的代码不会被执行
    double delaySeconds = 4;
    dispatch_time_t delay_time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delaySeconds * NSEC_PER_SEC));
//    long result = dispatch_group_wait(one_group, DISPATCH_TIME_FOREVER);
    long result = dispatch_group_wait(one_group, delay_time);
    if (result == 0) {
        NSLog(@"等到了!");
    }
    NSLog(@"等待超时或结束后执行");
}
```

如果等待时间是 `DISPATCH_TIME_FOREVER` ，最后两句需要5秒，才可以执行完毕，执行结果是：

```
block 1
block 2
block 3
等到了!
等待超时或结束后执行
```

如果把等待时间改为4秒，则第3个block需要5秒才能执行完毕，所以会超时，执行结果是：

```
block 1
block 2
等待超时或结束后执行
block 3
```

#### dispatch\_barrier\_async

栅栏函数，它就好像栅栏一样可以将多个任务分隔开，在它前面追加的任务先执行，然后执行它自己的任务，然后在它后面追加的任务才执行。

> 注意：`dispatch_barrier_async` 中传入的参数队列必须是由 `dispatch_queue_create` 方法创建的队列，否则，与 `dispatch_async` 无异，起不到“栅栏”的作用了，对于 `dispatch_barrier_sync` 也是同理。

```
dispatch_queue_t queue = dispatch_queue_create("name", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{NSLog(@"1");});
dispatch_async(queue, ^{NSLog(@"2");});
dispatch_async(queue, ^{NSLog(@"3");});
dispatch_barrier_async(queue, ^{NSLog(@"barrier");});
dispatch_async(queue, ^{ NSLog(@"4");});
dispatch_async(queue, ^{NSLog(@"5");});
```

执行结果：

```
2
1
3
barrier
4
5
```

#### dispatch\_barrier\_sync

作用于 `dispatch_barrier_async` 一样

不同点：

* `dispatch_barrier_sync` 将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们。
* `dispatch_barrier_async` 将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。

#### dispatch_apply

重复执行同一个任务。

```
dispatch_async(queue, ^{
    dispatch_apply(3, queue, ^(size_t index) {
        NSLog(@"%@",index);
    });
});
```

执行结果：

```
0
1
2
```

#### dispatch_one

保证在App生命周期内只执行一次任务且线程安全，比如单例模式。

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    // 只执行一次的任务
    ...
});
```

#### dispatch\_suspend & dispatch\_resume

`dispatch_suspend` : 挂起指定的队列
`dispatch_resume` : 恢复指定队列

```
dispatch_queue_t queue = dispatch_get_main_queue();
dispatch_suspend(queue); //暂停队列queue
dispatch_resume(queue);  //恢复队列queue
```

#### dispatch\_semaphore\_signal

信号量，如果信号量计数大于或等于1，那个允许线程执行，如果信号量当前计数为0，线程将阻塞。

创建信号量

```
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
```

发出等待信号（信号量计数减1）

```
dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
```

发出通行信号 （信号量计数加1）

```
dispatch_semaphore_signal(_semaphore);
```

```
// 其实这个没研究懂 =_=||
// 传递的参数是信号量最初值,下面例子的信号量最初值是1
dispatch_semaphore_t signal = dispatch_semaphore_create(1);
    
dispatch_time_t overTime = dispatch_time(DISPATCH_TIME_NOW, 5 * NSEC_PER_SEC);
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    
    // 当信号量是0的时候,dispatch_semaphore_wait(signal, overTime);这句代码会一直等待直到overTime超时.
    //这里信号量是1 所以不会在这里发生等待.
    dispatch_semaphore_wait(signal, overTime);
    NSLog(@"需要线程同步的操作1 开始");
    sleep(2);
    NSLog(@"需要线程同步的操作1 结束");
    long signalValue = dispatch_semaphore_signal(signal);//这句代码会使信号值 增加1
    //并且会唤醒一个线程去开始继续工作,如果唤醒成功,那么返回一个非零的数,如果没有唤醒,那么返回 0
    
    NSLog(@"%ld",signalValue);
});
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    sleep(1);
    dispatch_semaphore_wait(signal, overTime);
    NSLog(@"需要线程同步的操作2");
    dispatch_semaphore_signal(signal);
    long signalValue = dispatch_semaphore_signal(signal);
    
    NSLog(@"%ld",signalValue);
});
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    sleep(1);
    dispatch_semaphore_wait(signal, overTime);
    NSLog(@"需要线程同步的操作3");
    dispatch_semaphore_signal(signal);
    long signalValue = dispatch_semaphore_signal(signal);
    
    NSLog(@"%ld",signalValue);
});
```

## NSOperation

[iOS多线程之NSOperation](http://www.jianshu.com/p/c6650fcc6612)

[NSOperation 和 NSOperationQueue](http://www.jianshu.com/p/e97bab6d6520)

[Concurrency Programming Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW8)

`NSOperation` 不可以直接创建，其实用的比较多的是 `NSOperationQueue` 和 它的两个子类 `NSBlockOperation`  和 `NSInvocationOperation`（Swift 不支持）。

### NSOperationQueue

`NSOperationQueue` 底层是基于 GCD 的封装，queue 会根据 operation 的优先级、依赖等来决定如何执行添加进来的操作。所有的自定义队列，都是在子线程中执行的。

创建一个线程队列

```
NSOperationQueue *queue = [[NSOperationQueue alloc]init];
queue.maxConcurrentOperationCount = 2; // 设置最大并发数（同时并发的最大线程数）
```

获取main队列

```
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
```

### NSBlockOperation

`NSBlockOperation` 是 `NSOperation` 的子类，用于管理一个或多个 block 的**并发执行**，会开启一个或多个线程，这些线程都由 queue 自己管理。可以通过设置依赖来控制执行顺序。

```
NSOperationQueue *queue = [[NSOperationQueue alloc]init];

NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 %@",[NSThread currentThread]);
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"修饰图片 %@",[NSThread currentThread]);
}];
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"保存图片 %@",[NSThread currentThread]);
}];
NSBlockOperation *op4 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"更新UI %@",[NSThread currentThread]);
}];
    
// 设置执行顺序（依赖）
[op2 addDependency:op1];
[op3 addDependency:op2];
[op4 addDependency:op3];
    
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];

// 所有UI的更新需要在主线程上进行
[[NSOperationQueue mainQueue] addOperation:op4];
```

 执行结果：
 
```
2017-04-20 02:25:56.693 02-NSOperation[73278:10188500] 下载图片 <NSThread: 0x6180000619c0>{number = 3, name = (null)}
2017-04-20 02:25:56.693 02-NSOperation[73278:10188502] 修饰图片 <NSThread: 0x610000063b40>{number = 4, name = (null)}
2017-04-20 02:25:56.693 02-NSOperation[73278:10188500] 保存图片 <NSThread: 0x6180000619c0>{number = 3, name = (null)}
2017-04-20 02:25:56.697 02-NSOperation[73278:10188413] 更新UI <NSThread: 0x608000062c40>{number = 1, name = main}
```

### NSInvocationOperation

`NSInvocationOperation` 是 NSOperation 的子类，用于管理调用单个封装任务的执行，此类实现**非并发操作**。

```
- (void)performHelloWorld:(id)obj
{
    NSLog(@"%@ - %@",[NSThread currentThread], obj);
}

- (void)test
	// 需要定义一个方法，接收一个参数，使用起来不够灵活
	NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(performHelloWorld:) object:@"hello world"];
	[[NSOperationQueue mainQueue] addOperation:op];
}
```

### 进阶

NSOperation 有四种状态：

* `isReady`: 表示操作是否已经准备好被执行。
* `isExecuting`: 表示操作是否正在执行。
* `isFinished`: 表示操作是否执行成功了。
* `isCancelled`: 表示操作是否被取消了。

![状态](http://upload-images.jianshu.io/upload_images/131122-255e56addb4ada84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### cancel

NSOperation 的 `- (void)cancel` 方法作用是：建议操作对象停止执行任务。

此方法不会强制停止操作，而是更新对象的内部标志以反映状态的变化，但取消尚未执行的操作可以使该操作更早地从队列中删除。

* 如果操作已经完成，则此方法无效。
* 如果某个操作处于队列中，但正在等待未完成的依赖操作，则会从队列中删除该操作。
* 如果取消不在队列中的操作，则此方法会立即将对象标记为已完成。

#### CompletionBlock

每个 NSOperation 执行完毕之后，就会执行该block，提供了一个非常好的方式让你能在 View Controller 或者 Model里加入自己更多自己的代码逻辑。比如说，你可以在一个网络请求操作的completionBlock来处理操作执行完以后从服务器下载下来的数据。

```
NSOperationQueue *queue = [NSOperationQueue mainQueue];
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"执行操作");
}];
[operation setCompletionBlock:^{
    NSLog(@"执行操作完成");
}];
[queue addOperation:operation];
```

#### 手动调用 operation

待补充...

#### 自定义同步的 operation

待补充...

#### 自定义异步的 operation

待补充...

## NSThread

NSThread 是基于线程使用的、轻量级的多线程编程方法（相对GCD和NSOperation），一个NSThread对象代表一个线程，需要手动管理线程的生命周期，处理线程同步等问题。一般不推荐使用。

有两种创建方式：

```
NSThread * newThread = [[NSThread alloc]initWithTarget:self selector:@selector(threadRun) object:nil];
```

```
[NSThread detachNewThreadSelector:@selector(threadRun) toTarget:self withObject:nil];
```

获取当前线程

```
[NSThread currentThread];
```

获取主线程

```
[NSThread mainThread];
```

### 线程操作

#### 开启线程

```
[newThread start];
```

#### 暂停线程

```
// NSThread的暂停会有阻塞当前线程的效果
[NSThread sleepForTimeInterval:1.0];　（以暂停一秒为例）
[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:1.0]];
```

#### 取消线程

```
[newThread cancel];
```

#### 终止线程

```
[[NSThread currentThread] exit]];
```

应该避免调用此方法，因为它不会让你的线程有机会清理在执行过程中分配的任何资源，可能会导致内存问题。

#### 设置优先级

有以下几种优先级：

* NSQualityOfServiceUserInteractive：最高优先级，用于用户交互事件
* NSQualityOfServiceUserInitiated：次高优先级，用于用户需要马上执行的事件
* NSQualityOfServiceDefault：默认优先级，主线程和没有设置优先级的线程都默认为这个优先级
* NSQualityOfServiceUtility：普通优先级，用于普通任务
* NSQualityOfServiceBackground：最低优先级，用于不重要的任务

```
[newThread setQualityOfService: NSQualityOfServiceUserInitiated];
```

#### 执行

```
// 在当前线程中执行
[self performSelector:@selector(threadRun)];
[self performSelector:@selector(threadRun) withObject:nil];
[self performSelector:@selector(threadRun) withObject:nil afterDelay:2.0];

// 在指定的线程中执行
[self performSelector:@selector(threadRun) onThread:newThread withObject:nil waitUntilDone:YES];

// 在后台线程中执行
[self performSelectorInBackground:@selector(threadRun) withObject:nil];

// 在主线程中执行
[self performSelectorOnMainThread:@selector(threadRun) withObject:nil waitUntilDone:YES];
```