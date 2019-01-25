
.. _登录:

登录
=========================

只有登录 Juphoon Cloud Platform 平台才能进行平台上的各种业务操作。

本页介绍各开发平台的登录集成。

- :ref:`iOS 登录<iOS 登录>`

- :ref:`Android 登录<Android 登录>`

- :ref:`Windows 登录<Windows 登录>`

在使用 Juphoon Cloud Platform 服务之前，您需要选择使用一种登录模式用于集成。关于登录模式的详细信息请参阅 :ref:`登录模式说明<登录模式说明>` 。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _iOS 登录:

iOS 登录
------------------------------

完成 SDK 的初始化后即可进行登录集成。

登录涉及 JCClient 类及其回调 JCClientCallback 类，其主要作用是负责登录、登出管理及帐号信息存储。

.. highlight:: objective-c

登录之前，可以进行登录的相关设置，如是否使用代理服务器登录以及服务器地址的设置，方法如下
::

    /**
     *  @brief 设置配置相关参数
     *  JCClientConfigServer, JCClientConfigHttpsProxy 均需要在 login 之前调用
     *  @param key   参数关键字, 参见 JCClientConstants 中定义
     *  @param value 参数值
     *  @return 返回 true 表示设置成功，false 表示设置失败
     */
    -(bool)setConfig:(NSString*)key value:(NSString*)value;

设置登录相关参数后，可以调用下面的方法获取相关的配置
::

    /**
     *  @brief 获取配置相关参数
     *  @param key 参数关键字, 参见 JCClientConstants 中定义
     *  @return 成功返回字符串类型具体值, 失败返回 NULL
     */
    -(NSString*)getConfig:(NSString*)key;

登录参数设置之后，即可调用 login 接口发起登录操作::

    /**
     *  @brief 登录
     *  @param userId   用户名
     *  @param password 密码，免鉴权模式密码可以随意输入，但不能为空
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     *  @warning 目前只支持免鉴权模式，免鉴权模式下当账号不存在时会自动去创建该账号
     *  @warning 用户名为英文、数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符
     */
    -(bool)login:(NSString*)userId password:(NSString*)password;


代码示例如下：
::

    // 创建 JCClient 实例
    JCClient *client = [JCClient create:@"your appkey" callBack:callback extraParams:nil];
    // 登录
    [client login:@"userId" password:@"your password"];
    ...

登录的结果通过 onlogin 接口监听回调::

    /**
     *  @brief 登录结果回调
     *  @param result  true 表示登录成功，false 表示登录失败
     *  @param reason  当 result 为 false 时该值有效
     *  @see JCClientReason
     */
    -(void)onLogin:(bool)result reason:(JCClientReason)reason;

其中，JCClientReason 有
::

    /// 正常
    JCClientReasonNone,
    /// sdk 未初始化
    JCClientReasonSDKNotInit,
    /// 无效的参数
    JCClientReasonInvalidParam,
    /// 函数调用失败
    JCClientReasonCallFunctionError,
    /// 当前状态无法再次登录
    JCClientReasonStateCannotLogin,
    /// 超时
    JCClientReasonTimeOut,
    /// 网络异常
    JCClientReasonNetWork,
    /// appkey 错误
    JCClientReasonAppKey,
    /// 账号密码错误
    JCClientReasonAuth,
    /// 无该用户
    JCClientReasonNoUser,
    /// 被强制登出
    JCClientReasonServerLogout,
    /// 其他错误
    JCClientReasonOther,

登录成功返回 true，false 返回失败。

登录成功之后，SDK 会自动保持与服务器的连接状态，直到用户主动调用登出接口，或者因为帐号在其他设备登录导致该设备登出。帐号体系是共享的，同一个用户名只能同时登录在一台设备上。后一个登录的设备登录成功之后，将会把前一个设备踢下线。


**登出**

如果您登出 Juphoon Cloud Platform 平台，则不能进行平台上的各种业务操作。

登出调用方法::

    /**
     *  登出 Juphoon Cloud 平台，登出后不能进行平台上的各种业务
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     */
    -(bool)logout;

登出结果通过 onlogout 接口监听回调::

    /**
     *  @brief 登出回调
     *  @param reason 登出原因
     *  @see JCClientReason
     */
    -(void)onLogout:(JCClientReason)reason;


当登录状态发生改变时，会收到 onClientStateChange 回调：

::
    
    /**
     *  @brief 登录状态变化通知
     *  @param state    当前状态值
     *  @param oldState 之前状态值
     */
    -(void)onClientStateChange:(JCClientState)state oldState:(JCClientState)oldState;

示例代码::

    -(void)onClientStateChange:(JCClientState)state oldState:(JCClientState)oldState
    {
        if (state == JCClientStateIdle) { // 未登录
           ...
        } else if (state == JCClientStateLogining) { // 登录中
           ...
        } else if (state == JCClientStateLogined) {  // 登录成功
           ...
        } else if (state == JCClientStateLogouting) {  // 登出中
           ...
        }
    }


JCClientState 通常有::

    // 未初始化
    JCClientStateNotInit,
    // 未登录
    JCClientStateIdle,
    // 登录中
    JCClientStateLogining,
    // 登录成功
    JCClientStateLogined,
    // 登出中
    JCClientStateLogouting,

集成登录后，即可进行相关业务的集成。

``SDK 支持前后台模式，可以在应用进入前台或者后台时调用 JCClient 类中的 setForeground 方法进行设置``

::

    /**
     *  @brief 设置是否是前台，当应用进入前台标志为true，进入后台标志为false
     *  @param foreground  true 是前台，false 是后台
     */
    -(void)setForeground:(bool)foreground;

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Android 登录:

Android 登录
------------------------------

登录涉及 JCClient 类，其主要作用是负责登录、登出管理及帐号信息存储。

.. highlight:: java

登录之前，可以进行登录的相关设置，如是否使用代理服务器登录以及服务器地址的设置，方法如下
::

    /**
     * 设置配置相关参数<br>
     * CONFIG_KEY_SERVER_ADDRESS, CONFIG_KEY_HTTPS_PROXY 均需要在 login 之前调用<br>
     *
     * @param key    参数关键字
     * @param value  参数值
     * @return 返回 true 表示设置成功，false 表示设置失败
     * @see JCClient.ConfigKey
     */
    public abstract boolean setConfig(@ConfigKey String key, String value);

设置登录相关参数后，可以调用下面的方法获取相关的配置
::

    /**
     * 获取配置相关参数
     *
     * @param key 参数关键字
     * @return 成功返回字符串类型具体值, 失败返回 NULL
     * @see JCClient.ConfigKey
     */
    public abstract String getConfig(@ConfigKey String key);

登录参数设置之后，即可调用 login 接口发起登录操作::

    /**
     * 登录 Juphoon Cloud 平台，只有登陆成功后才能进行平台上的各种业务
     * 登录结果通过 JCCallCallback 通知<br>
     * 注意:用户名为英文数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符
     *
     * @param userId    用户名
     * @param password  密码，但不能为空
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     */
    public abstract boolean login(String userId, String password);

示例代码::

    JCClient client = JCClient.create(Context, "your appkey", this, null);
    client.login(userI, password);

登录操作执行之后，登录的结果通过 onLogin 接口监听回调::

    /**
     * 登录结果回调
     *
     * @param result  true 表示登陆成功，false 表示登陆失败
     * @param reason  当 result 为 false 时该值有效
     */
    void onLogin(boolean result, @JCClient.ClientReason int reason);

登录成功之后，SDK 会自动保持与服务器的连接状态，直到用户主动调用登出接口，或者因为帐号在其他设备登录导致该设备登出。现有帐号体系中一个用户名只能同时登录在一台设备上。后一个登录的设备登录成功之后，将会把前一个设备踢下线。


**登出**

如果您登出 Juphoon Cloud Platform 平台，则不能进行平台上的各种业务操作。

return 返回 true 表示正常执行调用流程，false 表示调用异常。

登出通过下面方法实现::

    /**
     * 登出 Juphoon Cloud 平台，登出后不能进行平台上的各种业务
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     */
    public abstract boolean logout();


登出结果通过 onLogout 接口监听回调::

    /**
     * 登出回调
     *
     * @param reason 登出原因
     */
    void onLogout(@JCClient.ClientReason int reason);
    
ClientReason 通常有::

    // 正常
    public static final int REASON_NONE = 0;
    // sdk 未初始化
    public static final int REASON_SDK_NOT_INIT = 1;
    // 无效参数
    public static final int REASON_INVALID_PARAM = 2;
    // 函数调用失败
    public static final int REASON_CALL_FUNCTION_ERROR = 3;
    // 当前状态无法再次登录
    public static final int REASON_STATE_CANNOT_LOGIN = 4;
    // 超时
    public static final int REASON_TIMEOUT = 5;
    // 网络异常
    public static final int REASON_NETWORK = 6;
    // appkey 错误
    public static final int REASON_APPKEY = 7;
    // 账号密码错误
    public static final int REASON_AUTH = 8;
    // 无该用户
    public static final int REASON_NOUSER = 9;
    // 其他错误
    public static final int REASON_OTHER = 100;


当登录状态发生改变时，会收到 onClientStateChange 回调：
::

    /**
     * 登录状态变化通知
     *
     * @param state    当前状态值
     * @param oldState 之前状态值
     */
    void onClientStateChange(@JCClient.ClientState int state, @JCClient.ClientState int oldState);


示例代码::

    public void onClientStateChange(@JCClient.ClientState int state, @JCClient.ClientState int oldState) {
         if (state == JCClient.STATE_IDLE) { // 未登录
           ...
        } else if (state == JCClient.STATE_LOGINING) { // 正在登录
           ...
        } else if (state == JCClient.STATE_LOGINED) { // 登录成功
           ... 
        } else if (state == JCClient.STATE_LOGOUTING) { // 登出中
           ...
        }
    }


ClientState 通常有::

    // 未初始化
    public static final int STATE_NOT_INIT = 0;
    // 未登录
    public static final int STATE_IDLE = 1;
    // 登录中
    public static final int STATE_LOGINING = 2;
    // 登录成功
    public static final int STATE_LOGINED = 3;
    // 登出中
    public static final int STATE_LOGOUTING = 4;


集成登录后，即可进行相关业务的集成。

``SDK 支持前后台模式，可以在应用进入前台或者后台时调用 JCClient 类中的 setForeground 方法进行设置``

::

    /**
     * 设置是否为前台, 在有控制后台网络的手机上当进入前台时主动触发
     *
     * @param foreground 是否为前台
     */
    public abstract void setForeground(boolean foreground);

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Windows 登录:

Windows 登录
------------------------------

登录涉及 JCClient 类，其主要作用是负责登录、登出管理及帐号信息存储。

.. highlight:: c#

登录之前，可以进行登录的相关设置，如是否使用代理服务器登录以及服务器地址的设置，方法如下
::

    /// <summary>
    /// 设置登录相关属性
    /// </summary>
    /// <param name="key">设置的属性key值</param>
    /// <param name="value">设置的属性对应值</param>
    /// <returns>返回 ture 表示设置成功，false设置失败</returns>
    public bool setConfig(string key, string value)

设置登录相关参数后，可以调用下面的方法获取相关的配置
::

    /// <summary>
    /// 获取设置属性的值
    /// </summary>
    /// <param name="key">需要获取的属性key值</param>
    /// <returns>需要获取的属性值</returns>
    public string getConfig(string key)

登录参数设置之后，即可调用 login 接口发起登录操作::

    /// <summary>
    /// 登录 Juphoon Cloud 平台，只有登录成功后才能进行平台上的各种业务
    /// 登录结果通过 JCCallCallback 通知
    /// <param name="username">用户标识</param>
    /// <param name="password">密码，如果设置为免鉴权则可以填任意字符串</param>
    /// <returns>true 表示正常执行调用流程，false 表示调用异常</returns>
    /// <remarks>注意:用户名为英文数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符</remarks>
    /// <remarks>当用户不存在时会自动创建该用户</remarks>
    public bool login(string username, string password)

结果返回 true 表示正常执行调用流程，false 表示调用异常。

示例代码::

    JCClient client = JCClient.create(app, "your appkey", this, null);
    client.login(userId, password);

登录的结果通过 onLogin 接口监听回调::

    /// <summary>
    /// 登录结果回调
    /// </summary>
    /// <param name="result">true 表示登录成功，false 表示登录失败</param>
    /// <param name="reason">当 result 为 false 时该值有效，了解具体原因</param>
    void onLogin(bool result, JCClientReason reason);
                
登录成功之后，SDK 会自动保持与服务器的连接状态，直到用户主动调用登出接口，或者因为帐号在其他设备登录导致该设备登出。现有帐号体系中一个用户名只能同时登录在一台设备上。后一个登录的设备登录成功之后，将会把前一个设备踢下线。


**登出**

如果您登出 Juphoon Cloud Platform 平台，则不能进行平台上的各种业务操作。

登出通过下面方法实现::

    /// <summary>
    /// 登出 Juphoon Cloud 平台
    /// </summary>
    /// <returns>返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知</returns>
    public bool logout();

登出结果通过 onLogout 接口监听回调::
    
    /// <summary>
    /// 登出回调
    /// </summary>
    /// <param name="reason">登出原因</param>
    void onLogout(JCClientReason reason);

JCClientReason 通常有::

     // 正常
     None,
     // sdk 未初始化
     SDKNotInit,
     // 无效参数
     InvalidParam,
     // 函数调用失败
     CallFucntionError,
     // 当前状态无法再次登录
     StateCannotLogin,
     // 超时
     TimeOut,
     // 网络异常
     NetWork,
     // appkey 错误
     AppKey,
     // 账号密码错误
     Auth,
     // 无该用户
     NoUser,
     // 其他错误
     Other = 100

当登录状态发生改变时，会收到 onClientStateChange 回调：
::

    /// <summary>
    /// 登录状态变化通知
    /// </summary>
    /// <param name="state">当前状态值</param>
    /// <param name="oldState">之前状态值</param>
    void onClientStateChange(JCClientState state, JCClientState oldState);

示例代码::

    private void onClientStateChange(JCClientState state, JCClientState oldState)
        {
            if (state == JCClientState.Idle) { // 未登录
               ...
            }
            else if (state == JCClientState.Logining) { // 登录中
               ...
            }
            else if (state == JCClientState.Logined) { // 登录成功
                ...
            }
            else if (state == JCClientState.Logouting) { // 登出中
                ...
            }
        }


JCClientState 通常有::

    // 未初始化
    NotInit,
    // 未登录
    Idle,
    // 登录中
    Logining,
    // 登录成功
    Logined,
    // 登出中
    Logouting,

集成登录后，即可进行相关业务的集成。

    