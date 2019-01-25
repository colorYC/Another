
.. _配置参数(ios):

配置参数-iOS
-------------------------------

您可以通过下面的配置参数进行频道的相关设置。

iOS 中配置参数(key值)如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - JCMediaChannelJoinParamRecord
     - 音视频录制参数，必须在 join 前通过 setConfig 设置
   * - JCMediaChannelJoinParamCdn
     - 推流Cdn，必须在 join 前通过 setConfig 设置
   * - JCMediaChannelConfigCapacity
     - 设置频道人数，必须在 join 前通过 setConfig 设置
   * - JCMediaChannelJoinParamPassword
     - 设置媒体通道密码
   * - JCMediaChannelConfigSipCallerMumber
     - SIP呼叫 主叫号码
   * - JCMediaChannelConfigSipCoreNetwork
     - 设置 SIP呼叫 核心网ID
   * - JCMediaChannelConfigSipParam
     - 设置 SIP呼叫 参数, 通过 JCMediaChannelUtils.buildSipParam 接口进行构造
   * - JCMediaChannelJoinParamMaxResolution
     - 设置频道最大分辨率
   * - JCMediaChannelConfigSmoothMode
     - 设置平滑模式，保证弱网环境下视频流畅

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _配置参数(android):

配置参数-Android
-------------------------------

您可以通过下面的配置参数进行频道的相关设置。

Android 中配置参数(key值)如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - CONFIG_CAPACITY = "config_capacity";
     - 设置频道人数
   * - CONFIG_SIP_CALLER_NUMBER = "config_sip_caller_number";
     - 设置 SIP呼叫 主叫号码
   * - CONFIG_SIP_CORE_NETWORK = "config_sip_core_network";
     - 设置 SIP呼叫 核心网ID
   * - CONFIG_NOTIFY_VOLUME_CHANGE = "config_notify_volume_change";
     - 设置是否上报音量变化，音量变化会比较频繁，默认为"1"，不需要则设置为"0"
   * - CONFIG_SIP_PARAM = "config_sip_param"
     - 设置 SIP呼叫 参数, 通过 JCMediaChannelUtils.buildSipParam 接口进行构造


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _配置参数(windows):

配置参数-Windows
-------------------------------

您可以通过下面的配置参数进行频道的相关设置。

Windows 中配置参数(key值)如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - JCMediaChannelConfigCapacity = "ConfigCapacity";
     - 设配置频道人数上限，必须在join前通过setConfig设置
   * - JCMediaChannelConfigSipCallerNumber = "ConfigSipCallerNumber";
     - 配置Sip呼叫主叫号码
   * - JCMediaChannelConfigSipCoreNetwork = "ConfigSipCoreNetwork";
     - 配置Sip核心网
   * - JCMediaChannelConfigNotifyVolumeChange = "ConfigNotifyVolumeChange";
     - 配置是否上报音量变化，音量变化会比较频繁，默认为1，不需要则设置为0

        