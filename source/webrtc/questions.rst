常见问题
--------------------

1. 关于录制文件的命名规则和路径问题。

 关于录制文件的命名规则：录制文件的命名规则由会议号和可读日期时间组成，比如 10316548_2017-10-30_14-15-21.mp4 表示会议号是 10316548，2017年10月30日14点15分21秒开始录制。

 如果视频录制超过设置的SplitFileSize 大小，就会分割成：会议号_日期时间 数字. 文件后缀。

 关于存储路径：存储路径在部署的时候设置好，代码里面写死，建议不要修改， 建议设个软连接到此目录上， 如果修改的话，要同步修改自动清理脚本文件。

2. 关于 Chrome 的屏幕共享插件安装问题。

 （1）在 https://www.justalkcloud.com/webrtc/ 网站上下载屏幕共享插件并解压；

 （2）打开 Chrome 输入 chrome://extensions/，勾上开发者模式，点击加载已解压的扩展程序，选择刚刚解压的屏幕共享插件。

  举例 (以Mac版Chrome为例，Windows版类似)：

  - 打开 https://www.justalkcloud.com/webrtc/。
  - 点击“屏幕共享插件下载”。
  - 下载屏幕共享插件 screen_share_kit.zip 到一个本地目录，然后解压。
  - 打开 Chrome 输入 chrome://extensions/，选中开发者模式(Developer mode)，并点击载入解压扩展包(Load unpacked extension)，如下图：

  .. image:: images/webrtc_screenshare1.png

  - 弹出窗口，选择刚刚解压的屏幕共享插件的目录，进入 ..\screen_share_kit\3.4_0 目录，点击选择(select)。

  .. image:: images/webrtc_screenshare2.png

  - 屏幕共享插件安装成功，可能会提示仅在 https 环境下使用。
  
  .. image:: images/webrtc_screenshare3.png


  - 关闭开发者模式，后续屏幕共享功能可正常使用。

  .. image:: images/webrtc_screenshare4.png

  .. image:: images/webrtc_screenshare5.png

3. onactorstate 接口成员状态改变时的各个状态代表什么意思。

  onactorstate 各个状态分别是 role, state 和 subscribe。

  其中：

::

    // role  是角色位整型：
             0x2 - 发送者，只有发送者才能被订阅
             0x4 - 演示者，其事件将被广播，打开连接时设置 mtcConfRoleKey
             0x8 - 拥有者，第一个加入会议的人 
    // state 是状态位整型：
             0x1 - 转发视频，打开连接时设置 mtcConfStateKey 
             0x2 - 转发音频，打开连接时设置 mtcConfStateKey 
             0x4 - 打开视频,本地是否打开摄像头
             0x8 - 打开音频,本地是否打开音频设备
    // subscribe 是订阅信息
             0   - 不订阅视频
             1   - 订阅视频

