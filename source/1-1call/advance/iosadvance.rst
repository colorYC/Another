iOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];
    JCCall *call = [JCCall create:client mediaDevice:mediaDevice callback:self];

.. _通话录音(iOS):

通话录音
-----------------------------

您可以在通话中进行录音，开启或关闭录音需要根据当前的录音状态（audioRecord）来决定。录音状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获得。接口如下：

::

    /**
     * 语音通话录音，通过 JCCallItem 对象中的audioRecord状态来决定开启关闭录音
     *
     * @param item              JCCallItem 对象
     * @param enable            开启关闭录音
     * @param filePath          录音文件路径
     * @return                  返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)audioRecord:(JCCallItem*)item enable:(bool)enable filePath:(NSString*)filePath;


示例代码::

    // 语音录制
    - (IBAction)audioRecord:(id)sender {
        JCCallItem *item = call.callItems[0];
        if (item.audioRecord) { // 正在录制中
           //录音结束的处理
           [call audioRecord:item enable:false filePath:@"your filePath"];
            ...
        } else {
            // 创建录音文件
            NSString *filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (filePath != nil) {
               // 开始录音
               [call audioRecord:item enable:true filePath:filePath];
                ...
            } else {
                // 录音失败的处理
            }
        } 
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _视频通话录制(iOS):

视频通话录制
----------------------------

菊风一对一通话视频录制为您提供 **本地视频源录制和远端视频源录制**。

开启或者关闭视频录制需要根据本端视频源录制状态（localVideoRecord）和远端视频源录制状态（remoteVideoRecord）来决定。视频录制状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获得。您可以对录制视频的高度和宽度以及存储路径进行设置，接口如下：
::

    /**
     * 视频通话录制，通过 JCCallItem 对象中的localVideoRecord状态和remoteVideoRecord状态来决定开启关闭录制
     *
     * @param item              JCCallItem 对象
     * @param enable            开启关闭录制
     * @param remote            是否为远端视频源
     * @param width             录制视频宽像素
     * @param height            录制视频高像素
     * @param filePath          录制视频文件存储路径
     * @return                  返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)videoRecord:(JCCallItem*)item enable:(bool)enable remote:(bool)remote width:(int)width height:(int)height filePath:(NSString*)filePath;

.. note:: 参数 remote 值为 true 时为远端视频源录制，值为 false 时为本地视频源录制。

示例代码::

    // 视频录制
    - (IBAction)videoRecord:(id)sender {
        JCCallItem *item = call.callItems[0];
        if (item.localVideoRecord) { //如果正在录制本地视频
            // 停止录制本地视频
            [call videoRecord:item enable:false remote:false width:0 height:0 filePath:@"your filePath"];
             ...
        } else if (item.remoteVideoRecord) { // 如果正在录制远端视频
            // 停止录制远端视频
            [call videoRecord:item enable:false remote:true width:0 height:0 filePath:@"your filePath"];
            ...
        } else {
            // 创建视频录制文件保存路径
            NSString *filePath; // 视频录制文件的绝对路径，SDK会自动创建视频录制文件
            if (filePath != nil) {
                // 远端视频录制
                [call videoRecord:item enable:true remote:true width:640 height:360 filePath:filePath];
                // 本端视频录制
                [call videoRecord:item enable:true remote:false width:640 height:360 filePath:filePath];
                ...
            } 
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _截屏(iOS):

截屏
------------------------------

在视频通话中，如果想对当前的通话界面进行保存，可以使用截屏功能，截屏分为 **本端视频源截图和远端视频源截图**，接口如下：

::

    /**
     *  @breif           视频通话截图
     *  @param width     截屏宽度像素，-1为视频源像素
     *  @param height    截屏高度像素，-1为视频源像素
     *  @param filePath  文件路径
     */
    -(bool)snapshot:(int)width heigh:(int)height filePath:(NSString*)filePath;

示例代码::

    - (IBAction)snapshot:(id)sender {
        JCCallItem *item = call.callItems[0];
        JCMediaDeviceVideoCanvas *localCanvas = [mediaDevice startCameraVideo:JCMediaDeviceRenderFullContent];
        JCMediaDeviceVideoCanvas *remoteCanvas = [mediaDevice startVideo:item.renderId renderType:JCMediaDeviceRenderFullContent];
        NSString *filePath; // 截屏文件的绝对路径，SDK会自动创建截屏文件
        // 本端视频源截图
        [localCanvas snapshot:-1 heigh:-1 filePath:filePath];
        // 远端视频源截图
        [remoteCanvas snapshot:-1 heigh:-1 filePath:filePath];
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(iOS1):

发送消息
-----------------------------

您可以在通话中发送消息::

    /**
     *  @brief           通过通话建立的通道发送数据
     *  @param item      需要发送数据的 JCCallItem 对象
     *  @param type      文本消息类型，用户可以自定义，例如text、xml等
     *  @param content   消息内容
     *  @return          返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)sendMessage:(JCCallItem *)item type:(NSString *)type content:(NSString *)content;

示例代码::

    JCCallItem *item = call.callItems[0];
    [call sendMessage:[item type:"自定义消息类型" content:"自定义消息内容"];

通话中收到消息会收到 onMessageReceive 回调
::

    /**
     *  @brief          通话中收到消息的回调
     *  @param item     JCCallItem 对象
     *  @param type     消息类型
     *  @param content  消息内容
     */
    -(void)onMessageReceive:(JCCallItem *)item type:(NSString *)type content:(NSString *)content;


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _涂鸦(iOS):

涂鸦
-----------------------------

涂鸦功能允许您进行画笔的涂鸦操作和贴图包的涂鸦操作，您在涂鸦的同时，另一端可以看到实时的涂鸦画面，涂鸦的详细集成请参考 :ref:`涂鸦<涂鸦>`。


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _推送(iOS):

推送
-----------------------------

通过集成推送，可以将通话信息即时告知用户，从而提高通话的接通率。推送分为 Android 端的小米推送、华为推送以及苹果端的 VoIP 推送，详细集成步骤请参考 :ref:`推送<推送>` 模块。
