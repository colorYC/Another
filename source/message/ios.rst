iOS
============================

准备工作
---------------------------

.. highlight:: objective-c

开始之前，请您先做好如下准备工作：

- `iOS 版 SDK 下载 <http://developer.juphoon.com/document/cloud-communication-ios-sdk#2>`_

- :ref:`iOS SDK 配置和初始化<iOS SDK 配置和初始化>`

- :ref:`iOS 登录<iOS 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。

业务集成
----------------------------------

即时消息集成涉及到即时消息对象类(JCMessageChannelItem)及其属性：

.. csv-table::
   :file: message-ios.csv

**即时消息中的枚举**

即时消息分为一对一消息和群组消息，其类型(JCMessageChannelType)如下：
::

    /// 一对一消息
    JCMessageChannelType1To1,
    /// 群组消息
    JCMessageChannelTypeGroup,

消息传输方向(JCMessageChannelItemDirection)有发送和接收两种：
::

    /// 发送
    JCMessageChannelItemDirectionSend,
    /// 接收
    JCMessageChannelItemDirectionReceive,

**即时消息集成**

.. image:: images/message_workflow.png

**开始集成消息功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMessageChannel *messageChannel = [JCMessageChannel create:client callback:self];

其中，创建 JCMessageChannel 实例的方法如下
::

    /**
     *  @brief 创建 JCMessageChannel 对象
     *  @param client JCClient 对象
     *  @param callback JCMessageChannelCallback 回调接口，用于接收 JCMessageChannel 相关通知
     *  @return 返回 JCMessageChannel 对象
     */
    +(JCMessageChannel*)create:(JCClient*)client callback:(id<JCMessageChannelCallback>)callback;

**开始集成**

消息类型分为两种：文本消息和文件消息。

假如用户 A 想给用户 B 发送即时消息，可以调用如下接口：

- 发送文本消息
::

    /**
     *  @brief 发送文本消息
     *  @param type 类型，参见 JCMessageChannelType
     *  @param keyId 对方唯一标识，当 type 为 JCMessageChannelType1To1 时为用户标识，当 type 为 JCMessageChannelTypeGroup 时为群组标识
     *  @param messageType 文本消息类型，用户可以自定义，例如text，xml等
     *  @param text 文本内容
     *  @param extraParams 自定义参数集
     *  @return 返回 JCMessageChannelItem 对象，异常返回 nil
     *  @warning 文本内容不要超过10KB
     */
    -(JCMessageChannelItem*)sendMessage:(JCMessageChannelType)type keyId:(NSString*)keyId messageType:(NSString*)messageType text:(NSString*)text extraParams:(NSDictionary*)extraParams;

示例代码::

    // 发送一对一文本消息
    JCMessageChannelItem *item = [messageChannel sendMessage:JCMessageChannelType1To1 keyId:userId messageType:@"text" text:text extraParams:@{@"1":@"1", @"2":@"2"}];
    // 发送群组文本消息
    JCMessageChannelItem *item = [messageChannel sendMessage:JCMessageChannelTypeGroup keyId:groupId messageType:@"text" text:text extraParams:nil];


- 发送文件消息
::

    /**
     *  @brief 发送文件消息
     *  @param type 类型，参见 JCMessageChannelType
     *  @param keyId 对方唯一标识，当 type 为 JCMessageChannelType1To1 时为用户标识，当 type 为 JCMessageChannelTypeGroup 时为群组标识
     *  @param messageType 文件消息类型，用户可以自定义，例如image，video等
     *  @param fileUri 文件链接地址
     *  @param thumbPath 缩略图路径，针对视频，图片等消息
     *  @param size 文件大小
     *  @param duration 文件时长，针对语音，视频等消息
     *  @param extraParams 自定义参数集
     *  @return 返回 JCMessageChannelItem 对象，异常返回 nil
     */
    -(JCMessageChannelItem*)sendFile:(JCMessageChannelType)type keyId:(NSString*)keyId messageType:(NSString*)messageType fileUri:(NSString*)fileUri thumbPath:(NSString*)thumbPath size:(int)size duration:(int)duration extraParams:(NSDictionary*)extraParams;

示例代码::

    // 发送一对一文件消息
    JCMessageChannelItem *item = [messageChannel sendFile:JCMessageChannelType1To1 keyId:userId messageType:@"image/png" fileUri:@"http://test.png" thumbPath:[[NSBundle mainBundle] pathForResource:@"fire" ofType:@"png"] size:100 duration:10 extraParams:nil];
    // 发送群组文件消息
    JCMessageChannelItem *item = [messageChannel sendFile:JCMessageChannelTypeGroup keyId:groupId messageType:@"image/png" fileUri:@"http://test.png" thumbPath:[[NSBundle mainBundle] pathForResource:@"fire" ofType:@"png"] size:100 duration:10 extraParams:nil];


A 发送即时消息后，会收到 onMessageSendUpdate 回调
::
    
    /**
     *  @brief 消息发送状态更新
     *  @param message IM消息对象，通过该对象可以获得消息的属性及状态
     *  @see JCMessageChannelItem
     */
    -(void)onMessageSendUpdate:(JCMessageChannelItem*)message;

其中，消息状态(JCMessageChannelItemState)有以下几种：
::

    /// 消息初始状态
    JCMessageChannelItemStateInit,
    /// 消息发送中状态
    JCMessageChannelItemStateSending,
    /// 消息发送成功状态
    JCMessageChannelItemStateSendOK,
    /// 消息发送失败状态
    JCMessageChannelItemStateSendFail,
    /// 收到消息
    JCMessageChannelItemStateRecveived,


示例代码::

    -(void)onMessageSendUpdate:(JCMessageChannelItem*)message {
        if (message.state == JCMessageChannelItemStateSendOK && !message.fileUri && message.type == JCMessageChannelTypeGroup) {
            // 群组文本消息发送成功
        } else if (message.state == JCMessageChannelItemStateSendOK && message.fileUri && message.type == JCMessageChannelTypeGroup) {
            // 群组文件消息发送成功
        } else if (message.state == JCMessageChannelItemStateSendOK && !message.fileUri && message.type == JCMessageChannelType1To1) {
            // 一对一文本消息发送成功
        } else if  (message.state == JCMessageChannelItemStateSendOK && message.fileUri && message.type == JCMessageChannelType1To1) {
            // 一对一文件消息发送成功
        }
    }


如果消息发送失败(当消息状态为 JCMessageChannelStateSendFail 时)，原因有以下几种：
::

    /// 正常
    JCMessageChannelReasonNone,
    /// 未登录
    JCMessageChannelReasonNotLogin,
    /// 消息内容太长
    JCMessageChannelReasonTooLong,
    /// 其他错误
    JCMessageChannelReasonOther,

**接收消息**

即时消息发送成功后，用户 B 会收到 onMessageRecv 回调
::

    /**
     *  @brief 收到消息通知
     *  @param message IM消息对象，通过该对象可以获得消息的属性及状态
     *  @see JCMessageChannelItem
     */
    -(void)onMessageRecv:(JCMessageChannelItem*)message;

示例代码::

    -(void)onMessageRecv:(JCMessageChannelItem*)message {
        if (message.state == JCMessageChannelItemStateRecveived && message.direction == JCMessageChannelItemDirectionReceive && !message.fileUri && item.type == JCMessageChannelTypeGroup) {
            // 收到群组文本消息
        } else if (message.state == JCMessageChannelItemStateRecveived && message.direction == JCMessageChannelItemDirectionReceive && message.fileUri && item.type == JCMessageChannelTypeGroup) {
            // 收到群组文件消息
        } else if (message.state == JCMessageChannelItemStateRecveived && message.direction == JCMessageChannelItemDirectionReceive && !message.fileUri && item.type == JCMessageChannelType1To1) {
            // 收到一对一文本消息
        } else if (message.state == JCMessageChannelItemStateRecveived && message.direction == JCMessageChannelItemDirectionReceive && message.fileUri && item.type == JCMessageChannelType1To1) {
            // 收到一对一文件消息
        }
    }