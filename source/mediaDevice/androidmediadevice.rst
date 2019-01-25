
.. _视频采集和渲染(Android):

视频采集、渲染和设备控制
=============================

.. highlight:: java

视频采集、渲染和设备控制涉及到以下两个类：

.. _JCMediaDevice(android):

- JCMediaDevice ：主要用于媒体设备的控制和管理。详细信息请参考 `JCMediaDevice 类说明 <http://developer.juphoon.com/portal/reference/android/>`_。

.. _JCMediaDeviceVideoCanvas(android):

- JCMediaDeviceVideoCanvas ：主要用于界面层操作视频。如渲染的控制、视图旋转、视频通话截图等。详细信息请参考 `JCMediaDeviceVideoCanvas 类说明 <http://developer.juphoon.com/portal/reference/android/>`_。

**初始化 JCMediaDevice 模块**

::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);

视频采集设置
------------------------

- 设置要开启的摄像头类型

视频采集前，可以指定要开启的摄像头，如前置摄像头或者后置摄像头
::

    /**
     * 指定要开启的摄像头
     * @param cameraIndex   摄像头索引
     */
    public abstract void specifyCamera(int cameraIndex);

camera 的值为下面几个常量::

    // 未获取到摄像头
    public static final int CAMERA_NONE = 0;
    // 前置摄像头
    public static final int CAMERA_FRONT = 1;
    // 后置摄像头
    public static final int CAMERA_BACK = 2;
    // 未知摄像头
    public static final int CAMERA_UNKNOWN = 3;


- 设置摄像头采集分辨率

您可以通过自定义摄像头采集参数实现不同的视频分辨率，如采集的高度、宽度和帧速率。

摄像头采集属性设置接口如下：

::

    /**
     * 设置摄像头采集属性
     * @param width     采集宽度，默认640
     * @param height    采集高度，默认360
     * @param frameRate 采集帧速率，默认30
     */
    public abstract void setCameraProperty(int width, int height, int frameRate);

- 设置摄像头采集角度

您可以调用下面的接口设置摄像头采集的角度，其中，角度需为 90 的倍数
::

    /**
     * 指定摄像头采集角度，为90的倍数
     * @param angle 角度
     */
    public abstract void specifyCameraAngle(int angle);

**示例代码**

::

    // 指定要开启的摄像头
    mediaDevice.specifyCamera(0);
    // 设置摄像头采集属性
    mediaDevice.setCameraResolution(640, 360, 30);


^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _创建本地和远端视图画面(android):

创建本地和远端视图画面
----------------------------

.. _创建本地视图画面(android):

- 本地视图渲染

本地视图渲染通过调用 startCameraVideo 接口获得本地视频对象用于 UI 界面显示，**该接口会打开摄像头**
::

    /**
     * 获得视频预览对象，通过此对象能获得视图用于UI显示
     *
     * @param renderType    渲染模式
     * @return              JCMediaDeviceVideoCanvas 对象
     * @see RenderType
     */
    public abstract JCMediaDeviceVideoCanvas startCameraVideo(@RenderType int renderType);


.. _渲染模式(android):

其中，渲染模式（JCMediaDeviceRender)有以下三种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - public static final int RENDER_FULL_SCREEN = 0
     - 铺满窗口
   * - public static final int RENDER_FULL_CONTENT = 1
     - 全图像显示，会有黑边，但在窗口跟图像比例相同的情况下不会有黑边
   * - public static final int RENDER_FULL_AUTO = 2
     - 自适应


.. _创建远端视图画面(android):

- 远端视图渲染

您可以调用 startVideo 方法获取对端视频对象并进行渲染
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


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**示例代码**

::

    // 打开本地视图预览
    JCMediaDeviceVideoCanvas localCanvas = mediaDevice.startCameraVideo(JCMediaDevice.RENDER_FULL_CONTENT);
    viewGroup.addView(localCanvas.getVideoView(), 0);
    // 远端视频渲染，renderId来源于通话对象，一对一为JCCallItem对象，多方为JCMediaChannelParticipant对象
    JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(renderId, JCMediaDevice.RENDER_FULL_CONTENT);
    viewGroup.addView(remoteCanvas.getVideoView(), 0);

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _销毁本地和远端视图画面(android):

销毁本地和远端视图画面
----------------------------

在视频通话结束或者视频通话中，如果想销毁视频画面，可以调用下面的接口
::

    /**
     * 停止视频
     *
     * @param canvas JCMediaDeviceVideoCanvas 对象，由 startVideo 获得
     */
    public abstract void stopVideo(JCMediaDeviceVideoCanvas canvas);

示例代码::

    JCMediaDeviceVideoCanvas localCanvas = mediaDevice.startCameraVideo(JCMediaDevice.RENDER_FULL_CONTENT);
    JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(renderId, JCMediaDevice.RENDER_FULL_CONTENT);
    if (localCanvas != null) {
        mContentView.removeView(localCanvas.getVideoView());
        mediaDevice.stopVideo(localCanvas);
        localCanvas = null;
    
    if (remoteCanvas != null) {
        mContentView.removeView(remoteCanvas.getVideoView());
        mediaDevice.stopVideo(remoteCanvas);
        remoteCanvas = null;
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

渲染控制
---------------------------

如果暂停画面的渲染，可以调用下面的接口：

::

    /**
     * 暂停视频渲染
     */
    public void pause();


- 恢复渲染

如果想对已暂停的画面继续进行渲染，可以调用下面的接口：
::

    /**
     * 继续视频渲染
     */
    public void resume();


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _设备控制(android):


设备控制
------------------------

**1. 音频设备**

- 开启关闭扬声器

::

    /**
     * 开启关闭扬声器
     *
     * @param enable 是否开启
     */
    public abstract void enableSpeaker(boolean enable);


- 开启关闭音频设备

::

    /**
     * 启动音频，一般正式开启通话前需要调用此接口
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean startAudio();

    /**
     * 停止音频，一般在通话结束时调用
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean stopAudio();


**2. 视频设备**

- 获取摄像头列表

::

    /**
     * 获取摄像头列表
     *
     * @return 摄像头列表
     */
    public abstract List<String> getCameras();

示例代码
::

    获取摄像头列表
    List<String> cameras = mediaDevice.getCameras();

- 开启关闭摄像头

::

    /**
     * 开启摄像头
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean startCamera();

    /**
     * 关闭摄像头
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean stopCamera();


- 切换摄像头

::

    /**
     * 切换摄像头，内部会根据当前摄像头类型来进行切换
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean switchCamera();

**示例代码**

::

    // 开启扬声器
    mediaDevice.enableSpeaker(true);
    // 开启音频设备
    mediaDevice.startAudio();
    // 关闭音频设备
    mediaDevice.stopAudio();
    // 打开摄像头
    mediaDevice.startCamera();
    // 关闭摄像头
    mediaDevice.stopCamera();
    // 切换摄像头
    mediaDevice.switchCamera();


