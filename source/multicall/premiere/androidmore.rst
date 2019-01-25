
.. _通话状态更新(android):

通话状态更新
----------------------------

.. highlight:: java


如果频道中的成员状态发生了改变，比如开启或关闭发送音频数据、开启本地静音等，其他成员会收到 onParticipantUpdate 的回调
::
    
    /**
     * 成员更新回调
     *
     * @param participant 成员对象
     * @param changeParam 更新标识类
     */
    void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam);


如果有新成员加入了频道，其他成员会收到成员加入的回调：
::

    /**
     * 新成员加入回调
     *
     * @param participant 成员对象
     */
    void onParticipantJoin(JCMediaChannelParticipant participant);

如果有成员离开了频道，其他成员会收到成员离开的回调：
::

    /**
     * 成员离开回调
     *
     * @param participant 成员对象
     */
    void onParticipantLeft(JCMediaChannelParticipant participant);


如果成员自身状态发生了改变会收到 onMediaChannelStateChange 的回调
::

    /**
     * 自身状态变化回调
     *
     * @param state    当前状态
     * @param oldState 变化前状态
     */
    void onMediaChannelStateChange(@JCMediaChannel.MediaChannelState int state, @JCMediaChannel.MediaChannelState int oldState);


其中，自身状态（JCMediaChannelState）有：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - STATE_IDLE
     - 空闲状态
   * - STATE_JOINING
     - 加入中
   * - STATE_JOINED
     - 已加入
   * - STATE_LEAVING
     - 离开中


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _通话过程控制(android):


通话过程控制
----------------------------

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


- 开启/关闭发送本地视频流

如果想开启或关闭发送本地视频流，可以调用下面方法，当 enable 值为 false 时，将会停止发送本地视频流，此时其他成员将看不到您的画面

::

    /**
     * 开启关闭发送本地视频流
     * 1.在频道中将会与服务器进行交互，服务器会更新状态并同步给其他用户
     * 2.未在频道中则标记是否上传视频流，在join时生效
     * 3.建议每次join前设置
     *
     * @param enable 是否开启本地视频流
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableUploadVideoStream(boolean enable);


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

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);
    // 开启音频输出
    mediaChannel.enableAudioOutput(true);
    // 上传本地视频流
    mediaChannel.enableUploadVideoStream(true);
    // 上传本地音频流
    mediaChannel.enableUploadAudioStream(true);


