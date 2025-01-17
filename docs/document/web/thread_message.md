# 管理子区消息

<Toc />

子区消息消息类型属于群聊消息类型，与普通群组消息的区别是需要添加 `isChatThread` 标记。本文介绍环信即时通讯 IM Web SDK 如何发送、接收以及撤回子区消息。

## 技术原理

环信即时通讯 IM Web SDK 支持你通过调用 API 在项目中实现如下功能：

- 发送子区消息
- 接收子区消息
- 撤回子区消息
- 获取子区消息

消息收发流程如下：

1. 用户 A 发送一条消息到环信的消息服务器;
2. 单聊时消息时，服务器投递消息给用户 B；对于群聊时消息，服务器投递给群内其他每一个成员;
3. 用户收到消息。

![img](@static/images/android/sendandreceivemsg.png)

子区创建和查看如下图：

![img](@static/images/web/web_group_chat_chreat_new_thread_step_01.png)

![img](@static/images/web/web_group_chat_new_thread_created.png)

## 前提条件

开始前，请确保满足以下条件：

- 完成 `4.0.7 及以上版本` SDK 初始化，详见 [快速开始](quickstart.html)；
- 了解环信即时通讯 IM API 的 [使用限制](/product/limitation.html)。
- 了解子区和子区成员数量限制，详见 [使用限制](/product/limitation.html)。
- 联系商务开通子区功能。

## 实现方法

本节介绍如何使用环信即时通讯 IM SDK 提供的 API 实现上述功能。

### 发送子区消息

发送子区消息和发送群组消息的方法基本一致，详情请参考 [发送消息](message_send_receive.html)。唯一不同的是，发送子区消息需要指定标记 `isChatThread` 为 `true`。

单设备登录时，子区所属群组的所有成员会收到 `onChatThreadChange` 回调。

示例代码如下：

```javascript
// 在 thread 内发送文本消息
function sendTextMessage() {
    let option = {
        chatType: 'groupChat',     // 会话类型，设置为群聊。
        type: 'txt',               // 消息类型。
        to: chatThreadId,          // 消息接收方（子区 ID)。
        msg: 'message content'     // 消息内容。
        isChatThread:true,   // thread 消息标记
    }
    let msg = WebIM.message.create(option); 
    connection.send(msg).then(() => {
        console.log('send private text Success');  
    }).catch((e) => {
        console.log("Send private text error");  
    })
};
```

### 接收子区消息

可以通过 `addEventHandler` 注册监听器接收各类消息的回调，详情参考 [接收消息](message_send_receive.html#接收消息)。

示例代码如下：

```javascript
// 监听收到的文本消息
connection.addEventHandler('THREADMESSAGE',{
  onTextMessage:(message) => {
    if(message.chatThread && JSON.stringify(message.chatThread)!=='{}'){
      console.log(message)
        // 接收到子区消息，添加处理逻辑。
      }
    },
});
```

### 撤回子区消息

撤回子区消息和撤回群组消息的方法基本一致，详情请参考 [撤回消息](message_send_receive.html#撤回消息)。唯一不同的是，撤回子区消息需要指定标记 `isChatThread` 为 `true`。

子区成员可以设置消息监听回调 `onRecallMessage` 对子区消息的撤回进行监听。

示例代码如下：

```javascript
// 撤回消息
/**
 * 发送要撤回的消息。
 * @param {Object} option - 
 * @param {Object} option.mid -  要撤回消息的 ID。
 * @param {Object} option.to -   消息接收对象。
 * @param {Object} option.type - 消息类型：chat (单聊)、groupchat (群聊)和 chatroom (聊天室)。
   */
let option = {
  mid: 'msgId',
  to: 'userID',
  chatType: 'groupChat'
  isChatThread: true
};
connection.recallMessage(option).then((res) => {
  console.log('success', res)
}).catch((error) => {
  // 消息撤回失败 (超过 2 分钟)。
  console.log('fail', error)
})

// 监听要撤回的消息：
conn.addEventHandler('MESSAGES',{
   onRecallMessage: => (msg) {
       // 接收到子区消息被撤回，添加处理逻辑。
       console.log('撤回成功'，msg) 
   }, 
})
```

### 从服务器获取子区消息 (消息漫游)

从服务器获取子区消息，请参考 [从服务器获取消息](message_retrieve.html)。