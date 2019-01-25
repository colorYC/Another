
.. _视频采集和渲染(Windows):

视频采集、渲染和设备控制
=============================

.. highlight:: csharp


视频采集、渲染和设备控制涉及到以下两个类：

.. _JCMediaDevice(windows):

- JCMediaDevice ： 主要用于媒体设备的控制和管理。详细信息请参考 `JCMediaDevice 类说明 <http://developer.juphoon.com/portal/reference/windows/html/034d5af6-ec04-5148-7ec5-04e27e93e8c2.htm>`_。

.. _JCMediaDeviceVideoCanvas(windows):

- JCMediaDeviceVideoCanvas ：主要用于界面层操作视频。如渲染的控制、视图旋转、视频通话截图等。详细信息请参考 `JCMediaDeviceVideoCanvas 类说明 <http://developer.juphoon.com/portal/reference/windows/html/6a5b853c-d890-c30e-d236-5728d789ace1.htm>`_。


**初始化 JCMediaDevice 模块**

::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);           
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);  

视频采集设置
------------------------

- 设置要开启的摄像头类型

视频采集前，可以指定要开启的摄像头
::

    /// <summary>
    /// 指定要开启的摄像头
    /// </summary>
    /// <param name="camera">摄像头对象</param>
    public bool specifyCamera(JCMediaDeviceCamera camera)

- 设置摄像头采集分辨率

您可以通过自定义摄像头采集参数实现不同的视频分辨率，如采集的高度、宽度和帧速率。

摄像头采集属性设置接口如下：

::

    /// <summary>
    /// 设定摄像头分辨率，请在调用startCamera()接口之前调用才会生效
    /// </summary>
    /// <param name="width">摄像头分辨率宽</param>
    /// <param name="height">摄像头分辨率高</param>
    /// <param name="framerate">帧速率</param>
    public void setCameraProperty(int width, int height, int framerate)

**示例代码**

::

    // 设置要开启的摄像头
    mediaDevice.specifyCamera(mediaDevice.cameraDevices[0]);
    
    // 设置摄像头采集属性
    mediaDevice.setCameraProperty(640, 360, 30);


.. _创建本地和远端视图画面(windows):

创建本地和远端视图画面
----------------------------

.. _创建本地视图画面(windows):

- 本地视图渲染

进行视图渲染前可通过 :ref:`获取摄像头列表<获取摄像头列表(windows)>` 接口获取摄像头列表。

本地视图渲染通过调用 startCameraVideo 接口获得本地视频对象用于 UI 界面显示，**该接口会打开摄像头**
::

    /// <summary>
    /// 获取预览视频对象，通过此对象能获得视图用于UI显示
    /// </summary>
    /// <param name="camera">摄像头对象</param>
    /// <param name="mode">渲染方式</param>
    /// <returns>JCMediaDeviceVideoCanvas对象</returns>
    public JCMediaDeviceVideoCanvas startCameraVideo(JCMediaDeviceCamera camera, JCMediaDeviceRenderMode mode)


.. _渲染模式(windows):

其中，渲染模式（JCMediaDeviceRender)有以下三种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - FULLSCREEN
     - 铺满窗口
   * - FULLCONTENT
     - 全图像显示，会有黑边，但在窗口跟图像比例相同的情况下不会有黑边
   * - AUTO
     - 自适应


.. _创建远端视图画面(windows):

- 远端视图渲染

您可以调用 startVideo 方法获取对端视频对象并进行渲染
::

    /// <summary>
    /// 获得视频对象，通过此对象能获得视图用于UI显示
    /// </summary>
    /// <param name="videoSource">渲染标识串，比如JCMediaChannelParticipant JCCallItem中的renderId</param>
    /// <param name="mode">渲染模式</param>
    /// <returns>JCMediaDeviceVideoCanvas对象</returns>
    public JCMediaDeviceVideoCanvas startVideo(string videoSource, JCMediaDeviceRenderMode mode)


**示例代码**

::

    // 获取摄像头列表
    List<JCMediaDeviceCamera> cameraDevices = mediaDevice.cameraDevices;
    
    // 打开本地视图预览
    JCMediaDeviceVideoCanvas localCanvas = mediaDevice.startCameraVideo(mediaDevice.cameraDevices[0], JCMediaDevice.JCMediaDeviceRenderMode.FULLCONTENT);  
    ImageBrush image = new ImageBrush(localCanvas.videoView);
    image.Stretch = Stretch.Uniform;
    this.label.Background = image;
    
    // 远端视频渲染，renderId来源于通话对象，一对一为JCCallItem对象，多方为JCMediaChannelParticipant对象        
    JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(renderId, JCMediaDevice.JCMediaDeviceRenderMode.FULLSCREEN);
    ImageBrush image = new ImageBrush(remoteCanvas.videoView);
    image.Stretch = Stretch.Uniform;
    this.label.Background = image;


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _销毁本地和远端视图画面(windows):

销毁本地和远端视图画面
----------------------------

在视频通话结束或者视频通话中，如果想销毁视频画面，可以调用下面的接口
::

    /// <summary>
    /// 停止视频
    /// </summary>
    /// <param name="canvas">JCMediaDeviceVideoCanvas对象，由startVideo获得</param>
    public void stopVideo(JCMediaDeviceVideoCanvas canvas)


示例代码::

    JCMediaDeviceVideoCanvas localCanvas = mediaDevice.startCameraVideo(mediaDevice.cameraDevices[0], JCMediaDevice.JCMediaDeviceRenderMode.FULLCONTENT);
    JCMediaDeviceVideoCanvas remoteCanvas = mediaDevice.startVideo(renderId, JCMediaDevice.JCMediaDeviceRenderMode.FULLSCREEN);
    if (localCanvas != null)
        {
            this.smvideoGrid.Background = null;
            mediaDevice.stopVideo(localCanvas);
            localCanvas = null;
        }
    if (remoteCanvas != null)
        {
            this.fullvideoGrid.Background = null;
            mediaDevice.stopVideo(remoteCanvas);
            remoteCanvas = null;
        }

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

渲染控制
---------------------------

- 暂停渲染

如果想替换当前摄像头视图画面，可以调用下面的接口
::

    /// <summary>
    /// 更新视频渲染标识
    /// </summary>
    /// <param name="videoSource">渲染标识</param>
    /// <returns>成功失败</returns>
    public bool replace(string videoSource)


- 暂停渲染

如果想暂停画面的渲染可以调用如下接口
::

    /// <summary>
    /// 暂停渲染
    /// </summary>
    /// <returns>成功失败</returns>
    public bool pause()


- 恢复渲染

如果想对已暂停的画面继续进行渲染，可以调用下面的接口
::

    /// <summary>
    /// 恢复渲染
    /// </summary>
    /// <returns>成功失败</returns>
    public bool resume()


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _设备控制(windows):

设备控制
------------------------


**1. 获取设备列表**

.. _获取窗口列表(windows):

- 获取窗口列表
::

    /// <summary>
    /// 窗口列表
    /// </summary>
    public List<JCMediaDeviceWindow> windowsDevices

其中，JCMediaDeviceWindow 有以下几个变量
::

    /// <summary>
    /// 窗口
    /// </summary>
    public class JCMediaDeviceWindow {
        /// <summary>
        /// 窗口名称
        /// </summary>
        public string windowName;
        /// <summary>
        /// 窗口id
        /// </summary>
        public string windowId;
    }


.. _获取桌面列表(windows):

- 获取桌面列表
::

    /// <summary>
    /// 桌面列表
    /// </summary>
    public List<JCMediaDeviceDesktop> desktopDevices

其中，JCMediaDeviceDesktop 有以下几个变量
::

    /// <summary>
    /// 桌面
    /// </summary>
    public class JCMediaDeviceDesktop {
        /// <summary>
        /// 桌面名称
        /// </summary>
        public string desktopName;
        /// <summary>
        /// 桌面id
        /// </summary>
        public string desktopId;
    }

- 获取音频输入设备列表
::

    /// <summary>
    /// 音频输入设备列表
    /// </summary>
    public List<JCMediaDeviceInput> audioInputDevices

其中，JCMediaDeviceInput 有以下几个变量
::
    
    /// <summary>
    /// 话筒
    /// </summary>
    public class JCMediaDeviceInput {
        /// <summary>
        /// 话筒名称
        /// </summary>
        public string inputName;
        /// <summary>
        /// 话筒id
        /// </summary>
        public string inputId;
    }


- 获取音频输出设备列表
::

    /// <summary>
    /// 音频输出设备列表
    /// </summary>
    public List<JCMediaDeviceOutput> audioOutputDevices

其中，JCMediaDeviceOutput 有以下几个变量::

    /// <summary>
    /// 扬声器
    /// </summary>
    public class JCMediaDeviceOutput {
        /// <summary>
        /// 扬声器名称
        /// </summary>
        public string outputName;
        /// <summary>
        /// 扬声器id
        /// </summary>
        public string outputId;
    }


.. _获取摄像头列表(windows):

- 获取摄像头列表
::

    /// <summary>
    /// 摄像头列表
    /// </summary>
    public List<JCMediaDeviceCamera> cameraDevices

其中，JCMediaDeviceCamera 有以下几个变量
::

    /// <summary>
    /// 摄像头
    /// </summary>
    public class JCMediaDeviceCamera {
        /// <summary>
        /// 摄像头名称
        /// </summary>
        public string cameraName;
        /// <summary>
        /// 摄像头id
        /// </summary>
        public string cameraId;
    }

示例代码::

    // 获取窗口列表
    List<JCMediaDeviceWindow> windowsDevices = mediaDevice.windowsDevices;

    // 获取桌面列表
    List<JCMediaDeviceDesktop> desktopDevices = mediaDevice.desktopDevices;

    // 获取音频输入设备列表
    List<JCMediaDeviceInput> audioInputDevices = mediaDevice.audioInputDevices;

    // 获取音频输出设备列表
    List<JCMediaDeviceOutput> audioOutputDevices = mediaDevice.audioOutputDevices;

    // 获取摄像头列表
    List<JCMediaDeviceCamera> cameraDevices = mediaDevice.cameraDevices;


**2. 音频设备**

- 开启/关闭音频设备
::

    /// <summary>
    /// 启动音频，一般正式开启通话前需要调用此接口
    ///</summary>
    ///<returns>启动成功失败</returns>
    public bool startAudio()

    /// <summary>
    /// 停止音频，一般在通话结束时调用
    /// </summary>
    /// <returns>停止音频成功失败</returns>
    public bool stopAudio()


- 打开输入设备
::

    /// <summary>
    /// 打开输入设备
    /// </summary>
    /// <param name="input">输入设备</param>
    /// <returns>打开输入设备成功失败</returns>
    public bool startInput(JCMediaDeviceInput input)


- 关闭输入设备
::

    /// <summary>
    /// 关闭输入设备
    /// </summary>
    /// <returns>关闭输入设备成功失败</returns>
    public bool stopInput(JCMediaDeviceInput input)


- 打开输出设备
::

    /// <summary>
    /// 打开输出设备
    /// </summary>
    /// <param name="output">输出设备</param>
    /// <returns> 打开输出设备成功失败</returns>
    public bool startOutput(JCMediaDeviceOutput output)


- 关闭输出设备
::

    /// <summary>
    /// 关闭输出设备
    /// </summary>
    /// <returns>关闭输出设备成功失败</returns>
    public bool stopOutput()


**3. 视频设备**

- 开启关闭摄像头
::

    /// <summary>
    /// 开启摄像头
    /// </summary>
    /// <returns>true为开启成功，false为开启失败</returns>
    public bool startCamera()

    /// <summary>
    /// 关闭摄像头
    /// </summary>
    /// <returns>true为关闭成功，false为关闭失败</returns>
    public bool stopCamera()


- 切换摄像头

::

    /// <summary>
    /// 切换摄像头
    /// </summary>
    /// <param name="camera">要切换的摄像头</param>
    /// <returns>true为切换成功，false为切换失败</returns>
    public bool switchCamera(JCMediaDeviceCamera camera)


示例代码::

    // 打开音频
    mediaDevice.startAudio();

    // 关闭音频
    mediaDevice.stopAudio();

    // 打开输入设备
    mediaDevice.startInput(mediaDevice.audioInputDevices[0]);

    // 打开输出设备
    mediaDevice.startOutput(mediaDevice.audioOutputDevices[0]); 

    // 打开摄像头
    mediaDevice.startCamera();

    // 关闭摄像头
    mediaDevice.stopCamera();

    // 切换摄像头
    mediaDevice.switchCamera(mediaDevice.cameraDevices[0]);
