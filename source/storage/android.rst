Android
==========================

准备工作
---------------------------

开始之前，请您先做好如下准备工作：

- `Android 版 SDK Download <http://developer.juphoon.com/document/cloud-communication-android-sdk#2>`_

- :ref:`Android SDK 配置和初始化<Android SDK 配置和初始化>`

- :ref:`Android 登录<Android 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。


业务集成
----------------------------------

文件存储模块主要涉及 JCStorageItem 类

.. csv-table::
   :file: storage.csv

文件传输方向有：

.. list-table::

   * - 上传
     - public static final int DIRECTION_UPLOAD = 0
   * - 下载
     - public static final int DIRECTION_DOWNLOAD = 1


想要集成文件存储，请按以下步骤操作：

.. highlight:: java

首先进行 ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCStorage storage = JCStorage.create(client, this);

其中，创建 JCStorage 对象的方法如下::

    /**
     * 创建 JCStorage 对象
     *
     * @param client   JCClient 对象
     * @param callback JCStorageCallback 回调接口，用于接收 JCStorage 相关通知
     * @return 返回 JCStorage 对象
     */
    public static JCStorage create(JCClient client, JCStorageCallback callback)

对象创建完成后，既可以进行文件的上传和下载：

- 上传文件
::

    /**
     * 上传文件
     *
     * @param path 文件路径
     * @return 返回 JCStorageItem 对象, 文件大小不要超过100MB，异常返回 null
     */
    public abstract JCStorageItem uploadFile(String path);


- 下载文件
::

    /**
     * 下载文件
     *
     * @param uri      文件地址
     * @param savePath 本地文件保存地址
     * @return 返回 JCStorageItem 对象，异常返回 null
     */
    public abstract JCStorageItem downloadFile(String uri, String savePath);

.. note:: 上传文件时，文件大小不能超过100MB，文件的存储期限为7天。

示例代码::

    // 上传文件
    storage.uploadFile(path);
    // 下载文件
    storage.downloadFile(uri, savePath);


如果想要取消正在上传或者下载的文件，则调用 cancelFile 接口::

    /**
     * 取消正在进行的文件上传下载
     *
     * @param item JCStorageItem对象，由 uploadFile，downloadFile 返回
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean cancelFile(JCStorageItem item);


示例代码::

    // 取消文件上传或下载
    storage.cancelFile(item);


文件状态包括文件的传输方向、传输状态、传输进度等，文件状态的改变可以通过调用 onFileUpdate 接口进行回调::

    /**
     * 文件状态更新通知
     *
     * @param item 文件消息对象，通过该对象可以获得当前文件传输的属性及状态
     * @see JCStorageItem
     */
    void onFileUpdate(JCStorageItem item);

其中，文件状态有：

.. list-table::

   * - 文件初始状态
     - public static final int ITEM_STATE_INIT = 0
   * - 文件传输中状态
     - public static final int ITEM_STATE_TRANSFERRING = 1
   * - 文件传输成功状态R
     - public static final int ITEM_STATE_OK = 2
   * - 文件传输失败状态
     - public static final int ITEM_STATE_FAIL = 3
   * - 文件传输取消状态
     - public static final int ITEM_STATE_CANCEL = 4


示例代码::

    public void onFileUpdate(JCStorageItem item) {
        if (item.getState() == JCStorage.ITEM_STATE_TRANSFERRING) {
            // 文件传输中
        } else if (item.getState() == JCStorage.ITEM_STATE_OK) {
            if (item.getDirection() == JCStorage.DIRECTION_UPLOAD) {
                // 上传文件成功
            } else if(item.getDirection() == JCStorage.DIRECTION_DOWNLOAD) {
                // 下载文件成功
            }
        } else if (item.getState() == JCStorage.ITEM_STATE_CANCEL) {
            // 取消文件传输
        }
        ...
    }


如果文件传输发生了失败，则原因有：

.. list-table::

   * - 无异常
     - public static final int REASON_NONE = 0
   * - 未登录
     - public static final int REASON_NOT_LOGIN = 1
   * - 超时
     - public static final int REASON_TIMEOUT = 2
   * - 网络异常
     - public static final int REASON_NETWORK = 3
   * - 文件太大
     - public static final int REASON_TOOLARGE = 4
   * - 文件过期
     - public static final int REASON_EXPIRE = 5
   * - 其他错误
     - public static final int REASON_OTHER = 100
