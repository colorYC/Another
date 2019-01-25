
.. _通话状态更新(windows):

通话状态更新
----------------------------

.. highlight:: csharp


如果频道中的成员状态发生了改变，比如开启或关闭发送音频数据、开启本地静音等，其他成员会收到 onParticipantUpdate 的回调
::
    
    /// <summary>
    /// 成员更新回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    /// <param name="changeParam">更新标识类</param>
    void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam);

如果有新成员加入了频道，其他成员会收到成员加入的回：
::

    /// <summary>
    /// 成员加入回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    void onParticipantJoin(JCMediaChannelParticipant participant);

如果有成员离开了频道，其他成员会收到成员离开的回调：
::

    /// <summary>
    /// 成员离开回调
    /// </summary>
    /// <param name="participant">成员对象</param>
    void onParticipantLeft(JCMediaChannelParticipant participant);


如果成员自身状态发生了改变会收到 onMediaChannelStateChange 的回调
::

    /// <summary>
    /// 自身状态变化回调
    /// </summary>
    /// <param name="state">当前状态</param>
    /// <param name="oldState">变化前状态值</param>
    void onMediaChannelStateChange(JCMediaChannelState state, JCMediaChannelState oldState);

其中，自身状态（JCMediaChannelState）有：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - Idle
     - 空闲状态
   * - Joining
     - 加入中
   * - Joined
     - 已加入
   * - JLeaving
     - 离开


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _通话过程控制(windows):

通话过程控制
----------------------------

- 开启/关闭音频输出

在通话中可以通过下面的方法开启或者关闭音频输出，当 enable 值为 false 时，您将听不到其他成员的声音

::

    /// <summary>
    /// 开启关闭音频输出，可实现静音功能，建议每次Join前设置
    /// </summary>
    /// <param name="enable">是否开启音频输出</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableAudioOutput(bool enable)


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
    public bool enableUploadAudioStream(bool enable)


- 发送/停止发送本地视频流

如果想开启或关闭发送本地视频流，可以调用下面方法，当 enable 值为 false 时，将会停止发送本地视频流，此时其他成员将看不到您的画面

::

    /// <summary>
    /// 开启关闭发送本地视频流
    /// 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
    /// 2.未在频道中则标记是否上传视频流，在Join时生效
    /// 2.建议每次Join前设置
    /// </summary>
    /// <param name="enable">开启关闭本地视频流</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableUploadVideoStream(bool enable)

示例代码::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);            
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);
    // 开启音频输出
    mediaChannel.enableAudioOutput(true);
    // 发送本地音频流
    mediaChannel.enableUploadAudioStream(audio);
    // 发送本地视频流
    mediaChannel.enableUploadVideoStream(true);


