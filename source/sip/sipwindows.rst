Windows
-------------------------------

.. image:: images/sipcall2.png

.. highlight:: csharp

**开始集成 Sip 通话功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);            
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);

其中，创建 JCMediaChannel 实例的方法如下
::

    /// <summary>
    /// 创建JCMediaChannel对象
    /// </summary>
    /// <param name="client"> JCClient 对象</param>
    /// <param name="mediaDevice">JCMediaDevice 对象</param>
    /// <param name="callback">JCMediaChannelCallback 对象，用于接收JCMediaDevice通知</param>
    /// <returns>JCMediaChannel对象</returns>
    public static JCMediaChannel create(JCClient.JCClient client, JCMediaDevice.JCMediaDevice mediaDevice, JCMediaChannelCallback callback);

**开始集成**

1. 加入通话
::

    /// <summary>
    /// 加入频道，当前只支持同时加入一个媒体频道
    /// </summary>
    /// <param name="channelId">频道标识</param>
    /// <param name="joinParams">加入频道参数（设置cdn,录制,通道密码，最大分辨率等）</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool join(string channelId, Dictionary<string,string>joinParams);

.. note:: 加入频道会自动打开音频设备。

示例代码::

    // 加入频道
    mediaChannel.join("channelId", null);

Client A 加入之后会收到 onjoin 回调

::

    /// <summary>
    /// 加入频道结果回调
    /// </summary>
    /// <param name="result">true表示加入成功，false表示加入失败</param>
    /// <param name="reason">加入失败原因，在result为false时该值有效</param>
    /// <param name="channelId">媒体频道标识</param>
    void onJoin(bool result, JCMediaChannelReason reason, string channelId);

Client A 加入成功后，调用 enableUploadAudioStream 方法请求开启发送本地音频流
::

    /// <summary>
    /// 开启关闭发送本地音频流
    /// 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
    /// 2.未在频道中则标记是否上传音频流，在Join时生效
    /// 2.建议每次Join前设置
    /// </summary>
    /// <param name="enable">开启关闭本地音频流</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableUploadAudioStream(bool enable);


示例代码::

    // 发送本地音频流
    mediaChannel.enableUploadAudioStream(true);

2.  邀请 SIP 用户加入通话

Client A 通过 UI 调用 inviteSipUser 接口邀请 SIP 用户 B 加入通话
::

    /// <summary>
    /// 邀请SIP用户，一般用于对接落地网关等
    /// </summary>
    /// <param name="userId">一般为号码</param>
    /// <param name="sipParam">nil 或者通过JCMediaChannelUtils.buildSipParam构造</param>
    /// <returns>成功返回值 >= 0，失败返回 -1</returns>
    public int inviteSipUser(string userId, string sipParam);

其中，sipParam 构造方法如下
::

    /// <summary>
    /// sip邀请param参数构造
    /// </summary>
    /// <param name="sipUri">JCMediaChannel.inviteSipUser 参数userId是号码还是sipUri</param>
    /// <param name="route">sipUri为true才生效，决定sip信令是否路由到userId的sip域里</param>
    /// <param name="displayName">sip用户加入会议后的昵称</param>
    /// <param name="mcu">JCMediaChannel.inviteSipUser参数userId是否为Mcu会议</param>
    /// <param name="video">是否需要视频接入</param>
    /// <param name="dtmfPassowrd">dtmf 密码</param>
    /// <returns>json 字符串</returns>
    public static string buildSipParam(bool sipUri, bool route, string displayName, bool mcu, bool video, string dtmfPassowrd)


示例代码::

    // 邀请SIP用户
    mediaChannel.inviteSipUser("userId", nil);

邀请操作执行后，Client A 将会收到 onInviteSipUserResult 回调
::

    /// <summary>
    /// 邀请SIP用户操作结果回调，成功后会触发 onParticipantJoin
    /// </summary>
    /// <param name="operationId">操作id</param>
    /// <param name="result">true表示成功，false表示失败</param>
    /// <param name="reason">原因</param>
    void onInviteSipUserResult(int operationId, bool result, int reason);

SIP 用户 B 加入成功后，Client A 和频道中的其他成员会收到新成员加入事件（onParticipantJoin）回调
::

    /// <summary>
    /// 成员加入回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    void onParticipantJoin(JCMediaChannelParticipant participant);

3. 离开通话

通话结束，Client A 可通过 UI 调用 Leave 接口离开通话，Client A 离开后会收到 onLeave 回调。B 则可以直接挂断以结束通话
::

    /// <summary>
    /// 离开频道
    /// </summary>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool leave();

    /// <summary>
    /// 离开频道结果标识
    /// </summary>
    /// <param name="reason">离开原因</param>
    /// <param name="channelId">媒体频道标识</param>
    void onLeave(JCMediaChannelReason reason, string channelId);


示例代码::

    // 离开频道
    mediaChannel.leave();


离开原因枚举值请参考 `JCMediaChannelReason <http://developer.juphoon.com/portal/reference/windows/html/4481d778-9d4d-43fe-f94d-fdfa690dd939.htm>`_。

如果 B 先离开通话，则 Client A 和频道中的其他成员将会收到成员离开事件（onParticipantLeft）
::

    /// <summary>
    /// 成员离开回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    void onParticipantLeft(JCMediaChannelParticipant participant);


**通话状态更新**

通话过程中，如果有成员状态发生了改变，则频道中的其他成员会收到 onParticipantUpdate 回调
::
    
    /// <summary>
    /// 成员更新回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    void onParticipantUpdate(JCMediaChannelParticipant participant);

**通话过程控制**

- 开启/关闭音频输出

在通话中可以通过下面的方法开启或者关闭音频输出，当 enable 值为 false 时，您将听不到其他成员的声音

::

    /// <summary>
    /// 开启关闭音频输出，可实现静音功能，建议每次Join前设置
    /// </summary>
    /// <param name="enable">是否开启音频输出</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableAudioOutput(bool enable);


- 开启/关闭发送本地音频流

如果想开启或关闭发送本地音频流，可以调用下面方法，当 enable 值为 false 时，将会停止发送本地音频流，此时其他成员将听不到您的声音，从而实现静音功能
::

    /// <summary>
    /// 开启关闭发送本地音频流
    /// 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
    /// 2.未在频道中则标记是否上传音频流，在Join时生效
    /// 2.建议每次Join前设置
    /// </summary>
    /// <param name="enable">开启关闭本地音频流</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableUploadAudioStream(bool enable);

示例代码::

    // 开启音频输出
    mediaChannel.enableAudioOutput(true);
    // 发送本地音频流
    mediaChannel.enableUploadAudioStream(audio);