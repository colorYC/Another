Android
========================

.. highlight:: java

1.Android 日志地址在哪里看？

- 日志路径如下:
  /sdcard/Android/data/工程包名/files/log/

2.通话音量如何调节？

::

    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_VOLUME_UP || keyCode == KeyEvent.KEYCODE_VOLUME_DOWN) {
            int direction = keyCode == KeyEvent.KEYCODE_VOLUME_UP ? AudioManager.ADJUST_RAISE : AudioManager.ADJUST_LOWER;
            int flags = AudioManager.FX_FOCUS_NAVIGATION_UP;
            AudioManager am = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
            am.adjustStreamVolume(AudioManager.STREAM_VOICE_CALL, direction, flags);
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }

3.混淆配置

如果你的 apk 最终会经过代码混淆，请在 proguard 配置文件中加入以下代码::

    -dontwarn  com.juphoon.*
    -keep class com.juphoon.**{*;}

4. Android 在被登出的时候收到多个 JCLogoutedNotification 的通知，是什么原因？

- 界面注册了多个广播监听导致。

5. 初始化问题

- 如果遇到初始化问题，可以参考 `此贴 <http://developer.juphoon.com/portal/cn/bbs/problem_details.php?t_id=1266>`_ 。

6. 主叫拨打电话，被叫没有收到 JCCallIncomingd 的通知，可能是什么原因？

- 主叫或者被叫处于断网的状况。
- 主叫调用 JCClient.Login 之后在收到 JCCliNotifyLoginOk 通知之前发起呼叫，这个时候 SDK 的环境还不存在，导致无法呼叫。