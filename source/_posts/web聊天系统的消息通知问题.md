---
title: web聊天系统的消息通知问题
toc: true
date: 2016-07-06 14:42:58
categories: javascript
tags: javascript
---

web消息提示无非三种方式：声音提示，桌面弹窗和title闪烁提醒。下面做一一介绍。

声音提示
----

注意声音提示前提示已经加载了声音文件，有文章写的很多是临时create一个audio对象，然后audio.src,这样做是非常不好的，因为你每次调用声音的时候都会去后台请求一下这个声音文件。所以先加载出来是最好的方法。

    <audio id="chat-audio" src="audio/system.wav" display="none"></audio>

    function playAudio() {
        document.getElementById('chat-audio').play(); 
        //pause()方法也可以暂停，具体可查html5的audio标签
    }
    //调用方式
    playAudio();

桌面弹窗
====

    function palyDeskNotice(theTitle, options) {
        if (Notification.permission !== "granted") {
            //先判断一下用户是否已经开启了桌面提示的权限，如果没有则提醒用户开启
            window.Notification.requestPermission(function(permission) {
                if (permission === "granted") showNotice(theTitle, options);
            });
        } else {
            showNotice(theTitle, options);
        }
    }
     
    function showNotice(theTitle, options) {
            //这个就是桌面弹窗
        var desknotice = new Notification(theTitle, options);
        desknotice.onclick = function() {
            //当用户点击弹窗的时候，要定位到聊天窗口
            window.focus();
            desknotice.close();
        };
        //页面退出时关闭提醒
        window.onbeforeunload = function() {
            desknotice.close();
        }
        //弹窗3秒后自动消失
        setTimeout(desknotice.close.bind(desknotice), 3000);
    }
    //调用方式
    palyDeskNotice('来自xxx', {
        body: '内容',
        icon: "images/xxx.jpg"
    });

title闪烁提醒的原理
------------


    var NewMsgNoticeflag = false,//闪烁标识
        newMsgNotinceTimer = null;
     
    function newMsgCount() {
        if (NewMsgNoticeflag) {
            NewMsgNoticeflag = false;
            document.title = '【☏新消息】您有新的即时消息';
        } else {
            NewMsgNoticeflag = true;
            document.title = '【　　　】您有新的即时消息';
        }
    }
    //兼容性
    var hiddenProperty = 'hidden' in document 
    ? 'hidden' : 'webkitHidden' in document 
    ? 'webkitHidden' : 'mozHidden' in document 
    ? 'mozHidden' : null;
     
    var visibilityChangeEvent = hiddenProperty.replace(/hidden/i, 'visibilitychange');
    var onVisibilityChange = function() {
            if (!document[hiddenProperty]) {
                clearInterval(newMsgNotinceTimer);
                newMsgNotinceTimer = null;
                document.title = 'beta-即时消息系统'; //窗口没有消息的时候默认的title内容
            }
        }
    document.addEventListener(visibilityChangeEvent, onVisibilityChange);
    //调用方式
    if (!newMsgNotinceTimer) newMsgNotinceTimer = setInterval("newMsgCount()", 200);


