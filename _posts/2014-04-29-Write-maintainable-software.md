---
layout: post
---

创建可维护的代码
================

1. 尽量少的重复代码， 同样的逻辑的代码包装到函数中去， 一个函数不应该太长，（Fix bug或者更改逻辑时更方便）


2. 删除不用的代码，避免干扰（optional）

3. 功能模块化，减少不同功能之间的耦合，单独的功能需要用借口包装好， 比如信用卡模块，登录模块

4. 尽量用显式逻辑替代隐式逻辑，比如一些重要的必须要设置的属性 可以强制用代码设为为no，而不用依赖默认值，还有避免hard code一些特殊字段， hard code 的index  需要用宏来定义，宏的名字本身就是对这些数字的解释

5. MVC 分层要明显，耦合不能太紧，比如view中不应该直接通过request 得到json，尽量只把需要用的属性给view，而不是整个数据，这样可以清晰知道哪些是被view用到的，更改时会更方便

6. 类的命名要确义，如果功能改了名字也需要更新，否则后来找相关文件会很阻碍， 尽量不要用缩写，如果缩写那么所有地方都必须缩写

7. 方法的命名规范要统一，
getXX 表示获取， addXX 表示添加，removeXX 表示移除 updateXX 表示更新，像其他同义词尽量不要一起用了，比如 fetch， increase, delete, refresh

8. 一个类不要太大，比如view 类，子view 太多，要考虑写成多个单独的view，或者用category 分开到不同文件中

**别人的表述：[Writing Maintainable Perl](http://modernperlbooks.com/books/modern_perl_2014/08-perl-style-efficiency.html)**