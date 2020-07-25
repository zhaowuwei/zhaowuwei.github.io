---
layout: article
title: 喜马IM系统 直播聊天室API设计
tags: IM
article_header:
  type: cover
  image:
    src: /screenshot.jpg
---

喜马IMSDK是基于喜马拉雅内部即时通信的技术积累，结合以往的业务使用经验打造的即时通信的中台服务组件。借助IMSDK提供的中台服务，开发者不需要耗费资源搭建服务即可将IM即时通信能力快速集成到自己的应用产品中。

针对不同场景，喜马IMSDK计划提供了一系列产品、技术解决方案，包括：客户端 IM 通信能力基础库、客户端 IM 界面组件以及服务端 API 等，利用这些解决方案，用户可以方便地在自身的应用中支持诸如私信、群聊、聊天室以及推送等IM业务服务。

![电脑插图](https://github.com/zhaowuwei/zhaowuwei.github.io/blob/master/_posts/post_imgs/img_computer.png)

## 客户端调用

喜马IM系统后续会包含数个子系统，如私信、社群、直播聊天室等，各子系统都会拥有一个提供功能服务的单例。
XmIMClient中存储了服务接口和对应的服务单例之间的对应关系，使用如下的API调用模式取得相应的功能单例，该设计参照android现有的`RouterServiceManager`设计。

```java
XmIMClient.getService(IChatRoomService.class)//聊天室服务
XmIMClient.getService(IPrivateMsgService.class)//私信服务
XmIMClient.getService(IGroupChatService.class)//社群服务
```
由于IM系统的重构升级先由直播聊天室服务起步，本文只阐述直播聊天室相关的API设计。

## 1 登陆聊天室

直播聊天室的登陆过程分为 `建立长链接Connect` 和 `加入聊天室Join` 两个步骤，初始进入直播时需要连续完成这两步，而切换直播间或者用户时，并不需要彻底断开连接，只需要在`退出直播间Leave`后再次完成Join操作即可。

在提供给外部的API中，不会具体体现这两个过程，只会提供统一的进入直播聊天室的接口，SDK内部将会判断底层的长连接是否存在，存在则只会进行join过程，不存在则会完成登陆全部步骤。这一过程并不会暴露给外部。

### 进入直播间Enter

聊天室服务建立需要两条长连接：一是推送连接PushConnection，负责接收推送的聊天消息；二是进行收发交互的控制连接ControlConnection，用户发送自己的聊天消息，以及重要的系统消息接收都由该连接负责。

所以初始的连接过程包含两条链路的操作，任意一条链路的失败则宣告登陆流程失败，需要提示用户进行重试。

>API原型

```java
/**
 * 登陆直播聊天室，包含建链和加入两个过程
 *
 * @param loginData    加入直播间所需业务数据
 * @param callback    结果回调
 */
void enterChatRoom(ImJoinChatRoomData loginData,
                   EnterChatRoomResultCallback callback);

```

>参数说明

由于SDK内部将会进行http请求获取部分连接所需数据，所以外部传入的数据较为有限。

输入参数 | 相关说明
--------- | -------------
loginData | 登陆流程需要的直播间业务数据
callback | 登陆结果回调


ImJoinChatRoomData说明

内容 | 相关说明
--------- | -------------
appId | 应用唯一标识
chatId | 聊天室id
userId | 用户id
nickName | 用户昵称
token | app登陆带的token


>代码示例

```java
ImJoinChatRoomData joinData;

XmIMClient.getService(IChatRoomService.class).enterChatRoom(joinData, new EnterChatRoomResultCallback() {
    @Override
    public void onSuccess(EnterChatRoomResultData result) {
        //登陆成功
    }
    @Override
    public void onFail(int code, String msg) {
        //登陆失败
    }
});

```

## 2 退出聊天室

退出直播聊天室也分为两个步骤，一是`停止和聊天室服务的通讯Leave`，不再和聊天室服务产生交互、或者接收消息；二是`彻底断开连接Release`，即彻底释放长连接资源。

由于聊天室服务基于两条长连接建立的，所以以上操作也是两条连接同时进行的。

### 2.1 停止服务Leave

>API原型

```java
/**
 * 没有断开连接的情况下，停止和某个直播间的通讯
 *
 * @param chatId   直播间id
 * @param callback 结果回调
 */
void leaveChatRoom(long chatId,
                   LeaveChatRoomResultCallback callback);
```

>代码示例

```java
XmIMClient.getService(IChatRoomService.class).leaveChatRoom(1233, new LeaveChatRoomResultCallback() {
    @Override
    public void onSuccess() {
        
        //成功退出聊天室
        
    }
    @Override
    public void onFail(int code, String msg) {
        
        //退出失败
    }
});
```


### 2.2 释放连接Release

当聊天室的服务连接没有存在必要时，需要释放连接，由于该操作一定可以成功，所以没有回调。

一般的完全退出流程应该是：先进行leave操作告知服务端退出聊天室，而后释放长连接。当leave操作不能成功的时候，如果需要也可以直接释放连接。

>API原型

```java
/**
 * 彻底断开长连接，释放连接相关资源
 */
void release();
```

>代码示例

```java

XmIMClient.getService(IChatRoomService.class).release();

```


## 3 消息收发

直播聊天室业务中，客户端和服务端的交互有两种形式。
1. 一问一答的形式：request发出会得到相应的response，最重要的业务场景就是聊天室中用户发送聊天消息；
2. 服务端直接推送：服务端直接将消息push给客户端，聊天室中其他用户的聊天消息、以及重要的系统通知都是以推送模式到达客户端的；


### 3.1 发送消息

>发送消息的产生

用户发送的聊天消息请求和回复，服务端设计的消息类型为 `RoomMsgReq`/`RoomMsgRsp`。为了便于二次开发使用，设计了`ChatRoomMsgBuilder`来创建可以发送的消息对象，然后就可以直接调用发送接口发出去即可。

ChatRoomMsgBuilder接口设计

```java
    //创建文本消息
    RoomMsgReq createChatRoomTextMessage(String chatId, String text);

    //创建图片消息
    RoomMsgReq createChatRoomImageMessage(String chatId, String picUrl);

    //创建音频消息
    RoomMsgReq createChatRoomAudioMessage(String roomId, String fileUrl, long duration);

    //创建视频消息
    RoomMsgReq createChatRoomVideoMessage(String roomId, String fileUrl,
                                       long duration, int width, int height);
```


>发送方法API原型

```java
/**
 * 聊天室发送聊天消息
 * 
 * @param msg 聊天消息内容
 * @param retryCount 重试次数
 * @param callback 发送结果回调
 */
void sendChatRoomMsg(Message msg, int retryCount, 
                     SendChatRoomMsgCallback callback);
```

>代码示例

```java
IChatRoomMsgBuilder msgBuilder;

Message textMessage 
        = msgBuilder.createChatRoomTextMessage("chat3132", "This is Text Message!");
        
XmIMClient.getService(IChatRoomService.class).sendChatRoomMsg(textMessage, 
        2, new SendChatRoomMsgCallback() {
    @Override
    public void onSuccess(long uniqueId) {
        
        //发送成功
        
    }
    @Override
    public void onFail(long uniqueId, int code, String msg) {
        
        //发送失败
    }
});
```

### 3.2 接收推送消息

推送的消息也分为2种，一种是聊天内容消息ChatMsg，另一种是系统下发的通知消息CustomMsg。

通过注册消息监听，在收到新消息时可以接收到推送通知。


>API原型

推送监听接口
```java
public interface IGetChatRoomPushMsgListener {

    //接收到聊天内容消息
    void onGetPushChatMsg(List<ChatMsg> chatMsgList);

    //接收到系统通知消息
    void onGetPushCustomMsg(List<CustomMsg> customMsgList);

}
```

注册、注销接收推送消息的监听

进入直播聊天室成功之后注册接收消息的监听，离开一个聊天室同时注销监听，以避免不同聊天室的消息互串；

```java
/**
 * 注册/注销接收推送消息监听
 * @param listener 变化监听
 */
void registerGetChatRoomPushMsgListener(IGetChatRoomPushMsgListener listener);
void unRegisterGetChatRoomPushMsgListener(IGetChatRoomPushMsgListener listener);
```

>代码示例

```java
XmIMClient.getService(IChatRoomService.class)
        .registerGetChatRoomPushMsgListener(new IGetChatRoomPushMsgListener() {
    @Override
    public void onGetPushChatMsg(List<ChatMsg> chatMsgList) {
        //收到推送的聊天消息
    }
    @Override
    public void onGetPushCustomMsg(List<CustomMsg> customMsgList) {
        
        //接收到推送的系统通知
    }
});
```


## 4 监听聊天室在线状态

进入聊天室成功后，SDK 底层链路管理模块会负责维护与服务器的长连接以及断线重连等工作。当用户在线状态发生改变时，会发出通知。通过注册监听的方式外部业务层也可以实时获得当前的连接状态。

>API原型

连接状态监听接口

```java
public interface IGetChatRoomStatusChangeListener {

    /**
     * 接收到连接的状态变化
     * 
     * @param chatId 聊天室ID
     * @param connectStatus 当前的连接状态
     * @param msg 描述信息
     */
    void onGetChatRoomStatus(String chatId, int connectStatus, String msg);

}
```

注册、注销状态监听的方法
```java
/**
 * 注册/注销聊天室在线状态变化监听
 * @param listener 变化监听
 */
void registerChatRoomStatusListener(IGetChatRoomStatusChangeListener listener);
void unRegisterChatRoomStatusListener(IGetChatRoomStatusChangeListener listener);
```

>代码示例

```java
XmIMClient.getService(IChatRoomService.class)
        .registerChatRoomStatusListener(new IGetChatRoomStatusChangeListener() {
            @Override
            public void onGetChatRoomStatus(String chatId, int connectStatus, String msg) {
                
            }
        });
```

