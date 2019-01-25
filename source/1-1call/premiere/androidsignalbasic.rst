Android
============================

.. _一对一信令通话-Android:

业务集成
---------------------------

准备工作
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

开始之前，请您先做好如下准备工作：

- `Android SDK 下载 <http://developer.juphoon.com/document/cloud-communication-android-sdk#2>`_

- :ref:`Android SDK 配置和初始化<Android SDK 配置和初始化>` **您在 AndroidManifest 中进行权限配置时，请确保您能够获得打开摄像头、音视频录制等相关权限**

- :ref:`Android 登录<Android 登录>`

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


接口的详细信息请参考 `API 说明文档 <http://developer.juphoon.com/portal/reference/android/>`_ 。

*接口调用逻辑和相关状态*

.. image:: 1-1workflowandroid.png

*说明：黑色字体表示接口，棕色字体表示通话状态*

.. note::

    通话方向（direction）及通话状态（state）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_  对象中的 `getDirection() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getDirection-->`_ 方法和 `getState() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCall.html#STATE_INIT>`_ 方法获得。

.. highlight:: java

**开始集成通话功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCCall call = JCCall.create(client, mediaDevice, this);

其中，创建 JCCall 实例的方法如下
::

    /**
     * 创建JCCall实例
     *
     * @param client        JCClient实例
     * @param mediaDevice   JCMediaDevice实例
     * @param callback      回调接口，用于接收 JCCall 相关回调事件
     * @return JCCall       JCCall实例
     */
    public static JCCall create(JCClient client, JCMediaDevice mediaDevice, JCCallCallback callback);


**开始集成**

**场景一 ：语音通话**

1. 拨打通话

主叫调用下面的接口发起语音通话，此时 video 传入值为 false
::

    /**
     * 一对一呼叫
     *
     * @param userId        用户标识
     * @param video         是否视频呼叫
     * @param extraParam    透传参数，设置后被叫方可获取该参数
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean call(String userId, boolean video, String extraParam);

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_  对象中的 `getExtraParam() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getExtraParam-->`_ 方法获取 extraParam 属性。

示例代码
::

    // 发起语音呼叫
    call.call("peer number", false, "自定义透传字符串");

通话发起后，主叫和被叫均会收到新增通话的回调
::

    /**
     * 新增通话回调
     *
     * @param item JCCallItem 对象
     */
    void onCallItemAdd(JCCallItem item);

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 新增通话回调
    }


2. 应答通话

被叫收到 onCallItemAdd 回调事件，此时可通过 JCCallItem 中的 `getVideo() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getVideo-->`_ 方法以及 `getDirection() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getDirection-->`_ 方法获取 video 和 direction 属性，并根据 video 属性的值以及 direction 属性的值 DIRECTION_IN 判断是视频呼入还是语音呼入，然后可以调用下面的接口进行应答，**语音通话只能进行语音应答**
::

    /**
     * 接听
     *
     * @param item  JCCallItem 对象
     * @param video 针对视频呼入可以选择以视频接听还是音频接听
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean answer(JCCallItem item, boolean video);

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 如果是语音呼入且在振铃中
        if (item.getState() == JCCall.STATE_PENDING) {
            if (item.getDirection() == JCCall.DIRECTION_IN && !item.getVideo()) {
                // 应答通话
                call.answer(item, false);
            }
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
     * 一对一呼叫
     *
     * @param userId        用户标识
     * @param video         是否视频呼叫
     * @param extraParam    透传参数，设置后被叫方可获取该参数
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean call(String userId, boolean video, String extraParam);

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_  对象中的 `getExtraParam() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getExtraParam-->`_ 方法获取 extraParam 属性。


通话发起后，主叫和被叫均会收到新增通话的回调
::

    /**
     * 新增通话回调
     *
     * @param item JCCallItem 对象
     */
    void onCallItemAdd(JCCallItem item);

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 新增通话回调
    }


**创建本地视图画面**

通话发起后，即可调用 JCMediaDevice 类中的 :ref:`startCameraVideo<创建本地视图画面(android)>` 方法打开本地视图预览，**调用此方法会打开摄像头**
::

    /**
     * 获得视频预览对象，通过此对象能获得视图用于UI显示
     *
     * @param renderType    渲染模式
     * @return              JCMediaDeviceVideoCanvas 对象
     * @see RenderType
     */
    public abstract JCMediaDeviceVideoCanvas startCameraVideo(@RenderType int renderType);
    

示例代码::

    // 发起视频呼叫
    call.call("peer number", true, "自定义透传字符串");
    // 打开本地视图预览
    JCMediaDeviceVideoCanvas localCanvas = mediaDevice.startCameraVideo(JCMediaDevice.RENDER_FULL_CONTENT);
    viewGroup.addView(localCanvas.getVideoView(), 0);


2. 应答通话

被叫收到 onCallItemAdd 回调事件，此时可通过 JCCallItem 中的 `getVideo() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getVideo-->`_ 方法以及 `getDirection() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getDirection-->`_ 方法获取 video 和 direction 属性，并根据 video 属性的值以及 direction 属性的值 DIRECTION_IN 判断是视频呼入还是语音呼入，然后可以调用下面的接口选择视频应答或者语音应答
::

    /**
     * 接听
     *
     * @param item  JCCallItem 对象
     * @param video 针对视频呼入可以选择以视频接听还是音频接听
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean answer(JCCallItem item, boolean video);

如果被叫应答通话成功，双方都会收到 onCallItemUpdate 的回调。

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 如果是视频呼入且在振铃中
        if (item.getDirection() == JCCall.DIRECTION_IN && item.getVideo()) {
            // 应答通话
            call.answer(item, true);
        }
    }


3. 通话建立

被叫接听通话后，双方将建立连接，此时，主叫和被叫都将会收到通话更新的回调（onCallItemUpdate）。连接成功之后，可以进行远端视图的渲染。

**创建远端视图画面**

远端视频画面的获取通过调用 JCMediaDevice 类中的 :ref:`startVideo<创建远端视图画面(android)>` 方法实现 
::

    /**
     * 获得视频对象，通过此对象能获得视图用于UI显示
     *
     * @param videoSource   渲染标识串，比如 JCMediaChannelParticipant JCCallItem 中的 renderId
     * @param renderType    渲染模式
     * @return              JCMediaDeviceVideoCanvas 对象
     * @see RenderType
     */
    public abstract JCMediaDeviceVideoCanvas startVideo(String videoSource, @RenderType int renderType);

现在您可以进行一对一视频通话了。

示例代码::

    public void onCallItemUpdate(JCCallItem item) {
        // 如果对端在上传视频流（uploadVideoStreamOther）
        if (item.getState() == JCCall.STATE_TALKING && remoteCanvas == null && item.getUploadVideoStreamOther()) {
            // 获取远端视图画面，renderId来源JCCallItem对象
            JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(item.renderId, JCMediaDevice.RENDER_FULL_CONTENT);
            viewGroup.addView(remoteCanvas.getVideoView(), 0);
        }
    }


4. 挂断通话

主叫或者被叫均可以调用下面的方法挂断通话
::

    /**
     * 挂断
     *
     * @param item          JCCallItem 对象
     * @param reason        挂断原因
     * @param description   挂断描述
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     * @see CallReason
     */
    public abstract boolean term(JCCallItem item, @CallReason int reason, String description);


示例代码::
    
    JCCallItem item = call.getCallItems().get(0);
    call.term(item, JCCall.REASON_NONE, null);

如果是视频通话，则在通话挂断后需要调用 :ref:`stopVideo<销毁本地和远端视图画面(android)>` 接口移除视频画面
::

    /**
     * 停止视频
     *
     * @param canvas JCMediaDeviceVideoCanvas 对象，由 startVideo 获得
     */
    public abstract void stopVideo(JCMediaDeviceVideoCanvas canvas);


通话挂断后，UI 会收到移除通话的回调
::

    /**
     * 移除通话回调
     *
     * @param item          JCCallItem 对象
     * @param reason        通话结束原因
     * @param description   通话结束原因的描述，只有被动挂断的时候，才会收到这个值，其他情况下则返回空字符串
     */
    void onCallItemRemove(JCCallItem item, @JCCall.CallReason int reason, String description);

示例代码::

    public void onCallItemRemove(JCCallItem item, @JCCall.CallReason int reason, String description) {
        // 移除通话回调
    }


其中，reason 有以下几种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - REASON_NONE = 0
     - 无异常
   * - REASON_NOT_LOGIN = 1
     - 未登录
   * - REASON_CALL_FUNCTION_ERROR = 2
     - 函数调用错误
   * - REASON_TIMEOUT = 3
     - 超时
   * - REASON_NETWORK = 4
     - 网络错误
   * - REASON_OVER_LIMIT = 5
     - 超出通话上限
   * - REASON_TERM_BY_SELF = 6
     - 自己挂断
   * - REASON_ANSWER_FAIL = 7
     - 应答失败
   * - REASON_BUSY = 8
     - 忙
   * - REASON_DECLINE = 9
     - 拒接
   * - REASON_USER_OFFLINE = 10
     - 用户不在线
   * - REASON_NOT_FOUND = 11
     - 无此用户
   * - REASON_OTHER = 100
     - 其他错误



**更多功能**

- :ref:`通话状态更新<通话状态更新(android1-1)>`

- :ref:`通话过程控制<通话过程控制(android1-1)>`

- :ref:`获取网络状态<获取网络状态(android1-1)>`

- :ref:`设备控制<设备控制(android)>`


**进阶**

在实现音视频通话的过程中，您可能还需要添加以下功能来增强您的应用：

- :ref:`通话录音<通话录音(android)>`

- :ref:`视频通话录制<视频通话录制(android)>`

- :ref:`截屏<截屏(android)>`

- :ref:`发送消息<发送消息(android1)>`

- :ref:`涂鸦<涂鸦(android)>`

- :ref:`推送<推送(android)>`
