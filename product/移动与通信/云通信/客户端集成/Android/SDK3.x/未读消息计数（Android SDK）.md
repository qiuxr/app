
## 1 未读消息

这里的未读消息是指用户没有进行已读上报的消息（而非对方是否已经阅读，这种情况需要开启已读回执才能实现，请参考[已读回执](/doc/product/269/9232#3.11-.E5.B7.B2.E8.AF.BB.E5.9B.9E.E6.89.A729)）。`TIMMessageExt` 方法 `isRead` 标识消息是否已读，要想显示正确的未读计数，需要开发者显式调用已读上报，告诉SDK某条消息是否已读，例如，当用户进入聊天界面，可以设置整个会话的消息已读。

> 注意：
> 对于聊天室，Server不保存未读计数，每次登录后跟Server同步未读计数后将会清零。

**原型：**

```
/**
 * 消息是否已读
 * @return 是否已读
 */
public boolean isRead()
```

**示例：**
```
//获取消息扩展实例，其中参数msg是TIMMessage
TIMMessageExt msgExt = new TIMMessageExt(msg);
boolean isReaded = msgExt.isRead();
Log.d(tag, "msg isReaded: " + isReaded);
```

## 2 获取当前未读消息数量

可通过 `TIMConversationExt` 的 `getUnReadMessageNum` 方法获取当前会话中未读消息的数量。
> 注意：
> 对于聊天室，Server不保存未读计数，每次登录后跟Server同步未读计数后将会清零。

**原型：**

```
/**
 * 获取未读消息数
 * @return 未读消息计数
 */
public long getUnreadMessageNum()
```

**示例：**

```
//获取会话扩展实例
TIMConversation con = TIMManager.getInstance().getConversation(TIMConversationType.C2C, peer);
TIMConversationExt conExt = new TIMConversationExt(con);
//获取会话未读数
long num = conExt.getUnreadMessageNum();
Log.d(tag, "unread msg num: " + num);
```
## 3 已读上报

当用户阅读某个会话的数据后，需要进行会话消息的已读上报，SDK根据会话中最后一条阅读的消息，设置会话中之前所有消息为已读。已读上报接口为`TIMConversationExt`中的`setReadMessage`。

**原型：**

```
/**
 * 将此消息之前的所有消息标记为已读
 * @param msg 最后一条已读的消息, 传null表示将当前会话的所有消息标记为已读
 * @param callback 回调
 */
public void setReadMessage(TIMMessage msg, TIMCallBack callback) 
```

**示例：**

```
//对单聊会话已读上报
String peer = "sample_user_1";  //获取与用户 "sample_user_1" 的会话
TIMConversation conversation = TIMManager.getInstance().getConversation(
        TIMConversationType.C2C,    //会话类型：单聊
        peer);                      //会话对方用户帐号
//获取会话扩展实例
TIMConversationExt conExt = new TIMConversationExt(conversation);
//将此会话的所有消息标记为已读
conExt.setReadMessage(null, new TIMCallBack() {
	@Override
	public void onError(int code, String desc) {
		Log.e(tag, "setReadMessage failed, code: " + code + "|desc: " + desc);
	}

	@Override
	public void onSuccess() {
		Log.d(tag, "setReadMessage succ");
	}
});
 
//对群聊会话已读上报
String groupId = "TGID1EDABEAEO";  //获取与群组 "TGID1LTTZEAEO" 的会话
conversation = TIMManager.getInstance().getConversation(
        TIMConversationType.Group,      //会话类型：群组
        groupId);                       //群组Id
//获取会话扩展实例
TIMConversationExt conExt = new TIMConversationExt(conversation);
//将此会话的lastMsg代表的消息及这个消息之前的所有消息标记为已读
conExt.setReadMessage(lastMsg, new TIMCallBack() {
	@Override
	public void onError(int code, String desc) {
		Log.e(tag, "setReadMessage failed, code: " + code + "|desc: " + desc);
	}

	@Override
	public void onSuccess() {
		Log.d(tag, "setReadMessage succ");
	}
});
```

单聊和群聊设置已读用法相同，区别在于会话类型。

## 4 多终端已读上报同步

在多终端情况下，未读消息计数由Server下发同步通知，SDK在本地更新未读计数后，通知用户更新会话。通知会通过`TIMRefreshListener`中的`onRefreshConversation`接口来进行回调，对于关注多终端同步的用户，可以在这个接口中进行相关的同步处理。请参考[会话刷新](/doc/product/269/9229#4.4-.E4.BC.9A.E8.AF.9D.E5.88.B7.E6.96.B0.E7.9B.91.E5.90.AC15) 。

**原型：**
```
/**
 * 部分会话刷新（包括多终端已读上报同步）
 * @param conversations 需要刷新的会话列表
 */
public void onRefreshConversation(List<TIMConversation> conversations);
```

## 5 禁用自动上报

出于性能考虑，未读消息由SDK拉回到本地，Server默认会删除未读消息，切换终端以后无法看到之前终端已经拉回的未读消息，即使没有主动进行已读上报。如果仅在一个终端，未读计数没有问题，如果需要多终端情况下仍然会有未读，可以通过`TIMUserConfigMsgExt`中的`enableAutoReport`方法禁用自动上报，禁用后IM通讯云不会代替用户已读上报。

> 注意：
> 1. 一旦禁用自动上报，需要开发者显式调用`setReadMessage`进行已读上报。
> 2. 需要在登录前设置，登录后设置无效。
```
/**
 * 设置是否开启自动已读上报功能（默认开启），登录前设置
 * @param autoReportEnabled true - 开启， false - 关闭
 */
public TIMUserConfigMsgExt enableAutoReport(boolean autoReportEnabled) 
```

