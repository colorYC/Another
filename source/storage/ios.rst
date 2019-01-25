iOS
===============================

准备工作
---------------------------

开始之前，请您先做好如下准备工作：

- `iOS 版 SDK Download <http://developer.juphoon.com/document/cloud-communication-ios-sdk#2>`_

- :ref:`iOS SDK 配置和初始化<iOS SDK 配置和初始化>`

- :ref:`iOS 登录<iOS 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。


业务集成
----------------------------------

文件存储模块主要涉及 JCStorageItem 类，其属性如下：

.. csv-table::
   :file: storage_ios.csv

文件传输方向（JCStorageItemDirection）有上传和下载两种：

.. list-table::

   * - 上传
     - JCStorageItemDirectionUpload
   * - 下载
     - JCStorageItemDirectionDownload


想要集成文件存储，请按以下步骤操作：

.. highlight:: objective-c

首先进行 ``模块的初始化``

::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCStorage *storage = [JCStorage create:client callback:self];

其中，创建 JCStorage 对象的方法如下
::

    /**
     *  @brief 创建 JCStorage 对象
     *  @param client JCClient 对象
     *  @param callback JCStorageCallback 回调接口，用于接收 JCStorage 相关通知
     *  @return 返回 JCStorage 对象
     */
    +(JCStorage*)create:(JCClient*)client callback:(id<JCStorageCallback>)callback;

对象创建完成后，既可以进行文件的上传和下载：

- 上传文件

::

    /**
     *  @brief 上传文件
     *  @param path 文件路径
     *  @return 返回 JCStorageItem 对象
     *  @warning 文件大小不要超过100MB，异常返回 nil
     */
    -(JCStorageItem*)uploadFile:(NSString*)path;

.. note:: 上传文件时，文件大小不能超过100MB，文件的存储期限为7天。

- 下载文件

::

    /**
     *  @brief 下载文件
     *  @param uri 文件地址
     *  @param savePath 本地文件保存地址
     *  @return 返回 JCStorageItem 对象，异常返回 nil
     */
    -(JCStorageItem*)downloadFile:(NSString*)uri savePath:(NSString*)savePath;


示例代码::

    // 上传文件
    [storage uploadFile:path];
    // 下载文件
    [storage downloadFile:uri savePath:savePath];


如果想要取消正在上传或者下载的文件，可以调用 cancelFile 接口::

    /**
     *  @brief 取消正在进行的文件上传下载
     *  @param item JCStorageItem对象，由 uploadFile，downloadFile 返回
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)cancelFile:(JCStorageItem*)item;

示例代码::

    // 取消上传或下载文件
    [storage cancelFile:item];

文件状态包括文件的传输状态、传输进度等，如果文件状态发生了改变，则会收到 onFileUpdate 回调::

    /**
     *  @brief 文件状态更新通知
     *  @param item 文件消息对象，通过该对象可以获得当前文件传输的属性及状态
     *  @see JCStorageItem
     */
    -(void)onFileUpdate:(JCStorageItem*)item;

其中，文件传输状态（JCStorageItemState）有：

.. list-table::

   * - 文件初始状态
     - JCStorageItemStateInit
   * - 文件传输中状态
     - JCStorageItemStateTransfering
   * - 文件传输成功状态
     - JCStorageItemStateOK
   * - 文件传输失败状态
     - JCStorageItemStateFail
   * - 文件传输取消状态
     - JCStorageItemStateCancel
    

示例代码::

    -(void)onFileUpdate:(JCStorageItem*)item {
      if (item.state == JCStorageItemStateTransfering) {
          if (item.direction == JCStorageItemDirectionUpload) {
              // 文件上传中
          } else if (item.direction == JCStorageItemDirectionDownload) {
              // 文件下载中
          }
      } else if (item.state == JCStorageItemStateCancel) {
          // 取消文件传输
      }
    }



如果文件传输发生了失败，则原因（JCStorageReason）有以下几种：

.. list-table::

   * - 无异常
     - JCStorageReasonNone
   * - 未登录
     - JCStorageReasonNotLogin
   * - 超时
     - JCStorageReasonTimeOut
   * - 网络异常
     - JCStorageReasonNetWork
   * - 文件太大
     - CStorageReasonTooLarge
   * - 文件过期
     - JCStorageReasonExpire