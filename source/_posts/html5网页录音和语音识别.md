---
title: html5网页录音和语音识别
toc: true
date: 2016-08-23 09:48:15
categories: html5
tags: [html5, 录音, 语言识别]
---

#**背景**

在输入方式上，人们总是在追寻一种更高效，门槛更低的方式，来降低用户使用产品的学习成本。语音输入也是一种尝试较多的方式，有些直接使用语音（如微信语音聊天），有些需要将语音转化为文字（语音识别）。接下来的内容是一次在pc浏览器上进行语音识别输入的一种尝试。 ### 实现 调研阶段，chrome是支持语音识别的。它支持了一系列的接口，可以进行语音识别。参考[HTML5的Speech API相关标准的现状][1]
但是使用这些接口有一些困难，连不上服务器。此路不通。

那么，可以使用笨点儿的方法，先录音再上传到指定语音识别服务器，进行语音识别。这里使用的是[百度语音开放平台](http://yuyin.baidu.com)的语音识别接口，支持8k,16k的单声道的wav文件，或者pcm。尝试了8k的识别效果，跟16k的差了好远。就使用了16k,单声道wav文件，上传到语音识别服务器。

关于录音，主要步骤就是使用navigator.getUserMedia来获取用户的输入设备，成功之后使用webkitAudioContext来创建音频实例。在录音结束之后，将录音的流导出为文件，上传即可。录音的可以参考这个[recorder.js][2]，只要稍微做一下修改就可以应用。其中需要处理音频采样率，默认的采样率为44.1k,这里需要做一个转换，具体方法可以参考[HTML5网页录音和压缩,边猜边做][3]

在浏览器扩展中，没有明确的方式去获取用户对录音的授权。可以在扩展的optionpage里面申请授权，之后在扩展的所有页面都有权限了。在较新的chrome浏览器里测过可以用。参考这里：[How do I give webkitGetUserMedia permission in a Chrome Extension popup window][4]
#**demo**

这里有一个chrome扩展的demo，实现了通过语音采样，生成wav文件上传到语音识别服务器的功能。其中做了一个比较简单的端点检测，通过音量的大小来确定输入的完成。
http://github.com/veizz/speech_io 
一些思考 * demo其实是用来参与公司举办的一届hackathon比赛，主要实现了语音在线识别，文字播报等功能。在后期还有想法加入了一些自然语音处理的功能，可以识别一些输入指令。如『打开百度首页』、『上淘宝买衣服』等功能。会打开指定网站，自动填写输入词，执行搜索。还可以做一些小功能，比如说语音输入『查询天气』、『买电影票』等常用功能，在popup的窗口里面打开等。
一切的想法都看起来很美好，但在大家都熟悉了打字输入的今天，还有多少人愿意使用语音识别做为输入方式？而对于不会打字的人，能否使用标准的普通话来进行语音识别的输入？ * 采样率的处理是通过js的文件操作来实现的。html5支持的fileapi强大如此，怪不得有人用js做视频解码器，不考虑性能的话，看起来很美好啊

#**参考**

http://www.cnblogs.com/jz1108/archive/2012/05/21/2511447.html
https://dvcs.w3.org/hg/speech-api/raw-file/tip/speechapi.html
http://codeartists.com/post/36746402258/how-to-record-audio-in-chrome-with-native-html5-apis
http://stackoverflow.com/questions/13076272/how-do-i-give-webkitgetusermedia-permission-in-a-chrome-extension-popup-window
http://www.cnblogs.com/blqw/p/3782420.html
http://ibillxia.github.io/blog/2013/05/22/audio-signal-processing-time-domain-Voice-Activity-Detection/
http://stackoverflow.com/questions/13333378/how-can-javascript-upload-a-blob
http://www.web-tinker.com/article/20498.html


  [1]: http://www.cnblogs.com/jz1108/archive/2012/05/21/2511447.html
  [2]: http://codeartists.com/post/36746402258/how-to-record-audio-in-chrome-with-native-html5
  [3]: http://www.cnblogs.com/blqw/p/3782420.html
  [4]: http://stackoverflow.com/questions/13076272/how-do-i-give-webkitgetusermedia-permission-in-a-chrome-extension-popup-window