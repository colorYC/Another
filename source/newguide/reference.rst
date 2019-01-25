参考文件
=====================

.. _AppKey 说明:

AppKey 说明
--------------------------------

AppKey 是应用在 Juphoon Cloud Platform 中的唯一标识。

在创建应用时，同一个控制台帐号中创建的多个应用属于同一个域。同域中的应用是可以互通的，所以同一帐号下的应用即时 AppKey 不同，也可以进行业务往来。例如，从一个应用登录的用户可以呼叫另一个应用登录的用户。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _iOS 导入动态库:

iOS 导入动态库
---------------------

在 Mac 环境下解压下载的压缩包，解压后的文件夹内有 JCSDK 文件夹。JCSDK 文件夹里包含了 JCSDKOC.framework。

**拷贝文件**

将 JCSDK 文件夹拷贝到您工程所在的目录下，如下图（仅供参考）：

.. image:: images/sdkshare1.png

**工程设置**

1. 导入文件和库

点击 ‘General’，在 “Embedded Binaries” 一栏，点击 ‘+’ 符号，然后导入 JCSDK 文件夹下的 JCSDKOC.framework

.. image:: images/frameworkshare.png

点击 ‘General’，点击 ‘+’ 符号，在 “Linked Frameworks and Libraries” 一栏，导入 SDK 依赖的其他库，如下图：

.. image:: images/sharelibinput.png

2. 设置 Framework Search Paths 路径

点击 ‘Build Settings’，找到 Framework Search Paths 和 Header Search Paths，在右侧输入路径。如下图：

.. image:: images/iOS_integration_AddDynamicFrameworkPath.png

.. image:: images/headerfile.png

.. note:: 在设置 Framework Search Paths 时，一般在完成第1步导入 JCSDKOC.framework 后，Xcode 会自动生成该路径
       如果 Xcode 没有自动生成路径，用户要根据 JCSDKOC.framework 和 lib 库文件所在目录，手动设置路径。

3. 设置 Enable Bitcode 为 NO

点击 ‘Build Settings’，找到 Enable Bitcode 设置为 NO，如下图：

.. image:: images/iOS_integration_DynamicBitcode.png

4. 设置预处理宏定义

点击 ‘Build Settings’，找到 Preprocessor Macros，在右侧输入 ZPLATFORM=ZPLATFORM_IOS，如下图：

.. image:: images/static_import_ios.png

5. 设置 Documentation Comments 为 NO

点击 ‘Build Settings’，找到 Documentation Comments 设置为 NO，如下图：

.. image:: images/static_import_ios1.png

6. 设置后台运行模式

点击 ‘Capabilities’，找到 Background Modes，勾选红框内的 Audio, AirPlay, and Picture in Picture，如下图：

.. image:: images/iOS_integration_DynamicBackgroundModes.png

**权限设置**

7. 设置麦克风和摄像头的权限

点击 ‘Info’，然后添加麦克风和摄像头权限，如下图：

.. image:: images/iOS_integration_DynamicPermissions.png

8. 编译运行

以上步骤进行完后，编译工程，如果没有报错，恭喜您，您已经成功配置 SDK，可以进行下一步了。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _登录模式说明:

登录模式说明
------------------------

Juphoon Cloud Platform 提供 2 种登录的鉴权模式：

- Cloud 鉴权
- 免鉴权

两种鉴权模式的详细信息如下：

.. list-table::
   :header-rows: 1

   * - 签权方式
     - 公有云账号
     - 说明
   * - Cloud签权
     - 帐号由用户自行生成
     - 用户名、密码等信息保存在 Juphoon Cloud Platform 服务器中，服务器会检查密码
   * - 免签权
     - 帐号由用户自行生成
     - 用户名、密码等信息保存在 Juphoon Cloud Platform 服务器中，服务器不检查密码

关于 “免鉴权模式下，客户使用自己的账号体系” 的说明请参考 :ref:`如何使用自己的账号体系<如何使用自己的账号体系>` 。

.. note:: 在同一个控制台帐号中创建的多个应用属于同一个域，同域的不同 Appkey 可以互通。底层允许同域可以互通，不同域不能互通。因此业务在同域的不同应用间是可以互通的。例如，在上层业务允许的情况下，从一个应用登录的用户可以呼叫另一个应用登录的用户。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Android 权限说明:

Android 权限说明
--------------------------

.. list-table::
   :header-rows: 1

   * - 权限
     - 介绍
   * - INTERNET
     - 网络权限，登录与通话必须
   * - ACCESS_NETWORK_STATE
     - 访问网络状态权限，登录与通话必须
   * - ACCESS_WIFI_STATE
     - 访问WIFI权限，登录与通话必须
   * - READ_PHONE_STATE
     - 访问设备信息权限，登录必须
   * - WAKE_LOCK
     - 保持后台在线权限，保持后台在线必须
   * - CAMERA
     - 摄像头权限，视频通话必须
   * - RECORD_AUDIO
     - 录音权限，通话必须
   * - MODIFY_AUDIO_SETTINGS
     - 修改音频设置权限，通话必须
   * - WRITE_EXTERNAL_STORAGE
     - 读写外部存储权限，存储必须
   * - VIBRATE
     - 手机振动
   * - BLUETOOTH
     - 蓝牙功能
   * - SYSTEM_ALERT_WINDOW
     - 悬浮窗功能