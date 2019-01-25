参考文件
=====================

.. _智能分屏:

调整智能模式的布局
-------------------------

setmergemode_i 调整智能模式的布局
::

	  setmergemode_i <mode> <scsMode> [uri]

		   mode: 智能分屏模式 1=freeLayout,2=rectLayout,3=bigSmallX2,4=bigSmallX3....,
			                 11=bigSmallX10,12=bigSmallTop,13=bigSmallBottom

		   scsMode: 智能分屏带屏幕共享模式 1=screen,2=platform,3=platformCover,4=speaker

		   uri: 大小屏默认放大的用户uri

.. note:: 确保合并模式为智能模式(@setmergemode 5), 否则调用完之后要将模式设置为智能模式后才能看到效果；
	      
	没有收到屏幕共享时，模式为mode,当收到屏幕共享时模式会切换成scsMode；

	当模式为mode并且mode>=3且<=13时,uri为大小屏设置的默认放大用户；

	uri若为空或无效用户，则默认第一个用户为大小屏用户。


智能模式的布局调整样例如下：

- 样例1-自由布局
::

    在没有屏幕共享时

    @setmergemode_i 1 4 [username:actor@sample.cloud.justalk.com]

.. image:: images/freelayout.png


- 样例2-矩形布局
::

    在没有屏幕共享时

    @setmergemode_i 2 4 [username:actor@sample.cloud.justalk.com]

.. image:: images/rectangle.png


- 样例3-大屏X2
::

	在没有屏幕共享时
	
	@setmergemode_i 3 4 [username:actor@sample.cloud.justalk.com]

	actor用户显示2倍

.. image:: images/bigscreenX2.png


- 样例4-大屏X3
::

	在没有屏幕共享时
	
	@setmergemode_i 4 4 [username:actor@sample.cloud.justalk.com]

	actor用户显示3倍

.. image:: images/bigscreenX3.png	


- 样例5-小屏底部
::

	在没有屏幕共享时
	
	@setmergemode_i 12 4 [username:actor@sample.cloud.justalk.com]

	actor用户显示在顶部，其它用户显示小屏在底部

.. image:: images/smallscreenb.png			


- 样例6-小屏顶部
::

	在没有屏幕共享时
	
	@setmergemode_i 13 4 [username:actor@sample.cloud.justalk.com]

	actor用户显示在底部，其它用户显示小屏在顶部

.. image:: images/smallscreent.png	


- 样例7-纯屏幕共享
::

	在收到屏幕共享时
	
	@setmergemode_i 1 1 [username:actor@sample.cloud.justalk.com]

.. image:: images/purescreenshare.png		


- 样例8-屏幕共享小屏底部
::

	在收到屏幕共享时
	
	@setmergemode_i 1 2 [username:actor@sample.cloud.justalk.com]

	屏幕共享显示在顶部，其它用户显示小屏在底部

.. image:: images/smallscreenb.png	


- 样例9-讲台模式
::

	在收到屏幕共享时
	
	@setmergemode_i 1 3 [username:actor@sample.cloud.justalk.com]

	屏幕共享显示在顶部，其它用户显示小屏在底部并且遮盖大屏

.. image:: images/teachermode.png		


- 样例10-主持人模式
::

	在收到屏幕共享时
	
	@setmergemode_i 1 4 [username:actor@sample.cloud.justalk.com]

.. image:: images/hostmode.png	


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


调整智能模式的布局和分辨率
---------------------------------

调整智能模式的布局和分辨率

::

    setmerge_i <width> <height> <mode> <scsMode> <fps> [uri]

	   width:宽
	   height:高
	   mode:智能分屏模式 1=freeLayout,2=rectLayout,3=bigSmallX2,4=bigSmallX3....,
			             11=bigSmallX10,12=bigSmallTop,13=bigSmallBottom
	   scsMode:智能分屏带屏幕共享模式 1=screen,2=platform,3=platformCover,4=speaker
	   fps:帧数 1-24
	   uri: 大小屏默认放大的用户uri

将合并模式设置为智能模式，宽和高可以设置任意的，目前实例中暂设为 90P，180P，360P，720P，1080P。

没有收到屏幕共享时，模式为mode，当收到屏幕共享时模式会切换成 scsMode。

当模式为 mode 并且 mode>=3 且 <=13 时，uri为大小屏设置的默认放大用户。

uri 若为空或无效用户，则默认第一个用户为大小屏用户。


分辨率设置样例如下：

- 样例1-1080P
::

	@setmerge_i 1920 1080 1 4 24 [username:actor@sample.cloud.justalk.com]


- 样例2-720P
::

	@setmerge_i 1280 720 1 4 24 [username:actor@sample.cloud.justalk.com]


- 样例3-360P
::

	@setmerge_i 640 360 1 4 24 [username:actor@sample.cloud.justalk.com]


- 样例4-180P
::

	@setmerge_i 320 180 1 4 24 [username:actor@sample.cloud.justalk.com]


- 样例5-90P
::

	@setmerge_i 160 90 1 4 24 [username:actor@sample.cloud.justalk.com]


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _修改录制或推流分辨率:

修改录制或推流分辨率
-------------------------

修改录制或推流分辨率
::

	json字符串需要base64加密
	{"cmd":"SetVideoLevel","MtcConfVideoLevelKey":%x}

	//MtcConfVideoLevelKey 视频尺寸,由小到到可选值:
	                                 0x100即256对应360P
	                                 0x200即512对应480P
	                                 0x300即768对应720P
	                                 0x400即1024对应1080P

	录制器或推流器也是一个confDelivery, 可以通过sendtext命令通知录制器/推流器来操作

	录制器或推流器uri为[username:delivery_会议号@delivery.cloud.justalk.com]


**修改录制或推流分辨率为360P**
::

    {"cmd":"SetVideoLevel","MtcConfVideoLevelKey":0x100}

    base64编码后为
    eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjB4MTAwfQ==

    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjB4MTAwfQ== [username:delivery_12345678@delivery.cloud.justalk.com]
 
**修改录制或推流分辨率为480P**
::

    {"cmd":"SetVideoLevel","MtcConfVideoLevelKey":512}

    base64编码后为
    eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjUxMn0=

    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjUxMn0= [username:delivery_12345678@delivery.cloud.justalk.com]

**修改录制分辨率为720P**
::

    {"cmd":"SetVideoLevel","MtcConfVideoLevelKey":0x300}

    base64编码后为
    eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjB4MzAwfQ==

    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjB4MzAwfQ== [username:delivery_12345678@delivery.cloud.justalk.com]

**修改录制分辨率为1080P**
::

    {"cmd":"SetVideoLevel","MtcConfVideoLevelKey":1024}

    base64编码后为
    eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjEwMjR9
    
    假设会议号为12345678，则指令
    sendtext eyJjbWQiOiJTZXRWaWRlb0xldmVsIiwiTXRjQ29uZlZpZGVvTGV2ZWxLZXkiOjEwMjR9 [username:delivery_12345678@delivery.cloud.justalk.com]


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _修改录制文件大小:

修改录制文件大小
-------------------------

**修改录制文件大小为100KB**
::

    {"cmd":"StartRecord","Video":true,"Storage":{"Protocol":"qiniu","AccessKey":"Th2jW_jP5VSk-wfdVMy5it1oK4iHa8pv8vBZMhbd","SecretKey":"Nh-_pgXrx9weiq8bkAtJPkrBlV3YP13U0Fx0fT-c","BucketName":"ajianzheng","FileKey":"12345678.mp4","SplitFileSize":100}}  
    
    base64编码后为  
    eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwMH19  
    
    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwMH19 [username:delivery_12345678@delivery.cloud.justalk.com]

**修改录制文件大小为1MB**
::

    {"cmd":"StartRecord","Video":true,"Storage":{"Protocol":"qiniu","AccessKey":"Th2jW_jP5VSk-wfdVMy5it1oK4iHa8pv8vBZMhbd","SecretKey":"Nh-_pgXrx9weiq8bkAtJPkrBlV3YP13U0Fx0fT-c","BucketName":"ajianzheng","FileKey":"12345678.mp4","SplitFileSize":1024}}  
   
    base64编码为
    eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwMjR9fQ==
    
    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwMjR9fQ== [username:delivery_12345678@delivery.cloud.justalk.com]

**修改录制文件大小为1GB**
::

    {"cmd":"StartRecord","Video":true,"Storage":{"Protocol":"qiniu","AccessKey":"Th2jW_jP5VSk-wfdVMy5it1oK4iHa8pv8vBZMhbd","SecretKey":"Nh-_pgXrx9weiq8bkAtJPkrBlV3YP13U0Fx0fT-c","BucketName":"ajianzheng","FileKey":"12345678.mp4","SplitFileSize":1048576}}  
    
    base64编码后为
    eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwNDg1NzZ9fQ==
    
    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOjEwNDg1NzZ9fQ== [username:delivery_12345678@delivery.cloud.justalk.com]

**修改录制文件大小为6GB**
::

    {"cmd":"StartRecord","Video":true,"Storage":{"Protocol":"qiniu","AccessKey":"Th2jW_jP5VSk-wfdVMy5it1oK4iHa8pv8vBZMhbd","SecretKey":"Nh-_pgXrx9weiq8bkAtJPkrBlV3YP13U0Fx0fT-c","BucketName":"ajianzheng","FileKey":"12345678.mp4","SplitFileSize": 6291456}}  
    
    base64编码后为
    eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOiA2MjkxNDU2fX0=
    
    假设会议号为12345678，则指令为
    sendtext eyJjbWQiOiJTdGFydFJlY29yZCIsIlZpZGVvIjp0cnVlLCJTdG9yYWdlIjp7IlByb3RvY29sIjoicWluaXUiLCJBY2Nlc3NLZXkiOiJUaDJqV19qUDVWU2std2ZkVk15NWl0MW9LNGlIYThwdjh2QlpNaGJkIiwiU2VjcmV0S2V5IjoiTmgtX3BnWHJ4OXdlaXE4YmtBdEpQa3JCbFYzWVAxM1UwRngwZlQtYyIsIkJ1Y2tldE5hbWUiOiJhamlhbnpoZW5nIiwiRmlsZUtleSI6IjEyMzQ1Njc4Lm1wNCIsIlNwbGl0RmlsZVNpemUiOiA2MjkxNDU2fX0= [username:delivery_12345678@delivery.cloud.justalk.com]