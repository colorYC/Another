iOS
==============================

.. _一对一信令通话-iOS:

业务集成
---------------------------

准备工作
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

开始之前，请您先做好如下准备工作：

- `iOS SDK 下载 <http://developer.juphoon.com/document/cloud-communication-ios-sdk#2>`_

- :ref:`iOS SDK 配置和初始化<iOS SDK 配置和初始化>`

- :ref:`iOS 登录<iOS 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。


业务集成
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

一对一通话涉及以下类：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - JCCall
     - 通话类，包含一对一语音和视频通话功能
   * - JCCallItem
     - 通话对象类，此类主要记录通话的一些状态，UI 可以根据其中的状态进行显示逻辑
   * - JCCallCallback
     - 通话回调接口
   * - JCMediaDevice
     - 设备模块，主要用于视频、音频设备的管理
   * - JCMediaDeviceVideoCanvas
     - 视频对象，主要用于 UI 层视图显示、渲染的控制


接口的详细信息请参考 `API 说明文档 <http://developer.juphoon.com/portal/reference/ios/>`_ 。

*接口调用逻辑和相关状态*

.. image:: 1-1workflowios.png

*说明：黑色字体表示接口，棕色字体表示通话状态*

.. note::

    通话方向（呼入或呼出）及通话状态（振铃、连接中、通话中等）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_  对象中的 `direction <http://developer.juphoon.com/portal/reference/ios/Constants/JCCallDirection.html>`_ 和 `state <http://developer.juphoon.com/portal/reference/ios/Constants/JCCallState.html>`_ 获得。


.. highlight:: objective-c

**开始集成通话功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];
    JCCall *call = [JCCall create:client mediaDevice:mediaDevice callback:self];

其中，创建 JCCall 实例的方法如下
::

    /**
     *  @brief                  创建 JCCall 实例
     *  @param client           JCClient 实例
     *  @param mediaDevice      JCMediaDevice 实例
     *  @param callback         JCCallCallback 回调接口，用于接收 JCCall 相关回调事件
     *  @return                 返回 JCCall 实例
     */
    +(JCCall*)create:(JCClient*)client mediaDevice:(JCMediaDevice*)mediaDevice callback:(id<JCCallCallback>)callback;


**开始集成**

**场景一 ：语音通话**

1. 拨打通话

主叫调用下面的接口发起语音通话，此时 video 传入值为 false
::

    /**
     *  @brief                  一对一呼叫
     *  @param userId           用户标识
     *  @param video            是否为视频呼叫
     *  @param extraParam       透传参数，被叫方可获取透传参数
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)call:(NSString*)userId video:(bool)video extraParam:(NSString *)extraParam;

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_  对象中的 `extraParam <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/extraParam>`_ 属性获得。


示例代码
::

    // 发起语音呼叫
    [call call:@"peer number" video:false extraParam:@"自定义透传字符串"];

通话发起后，主叫和被叫均会收到新增通话的回调
::

    /**
     *  @brief 新增通话回调
     *  @param item JCCallItem 对象
     */
    -(void)onCallItemAdd:(JCCallItem*)item;

示例代码::

    -(void)onCallItemAdd:(JCCallItem*)item {
        // 收到新增通话回调
    }


2. 应答通话

被叫收到 onCallItemAdd 回调事件，并通过 JCCallItem 中的 `video <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/video>`_ 属性以及 `direction <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/direction>`_  属性值 JCCallDirectionIn 判断是视频呼入还是语音呼入，此时可以调用下面的接口进行应答，**语音通话只能进行语音应答**
::

    /**
     *  @brief                  接听
     *  @param item             JCCallItem 对象
     *  @param video            针对视频呼入可以选择以视频接听还是音频接听
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)answer:(JCCallItem*)item video:(bool)video;

示例代码::

    -(void)onCallItemAdd:(JCCallItem*)item {
        // 如果是语音呼入且在振铃中
        if (item && item.direction == JCCallDirectionIn && !item.video) {
             // 应答通话
             [call answer:item video:false];
        }
    }


3. 通话建立

被叫接听通话后，双方将建立连接，此时，主叫和被叫都将会收到通话更新的回调，连接成功之后，通话将建立。

现在您可以进行一对一语音通话了。

如果已经在语音通话中，但又有新通话进来，可以选择接听或挂断，如果选择接听，则原来的一路通话将被保持。


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**场景二 ：视频通话**


1. 拨打通话

主叫调用下面的接口发起视频通话，此时 video 传入值为 true
::

    /**
     *  @brief                  一对一呼叫
     *  @param userId           用户标识
     *  @param video            是否为视频呼叫
     *  @param extraParam       透传参数，被叫方可获取透传参数
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)call:(NSString*)userId video:(bool)video extraParam:(NSString *)extraParam;

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_  对象中的 `extraParam <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/extraParam>`_ 属性获得。

通话发起后，主叫和被叫均会收到新增通话的回调
::

    /**
     *  @brief 新增通话回调
     *  @param item JCCallItem 对象
     */
    -(void)onCallItemAdd:(JCCallItem*)item;

示例代码::

    -(void)onCallItemAdd:(JCCallItem*)item {
        // 收到新增通话回调
        ...
    }


**创建本地视图画面**

通话发起后，即可调用 JCMediaDevice 类中的 :ref:`startCameraVideo<创建本地视图画面>` 方法打开本地视图预览，**调用此方法会打开摄像头**
::

    /**
     *  @brief           获得预览视频对象，通过此对象能获得视图用于UI显示
     *  @param type      渲染模式，@ref JCMediaDeviceRender
     *  @return          JCMediaDeviceVideoCanvas 对象
     */
    -(JCMediaDeviceVideoCanvas*)startCameraVideo:(int)type;


示例代码::

    // 发起视频呼叫
    [call call:@"peer number" video:true extraParam:@"自定义透传字符串"];
    // 本地视图渲染
    JCMediaDeviceVideoCanvas *localCanvas = [mediaDevice startCameraVideo:JCMediaDeviceRenderFullContent];
    localCanvas.videoView.frame = CGRectMake(20, 20, 90, 160);
    [self.view addSubview:localCanvas.videoView];


2. 应答通话

被叫收到 onCallItemAdd 回调事件，并通过 JCCallItem 中的 `video <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/video>`_ 属性以及 `direction <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/direction>`_  属性值 JCCallDirectionIn 判断是视频呼入还是语音呼入，此时可以调用以下接口选择视频应答或者语音应答

::

    /**
     *  @brief                  接听
     *  @param item             JCCallItem 对象
     *  @param video            针对视频呼入可以选择以视频接听还是音频接听
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)answer:(JCCallItem*)item video:(bool)video;

如果被叫应答通话成功，双方都会收到 onCallItemUpdate 的回调。

示例代码::

    -(void)onCallItemAdd:(JCCallItem*)item {
        // 如果是视频呼入且在振铃中
        if(item && item.state == JCCallStatePending) {
            if (item.direction == JCCallDirectionIn && item.video) {
                 // 应答通话
                 [call answer:item video:true];
            }
        }
    }


3. 通话建立

被叫接听通话后，双方将建立连接，此时，主叫和被叫都将会收到通话更新的回调（onCallItemUpdate）。连接成功之后，可以进行远端视图的渲染。

**创建远端视图画面**

远端视频画面的获取通过调用 JCMediaDevice 类中的 :ref:`startVideo<创建远端视图画面>` 方法实现
::

    /**
     *  @brief              获得预览视频对象，通过此对象能获得视图用于UI显示
     *  @param videoSource  渲染标识串，比如 JCMediaChannelParticipant JCCallItem 中的 renderId
     *  @param type         渲染模式，@ref JCMediaDeviceRender
     *  @return             JCMediaDeviceVideoCanvas 对象
     */
    -(JCMediaDeviceVideoCanvas*)startVideo:(NSString*)videoSource renderType:(int)type;

现在您可以进行一对一视频通话了。

示例代码::

    -(void)onCallItemUpdate:(JCCallItem*)item {
        // 如果对端在上传视频流（uploadVideoStreamOther）
        if (item.state == JCCallStateTalking && remoteCanvas == nil && item.uploadVideoStreamOther) {
            // 获取远端视图画面，renderId来源JCCallItem对象
            JCMediaDeviceVideoCanvas *remoteCanvas = [mediaDevice startVideo:item.renderId renderType:JCMediaDeviceRenderFullContent];
            remoteCanvas.videoView.frame = self.view.frame;
            [self.view addSubview:remoteCanvas.videoView];
        }
    }


4. 挂断通话

主叫或者被叫均可以调用下面的方法挂断通话
::

    /**
     *  @brief                  挂断
     *  @param item             JCCallItem 对象
     *  @param reason           挂断原因
     *  @param description      挂断描述
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     *  @see JCCallReason
     */
    -(bool)term:(JCCallItem*)item reason:(JCCallReason)reason description:(NSString*)description;


示例代码
::

    // 挂断通话
    JCCallItem *item = call.callItems[0];
    [call term:item reason:JCCallReasonNone description:@"test"];


如果是视频通话，则在通话挂断后需要调用 :ref:`stopVideo<销毁本地和远端视图画面>` 接口移除视频画面
::

    /**
     *  @brief 停止视频
     *  @param canvas JCMediaDeviceVideoCanvas 对象，由 startVideo 获得
     */
    -(void)stopVideo:(JCMediaDeviceVideoCanvas*)canvas;


通话挂断后，UI 会收到移除通话的回调
::

    /**
     *  @brief              移除通话
     *  @param item         JCCallItem 对象
     *  @param reason       通话结束原因
     *  @param description  通话结束原因的描述，只有被动挂断的时候，才会收到这个值，其他情况下则返回空字符串
     *  @see JCCallReason
     */
    -(void)onCallItemRemove:(JCCallItem*)item reason:(JCCallReason)reason description:(NSString *)description;

示例代码::

    -(void)onCallItemRemove:(JCCallItem*)item reason:(JCCallReason)reason description:(NSString *)description {
        //移除通话回调
    }


其中，reason 有以下几种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - JCCallReasonNone
     - 无异常
   * - JCCallReasonNotLogin
     - 未登录
   * - JCCallReasonCallFunctionError
     - 函数调用错误
   * - JCCallReasonTimeOut
     - 超时
   * - JCCallReasonNetWork
     - 网络错误
   * - JCCallReasonCallOverLimit
     - 超出通话上限
   * - JCCallReasonTermBySelf
     - 自己挂断
   * - JCCallReasonAnswerFail
     - 应答失败
   * - JCCallReasonBusy
     - 忙
   * - JCCallReasonDecline
     - 拒接
   * - JCCallReasonUserOffline
     - 用户不在线
   * - JCCallReasonNotFound
     - 无此用户
   * - JCCallReasonOther
     - 其他错误


**更多功能**

- :ref:`通话状态更新<通话状态更新(ios1-1)>`

- :ref:`通话过程控制<通话过程控制(ios1-1)>`

- :ref:`获取网络状态<获取网络状态(ios1-1)>`

- :ref:`设备控制<设备控制(ios)>`


**进阶**

在实现音视频通话的过程中，您可能还需要添加以下功能来增强您的应用：

- :ref:`通话录音<通话录音(iOS)>`

- :ref:`视频通话录制<视频通话录制(iOS)>`

- :ref:`截屏<截屏(iOS)>`

- :ref:`发送消息<发送消息(iOS1)>`

- :ref:`涂鸦<涂鸦(iOS)>`

- :ref:`推送<推送(iOS)>`