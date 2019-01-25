Android 群组集成
===================================

.. highlight:: java


准备工作
---------------------------

开始之前，请您先做好如下准备工作：

- `Android 版 SDK 下载 <http://developer.juphoon.com/document/cloud-communication-android-sdk#2>`_

- :ref:`Android SDK 配置和初始化<Android SDK 配置和初始化>`

- :ref:`Android 登录<Android 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。


业务集成
----------------------------------

群组涉及群组实例类（JCGroupItem）和群组成员类（JCGroupMember）。

JCGroupItem 有以下属性：

.. list-table::

   * - 群组标识
     - String groupId
   * - 群组名称
     - String name
   * - 最新一次更新时间
     - int changeState

JCGroupMember 有以下属性：

.. list-table::

   * - 群组标识
     - String groupId
   * - 用户标识
     - String userId
   * - 服务器端用户标识

        @warning 当通知成员变化时，changeState 为 
        
        GROUP_CHANGE_STATE_REMOVE 时只能通过此参数来判断，
       
        不能通过 userId
     - String uid
   * - 用户昵称
     - String displayName
   * - 角色类型，参见 JCGroupMemberType
     - int memberType
   * - 成员改变状态
     - int changeState

群成员类型（JCGroupMemberType）有以下三种：

.. list-table::

   * - 拥有者
     - public static final int GROUP_MEMBER_TYPE_OWNER = 0
   * - 管理者
     - public static final int GROUP_MEMBER_TYPE_MANAGER = 1
   * - 群成员
     - public static final int GROUP_MEMBER_TYPE_MEMBER = 2


**群组集成**

.. image:: images/group_management2.png

**开始集成群组功能前，请先进行** ``模块的初始化``
::
    
    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(context, "your appkey", this, null);
    JCGroup group = JCGroup.create(client, this);

其中，创建 JCGroup 实例的方法如下
::

    /**
     *  创建 JCGroup 对象
     *  @param client JCClient 对象
     *  @param callback JCGroupCallback 回调接口，用于接收 JCGroup 相关通知
     *  @return 返回 JCGroup 对象
     */
    public static JCGroup create(JCClient client, JCGroupCallback callback) 

**开始集成**

**群组管理**

1. 创建群
::

    /**
     *  创建群
     *  @param members JCGroupMemeber 队列
     *  @param groupName 群名称
     *  @return 返回操作id
     */
    public abstract int createGroup(List<JCGroupMember> members, String groupName);

示例代码::

    List<JCGroupMember> members = new ArrayList<>();
    for (String userId: userIds) {
        members.add(new JCGroupMember(null, userId, userId, JCGroup.GROUP_MEMBER_TYPE_MEMBER, JCGroup.GROUP_CHANGE_STATE_ADD));
    }
    group.createGroup(members, "test");


创建群回调
::

    /**
     *  创建群回调
     *  @param operationId 操作表示，由 createGroup 接口返回
     *  @param result true 表示登陆成功，false 表示登陆失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupItem JCGroupItem 对象
     *  @see JCGroup.Reason
     */
    void onCreateGroup(int operationId, boolean result, @JCGroup.Reason int reason,
                       JCGroupItem groupItem);


其中，JCGroup.Reason 有以下几种：

.. list-table::

   * - 无异常
     - public static final int REASON_NONE = 0
   * - 未登录
     - public static final int REASON_NOT_LOGIN = 1
   * - 函数调用失败
     - public static final int REASON_CALL_FUNCTION_ERROR = 2
   * - 超时
     - public static final int REASON_TIME_OUT = 3
   * - 网络异常
     - public static final int REASON_NETWORK = 4
   * - 参数错误
     - public static final int REASON_PARAM_INVALID = 5
   * - 其他错误
     - public static final int REASON_OTHER = 100


2. 获取群列表
::

    /**
     *  获取当前用户所有加入的群列表，结果通过 JCGroupCallback 中相应回调返回
     *  @param updateTime 最新一次记录的群列表服务器更新时间
     *  @return 返回操作id
     */
    public abstract int fetchGroups(long updateTime);

示例代码::

    group.fetchGroups(updateTime);


获取群列表结果回调
::

    /**
     *  拉取群列表结果回调
     *  @param operationId 操作表示，由 fetchGroups 接口返回
     *  @param result true 表示获取成功，false 表示获取失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groups 群列表
     *  @param updateTime 服务器更新时间
     *  @param fullUpdated 是否全更新
     *  @see JCGroup.Reason
     */
    void onFetchGroups(int operationId, boolean result, @JCGroup.Reason int reason,
                       List<JCGroupItem> groups, long updateTime, boolean fullUpdated);


当群列表发生了改变，会收到 onGroupListChange 回调，此时可以调用 fetchGroups 接口获取更新
::

    /**
     *  群列表更新，调用 JCGroup fetchGroups 获取更新
     */
    void onGroupListChange();

3. 获取群信息
::

    /**
     *  刷新群组信息
     *  @param groupId 群标识
     *  @param updateTime 最新一次记录的该群服务器更新时间
     *  @return 返回操作id
     */
    public abstract int fetchGroupInfo(String groupId, long updateTime);

示例代码::

    group.fetchGroupInfo(groupId, updateTime);


获取群详情结果回调
::

    /**
     *  拉取群详情结果回调
     *  @param operationId 操作表示，由 fetchGroupInfo 接口返回
     *  @param result true 表示获取成功，false 表示获取失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupItem JCGroupItem 对象
     *  @param members 成员列表
     *  @param updateTime 服务器更新时间
     *  @param fullUpdated 是否全更新
     *  @see JCGroup.Reason
     */
    void onFetchGroupInfo(int operationId, boolean result, @JCGroup.Reason int reason,
                          JCGroupItem groupItem, List<JCGroupMember> members,
                          long updateTime, boolean fullUpdated);


当群信息发生了改变，会收到 onGroupInfoChange 回调，此时可以调用 fetchGroupInfo 接口获取更新
::

    /**
     *  群信息更新，调用 JCGroup fetchGroupInfo 获取更新
     *  @param groupId 群标识
     */
    void onGroupInfoChange(String groupId);


4. 添加、删除、更新群成员
::

    /**
     *  操作成员
     *  @param groupId 群标识
     *  @param members JCGroupMemeber 对象列表，通过 changeState 值来表明增加，更新，删除成员操作
     *  @return 返回操作id
     */
    public abstract int dealMembers(String groupId, List<JCGroupMember> members);

.. note:: 只有群主才可以删除成员。

其中，群变化状态（JCGroupChangeState）有以下几种：

.. list-table::

   * - 无
     - public static final int GROUP_CHANGE_STATE_NONE = 0
   * - 新增
     - public static final int GROUP_CHANGE_STATE_ADD = 1
   * - 更新
     - public static final int GROUP_CHANGE_STATE_UPDATE = 2
   * - 删除
     - public static final int GROUP_CHANGE_STATE_REMOVE = 3


示例代码::

    List<JCGroupMember> members = new ArrayList<>();
    for (String userId: userIds) {
      members.add(new JCGroupMember(groupId, userId, userId, JCGroup.GROUP_MEMBER_TYPE_MEMBER, JCGroup.GROUP_CHANGE_STATE_ADD));
    }
    group.dealMembers(groupId, members);


dealMembers 结果回调
::

    /**
     *  dealMembers 结果回调
     *  @param operationId 操作表示，由 dealMembers 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 Reason
     */
    void onDealMembers(int operationId, boolean result, @JCGroup.Reason int reason);
    

5. 修改昵称
::
    
    /**
     *  更新昵称
     *  @param selfInfo JCGroupMember 对象，请传入 groupId，displayName，memberType(保持不变)
     *  @return 返回操作id
     */
    public abstract int updateSelfInfo(JCGroupMember selfInfo);


示例代码::

    group.updateSelfInfo(selfInfo);

6. 更新群，修改群名称
::

    /**
     *  更新群
     *  @param groupItem JCGroupItem 对象，其中 JCGroupItem 中 changeState 值不影响具体操作
     *  @return 返回操作id
     */
    public abstract int updateGroup(JCGroupItem groupItem);


示例代码::

    group.updateGroup(groupItem);


更新群信息回调
::

    /**
     *  更新群信息调用回调
     *  @param operationId 操作表示，由 updateGroup 接口返回
     *  @param result true 表示登陆成功，false 表示登陆失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupId 群标识
     *  @see JCGroup.Reason
     */
    void onUpdateGroup(int operationId, boolean result, @JCGroup.Reason int reason, String groupId);


7. 离开群组
::
    
    /**
     *  离开群组
     *  @param groupId 群标识
     *  @return 返回操作id
     */
    public abstract int leave(String groupId);

示例代码::

    group.leave(groupId);


离开群组回调
::

    /**
     *  离开群组回调
     *  @param operationId 操作表示，由 leave 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 Reason
     *  @param groupId 群标识
     */
    void onLeave(int operationId, boolean result, @JCGroup.Reason int reason, String groupId);

8. 解散群组
::

    /**
     *  解散群组，Owner才能解散群组
     *  @param groupId 群标识
     *  @return 返回操作id
     */
    public abstract int dissolve(String groupId);

示例代码::

    group.dissolve(groupId);

解散群组回调
::

    /**
     *  解散群组回调
     *  @param operationId 操作表示，由 dissolve 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 Reason
     *  @param groupId 群标识
     */
    void onDissolve(int operationId, boolean result, @JCGroup.Reason int reason, String groupId);


**收发群组消息**

在群组中发送和接收消息参见 :ref:`消息<消息>`。