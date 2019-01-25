
.. _通话状态更新(ios1-1):

通话状态更新
-----------------------------

.. highlight:: objective-c

通话过程中，如果通话状态发生了改变，如开启关闭静音、开启关闭通话保持、活跃状态切换、开启关闭视频流发送等，将会收到通话状态更新的回调
::

    /**
     *  @brief                  通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *  @param item             JCCallItem 对象
     *  @param changeParam      更新标识类
     */
    -(void)onCallItemUpdate:(JCCallItem*)item changeParam:(JCCallChangeParam *)changeParam;

关于 JCCallChangeParam 的说明请参考 JCCallItem.h 文件。

.. note::
     
       静音状态、通话保持状态、活跃状态、视频流发送状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获得。

示例代码::

    -(void)onCallItemUpdate:(JCCallItem*)item changeParam:(JCCallChangeParam *)changeParam {
        if (item.mute) { // 开启静音
            ...
        } else if (item.hold) { // 挂起通话
            ...
        } else if (item.held) { // 被挂起
            ...
        } else if (item.active) { // 激活状态
            ...
        } else if (item.uploadVideoStreamSelf) { // 本端在上传视频流
            ...
        } else if (item.uploadVideoStreamOther) { // 远端在上传视频流
            ...
        } 
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _通话过程控制(ios1-1):

通话过程控制
-----------------------------

**通话静音**

您可以通过下面的方法开启或关闭静音，开启关闭静音需要根据 JCCallItem 中的静音状态（`mute <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/mute>`_）来决定，静音开启后，对方将听不到您的声音
::

    /**
     *  @brief                  静音，通过 JCCallItem 对象中的静音状态来决定开启关闭静音
     *  @param item             JCCallItem 对象
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)mute:(JCCallItem*)item;


**开启关闭呼叫保持**

您可以调用下面的方法对通话对象进行呼叫保持或解除呼叫保持，开启或关闭呼叫保持需要根据 JCCallItem 对象中（`hold <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/hold>`_）的呼叫保持状态来决定
::

    /**
     *  @brief                  呼叫保持，通过 JCCallItem 对象中的呼叫保持状态来决定开启关闭呼叫保持
     *  @param item             JCCallItem 对象
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)hold:(JCCallItem*)item;


**开启关闭视频流发送**

您可以调用下面的方法开启或关闭视频流发送，调用后对端会 **收到 onCallItemUpdate 回调**

::

    /**
     *  @brief          开启关闭视频流发送，用于视频通话中
     *  @param item     JCCallItem 对象
     *  @return         返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableUploadVideoStream:(JCCallItem*)item;


该接口的具体作用机制如下图所示：

.. image:: enablevideostream.png

- 如果 A 开启发送视频流，则 A 的 uploadVideoStreamSelf 属性值为 true，B 则通过 uploadVideoStreamOther 属性值（此处为true）判断 A 的视频流发送状态。

- 如果 A 关闭发送视频流，则 A 的 uploadVideoStreamSelf 属性值为 false，B 则通过 uploadVideoStreamOther 属性值（此处为false）判断 A 的视频流发送状态。此时 B 将看不到 A 的画面。

示例代码
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];
    JCMediaChannel *mediaChannel = [JCMediaChannel create:client mediaDevice:mediaDevice callback:self];
    JCCallItem *item = call.callItems[0];
    // 开启或关闭静音
    [call mute:item];
    // 开启关闭呼叫保持
    [call hold:item];
    // 开启关闭视频流发送
    [call enableUploadVideoStream:item];

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _获取网络状态(ios1-1):

获取网络状态
----------------------------

当网络状态发生变化时，会收到 onNetChange 回调
::

    /**
     *  @brief 网络变化
     *  @param judgeType 网络判断类型
     *  @param newNetType 当前网络类型
     *  @param oldNetType 之前网络类型
     *  @see JCNetJudgeType JCNetType
     */
    -(void)onNetChange:(JCNetJudgeType)judgeType newNetTyp:(JCNetType)newNetType oldNetType:(JCNetType)oldNetType;

可以通过下面的方法获取网络状态
::

    - (NSString *)genNetStatus:(JCCallItem *)item {
        if (item.state != JCCallStateTalking) {
            return @"";
        }
        switch (item.netStatus) {
            case JCCallNetWorkDisconnected:
                return @"无网络";
            case JCCallNetWorkVeryBad:
                return @"很差";
            case JCCallNetWorkBad:
                return @"差";
            case JCCallNetWorkNormal:
                return @"一般";
            case JCCallNetWorkGood:
                
                return @"好";
            case JCCallNetWorkVeryGood:
                return @"非常好";
            default:
                return @"";
        }
    }


