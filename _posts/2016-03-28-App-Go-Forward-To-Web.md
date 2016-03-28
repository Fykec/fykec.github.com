---
layout: post
---

{{ 客户端的前端化 之 异步数据管理 }}
================


App的开发，生长于移动互联网时代。手机的无处不在，给了人们访问网络的便捷性，同时也给了开发者并发的挑战。手机虽然计算能力和存储能力相对于PC受到限制，但人们从未降低对互联网访问可用性的期待。所以这就促使开发者只能更多的把数据处理和页面渲染进行分离。剥离客户端不一定需要做的，尽力让客户端更好地做好它必须要做的事情。所以现在的客户端有一种前端化的趋势。因为数据存储和处理被挪向了后端。客户端要做的事情只剩下了，渲染页面，异步数据管理，这时的客户端架构，在接近web的架构。所以近年来，App上火热的技术，有些是跟随Web技术的发展趋势的。比如[Futures and Promise](https://en.wikipedia.org/wiki/Futures_and_promises)是JavaScript 先火起来，然后传到App。还比如React，响应式编程也是从Web走向了客户端。

未来的App 会更加凸现它**User Interface**这一本质作用。它扮演输出，数据通过它呈现给用户，它是扮演输入，用户的动作，声音，图像，位置等，都会被它接收传递给后端。

在手机上，人们不但是没有降低对互联网可用性的期待，反而是提高了期待。人们在CS（Client-Server）时代和Web时代（Browser-Server）享受过的红利，在Mobile时代，人们依然要求去享受，而且要的更多。所以人们需要App像CS时代一样有native应用该有的易用性，也需要像Web时代一样，拥有网页的及时更新的能力。

因此这给当下客户端架构，提出以下几点要求

1. 复杂异步数据管理的能力

虽然App中可以没有后端处理，但是App中是不能没有数据，所以必须要有异步通信，而且要善于异步通信。

2. 及时自我更新的能力

Web上刷新一下，网页就更新了，App上不行吗？对于特别重业务的App，这在某种程度上是个刚需，比如天猫，每天都可以有商业推广活动，不更新到App上，这不是阻碍业务推广吗？所以天猫的应用就可以通过数据进行模块化配置。

3. 在线Hotfix的能力

Fix bug，而且是hot的。是程序都会有bug，但是如果有bug，要一两个星期后（比如App Store 的审核）用户才能收到fix，这对于很多App来说是不能容忍的。一个公司同样的业务，出现bug，Web上改一下上线就好了，App改一下，确还要等审核这不是摧残产品和程序员的心灵吗？大产品出了bug，影响太大了，小产品出了bug，不及时fix会影响到生存。所以不管怎样的产品，想做好都是需要考虑hotfix的。


而这三点之中，异步数据管理是基础性的。所以这次我重点讲下异步数据管理。 讨论哪些技术会让App更善于管理异步数据。

善于管理异步数据，其实是要解决以下两个问题

1. 怎么管理更多，更复杂的异步任务

2. 怎么让异步程序更易懂，而不是随着异步任务的增加，程序的可读性快速衰减。


## 管理更多的异步？

对于数据量的快速增加，简单的去写重复代码，一开始还行，但是越往后就越不行，往后会要求进行组织架构的改变，甚至是设计指导思想的改变。

让我们回顾下，异步任务管理技术的几次升级。

### 单点式

最开始的异步，是单点式，一个来源，一个监听方，这对于异步任务很少时，仍然是很简洁的。

这个时候的代表是Delegate模式，或者换一种说法也就是回调的模式。这个时候的代码，是下面这样子, 异步的开始代码和结束代码是分开的。回调少时还是OK的，多了的话，多个回调函数各自的顺序，相互共享变量，都会让代码难以维护。

```

self.connection = [[NSURLConnection alloc] initWithRequest:self.request delegate:self startImmediately:NO];
				[self.connection scheduleInRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
				[self.connection start];
				self.executing = YES;
                
- (void)connection:(NSURLConnection *)connection didCancelAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge {
	@synchronized (self) {
		self.failed = YES;
		self.finished = YES;
		self.executing = NO;
		self.failedAuthentication = YES;
		if (self.delegate && [self.delegate respondsToSelector:@selector(connectionFailed:)]){
			[self.delegate performSelectorOnMainThread:@selector(connectionFailed:) withObject:self waitUntilDone:YES];
		} else {
			ATLogError(@"Orphaned connection. No delegate or nonresponsive delegate.");
		}
	}
}

- (void)connection:(NSURLConnection *)connection didSendBodyData:(NSInteger)bytesWritten totalBytesWritten:(NSInteger)totalBytesWritten totalBytesExpectedToWrite:(NSInteger)totalBytesExpectedToWrite {
	if (self.delegate && [self.delegate respondsToSelector:@selector(connectionDidProgress:)]) {
		self.percentComplete = ((float)totalBytesWritten)/((float) totalBytesExpectedToWrite);
		[self.delegate performSelectorOnMainThread:@selector(connectionDidProgress:) withObject:self waitUntilDone:YES];
	} else {
		ATLogError(@"Orphaned connection. No delegate or nonresponsive delegate.");
	}
}
```

### 多点式

随着异步任务的增多，要同时开启多个request是很常见的需求，怎样做到，让代码写起来简洁，没有重复的request代码，同时又能权衡CPU性能和内存占用呢？ 

答案是**queue**。把所有的异步任务都放在一个queue中，由queue去管理任务的启动和结束，调用的代码依然只关心传入什么，回调函数中改写什么就可以。
在iOS这方面来说[GCD](http://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html)就是一个很优秀的实现， 调用时候就很直接，比如：

```
double delayInSeconds = 2.0;
                    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds * NSEC_PER_SEC));
                    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
                        if (self.activityViewController) {
                            [self showProgressHUDWithMessage:nil];
                        }
                    });
```

只需要指定queue和相关参数，写好callback代码就可以了。


### 链式，流式

无论前面的单点式，还是多点式，都是处理一层的异步，如果遇到异步任务的嵌套呢？即一个异步任务的开始依赖于另一个异步任务的结束。 对于现在这种App前端化的趋势。异步之间的嵌套是很正常的。写嵌套的层次多了，就会形成讨厌的[回调金字塔(Pyramid of Doom)](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming))，如果不使用block，直接用deleaget来实现嵌套，那么就更不好，极易出现bug。像这种情况下，去拼命的优化代码长度，代码位置，函数名称等，效果是有限的。即使代码被你整的很清晰了，那也免不了花去很多精力。所以这时候，是改变思路的时候了。

#### 怎样去嵌套呢？

嵌套，怎么更清晰呢？最直接的办法，是把代码铺平。怎么铺平？统一回调函数的接口才能铺平，有了统一的接口，才能方便任务间的结合。

拿[Bolts](https://github.com/BoltsFramework/Bolts-iOS)举个例子, 而最初的思想来源.NET中的[Task Parallel Library](https://en.wikipedia.org/wiki/Parallel_Extensions#Task_Parallel_Library)

这个库的核心是Task， 一个Task会把一个异步任务的所有相关的部分都包装在一起。任务执行的代码，任务执行后的结果，任务执行过程中遇到的错误，任务的取消逻辑，都在Task中。

链式形式如下

```
- (BFTask *)test {
    return [[[self method:@"GET" URLString:@"http://www.baidu.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil] continueWithBlock:^id(BFTask *task) {
        if (task.error) {
        }
        else if (task.exception) {
        }
        else if (task.isCancelled) {
        }
        else {
            //handle the result
        }
        return [self method:@"GET" URLString:@"http://www.hao123.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    }] continueWithBlock:^id(BFTask *task) {
        if (task.error) {
        }
        else if (task.exception) {
        }
        else if (task.isCancelled) {
        }
        else {
            //handle the result
        }
        
        return task;
    }];
}
```

所有回调中需要考虑到的，成功，失败，取消处理，而这些都被统一到了一个BFTask对象中，本来需要把第二个request的处理代码嵌套写在第一个request的回调block中的，通过统一的接口 **continueWithBlock** 返回一个BFTask 对象直接传到下一个处理block了，每个block只需要关心自己面对的BFTask对象既可以，这样就避免了嵌套。而嵌套的移除，自然就形成了链式结构。

异步任务，除了需要嵌套，还需要合并和转换等，当然这些对于已经形成统一接口的链式结构都不在话下。

比如合并的例子如下，外部调用者看到的也只是一个Task

```
- (BFTask *)test2 {
    BFTask *task1 = [self method:@"GET" URLString:@"http://www.baidu.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    BFTask *task2 = [self method:@"GET" URLString:@"http://www.hao123.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    return [BFTask taskForCompletionOfAllTasks:@[task1, task2]];
}
```


## 让异步程序看起来易懂

易懂的关键点在直观，最直观的方法是顺着人类的思维，让异步代码看起来像是同步的代码，让代码的文本顺序就是它的执行顺序，这样人就能一下子看懂。

这其中有两个技术功劳很大

### Block 闭包技术

让一个异步任务自包含，回调函数和开启任务的代码在一起。 回调函数和开启任务在一起，不仅直观了，而且也方便了任务内变量的共享，这样可以把需要共享的变量，从类里面的全局变量，转化成方法里面的局部变量。


### 链式结构

除了Block。还有一个就是链式结构。 如果说Block是让每个任务单元自包含了，那么链式结构就是让任务之间相连，方便形成串联，并联，等复杂的组合。也因为能连在一起，所以链式结构间也方便进行任务间的变量的共享，任务间的变量也可以从全局的变量，转化到任务间的参数。


## 小结

总的来说，异步数据管理，是当下App一个必备的重要能力。某种程度上说，App的开发能否跟上未来数据科学的发展，扮演好自己作为用户端口的角色，起决定性因素的就是异步数据管理能力的强与否。


## 扩展阅读

[天猫App的动态化配置中心实践](http://www.tuicool.com/articles/ZVjqea3)
[理解 Promise 的工作原理](https://blog.coding.net/blog/how-do-promises-work)
