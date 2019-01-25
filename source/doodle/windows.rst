Windows
-----------------------

.. highlight:: csharp

涂鸦模块包含涂鸦组件和涂鸦控件，您可以使用我们的涂鸦控件，也可以自定义涂鸦控件，无论您是使用我们的控件还是自定义控件，都只需将界面控件和 SDK 绑定并完成数据收发层的集成即可，下面将介绍涂鸦模块的集成操作。

开始之前，先下载 `SDK <http://developer.juphoon.com/document/cloud-communication-windows-sdk#2>`_。

涂鸦涉及 `JCDoodle 类 <http://developer.juphoon.com/portal/reference/windows/html/0a03c2b0-e7de-bb6e-438a-568c1b41726c.htm>`_ 、`JCDoodleCallback 类 <http://developer.juphoon.com/portal/reference/windows/html/5c1f5252-6a3f-bd01-d67c-0db3220828e0.htm>`_ 、 `JCDoodleInteractor 类 <http://developer.juphoon.com/portal/reference/windows/html/117cd9e6-429d-bc19-1468-8d962c00048f.htm>`_ 和 `JCDoodleAction 类 <http://developer.juphoon.com/portal/reference/windows/html/e270f295-5d53-cd80-19dc-b793ae74e65a.htm>`_ 。

具体请参考 `API 说明文档 <http://developer.juphoon.com/portal/reference/windows/html/c134a0d9-74d2-4872-28ed-5b62b207aa8c.htm>`_。

涂鸦模块的调用逻辑如下图所示：

.. image:: doodle_principle.png
   :width: 986
   :height: 456

**开始集成**

1.首先创建涂鸦对象
::

    /// <summary>
    /// 创建JCDoodle 对象
    /// </summary>
    /// <param name="doodleCallback">JCDoodle的回调,用于接收未处理或自定义的JCDoodleAction</param>
    /// <returns>返回JCDoodle对象</returns>
    public static JCDoodle create(JCDoodleCallback doodleCallback)

示例代码::

    JCDoodle doodle = JCDoodle.create(this);

2.绑定 doodle 交互器

您可以调用 bindDoodleInteractor 接口将 UI 控件和 SDK 进行绑定
::

    /// <summary>
    /// 绑定doodle的UI控件
    /// </summary>
    /// <param name="interactor">UI 控件示例</param>
    public void bindDoodleInteractor(JCDoodleInteractor interactor) 

示例代码::

    JCDoodle doodle = JCDoodle.create(this);
    doodle.bindDoodleInteractor(interactor);

3.将涂鸦数据注入 SDK

当界面层产生涂鸦数据后，调用 generateDoodleAction 接口将涂鸦动作对象注入 SDK
::

    /// <summary>
    /// 向SDK注入JCDoodleAction对象，SDK 会通过 onDoodleActionGenerated(JCDoodleAction)回调此 Doodle 动作。本方法仅供 UI 控件调用
    /// </summary>
    /// <param name="doodleAction">涂鸦动作对象</param>
    public void generateDoodleAction(JCDoodleAction doodleAction)

示例代码::

    JCDoodle doodle = JCDoodle.create(this);
    doodle.generateDoodleAction(doodleAction);

4.实现 JCDoodleCallback 的回调

界面层产生涂鸦数据后，SDK 会收到 onDoodleActionGenerated 回调
::

    /// <summary>
    /// 生成涂鸦对象的回调
    /// </summary>
    /// <param name="doodleAction">JCDoodleAction 对象，用户可以调用 stringFromDoodleAction(JCDoodleAction) 将其转为Json字符串后通过消息通道发送。</param>
    void onDoodleActionGenerated(JCDoodleAction doodleAction);

5.涂鸦数据处理和发送

回调的涂鸦数据需要在数据收发层进行处理（把 doodleAction 对象转为 String）
::

    /// <summary>
    /// 将涂鸦动作对象转换成json字符串，可通过 doodleActionFromString(string) 转换回对应的 JCDoodleAction
    /// </summary>
    /// <param name="doodleAction">涂鸦动作对象</param>
    /// <returns>返回json字符串</returns>
    public String stringFromDoodleAction(JCDoodleAction doodleAction)

示例代码::

    JCDoodle doodle = JCDoodle.create(this);
    string doodlestring = doodle.stringFromDoodleAction(doodleAction);

涂鸦数据转换完成后，即可调用相应的数据传输通道发送涂鸦数据
::
    
    public override void handleDoodleActionGenerated(JCDoodleAction doodleAction)
    {
        JCDoodle doodle = JCDoodle.create(this);
        JJCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);
        // 将涂鸦动作转换成json字符串
        string doodlestring = doodle.stringFromDoodleAction(doodleAction);
        // 发送涂鸦数据
        mediaChannel.sendMessage("消息类型", doodlestring, null);
    }

6.接收涂鸦数据

涂鸦数据发送后，接收方需要在数据收发层完成字符串转换成 JCDoodleAction 对象的操作
::

    /// <summary>
    /// 解析涂鸦数据，涂鸦json字符串转换为涂鸦动作对象
    /// </summary>
    /// <param name="doodleActionData">涂鸦json字符串</param>
    /// <returns>涂鸦动作对象</returns>
    public JCDoodleAction doodleActionFromString(string doodleActionData)

涂鸦数据转换操作完成后会触发 onDoodleReceived 回调，UI 控件通过该方法回调收到的 JCDoodleAction 对象
::

    /// <summary>
    /// 收到涂鸦动作的回调
    /// </summary>
    /// <param name="doodleAction">涂鸦动作对象</param>
    void onDoodleReceived(JCDoodleAction doodleAction);

示例代码::

    public void onDoodleReceived(JCDoodleAction doodleAction)
    {
        JCDoodle doodle = JCDoodle.create(this);
        JCDoodleAction doodleAction = doodle.doodleActionFromString("涂鸦字符串");
    }

以上步骤完成后，即可完成涂鸦的集成。更多信息请参考我们的 Demo。