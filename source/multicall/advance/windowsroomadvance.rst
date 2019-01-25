Windows 进阶
==========================

.. highlight:: csharp

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);           
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);             
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);

.. _查询频道(windows):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /// <summary>
    /// 查询频道相关信息，例如是否存在、人数等
    /// </summary>
    /// <param name="channelId">频道标识</param>
    /// <returns>操作Id，与onQuery回调中的operationI对应</returns>
    public int query(string channelId)

示例代码

::

    mediaChannel.query("channelId");


查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /// <summary>
    /// 查询频道信息结果回调
    /// </summary>
    /// <param name="operationId">操作id，由query接口返回</param>
    /// <param name="result">ture表示查询成功，false表示查询失败</param>
    /// <param name="reason">查询失败原因，当result为false时该值有效</param>
    /// <param name="queryInfo">查询到的频道信息</param>
    public void onQuery(int operationId, bool result, JCMediaChannelReason reason, JCMediaChannelQueryInfo queryInfo);

示例代码::

    public void onQuery(int operationId, bool result, JCMediaChannelReason reason, JCMediaChannelQueryInfo queryInfo) {
       // 查询成功
       if (result) {
            // 查询频道标识
            String channelId = queryInfo.channelId;
            // 查询频道号
            int number = queryInfo.number;
            // 查询频道成员数
            int clientCount = queryInfo.clientCount;
            // 查询频道成员列表
            List<string> members = queryInfo.members;
       } else {
            // 查询失败
       }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _屏幕共享(windows):

桌面或窗口共享
----------------------

桌面或窗口可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用桌面或窗口共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

开启桌面或窗口共享前可以调用 :ref:`获取桌面列表<获取桌面列表(windows)>` 接口或者 :ref:`获取窗口列表<获取窗口列表(windows)>` 接口获取桌面列表和窗口列表。

设备列表获取后可通过下面的接口开启或关闭桌面或窗口共享
::

    /// <summary>
    /// 开启关闭桌面屏幕共享，内部根据当前状态决定是否开启
    /// </summary>
    /// <param name="enable">是否开启屏幕共享</param>
    /// <param name="videoSource">桌面或窗口id</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableScreenOrWindowShare(bool enable, string videoSource);
   

示例代码

::

    // 获取桌面列表
    List<JCMediaDeviceDesktop> desktopDevices = mediaDevice.desktopDevices;
    // 开启或关闭桌面或窗口共享
    mediaChannel.enableScreenOrWindowShare(mediaDevice.desktopDevices[0]);


您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /// <summary>
    /// 设置屏幕桌面共享采集属性
    /// </summary>
    /// <param name="width">采集宽度</param>
    /// <param name="height">采集高度</param>
    /// <param name="framerate">帧速率</param>
    public void setScreenCaptureProperty(int width, int height, int framerate)

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 screenUserId 属性获得发起屏幕共享的用户标识。
::

    /// <summary>
    /// 属性变化回调，目前主要关注屏幕共享和窗口共享状态的更新
    /// </summary>
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);

此时可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::
    
    /// <summary>
    /// 请求屏幕共享的视频流
    /// 当pictureSize未None表示关闭请求
    /// </summary>
    /// <param name="screenUri">屏幕分享uri</param>
    /// <param name="pictureSize">视频请求尺寸类型</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool requestScreenVideo(string screenUri, JCMediaChannelPictureSize pictureSize)

示例代码

::

    public void onMediaChannelPropertyChange() {
        // 请求屏幕共享的视频流
        JCMediaDeviceVideoCanvas canvas = mediaDevice.startVideo(screenURI,JCMediaDevice.JCMediaDeviceRenderMode.FULLCONTENT);
        mediaChannel.requestScreenVideo(screenURI, JCMediaChannelPictureSize.Large);
    }
    

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CDN 推流(windows):


CDN 推流
----------------------

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

**开启 CDN 推流**

如要开启 CDN 推流，需在加入频道前进行 CDN 推流地址的设置。

只有 CDN 当前状态不为 JCMediaChannelCdnStateNone 时才可以进行 CDN 推流。其中，CDN 推流状态有以下几种：
::

    /// 无法进行CDN推流
    None,
    /// 可以开启CDN推流
    Ready,
    /// CDN推流中
    Running


开启或关闭 CDN 推流调用如下接口
::

    /// <summary>
    /// 开关Cdn推流，内部根据当前状态决定是否开启
    /// 在收到onMediaChannelPropertyChange回调时检查cdnState
    /// </summary>
    /// <param name="enable">是否开启cdn推流</param>
    /// <param name="keyInterval">推流关键帧间隔(毫秒)，当 enable 为 true 时有效，-1表示使用默认值(5000毫秒)</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableCdn(bool enable, int keyInterval)

示例代码

::

    // 设置 CDN 推流地址
    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_CDN, "your cdnurl");
    // 加入频道
    mediaChannel.join("channelId", joinparams);
    public void onJoin(bool result, JCMediaChannelReason reason, string channelId) {
        // 根据CDN推流状态判断是否开启推流
        if (mediaChannel.cdnState == JCMediaChannelCdnState.None) {
            // 无法使用 CDN 推流
        } else if (mediaChannel.cdnState == JCMediaChannelCdnState.Ready) {
            // 可以开启 CDN 推流
            mediaChannel.enableCdn(true);
        } else if (mediaChannel.cdnState == JCMediaChannelCdnState.Running) {
            // CDN 推流中，可以关闭 CDN 推流
            mediaChannel.enableCdn(false);
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音视频录制(windows):

服务器音视频录制
------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前设置录制参数，然后在加入频道的时候传入录制参数。

录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有::

    /// 无法进行视频录制
    None,
    /// 可以开启视频录制
    Ready,
    /// 视频录制中
    Running

录制状态获取后，即可调用下面的接口开启或关闭音视频录制
::

    /// <summary>
    /// 开关视频录制，内部根据当前状态决定是否开启
    /// <param name="enable">是否开启屏幕录制</param>
    /// </summary>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableRecord(bool enable)
   

示例代码

::

    // 设置录制参数
    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_RECORD, "{\"MtcConfIsVideoKey\":\"true\",
                 \"Storage\":{
                 \"Protocol\":\"qiniu\",
                 \"BucketName\": \"用户填入\",
                 \"SecretKey\": \"用户填入\",
                 \"AccessKey\": \"用户填入\", 
                 \"FileKey\": \" * *.mp4\"
                 }
             }"
        );
    string recordParam = JCConfUtils.qiniuRecordParam(Properties.Settings.Default.RecordVideo, Properties.Settings.Default.RecordBucketName, Properties.Settings.Default.RecordSecretKey, Properties.Settings.Default.RecordAccessKey, Properties.Settings.Default.RecordFileName);
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_REGION, JCMediaChannelRegion.REGION_CHINA.ToString());
    // 加入频道
    mediaChannel.join("channelId", joinparams);
    public void onJoin(bool result, JCMediaChannelReason reason, string channelId) {
        // 根据音视频录制状态判断是否开启音视频录制
        if (mediaChannel.recordState == JCMediaChannelRecordState.None) {
            // 无法进行音视频录制
        } else if (mediaChannel.recordState == JCMediaChannelRecordState.Ready) {
            // 可以开启音视频录制
            mediaChannel.enableRecord(true);
        } else if (mediaChannel.recordState == JCMediaChannelRecordState.Running) {
            // 音视频录制中，可以关闭音视频录制
            mediaChannel.enableRecord(false);
        }
    }

.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 false。


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(windows):


发送消息
-------------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /// <summary>
    /// 频道中发送消息，当 toUserId 不为 null 时，content 不能大于 4k
    /// </summary>
    /// <param name="type">消息类型</param>
    /// <param name="content">消息内容</param>
    /// <param name="toUserId">接收方成员的userid，值为null发送给所有人</param>
    /// <returns>是否发送成功</returns>
    public bool sendMessage(string type,string content,string toUserId)

其中，消息类型（type）为自定义类型。

示例代码

::

    public void onJoin(bool result, JCMediaChannelReason reason, string channelId) {
        // 发送给所有成员
        mediaChannel.sendMessage("text", "content", null);
        // 发送给某个成员
        mediaChannel.sendMessage("text", "content", "userId");
    }


当频道中的其他成员收到消息时，会收到 onMessageReceive 回调
::

    /// <summary>
    /// 接收频道消息的回调
    /// </summary>
    /// <param name="type">消息类型</param>
    /// <param name="content">消息内容</param>
    /// <param name="fromUserId">消息发送成员userId</param>
    public void onMessageReceive(string type, string content, string fromUserId);


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送指令(windows):

发送指令
-----------------------------

频道中可以发送控制指令，如批量修改成员状态，角色，昵称、设置推流布局模式等。

发送指令命令调用 sendCommand 接口
::
 
    /// </summary
    /// 发送指令
    /// </summary>
    /// <param name="name">指令名</param>
    /// <param name="param">指令参数</param>
    /// <returns>是否发送成功</returns>
    public bool sendCommand(string name, string param)


示例代码

::

    public void onJoin(bool result, JCMediaChannelReason reason, string channelId) { {
        // 发送修改频道标题指令
        mediaChannel.sendCommand("ChangeTitle", "{"MtcConfTitleKey":"321"}");
    }


指令名称和指令参数详细信息如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - StartForward

       请求服务器开始转发音视频
     - 参数格式：{"MtcConfUserUriKey": "用户Uri", "MtcConfMediaOptionKey": "类型"}
        - 用户Uri：通过调用底层Mtc接口获取 MtcUser.Mtc_UserFormUri(EN_MTC_USER_ID_USERNAME, userId);
        - 类型：服务器转发分三种 音频、视频、音视频，具体可参考底层mtc_conf.h下的MtcConfMedia的枚举值。
        - 注意1:指令发送成功后会收到 onParticipantUpdate 回调 

       举例：
       {"MtcConfUserUriKey": "[username:justin@sample.cloud.justalk.com]", "MtcConfMediaOptionKey": 3}
   * - StopForward

       请求服务器停止转发音视频
     - 参数格式：{"MtcConfUserUriKey": "用户URL", "MtcConfMediaOptionKey": "类型"}
        - 用户Uri：通过调用底层Mtc接口获取 MtcUser.Mtc_UserFormUri(EN_MTC_USER_ID_USERNAME, userId);
        - 类型：服务器转发分三种 音频、视频、音视频，具体可参考底层mtc_conf.h下的MtcConfMedia的枚举值。
        - 注意1:指令发送成功后会收到 onParticipantUpdate 回调 

       举例：
       {"MtcConfUserUriKey": "[username:justin@sample.cloud.justalk.com]", "MtcConfMediaOptionKey": 3}
   * - ChangeTitle

       请求修改会议主题
     - 参数格式：{"MtcConfTitleKey":"修改的内容"}
        - 修改的内容：比如原来主题设置的是"123"，现在改为"321"。
        - 注意1：指令发送成功后会收到 onMediaChannelPropertyChange 回调
        - 注意2：可通过 JCManager.shared().MediaChannel.title 获取主题
       举例：{"MtcConfTitleKey": "321"}
   * - SetPartpProp

       批量修改成员状态，角色，昵称
     - 参数格式：{"MtcConfStateKey"：要修改的成员状态,"MtcConfDisplayNameKey":"要修改的成员昵称","MtcConfPartpLstKey":["用户Uri",...],"MtcConfRoleKey":7}
        - 要修改的成员状态：具体可参考底层 mtc_conf.h 下的 MtcConfState 的枚举值
        - 要修改的成员角色：具体可参考底层 mtc_conf.h 下的 MtcConfRole 的枚举值
        - 要修改的成员昵称：比如"123"
        - 用户Uri：通过调用底层Mtc接口获取 MtcUser.Mtc_UserFormUri(EN_MTC_USER_ID_USERNAME, userId); 
        - 注意1：指令发送成功后会收到 onParticipantUpdate 回调 
        - 注意2：MtcConfStateKey、MtcConfDisplayNameKey、MtcConfRoleKey这三个字段，可根据用户想修改哪个值，就在json字符串里面加入哪个。
        - 注意3：MtcConfPrtpLstKey 可包含多个用户uri进行批量修改

       举例：
       {"MtcConfStateKey":1,"MtcConfDisplayNameKey":"1314","MtcConfPartpLstKey":["[username:10086@sample.cloud.justalk.com]"],"MtcConfRoleKey":7}
   * - ReplayApplyMode 

       设置推流布局模式
     - 参数格式：{"MtcConfCompositeModeKey": 参数值}
        - 参数值：
        - 1 平铺模式,所有视频均分平铺    
        - 2 讲台模式,共享为大图,其他视频为小图
        - 3 演讲模式,共享为大图,共享者视频为小图,其他不显示  
        - 4 自定义模式,由ReplayApplyLayout指令设置所有视频布局
        - 5 智能模式

       举例：
       输入指令参数{"MtcConfCompositeModeKey": 2}就是讲台模式
   * - ReplayApplyLayout

       为多用户设置自定义推流布局 
     - 参数格式：{[{"MtcConfUserUriKey": "用户uri","MtcConfPictureSizeKey": 视频尺寸,"MtcConfRectangleKey": 图像矩形的具体方位和长宽}]，...}
        - 用户uri：通过调用底层Mtc接口获取MtcUser.Mtc_UserFormUri((uint)EN_MTC_USER_ID_TYPE.EN_MTC_USER_ID_USERNAME，userId)
        - 视频尺寸：一共5个枚举值，具体枚举值请参考底层mtc_conf.h下的MtcConfPs枚举
        - 图像矩形的具体方位和长宽：这是一个Json格式的Array对象表示这个图像的位置和大小。
           - 第一个值是图像左上角的x坐标(0~1)
           - 第二个值是图像左上角的y坐标(0~1)
           - 第三个值是图像的宽(0~1)
           - 第四个值是图像的高(0~1)
           - 比如[0.5,0.5,0.5,0.5]表示图像在右下角长宽是原始屏幕的一半
           
       举例：[{"MtcConfUserUriKey":"[username:zhang@xxxx.cloud.justalk.com]","MtcConfPictureSizeKey":512,"MtcConfRectangleKey":[0.5,0.5,0.5,0.5]}]
        - 表示成员zhang小尺寸的视频在屏幕右下角位置，长宽是原始屏幕的一半


