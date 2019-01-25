Android
-------------------------------

.. image:: images/sipcall2.png

.. highlight:: java

**开始集成 Sip 通话功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);

其中，创建 JCMediaChannel 实例的方法如下
::

    /**
     * 创建 JCMediaChannel 对象
     *
     * @param client      JCClient 对象
     * @param mediaDevice JCMediaDevice 对象
     * @param callback    JCMediaChannelCallback 回调接口，用于接收 JCMediaChannel 相关通知
     * @return 返回 JCMediaChannel 对象
     */
    public static JCMediaChannel create(JCClient client, JCMediaDevice mediaDevice, JCMediaChannelCallback callback);

**开始集成**

1. 加入通话
::

    /**
     * 加入频道，当前只支持同时加入一个媒体频道
     *
     * @param channelId 媒体频道标识
     * @param params    参数，KEY值参考JoinParam，没有则填null
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     * @see Region
     * @see MaxResolution
     * @see JoinParam
     */
    public abstract boolean join(String channelId, Map<String, String> params);

.. note:: 加入频道会自动打开音频设备。

示例代码
::

    // 加入频道
    mediaChannel.join("channelId", null);


加入频道后，Client A 会收到 onJoin 回调

::

    /**
     * 加入频道结果回调
     *
     * @param result    true 表示成功，false 表示失败
     * @param reason    加入失败原因，当 result 为 false 时该值有效
     * @param channelId 频道标识符
     */
    void onJoin(boolean result, @JCMediaChannel.MediaChannelReason int reason, String channelId);

示例代码::

    // 加入频道结果回调
    public void onJoin(boolean result, @JCMediaChannel.MediaChannelReason int reason, String channelId) {
        if (result) {
            // 加入频道成功
        } else {
            // 加入频道失败
        }
    }

Client A 加入成功后，调用 enableUploadAudioStream 方法请求开启发送本地音频流

::

    /**
     * 开启关闭发送本地音频流
     * 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
     * 2.未在频道中则标记是否上传音频流，在join时生效
     * 3.建议每次join前设置
     *
     * @param enable 是否开启本地音频流
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableUploadAudioStream(boolean enable);


2. 邀请 SIP 用户 B 加入通话

Client A 调用 inviteSipUser 接口邀请 SIP 用户 B 加入通话

::

    /**
     * 邀请SIP用户，一般用于对接落地网关等
     *
     * @param userId 一般为号码
     * @param sipParam  sipParam nil 或者通过 JCMediaChannelUtils.buildSipParam 构造
     * @return 成功返回值 >= 0，失败返回 -1
     */
    public abstract int inviteSipUser(String userId, String sipParam);

其中，sipParam 的构造方法如下
::

    /**
     * @brief sip邀请param参数构造
     * @param sipUri       JCMediaChannel.inviteSipUser 参数 userId 是号码还是 sipUri
     * @param route        sipUri 为 true 才生效，决定 sip 信令是否路由到 userId 的 sip 域里
     * @param displayName  sip用户加入会议后的昵称
     * @param mcu          JCMediaChannel.inviteSipUser 参数 userId 是否为 Mcu 会议
     * @param video        是否需要视频接入
     * @param dtmfPassowrd dtmf 密码
     * @return json 字符串
     */
    public static String buildSipParam(boolean sipUri, boolean route, String displayName, boolean mcu, boolean video, String dtmfPassowrd) {


示例代码
::

    // 邀请SIP用户
    mediaChannel.inviteSipUser("userId", nil);

邀请操作执行后，Client A 会收到 onInviteSipUserResult 回调
::

    /**
     * 邀请SIP用户操作结果回调，成功后会触发 onParticipantJoin
     *
     * @param operationId 操作id
     * @param result true表示成功，false表示失败
     * @param reason 原因
     */
    void onInviteSipUserResult(int operationId, boolean result, int reason);

SIP 用户 B 加入成功后，Client A 会收到 onParticipantJoin回调
::

    /**
     * 成员加入回调
     *
     * @param participant 成员对象
     */
    void onParticipantJoin(JCMediaChannelParticipant participant);

3. 离开通话

通话结束，Client A 可通过 UI 调用 Leave 接口离开通话，此时 Client A 会收到 onLeave 回调。B 则通过挂断直接结束通话。

::
    
    /**
     * 离开媒体通道，当前只支持同时加入一个媒体通道
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean leave();

    /**
     * 离开媒体通道结果回调
     *
     * @param reason    离开原因
     * @param channelId 媒体频道标识符
     */
    void onLeave(@JCMediaChannel.MediaChannelReason int reason, String channelId);

离开通话时，JCMediaChannelReason 通常有以下几种结果：
::
    
    // 正常
    public static final int REASON_NONE = 0;
    // 未登录
    public static final int REASON_NOT_LOGIN = 1;
    // 超时
    public static final int REASON_TIMEOUT = 2;
    // 网络异常
    public static final int REASON_NETWORK = 3;
    // 函数调用失败
    public static final int REASON_CALL_FUNCTION_ERROR = 4;
    // 已加入
    public static final int REASON_ALREADY_JOINED = 5;
     //被踢
    public static final int REASON_KICKED = 6;
    // 掉线
    public static final int REASON_OFFLINE = 7;
    // 主动离开
    public static final int REASON_QUIT = 8;
    // 频道关闭
    public static final int REASON_OVER = 9;
    // 成员满
    public static final int REASON_FULL = 10;
    // 无效密码
    public static final int REASON_INVALID_PASSWORD = 11;
    // 其他错误
    public static final int REASON_OTHER = 100;


示例代码::

    // 离开频道
    mediaChannel.leave();

如果  B 先离开，则 A 和其他成员将会收到 onParticipantLeft 回调

::
    
    /**
     * 成员离开回调
     *
     * @param participant 成员对象
     */
    void onParticipantLeft(JCMediaChannelParticipant participant);


**通话状态更新**

通话过程中，UI 将通过以下方法监听回调成员状态的改变并进行相应的更新。

::
    
    /**
     * 成员更新回调
     *
     * @param participant 成员对象
     */
    void onParticipantUpdate(JCMediaChannelParticipant participant);

**通话过程控制**

- 开启/关闭音频输出

在通话中可以通过下面的方法开启或者关闭音频输出，当 enable 值为 false 时，您将听不到其他成员的声音

::

    /**
     * 开启关闭音频输出，可实现静音功能，建议每次join前设置
     *
     * @param enable 是否开启音频输出
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableAudioOutput(boolean enable);


- 开启/关闭发送本地音频流

如果想开启或关闭发送本地音频流，可以调用下面方法，当 enable 值为 false ，将会停止发送本地音频流，此时其他成员将听不到您的声音，从而实现静音功能

::

    /**
     * 开启关闭发送本地音频流
     * 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
     * 2.未在频道中则标记是否上传音频流，在join时生效
     * 3.建议每次join前设置
     *
     * @param enable 是否开启本地音频流
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableUploadAudioStream(boolean enable);


**示例代码**

::

    // 开启音频输出
    mediaChannel.enableAudioOutput(true);
    // 上传本地音频流
    mediaChannel.enableUploadAudioStream(true);


**设备控制**

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


示例代码
::

    // 开启扬声器
    mediaDevice.enableSpeaker(true);
    // 开启音频设备
    mediaDevice.startAudio();
    // 关闭音频设备
    mediaDevice.stopAudio();