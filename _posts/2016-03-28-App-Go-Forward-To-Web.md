---
layout: post
---

{{ 客户端的前端化 之 异步数据管理 }}
================



App的开发，生长于移动互联网时代。手机的无处不在，给了人们访问网络的便捷性，同时也给了开发者并发的挑战。手机虽然计算能力和存储能力相对于PC受到限制，但人们从未降低对互联网访问可用性的期待。所以这就促使开发者把数据处理和页面渲染进行分离。
尽力让客户端只做必须要做的事情。所以现在的客户端有一种前端化的趋势。数据存储和处理被挪向了后端。客户端要做的事情只剩下了，渲染页面，异步数据管理，这时客户端，在接近web。所以近年来，App上火热的技术，有些是跟随Web技术的发展的。
比如iOS上Auto Layout技术，就类似于HTML加CSS的架构，去让页面结构和样式分开，还有[Futures and Promise](https://en.wikipedia.org/wiki/Futures_and_promises)是javascript 先火起来，但是现在App上也有很多Promise概念的库，让你去管理异步。还比如React也是从Web走向了客户端。

未来的App 会更加凸现它user interface的本质作用。它是一个输出，数据通过它呈现给用户，它是一个输入，用户的动作，声音，图像，位置等，都会被它接收传递给后端。这也意味有两大能力会成为App的刚需，第一就是异步数据管理。
第二就是UI的易用性

今天我想先谈一下第一个，异步数据管理。话接上文，在手机上，其实人们不但是没有降低对互联网可以用性的期待，反而是提高了期待。人们在CS（Client-Server）时代和Web时代（Browser-Server）享受过的红利，在mobile时代是依然要求去享受，而且要的更多。

所以这给其实给当下客户端架构，提出一下几个要求

1. 复杂异步数据管理的能力

虽然App中没有了后端，但是App中不能没有数据，所以必须要有异步通信，而且要善于异步通信。

2. App可以通过数据进行模块化配置的能力

更新，Web上刷新一下，网页就更新了，App上不行吗？对于特别重业务的App，这在某种程度上是个刚需，比如淘宝，每天都可以有活动，不更新到App上，这不是阻碍业务推广吗？

3. 在线Hotfix的能力

Fix bug，而且是hot。是程序都会有bug，但是如果有bug，要一两个星期后（比如App Store 的审核）用户才能收到fix，这对于很多App来说是不能容忍的。一个公司同样的业务，除了bug，Web上改一下上线就好了，App改一下，还要等审核
这不是摧残产品和程序员的心灵吗？大产品出了bug，影响太大了，小产品出了bug，不及时fix会影响到生存。所以不管大小，想做好，都是需要hotfix的。幸运的是，大家的愿望是一样的，也有人做出来了。比如JSPatch


下面我重点讲下异步通信管理。哪些技术会让App更善于管理异步通信。

善于管理异步数据，其实是要解决一下两个问题

1. 怎么管理更多的异步数据

2. 怎么让异步程序更易懂，而不会随着异步的增加，程序的可读性快速衰减。


## 管理更多的异步？
这其实要求组织架构的改变，甚至的组织知道思想的改变才能做到的。为什么这么说呢？

### 单点式

最开始的异步，是单点式，一个来源，一个监听方，这对于异步任务很少时，很简洁。

这个时候的代表是Delegate模式，代理模式，或者换一种说法也就是回调函数。这个时候的代码，是下面这样子, 异步的开始代码和结束代码是分开的。回调少时还是OK的，多了的话，多个回调函数各自的顺序，相互share变量，都是复杂度。

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

随着异步数组增多，要同时开启多个request是很常见的需求，怎样做到，让代码写起来简洁，没有重复的request代码，同时能权衡CPU性能和内存占用呢？ 答案是queue。把所有的异步任务都放在一个queue中，由queue去管理任务的启动和结束就可以了，调用的代码依然只关心传入什么，回调函数中改写什么就可以。
对iOS这方面来说GCD是一个很优秀的实现， 调用时候就很直接，比如：

```
double delayInSeconds = 2.0;
                    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds * NSEC_PER_SEC));
                    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
                        if (self.activityViewController) {
                            [self showProgressHUDWithMessage:nil];
                        }
                    });
```

只需要指定queue和参数，写好callback代码就可以了。


### 链式，流式

无论前面的单点式，还是多点式，都是处理一层的异步，如果遇到异步任务的嵌套呢？即一个异步任务依赖与另一个异步任务的结果的情况。 对于现在这种App前端化的趋势。异步之间的嵌套是很正常的。写起来是可以的，但是看起来就麻烦了。在使用block这种闭包技术的前提下，写嵌套就很烧脑，这个就是回调金字塔(Pyramid of Doom)，如果不使用block，直接用deleaget来实现嵌套，难么极易出现bug。
像这种情况下，去拼命的优化代码长度，代码位置，函数名称等，效果是有的，而且还不是很理想，即是代码被你整的很清晰了，但是也花费了很多琐碎的精力。这是时候改变思路了。

#### 怎样去嵌套呢？

统一回调函数的接口，把所有的回调都铺平。

那Bolts这个库来举个例子，BFTask就这样一个神奇的包装，BFTask属于[Bolts](https://github.com/BoltsFramework/Bolts-iOS), 而最初的思想来源.NET中的[Task Parallel Library](https://en.wikipedia.org/wiki/Parallel_Extensions#Task_Parallel_Library)

Task 把一个任务的所有相关的部分都包装在一起，任务执行的代码，任务执行后的结果，任务执行过程中遇到的错误，任务的取消逻辑。

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

所有回调中需要考虑到的，成功，失败，取消处理都被统一到了一个BFTask对象中，本来需要把第二个request的处理代码嵌套写在第一个的回调block中的，通过统一的接口 *continueWithBlock* 处理就给给铺平了。

异步任务，除了需要嵌套，还需要合并和转换等。

比如合并的例子如下，也很简单，调用者看到的只是一个Task

```
- (BFTask *)test2 {
    BFTask *task1 = [self method:@"GET" URLString:@"http://www.baidu.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    BFTask *task2 = [self method:@"GET" URLString:@"http://www.hao123.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    return [BFTask taskForCompletionOfAllTasks:@[task1, task2]];
}
```

异步和同步并存的合并

```
- (BFTask *)test3 {
    BFTask *task1 = [BFTask taskWithResult:@"result 1"];
    BFTask *task2 = [self method:@"GET" URLString:@"http://www.hao123.com" parameters:nil resultClass:nil resultKeyPath:nil cancellationToken:nil];
    return [BFTask taskForCompletionOfAllTasks:@[task1, task2]];
}

```


完成了管理更多的任务，怎么让异步程序看起来易懂呢？

关键点是直观，直观的方法是顺着人类的思维，让异步开起来像是同步的，让代码执行的顺序接近代码出现顺序，这样人就能一下子看懂。

这其中有两个技术功劳很大

### Block 闭包技术

让回调函数和开启任务的代码在一起。大家感受下，同样需要用NSURLConnection的代码，block版本如下

```
//show progress hud
    [SVProgressHUD show];
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue currentQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {

        if (!connectionError && data && [data length] > 0)
        {
            NSError *jsonError = nil;
            NSDictionary *resp = [NSJSONSerialization JSONObjectWithData:data options:0 error:&jsonError];

            if (!jsonError)
            {
                if ([self isResponseReturnOK:resp])
                {
                    if ([self isResponseHaveUpdate:resp])
                    {
                        [SVProgressHUD dismiss];
                        NSDictionary  *updateInfo = [resp objectForKey:@"val"];
                        [self showAlertWithChangeLogs:updateInfo];
                    }
                    else
                    {
                        [SVProgressHUD showSuccessWithStatus:NSLocalizedString(@"You already have the lastest version", nil)];
                    }
                }
                else
                {
                    [SVProgressHUD showErrorWithStatus:[NSString stringWithFormat:@"%@ %@", NSLocalizedString(@"App Update:", nil), [resp objectForKey:@"msg"]]];
                }
            }
            else
            {
                [SVProgressHUD showErrorWithStatus:[NSString stringWithFormat:@"%@ %@", NSLocalizedString(@"App Update:", nil), [[jsonError userInfo] objectForKey:@"NSLocalizedDescription"]]];
            }
        }
    }];
```

回调函数和开启任务在一起，直观是一方面，同时也方便了变量的共享，这样可以把需要共享的变量，从类里面的全局变量，限制成方法里面的局部变量。


除了Block。还有一个就是Futures and Promises，上面提到的BFTask就是其中的一种实现。Promise的作用，也就是把嵌套铺平串联起来，增加可读性

如果说Block是让每个任务单元在一起了，那么，Futures 建立了一个时间纬度上管道（pipeline）， 就像给每个任务去建立了twitter 的timeline一样，这个任务中，你就可以把针对不同的结果在链式结构里把安排的任务预订好。

拿[PromiseKit](https://github.com/mxcl/PromiseKit) 举下例子

```
UIApplication.sharedApplication().networkActivityIndicatorVisible = true

when(fetchImage(), getLocation()).then { image, location in
    self.imageView.image = image;
    self.label.text = "Buy your cat a house in \(location)"
}.always {
    UIApplication.sharedApplication().networkActivityIndicatorVisible = false
}.error { error in
    UIAlertView(…).show()
}
```

PromiseKit 所有方法都是表意的，尽量接近人的思维。when（） 的作用把 fetchImage() 和getLocation()返回结果包装进Promise对象里，then（） 方法就是Promise成功时去处理。always，就是无论成功失败都调用。
error 就是失败时去处理。仔细观察，你发现 then，always 和error，不是像BFTask一样放在一个block中去处理，而是可以一个接着一个，但是，想then 和error其实是二选一，只能有一个可以执行的。所以 then always error其实是 注册函数，注册未来发生是什么情况下，该做什么。
正因为他们是注册函数，而且 then always error 返回的结果都是Promise对象，这样一个Promise的多中回调，和多个Promise多可以很方便串到一起了。

## 小结



异步数据管理就先讲到这里，欢迎交流。



## 参考文章

https://blog.coding.net/blog/how-do-promises-work
