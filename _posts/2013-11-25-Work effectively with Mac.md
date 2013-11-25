---
layout: post
---

{{ page.title }}
================


##使用Mac高效工作

回顾过去，使用Mac已经快有2年了，从黑mac到mac mini，再大iMac，一步一个提升，越来越喜欢mac了，主要是因为mac结合了桌面操作和命令行的优点，太适合一个开发人员了。


###硬件篇

要高效工作，人必须得舒服，对于我主要调整了三个方面。

####椅子
不要有转轴，要有四个脚的，稳定，这样脊柱不会不稳，不会容易变形，人得气容易沉的下去。

####键盘
我使用微软人体工程学4000键盘，手可以展开，手腕也很舒服。我自己在mac对ms的键盘改键了，用了[keyremap4macbook](https://github.com/tekezo/KeyRemap4MacBook), Ctrl 和Cap对换，更放便在emacs中ctrl的使用，emacs出来时ctrl就在哪个位置，后来才挪到下面来，cmd和alt换了这样方便在mac中使用cmd，也和mac自己的键盘一致

####鼠标
我使用得是立式鼠标，[Minicute 人体工程学鼠标](http://detail.tmall.com/item.htm?spm=a1z10.1.w6174434-4544904258.1.fuv5dy&id=8354698667)，这样手腕和小臂就不会扭了，具体可以看[这些资料](http://www.minicute.cn/product/620.htm)，手腕痛得现象就没有

####补充
但是使用了这些，工作几年来，身体还是有不适的，屁股坐长了很疼，以后会尝试站着工作，胸口中间还是没有完全放松，以后会尝试分体式的人体工程学键盘。工作时间长了眼睛会痛，以后需要工作一段时间就休息，注意休息，提高效率。

###软件篇

软件的使用，有利于提高效率

####桌面软件
1. [emacs.app](http://emacsformacosx.com/) 编辑利器，在加上purcell的[config](https://github.com/purcell/emacs.d),编辑东西很方便，而且由于这个config很大，我把git的editor也设为了emacs，我做了[简单的basic的配置](https://gist.github.com/Fykec/7635697)，专门在git时用,使用时强制加载这个config就好了

		editor = "emacs -nw -q --load \"~/simple.emacs.d/init.el\""
		texteditor = "emacs -nw -q --load \"~/simple.emacs.d/init.el\""
2. [Mou](http://mouapp.com/) 我的markdown编辑器，简单好用.
3. Evernote, Evernote WebClipper插件，记笔记，而且通过clipper，把搜索到的知识点直接保存到evernote很方便
4. [Foxmail](http://foxmail.com.cn/mac/) 很好用的邮件客户端，简洁漂亮
5. Chrome， Firefox
6. Textmate，编辑时用，TextWrangler 远程ftp编辑时用
7. [Gitbox](http://gitboxapp.com/) 大部分情况下我是用命令行，但是这个软件有个功能特别好用，全文搜索历史，输入任何文本，comimit的hash值，就能搜到这个change， 某一行代码，就能知道哪个几个change影响了这行代码，输入文件名和文件夹名，就能知道哪几个change改动这些文件和文件夹，非常适合找问题，找到别人为什么要写这行代码，改什么bug，如果别人提交历史写的够清楚，我甚至可以找到还有哪些相关bug，很方便回溯历史，而且界面很简洁明了，sourcetree太复杂了
8. [Dash](http://kapeli.com/dash) 查文档很方便，而且还关联了stackoverflow，可以直接点进去，同时也有Xcode等集成插件，
9. [Alfred 2](http://www.alfredapp.com/) 快速搜索，打开软件，减少点击就是提高效率，还有很有用的powerpack，目前还没有尝试
10. [Marboo](http://marboo.biz/zh_CN/) 我自己记录的小笔记，开发遇到的小知识点不成文的，我以前都用[wikidpad](http://wikidpad.sourceforge.net/) 记录在了本地wiki上，但是不方便转移，后来我都通过[pandoc](http://johnmacfarlane.net/pandoc/)转换成了markdown格式了，本里用marboo来管理，更新后可以上传到我自己的github上，[my-notes](https://github.com/Fykec/my-notes) 上面有这几年我记录下来的点点滴滴的android，iOS，shell的一些小知识点，目前marboo还不是很完善好用，我以后还会调整，不过用纯文本markdown来记录笔记，迁移起来很方便
11. [AppCleaner](http://appcleanermac.com/) 卸载软件方便，也更彻底写，免得卸载后还留下一些配置文件
12. [Squidman](http://squidman.net/squidman/) 在mac上简单搭建一个代理服务器
13. [Axure RP](http://www.axure.com/) 设计些功能是做原型很有用
14. [Property List Editor](http://bbs.weiphone.com/read-htm-tid-122049.html) 编辑plist文件很好用
15. [MAMP](http://www.mamp.info/en/index.html) 本地起mysql Apache server的GUI软件，

####命令行软件
1. [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) zsh很好用，优点可看[这里](http://macshuo.com/?p=676)
2. git 版本控制 [my git config](https://gist.github.com/Fykec/7636248)
3. [git-extras](https://github.com/Fykec/git-extras) 使用git时，增强功能我个人喜欢 add 和commit成一步，所以就自造了一个add－commit的命令，利用git本身插件的机制，就用了
4. [tig](https://github.com/jonas/tig) 使用git时，在terminal下看change 看log很方便，而且我还把tig设为了git的pager
		
		pager = tig

	这样我用

		git log
	时生成的结果也是彩色的，更清晰

5. [Homebrew](http://brew.sh/) 下载很多命令好工具很方便，比如wget
6. [Cocoapods](https://github.com/cocoapods/cocoapods), 管理iOS程序依赖的第三方库[很happy](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)
7. [dic.py](https://gist.github.com/Fykec/7636366) 一个小字典脚本，利用了mac程序本身的字典, [ydcv](https://github.com/felixonmars/ydcv)有道字典的命令行版本，[为mac自带的词典添加词库，这杨就更happy，方便了](http://jiabin.tk/2013/06/23/add-simplified-chinese-to-english-dictionary-for-mac/)
8. [sl](https://github.com/Fykec/sl-mac) 一个输入ls错成sl后会跑火车的程序，我很喜欢
9. [most](http://www.jedsoft.org/most/) 一个man 的pager，彩色的
10. [xctool](https://github.com/facebook/xctool) facebook 做的build工具，适合自动打包，搭建daily build

####其他

#####Xcode
1. Open quickly... CMD＋T 我改成了和textmate一样，很适合输入文件名找文件，用过textmate这个功能肯定很有体会
2. [Xcode Theme](https://github.com/tursunovic/xcode-themes)
3. Xcode plugin, [Dash Plugin](https://github.com/omz/Dash-Plugin-for-Xcode)

####Emacs
命令行使用Emacs.app

	alias emacs='/Applications/Emacs.app/Contents/MacOS/Emacs'
	alias e="emacs -nw -q --load \"~/simple.emacs.d/init.el\""
	alias em="emacs -nw -q --load \"~/simple.emacs.d/init.el\""
	
	
