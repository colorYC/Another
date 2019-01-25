Windows 进阶
==========================

.. highlight:: csharp

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);           
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);               
    JCCall call = JCCall.create(client, mediaDevice, this);

.. _通话录音(windows):

通话录音
-----------------------------

您可以在通话中进行录音，并设置录音文件的路径。开启或关闭录音需要根据当前的录音状态（audioRecord）来决定。录音状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_ 对象获得。接口如下：

::

    /// <summary>
    /// 通话录音，通过JCCallItem对象中的呼叫保持状态来决定开启关闭呼叫保持
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="enable">开启关闭录音</param>
    /// <param name="filePath">录音文件路径</param>
    /// <returns>返回true表示正常执行调用流程，false表示调用异常</returns>
    public bool audioRecord(JCCallItem item, bool enable, string filePath)
   

示例代码::

        JCCallItem item = call.callItems[0];
        if (item.audioRecord)
        {
            // 录音结束
            call.audioRecord(item, false, "your filePath");
        }
        else
        {
            // 创建录音文件保存路径
            String filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (filtPath.Length > 0)
            {
                // 开始录音
                call.audioRecord(item, true, filePath);
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _视频通话录制(windows):

视频通话录制
----------------------------

菊风一对一通话视频录制为您提供 **本地视频源录制和远端视频源录制**。

开启或者关闭视频录制需要根据本端视频源录制状态（localVideoRecord）和远端视频源录制状态（remoteVideoRecord）来决定。视频录制状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_ 对象获得。您可以对录制视频的高度和宽度以及存储路径进行设置，接口如下：
::

    /// <summary>
    /// 视频通话录制，通过JCCallItem对象中的localVideoRecord和remoteVideoRecord状态来决定开启关闭录制
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="enable">开启关闭录制</param>
    /// <param name="remote">是否为远端视频源</param>
    /// <param name="width">录制视频宽像素</param>
    /// <param name="height">录制视频高像素</param>
    /// <param name="filePath">录制视频文件存储路径</param>
    /// <returns>返回true表示正常执行调用流程，false表示调用异常</returns>
    public bool videoRecord(JCCallItem item, bool enable, bool remote, int width, int height, string filePath)


示例代码::

        JCCallItem item = call.callItems[0];
        if (item.getRemoteVideoRecord)
        {
            // 视频录制结束
            call.videoRecord(item, false, true, 0, 0, "your filePath");
        }
        else
        {
            // 创建视频录制文件保存路径
            String filePath; // 视频录制文件的绝对路径，SDK会自动创建视频录制文件
            if (filtPath.Length > 0)
            {
                // 开始视频录制
                call.videoRecord(item, true, true, 640, 360, filePath);
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _截屏(windows):

截屏
------------------------------

在视频通话中，如果想对当前的通话界面进行保存，可以使用截屏功能，截屏分为 **本端视频源截图和远端视频源截图**，接口如下：
::

    /// <summary>
    /// 视频通话截图
    /// </summary>
    /// <param name="width">截屏宽度像素，-1为视频源像素</param>
    /// <param name="height">截屏高度像素，-1为视频源像素</param>
    /// <param name="filePath">文件路径</param>
    /// <returns></returns>
    public bool snapshot(int width, int height, string filePath)

示例代码::

    JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(renderId, JCMediaDevice.JCMediaDeviceRenderMode.FULLSCREEN);
    String filePath; // 截屏文件的绝对路径，SDK会自动创建截屏文件
    RemoteCanvas.snapshot(-1, -1, filePath);


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(windows1):

发送消息
-----------------------------

您可以在通话中发送消息::

    /// <summary>
    /// 通过通话建立后的通道发送数据
    /// </summary>
    /// <param name="item">需要发送数据的JCCallItem对象</param>
    /// <param name="type">文本消息类型，用户可以自定义，例如text，xml等</param>
    /// <param name="content">文本内容</param>
    /// <returns>返回 true 表示正常执行调用流程，false 表示调用异常</returns>
    public bool sendMessage(JCCallItem item, string type, string content)

示例代码::

    call.sendMessage(JCCallItem, "自定义消息类型", "自定义消息内容");

通话中收到消息会收到 onMessageReceive 回调
::

    /// <summary>
    /// 通话中收到消息的回调
    /// </summary>
    /// <param name="type">消息类型</param>
    /// <param name="content">消息内容</param>
    /// <param name="item">JCCallItem对象</param>
    void onMessageReceive(string type, string content, JCCallItem item);


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _涂鸦(windows):

涂鸦
-----------------------------

涂鸦功能允许您进行画笔的涂鸦操作和贴图包的涂鸦操作，您在涂鸦的同时，另一端可以看到实时的涂鸦画面，涂鸦的详细集成请参考 :ref:`涂鸦<涂鸦>`。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _推送(windows):

推送
-----------------------------

通过集成推送，可以将通话信息即时告知用户，从而提高通话的接通率。推送分为 Android 端的小米推送、华为推送以及苹果端的 VoIP 推送，详细集成步骤请参考 :ref:`推送<推送>` 模块。
