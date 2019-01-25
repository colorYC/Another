
.. _视频采集和渲染(ios):

视频采集、渲染和设备控制
=============================

.. highlight:: objective-c


视频采集、渲染和设备控制涉及到以下两个类：

.. _JCMediaDevice:

- JCMediaDevice ： 主要用于媒体设备的控制和管理。详细信息请参考 `JCMediaDevice 类说明 <http://developer.juphoon.com/portal/reference/ios/Classes/JCMediaDevice.html>`_。

.. _JCMediaDeviceVideoCanvas:

- JCMediaDeviceVideoCanvas ：主要用于界面层操作视频。如渲染的控制、视图旋转、视频通话截图等。详细信息请参考 `JCMediaDeviceVideoCanvas 类说明 <http://developer.juphoon.com/portal/reference/ios/Classes/JCMediaDeviceVideoCanvas.html>`_。

**初始化 JCMediaDevice 模块**

::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];

视频采集设置
---------------------------

- 设置要开启的摄像头类型

视频采集前，可以指定要开启的摄像头，如前置摄像头或者后置摄像头
::

    /**
     *  @breif 指定要开启的摄像头
     *  @param camera 摄像头标识
     */
    - (void)specifyCamera:(NSString *)camera;

camera 的值为下面几个变量的值::

    // 前置摄像头
    extern NSString* JCMediaDeviceCameraFront;
    // 后置摄像头
    extern NSString* JCMediaDeviceCameraBack;

- 设置摄像头采集分辨率

您可以通过自定义摄像头采集参数实现不同的视频分辨率，如采集的高度、宽度和帧速率。

摄像头采集属性设置接口如下：

::

    /**
     *  @breif 设置摄像头采集属性
     *  @param width 采集宽度，默认640
     *  @param height 采集高度，默认360
     *  @param framerate 帧速率，默认30
     */
    - (void)setCameraProperty:(int)width height:(int)height framerate:(int)framerate;

- 设置摄像头采集角度

您可以调用下面的接口设置摄像头采集的角度，其中，角度需为 90 的倍数
::

    /**
     * @breif 指定摄像头采集角度，为90的倍数
     * @param angle 角度
     */
    -(void)specifyCameraAngle:(int)angle;

**示例代码**

::

    // 设置要开启的摄像头类型
    [mediaDevice specifyCamera:JCMediaDeviceCameraFront];

    // 设置摄像头采集属性
    [mediaDevice setCameraProperty:640 height:360 framerate:30];

    // 设置摄像头采集角度
    [mediaDevice specifyCameraAngle:90];


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _创建本地和远端视图画面:


创建本地和远端视图画面
----------------------------

.. _创建本地视图画面:

- 本地视图渲染

本地视图渲染通过调用 startCameraVideo 接口获得本地视频对象用于 UI 界面显示，**该接口会打开摄像头**
::

    /**
     *  @brief 获得预览视频对象，通过此对象能获得视图用于UI显示
     *  @param type 渲染模式，@ref JCMediaDeviceRender
     *  @return JCMediaDeviceVideoCanvas 对象
     */
    -(JCMediaDeviceVideoCanvas*)startCameraVideo:(int)type;


.. _渲染模式:

其中，渲染模式（JCMediaDeviceRender)有以下三种：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - JCMediaDeviceRenderFullScreen = 0
     - 铺满窗口
   * - JCMediaDeviceRenderFullContent
     - 全图像显示，会有黑边，但在窗口跟图像比例相同的情况下不会有黑边
   * - JCMediaDeviceRenderFullAuto
     - 自适应

.. _创建远端视图画面:

- 远端视图渲染

您可以调用 startVideo 方法获取对端视频对象并进行渲染
::

    /**
     *  @brief 获得预览视频对象，通过此对象能获得视图用于UI显示
     *  @param videoSource 渲染标识串，比如 JCMediaChannelParticipant JCCallItem 中的 renderId
     *  @param type        渲染模式，@ref JCMediaDeviceRender
     *  @return JCMediaDeviceVideoCanvas 对象
     */
    -(JCMediaDeviceVideoCanvas*)startVideo:(NSString*)videoSource renderType:(int)type;

**示例代码**

::
    
    // 创建本地视频画面对象
    JCMediaDeviceVideoCanvas *local = [mediaDevice startCameraVideo:JCMediaDeviceRenderFullContent];
    local.videoView.frame = CGRectMake(0, 0, 100, 100);
    [self.view addSubview:local.videoView];
    
    // 创建远端视频画面对象，renderId来源于通话对象，一对一为JCCallItem对象，多方为JCMediaChannelParticipant对象
    JCMediaDeviceVideoCanvas *remote = [mediaDevice startVideo:renderId renderType:JCMediaDeviceRenderFullContent];
    remote.videoView.frame = CGRectMake(100, 0, 100, 100);
    [self.view addSubview:remote.videoView];


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _销毁本地和远端视图画面:

销毁本地和远端视图画面
----------------------------

在视频通话结束或者视频通话中，如果想销毁视频画面，可以调用下面的接口
::

    /**
     *  @brief 停止视频
     *  @param canvas JCMediaDeviceVideoCanvas 对象，由 startVideo 获得
     */
    -(void)stopVideo:(JCMediaDeviceVideoCanvas*)canvas;

示例代码::

    JCMediaDeviceVideoCanvas *localCanvas = [mediaDevice startCameraVideo:JCMediaDeviceRenderFullContent];
    JCMediaDeviceVideoCanvas *remoteCanvas = [mediaDevice startVideo:renderId renderType:JCMediaDeviceRenderFullContent];
    if (localCanvas) {
        // 移除本地视图
        [mediaDevice stopVideo:localCanvas];
        [localCanvas.videoView removeFromSuperview];
        localCanvas = nil;
    }
    if (remoteCanvas) {
        // 移除远端视图
        [mediaDevice stopVideo:remoteCanvas];
        [remoteCanvas.videoView removeFromSuperview];
        remoteCanvas = nil;
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

渲染控制
---------------------------

- 暂停渲染

如果想暂停画面的渲染可以调用如下接口：

::

    /**
     *  @brief 暂停渲染
     *  @return 成功返回 true，失败返回 false
     */
    -(void)pause;

- 恢复渲染

如果想对已暂停的画面继续进行渲染，可以调用下面的接口：
::

    /**
     *  @brief 恢复渲染
     *  @return 成功返回 true，失败返回 false
     */
    -(void)resume;


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _设备控制(ios):


设备控制
--------------------------------

**1. 音频设备**

- 开启/关闭扬声器

UI 通过下面的方法开启和关闭扬声器::

    /**
     *  @brief 开启关闭扬声器
     *  @param enable 是否开启
     */
    -(void)enableSpeaker:(bool)enable;


- 开启/关闭音频设备
::

    /**
     *  @brief 启动音频，一般正式开启通话前需要调用此接口
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)startAudio;

    /**
     *  @brief 停止音频，一般在通话结束时调用
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)stopAudio;


**2. 视频设备**

- 开启关闭摄像头

::

    /**
     *  @breif 开启摄像头，一般在只需开启摄像头时调用
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)startCamera;

    /**
     *  @breif 关闭摄像头，一般和 startCamera 配对使用
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)stopCamera;


- 切换摄像头

::

    /**
     *  @breif 切换前后摄像头，内部会根据当前摄像头类型来进行切换
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)switchCamera;


**示例代码**

::

    // 开启关闭扬声器
    [mediaDevice enableSpeaker:true];

    // 关闭音频设备
    [mediaDevice stopAudio];

    // 开启音频设备
    [mediaDevice startAudio]

    // 打开摄像头
    [mediaDevice startCamera];

    // 关闭摄像头
    [mediaDevice stopCamera];

    // 切换摄像头
    [mediaDevice switchCamera];
