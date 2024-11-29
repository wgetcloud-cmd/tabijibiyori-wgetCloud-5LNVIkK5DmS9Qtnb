
[上一篇逆向WeChat(七)是逆向微信客户端本地数据库相关事宜。](https://github.com)


[本篇逆向微信客户端本地日志**xlog**相关的事宜。](https://github.com)


本篇在博客园地址<https://github.com/bbqzsl/p/18459562>


现在开始本篇。


如果问AI, "微信xlog文件打开方法"。(提问AI的时间在较早的时候，下面的回答不代表最新的回答。）


百度AI:


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241015014548702-437939665.png)


 Gemini:


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241015015118387-1457994047.png)


 gpt3\.5:


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241015015427295-1353430253.png)


腾讯混元 g3\.5


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241016020228471-1330129644.png)


 


 百度AI 跟 Gemini 出现幻觉一本正经，它们还好像停留在搜索引擎爬网页的阶段。哦，它们本业就是搜索引擎。它们被[https://m.300\.cn/itzspd/634130\.html](https://github.com)这编2021\-0523的旧POST污染了。这编POST也是一本正经说微信的xlog文件是语音聊天记录文件。腾讯混元也是一本正经胡说八道，它自家的东西还不比谁都清楚，也不抖一点出来。对比后，这就侧面证明了AI能有多聪明，视乎你都教了它些什么，或者让它学了什么。填鸭式的井蛙式的AI会更聪明。（题外一下，gpt也有掉链子的时候。我在写（五），（六）的时候，问gpt，“微信的WMPF使用chromium哪些代码编译”。gpt三申五令chromium是google浏览器开源项目，并且强调WMPF是微信团队独立研发的小程序框架，不是浏览器，两者独立毫无关系，小程序框架不存在使用chromium代码。我然后重复4次，”微信WMPF是不是使用了chromium代码“之类变换问法，gpt给我相同的回答三申五令并且强调微信团队独立研发，最后还发起脾气罢答。那时候openai的确公告过出现罢答的bug。我最后问”我逆向过WMPF，它的确使用了chrominum代码编译“。gpt才变了回答，微信的WMPF可能使用了chrominum某些技术，纵然这样WMPF也是微信团队独立研发的。我去，它是你Ex吗？）。


百度AI更多地爬特定地区的新网页，爬到了有人发现mars的github有一未合并的分支，有人上传了decode\_mars\_nocrpyt\_log\_file.py等解码代码这一消息。但它并不知道xlog是加密的。Gemini爬不到这特定地区的网页，漏过了这一消息。只有gpt最不靠谱。


gpt还提出了两点建议，逆向破解加密跟找明文输出。我是一翻逆向分析验证后，做好了演示材料，定了稿的大纲，才问AI，没想到gpt将我做的都说了。


 


**本篇主要内容**就是**逆向破解加密**跟**找明文输出**。


首先来看**逆向破解加密。**


对于第一点，也不浪费大家时间，这是基本不可能的。xlog使用Elliptic Curve Cryptography进行加密。gen\_key(srv\_pub, client\_priv) \=\= gen\_key(client\_pub, srv\_priv)。每次xlog初始化，会随机生成client\_pub跟client\_priv，在gen\_key(srv\_pub, client\_priv)产生密钥后，丢弃client\_priv。这样每次启动微信，日志的密钥都不相同。每段加密内容都会附加上client\_pub，只有用srv\_priv才能产生出正确的密钥。除非你能够将每次丢弃的client\_priv跟client\_pub一同保存起来。通过加密段内容的附带的client\_pub，找回client\_priv，并使用srv\_pub就可以产生密钥了。


下图演示，使用wechatwin.dll!xlog方法uECC\_make\_key()生成两组ECC密钥对。uECC\_shared\_secret()分别对交换公钥后密钥pair，生成出相同的加密密钥ECDH\_KEY。


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241016035913205-277593659.png)


正因为这样，微信开发团队才可以无忌惮地大写特写日志，简直如同写调试信息一样，方便它们搞调试，搞不好你的一天的日志文件就会有几百个M。微信还有setDebugHost功能，还有调试服务器。猜想一下，服务器需要向客户端进行命令交互，这是十分简单的事，比如列出日志，上传日志。日志基本上只能它们用srv\_priv进行查看。日志里几乎都是流水帐的调试信息。不单是错误调试。


（再题外一下，微信在后台耗资源。主程序是可以通过主界面设置停止更新，但是其它子服务就不适用这个设置。WMPF是作为子服务的，我们无法阻止它在后台下载更新。现在的WMPF有500MB，而且更新得贼频繁。并且下载后，它会使得主程序突然耗尽你所有的CPU，强迫你重新启动微信让它可以顺利加载最新的WMPF。如果不想重开，删除它最新下载解压后的WMPF，主程序会停止耗CPU。但是它依然会锲而不舍在后台偷偷下载。 我原本借用手机电话卡的流量，查一查网页，挂一下微信。没想到一个下午跟夜晚，手机突然欠费停机。后来一查，我的套餐流量被全部耗尽还刷套餐外流量刷了三十块。套餐外流量1MB\-0\.3元。我才想起，我删了几次它反复下载的WMPF，流量被它狂耗。要不是我的话费不多，还要冤。可能这三十块，对于大厂里人均几十k（加上年终各种福利随便就double,triple）的大卡大佬大神仙人们来说，连一个工作午餐的最低餐费标准都算不上，可能还不及下班后打个车的车费，或者是买个Figure漂洋过海的运费的零头都不够。又怎么会顾及我们这些尘世间的小虾蟹囊中羞涩戳襟见肘的艰辛。你们不为用户考虑节省流量就算了，还阴阴湿湿在后台开水喉一样耗流量，pukgaai。）


回到正题，正因为它们可以无后顾之忧地写日志，我们逆向时才能够获得这么多参考信息，虽然有点矛盾，但还是要多谢这些海量日志信息。


即使mars的github仓库有分支提供了解码工具的代码，但是没有微信在服务端的私有密钥，你能够将其它人的日志解码出来吗？我水平有限常识浅疏，认为基本是不可能，不排除有泰斗可以做到。对于自己用的话，那是办法不只一个。除了我在上面说的，将客户端的随机私钥统统保存这个方法外，还可以伪造公钥patch掉微信程序里的公钥，这样就可用自己的伪造私钥来解码日志了。


patch方式有两种，一种只要用hex编辑器即可以，将wechatwin.dll的'1dac3876bd566b60c7dcbffd219ca6af2d2c07f045711bf2a6d111a2b1fc27c4d'开头的64字节替换成你伪造的ECC公开密钥。另一种就是在运行时，修改对应的动态内存，因为公开密钥已经加载到xlog对象的成员了。对于动态改内存，方法很多，最简单就是用windbg了。s\-a命令搜索，ea命令修改。


换句话说，只要包含有这个公开密钥的模块都会有自己的一个xlogger。这些模块分别有wechatwin.dll，wmpf\_host\_export.dll，AppEx。它们对应的日志文件名分别以下列前缀开后，MM\_, host\_, main\_。日志文件全部都在AppData/Roaming/Tencent/WeChat/log目录。


 


然后来看**找明文输出**。


我们的切入点是appender，有一个关键的函数指针\_\_xlogger\_Write\_impl。简介如下，详细去github看代码。


![](https://img2024.cnblogs.com/blog/665551/202410/665551-20241014232704462-790738866.png)


xlog的接口XLog设计在mars/comm/xlogger。它的实现XLoggerAppender在mars/xlog。


XLog设计跟Chrome::base::LogMessage相似，是对logging行为的封装。初始化，设置本条日志的说明信息，包括源代码文件名，行号，日志对象。流串输入操作符\<\<添加日志内容。析构函数结束本条日志，并且提交到XLoggerAppender。


XLoggerAppender相当于日志设备的实现，负责将XLog提交的日志写到存储设备上，过程包含加密。


所有WeChat模块都使用mars::xlog作为日志系统，但每个EXE都将它自身的appender设置给所有模块的mars::xlog。这样每个模块都统一使用EXE的appender作为日志系统。具体方式就是将其它模块的appender的\_\_xlogger\_Write\_impl指向同一个appender的\_\_xlogger\_Write\_impl。没错，这个函数就是我们可以要找的枢纽点，所有明文日志的入口点。要注意的是，wechatwin.dll生成的日志以MMPC\_开头，它们是以utf8作为编码的。而其它的却是用gbk进行编码的。


小程序平台WeChatAppEx同时使用xlog进行日志处理，它是将mars/xlog封装成一个服务xlogger.mojom.XLogger，并且将chrome的日志系统指向这个日志服务。


 


由于日志会泄漏个资，所以只能给一些无关重要的日志作为演示。


![](https://img2024.cnblogs.com/blog/665551/202411/665551-20241126223032002-2003954153.gif)


 


本篇到这里，下一篇再见。


[逆向WeChat(八，日志XLog）](https://github.com)


[逆向WeChat(七，查找sqlcipher的DBKey，查看protobuf文件）](https://github.com)


[逆向WeChat(六，通过嗅探mojo抓包小程序https，打开小程序devtool)](https://github.com/bbqzsl/p/18370679 "发布于 2024-05-28 21:13"):[MeoMiao 萌喵加速](https://biqumo.org)


[逆向WeChat(五，mmmojo, wmpfmojo)](https://github.com/bbqzsl/p/18216717 "发布于 2024-05-28 21:13")


[逆向通达信 x 逆向微信 x 逆向Qt (趣味逆向，你未曾见过的signal\-slot用法)](https://github.com)


[逆向WeChat(四，mars, 网络模块)](https://github.com/bbqzsl/p/18209439 "发布于 2024-05-28 21:13")


[逆向WeChat(三, EventCenter, 所有功能模块的事件中心)](https://github.com/bbqzsl/p/18198572 "发布于 2024-05-23 21:48")


[逆向WeChat (二, WeUIEngine, UI引擎)](https://github.com/bbqzsl/p/18187099 "发布于 2024-05-17 20:12")


[逆向wechat(一, 计划热身)](https://github.com)


 


