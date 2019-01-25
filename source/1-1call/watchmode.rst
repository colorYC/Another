手表模式
=========================

一对一通话支持手表模式，您的 Android 手机或者 iPhone 可以和手表进行互通。**如果您已经集成了一对一通话**，则在此基础上只需添加简单的配置即可支持手表模式。

手表模式接口简洁，只需调用极少的接口即可集成，下面将介绍在手表模式下， iPhone 和 Android 手机以及 Android 手表的集成。

手机端设置
-------------------------

iOS
>>>>>>>>>>>>>>>>>>>>>>>>>>

.. highlight:: objective-c

手表模式下，可以在 **发起通话前** 调用 setCameraProperty 方法对视频采集参数进行设置
::

    // 设置帧率为10
    JCMediaDevice *mediaDevice = [[JCMediaDevice create:client callback:self];
    [mediaDevice setCameraProperty:640 height:480 framerate:10];
    // 设置帧率为15
    [mediaDevice setCameraProperty:640 height:480 framerate:15];

针对视频质量，可以在 **发起通话前** 调用下面的方法设置抗锯齿
::

    /**
     *  @brief 设置抗锯齿
     *  @param sawtooth 是否抗锯齿
     */
    -(void)setSawtooth:(bool)sawtooth;

示例代码
::

    JCMediaDevice *mediaDevice = [[JCMediaDevice create:client callback:self];
    [mediaDevice setSawtooth:true];

调用以上两个方法即可完成手表模式下 iPhone 手机的适配。

Android
>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. highlight:: java

首先在 **登录前** 设置手表模式为 false
::

    /**
     * 设置手表模式, 登录前设置
     * @param watchMode
     */
    public abstract void setWatchMode(boolean watchMode);

示例代码::

    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setWatchMode(false);

手表模式下，可以在 **发起通话前** 调用 setCameraProperty 方法对视频采集参数进行设置
::

    // 设置帧率为10
    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setCameraProperty(640, 480, 10);
    // 设置帧率为15
    mediaDevice.setCameraProperty(640, 480, 15);

针对视频质量，可以在 **发起通话前** 调用下面的方法设置抗锯齿
::

    /**
     * 设置抗锯齿
     *
     * @param sawtooth 是否抗锯齿
     */
    public abstract void setSawtooth(boolean sawtooth);

示例代码
::

    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setSawtooth(true);

调用以上两个方法即可完成手表模式下 Android 手机的适配。


手表端集成
----------------------

手表端 SDK 集成调用以下接口：

设置手表模式，需要在 **登录前** 设置
::

    /**
     * 设置手表模式, 登录前设置
     * @param watchMode
     */
    public abstract void setWatchMode(boolean watchMode);

示例代码::

    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setWatchMode(true);

获取手表模式，通过该方法判断是否手表模式
::

    /**
     * 获取是否是手表模式
     *
     * @return true 表示手表模式,false 表示手机模式
     */
    public abstract boolean getWatchMode();

示例代码::

    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.getWatchMode();


在 **发起视频通话前** 调用 setCameraProperty 方法对视频采集参数进行设置
::

    // 设置帧率为10
    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setCameraProperty(352, 282, 10);
    // 设置帧率为15
    mediaDevice.setCameraProperty(352, 282, 15);

针对视频质量，可以在 **发起通话前** 调用下面的方法设置抗锯齿
::
 
    /**
     * 设置抗锯齿 
     *
     * @param sawtooth 是否抗锯齿
     */
    public abstract void setSawtooth(boolean sawtooth);

示例代码
::

    JCMediaDevice mediaDevice = JCMediaDevice.create(client,this);
    mediaDevice.setSawtooth(true);