Windows
=============================

准备工作
---------------------------

开始之前，请您先做好如下准备工作：

- `Windows 版 SDK Download <http://developer.juphoon.com/document/cloud-communication-windows-sdk#2>`_

- :ref:`Windows SDK 配置和初始化<Windows SDK 配置和初始化>`

- :ref:`Windows 登录<Windows 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。


业务集成
---------------------------------

文件存储模块主要涉及 JCStorageItem 类

.. csv-table::
   :file: storage_windows.csv

文件储存对象传输方向有：

.. list-table::

   * - 上传
     - Upload
   * - 下载
     - Download


想要集成文件存储，请按以下步骤操作：

.. highlight:: csharp

首先进行 ``模块的初始化``

::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);
    JCStorage storage = JCStorage.create(client, this);

其中，创建 JCStorage 对象的方法如下::

    /// <summary>
    /// 创建JCStorage对象
    /// </summary>
    /// <param name="client">JCClient对象</param>
    /// <param name="callback">JCStorageCallback 回调接口，用于接收JCStorage通知</param>
    /// <returns>JCStorage对象</returns>
    public static JCStorage create(JCClient.JCClient client, JCStorageCallback callback);

对象创建完成后，既可以进行文件的上传和下载：

- 上传文件
::

    /// <summary>
    /// 上传文件
    /// </summary>
    /// <param name="path">文件路径</param>
    /// <returns>JCStorageItem对象</returns>
    public JCStorageItem uploadFile(string path);

- 下载文件
::

    /// <summary>
    /// 下载文件
    /// </summary>
    /// <param name="uri">文件地址</param>
    /// <param name="savaPath">本地保存地址</param>
    /// <returns>JCStroageItem 对象</returns>
    public JCStorageItem downloadFile(string uri, string savaPath);

.. note:: 上传文件时，文件大小不能超过100MB，文件的存储期限为7天。

示例代码::

    // 上传文件
    storage.uploadFile(path);
    // 下载文件
    storage.downloadFile(uri, savePath);


如果想要取消正在上传或者下载的文件，则调用 cancelFile 接口::

    /// <summary>
    /// 取消传输文件
    /// </summary>
    /// <param name="item">JCStorageItem对象</param>
    /// <returns>是否取消成功</returns>
    private bool cancleFile(JCStorageItem item);

示例代码::

    // 取消上传或下载文件
    storage.cancleFile(item);


文件状态包括文件的传输方向、传输状态、传输进度等，文件状态的改变可以通过调用 onFileUpdate 接口进行回调::

    /// <summary>
    /// 文件状态更新通知
    /// </summary>
    /// <param name="item">文件消息对象，通过该对象可以获得当前文件传输的属性及状态</param>
    void onFileUpdate(JCStorageItem item);

示例代码::

    public void onFileUpdate(JCStorageItem item) {
        if (item.state == JCStorageItemState.Transfering)
        {
            // 文件传输中
        }
        if (item.state == JCStorageItemState.OK)
        {
            // 文件传输成功
        }
        if (item.state == JCStorageItemState.Fail)
        {
            // 文件传输失败
        }
        if(item.state == JCStorageItemState.Cancel)
        {
            // 文件传输取消
        }
    }


其中，文件状态有：

.. list-table::

   * - 文件初始状态
     - Init
   * - 文件传输中状态
     - Transfering
   * - 文件传输成功状态R
     - OK
   * - 文件传输失败状态
     - Fail
   * - 文件传输取消状态
     - Cancel


如果文件传输发生了失败，则原因有：

.. list-table::

   * - 无异常
     - None
   * - 未登录
     - NotLogin
   * - 超时
     - TimeOut
   * - 网络异常
     - NetWork
   * - 文件太大
     - TooLarge
   * - 文件过期
     - Expire
   * - 其他错误
     - Other = 100