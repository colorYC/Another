iOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];
    JCMediaChannel *mediaChannel = [JCMediaChannel create:client mediaDevice:mediaDevice callback:self];

.. _查询频道(iOS):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /**
     *  @brief              查询媒体频道相关信息，例如是否存在，人数等
     *  @param channelId    媒体通道标识
     *  @return             返回操作id，与 onQuery 回调中的 operationId 对应
     */
    -(int)query:(NSString*)channelId;

示例代码::

    [mediaChannel query:@"channelId"];

查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /**
     *  @brief                  查询媒体频道信息结果回调
     *  @param operationId      操作id，由 query 接口返回
     *  @param result           true 表示查询成功，false 表示查询失败
     *  @param reason           查询失败原因 当 result 为 false 时该值有效
     *  @param queryInfo        查询到的信息
     *  @see JCMediaChannelReason
     */
    -(void)onQuery:(int)operationId result:(bool)result reason:(JCMediaChannelReason)reason queryInfo:(JCMediaChannelQueryInfo*)queryInfo;

示例代码::

    -(void)onQuery:(int)operationId result:(bool)result reason:(JCMediaChannelReason)reason queryInfo:(JCMediaChannelQueryInfo *)queryInfo {
        // 查询成功
       if (result) {
            // 查询频道标识
            NSString* channelId = queryInfo.channelId;
            // 查询频道号
            int number = queryInfo.number;
            // 查询频道成员数
            int clientCount = queryInfo.clientCount;
            // 查询频道成员列表
            NSMutableArray *members = queryInfo.members;
       } else {
            // 查询失败
       }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _屏幕共享(iOS):

屏幕共享
----------------------

屏幕共享可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用屏幕共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

.. note:: 发起屏幕共享需要 iOS 11.0 及以上。目前 iOS 只支持应用内的屏幕共享。

开启或关闭屏幕共享接口如下::

    /**
     * @brief 开关屏幕共享
     * @param enable 是否开启屏幕共享
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableScreenShare:(bool)enable;

示例代码::

    // 开启或关闭屏幕共享
    [mediaChannel enableScreenShare:true];

您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /**
     *  @breif              设置屏幕共享采集属性
     *  @param width        采集宽度，默认640
     *  @param height       采集高度，默认360
     *  @param framerate    帧速率，默认10
     */
    - (void)setScreenCaptureProperty:(int)width height:(int)height framerate:(int)framerate;

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 screenUserId 属性获得发起屏幕共享的用户标识。
::

    /**
     *  @brief 属性变化回调，目前主要关注屏幕共享状态的更新
     *  @param changeParam 变化标识集合
     */
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam;


此时可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::

    /**
     *  @brief               请求屏幕共享的视频流
     *  @param screenUri     屏幕分享uri
     *  @param pictureSize   视频请求尺寸类型
     *  @return              返回 true 表示正常执行调用流程，false 表示调用异常
     *  @see JCMediaChannelPictureSize
     *  @warning 当 pictureSize 为 JCMediaChannelPictureSizeNone 表示关闭请求
     */
    -(bool)requestScreenVideo:(NSString*)screenUri pictureSize:(JCMediaChannelPictureSize)pictureSize;


示例代码::
    
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam {
        // 请求屏幕共享的视频流
        JCMediaDeviceVideoCanvas *screen = [mediaDevice startVideo:mediaChannel.screenRenderId renderType:JCMediaDeviceRenderFullContent];
        [mediaChannel requestScreenVideo:mediaChannel.screenRenderId pictureSize:JCMediaChannelPictureSizeLarge];
    }

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CDN 推流(iOS):


CDN 推流
----------------------

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

**开启 CDN 推流**

如要开启 CDN 推流，需在加入频道前进行 CDN 推流地址的设置。
    
示例代码
::

    // 设置 CDN 推流地址
    NSMutableDictionary *dic = [NSMutableDictionary dictionary];
    [dic setObject:@"your cdnurl" forKey:JCMediaChannelJoinParamCdn];
    // 加入频道
    [mediaChannel join:@"channelId" params:dic];

只有 CDN 当前状态不为 JCMediaChannelCdnStateNone 时才可以进行 CDN 推流。其中，CDN 推流状态有以下几种：
::

    /// 无法进行CDN推流
    JCMediaChannelCdnStateNone,
    /// 可以开启CDN推流
    JCMediaChannelCdnStateReady,
    /// CDN推流中
    JCMediaChannelCdnStateRunning,


开启或关闭 CDN 推流调用如下接口
::

    /**
     *  @brief              开关Cdn推流
     *  @param enable       是否开启Cdn推流
     *  @param keyInterval  推流关键帧间隔(毫秒)，当 enable 为 true 时有效，-1表示使用默认值(5000毫秒)，有效值需要>=1000
     *  @return             返回 true 表示正常执行调用流程，false 表示调用异常
     *  @warning 在收到 onMediaChannelPropertyChange 回调后检查是否开启
     */
    -(bool)enableCdn:(bool)enable keyInterval:(int)keyInterval;


示例代码
::

    -(void)onJoin:(bool)result reason:(JCMediaChannelReason)reason channelId:(NSString*)channelId {
        // 根据CDN推流状态判断是否开启推流
        if (mediaChannel.cdnState == JCMediaChannelCdnStateNone) {
            // 无法使用 CDN 推流
        } else if (mediaChannel.cdnState == JCMediaChannelCdnStateReady) {
            // 可以开启 CDN 推流
            [mediaChannel enableCdn:true keyInterval:0];
        } else if (mediaChannel.cdnState == JCMediaChannelCdnStateRunning) {
            // CDN 推流中，可以关闭 CDN 推流
            [mediaChannel enableCdn:false keyInterval:0];
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音视频录制(iOS):

服务器音视频录制
----------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前设置录制参数，然后在加入频道的时候传入录制参数。

示例代码::

    // 设置录制参数
    NSDictionary *paramDic = @{@"Protocol" : @"qiniu",
                                   @"AccessKey" : accessKey,
                                   @"SecretKey" : secretKey,
                                   @"BucketName" : bucketName,
                                   @"FileKey" : fileKey};
    NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @YES, @"Storage" : paramDic};
    [dic setObject:storageDic forKey:JCMediaChannelJoinParamRecord];
    // 加入频道
    [mediaChannel join:@"channelId" params:paramDic];


.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 NO。即：NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @NO, @"Storage" : paramDic};


录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /// 无法进行视频录制
    JCMediaChannelRecordStateNone,
    /// 可以开启视频录制
    JCMediaChannelRecordStateReady,
    /// 视频录制中
    JCMediaChannelRecordStateRunning,

录制状态获取后，即可调用下面的接口开启或关闭音视频录制
::

    /**
     *  @brief 开关视频录制
     *  @param enable 是否开启屏幕录制
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableRecord:(bool)enable;


示例代码::

    -(void)onJoin:(bool)result reason:(JCMediaChannelReason)reason channelId:(NSString*)channelId {
        // 根据音视频录制状态判断是否开启音视频录制
        if (mediaChannel.recordState == JCMediaChannelRecordStateNone) {
            // 无法进行音视频录制
        } else if (mediaChannel.recordState == JCMediaChannelRecordStateReady) {
            // 可以开启音视频录制
            [mediaChannel enableRecord:true];
        } else if (mediaChannel.recordState == JCMediaChannelRecordStateRunning) {
            // 音视频录制中，可以关闭音视频录制
            [mediaChannel enableRecord:false];
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(iOS):


发送消息
----------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /**
     * @brief 发送消息
     *
     * @param type       消息类型
     * @param content    消息内容，当 toUserId 不为 nil 时，content 不能大于 4k
     * @param toUserId   接收者id，null则发给频道所有人员
     * @return           返回 true 表示成功，false表示失败
     */
    -(bool)sendMessage:(NSString *)type content:(NSString *)content toUserId:(NSString *)toUserId;

其中，消息类型（type）为自定义类型。


示例代码::
    
    -(void)onJoin:(bool)result reason:(JCMediaChannelReason)reason channelId:(NSString*)channelId {
        // 发送给所有成员
        [mediaChannel sendMessage:@"text" content:@"content" toUserId:nil];
        // 发送给某个成员
        [mediaChannel sendMessage:@"text" content:@"content" toUserId:@"接收者id"];
    }


当频道中的其他成员收到消息时，会收到 onMessageReceive 回调
::

    /**
     * @brief                接收频道消息的回调
     *
     * @param type           消息类型
     * @param content        消息内容
     * @param fromUserId     消息发送成员的userId
     */
    -(void)onMessageReceive:(NSString *)type content:(NSString *)content fromUserId:(NSString *)fromUserId;



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送指令(iOS):

发送指令
----------------------

频道中可以发送控制指令，如批量修改成员状态，角色，昵称、设置推流布局模式等。

发送指令命令调用 sendCommand 接口
::

    /**
     * @brief 发送指令
     *
     * @param commandName   指令名
     * @param commandParam  指令参数
     * @return              true表示成功，false表示失败
     */
    -(bool)sendCommand:(NSString *)name commandParam:(NSString *)param;

示例代码::

    -(void)onJoin:(bool)result reason:(JCMediaChannelReason)reason channelId:(NSString*)channelId {
        // 发送修改频道标题的指令
        [mediaChannel sendCommand:@"ChangeTitle" commandParam:@"{"MtcConfTitleKey":"321"}"];
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
        - 用户uri：通过调用底层Mtc接口获取 Mtc_UserFormUri((uint)EN_MTC_USER_ID_TYPE.EN_MTC_USER_ID_USERNAME，userId)
        - 视频尺寸：一共5个枚举值，具体枚举值请参考底层 mtc_conf.h 下的 MtcConfPs 枚举
        - 图像矩形的具体方位和长宽：这是一个Json格式的Array对象表示这个图像的位置和大小。
           - 第一个值是图像左上角的x坐标(0~1)
           - 第二个值是图像左上角的y坐标(0~1)
           - 第三个值是图像的宽(0~1)
           - 第四个值是图像的高(0~1)
           - 比如[0.5,0.5,0.5,0.5]表示图像在右下角长宽是原始屏幕的一半
       举例：[{"MtcConfUserUriKey":"[username:zhang@xxxx.cloud.justalk.com]","MtcConfPictureSizeKey":512,"MtcConfRectangleKey":[0.5,0.5,0.5,0.5]}]
        - 表示成员zhang小尺寸的视频在屏幕右下角位置，长宽是原始屏幕的一半

