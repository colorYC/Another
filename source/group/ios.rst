iOS 群组集成
=================================

.. highlight:: objective-c

准备工作
---------------------------

开始之前，请您先做好如下准备工作：

- `iOS 版 SDK 下载 <http://developer.juphoon.com/document/cloud-communication-ios-sdk#2>`_

- :ref:`iOS SDK 配置和初始化<iOS SDK 配置和初始化>`

- :ref:`iOS 登录<iOS 登录>`

如果您已经做好相关准备工作，即可继续以下的内容。

业务集成
----------------------------------

群组涉及群组实例类（JCGroupItem）和群组成员类（JCGroupMember）。

JCGroupItem 有以下属性：

.. list-table::

   * - 群组标识
     - NSString* groupId
   * - 群组名称
     - NSString* name
   * - 群组改变状态
     - JCGroupChangeState changeState
   * - 最新一次更新时间
     - long long updateTime

JCGroupMember 有以下属性：

.. list-table::

   * - 群组标识
     - NSString* groupId
   * - 用户标识
     - NSString* userId
   * - 服务器端用户标识

        @warning 当通知成员变化时，changeState 为 
        
        JCGroupChangeStateRemove 时，
       
        只能通过此参数来判断，不能通过 userId
     - NSString* uid
   * - 用户昵称
     - NSString* displayName
   * - 角色类型，参见 JCGroupMemberType
     - JCGroupMemberType memberType
   * - 成员改变状态
     - JCGroupChangeState changeState

群成员类型（JCGroupMemberType）有以下三种：

.. list-table::

   * - 拥有者
     - JCGroupMemberTypeOwner
   * - 管理者
     - JCGroupMemberTypeManager
   * - 群成员
     - JCGroupMemberTypeMember


**群组集成**

群组功能及对应接口

.. image:: images/group_management2.png

**开始集成群组功能前，请先进行** ``模块的初始化``
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCGroup* group = [JCGroup create:client callback:self];

其中，创建 JCGroup 实例的方法如下
::

    /**
     *  @brief 创建 JCGroup 对象
     *  @param client JCClient 对象
     *  @param callback JCGroupCallback 回调接口，用于接收 JCGroup 相关通知
     *  @return 返回 JCGroup 对象
     */
    +(JCGroup*)create:(JCClient*)client callback:(id<JCGroupCallback>)callback;

**开始集成**

**群组管理**

1. 创建群
::

    /**
     *  @brief 创建群
     *  @param members JCGroupMemeber 队列
     *  @param groupName 群名称
     *  @return 返回操作id
     */
    -(int)createGroup:(NSArray*)members groupName:(NSString*)groupName;

示例代码::

    NSMutableArray* members = [[NSMutableArray alloc] init];
    [members addObject:[[JCGroupMember alloc] init:@"test" userId:client.userId displayName:client.displayName memberType:JCGroupMemberTypeOwner changeState:JCGroupChangeStateNone]];
    [group createGroup:members groupName:groupName];

群创建后会收到 onCreateGroup 回调
::

    /**
     *  @brief 创建群回调
     *  @param operationId 操作表示，由 createGroup 接口返回
     *  @param result true 表示登录成功，false 表示登录失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupItem JCGroupItem 对象
     *  @see JCGroupReason
     */
    -(void)onCreateGroup:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupItem:(JCGroupItem*)groupItem;

示例代码::

    -(void)onCreateGroup:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupItem:(JCGroupItem*)groupItem
    {
        if (result) {
            NSLog(@"创建群成功");
        } else {
            NSLog(@"创建群失败");
        }
    }


其中，群原因（JCGroupReason）有以下几种：

.. list-table::

   * - 正常
     - JCGroupReasonNone
   * - 未登录
     - JCGroupReasonNotLogin
   * - 函数调用失败
     - JCGroupReasonCallFunctionError
   * - 超时
     - JCGroupReasonTimeOut
   * - 网络异常
     - JCGroupReasonNetWork
   * - 参数错误
     - JCGroupResonInvalid
   * - 其他错误
     - JCGroupReasonOther = 100


2. 获取当前用户加入的所有群列表
::

    /**
     *  @brief 获取当前用户所有加入的群列表，结果通过 JCGroupCallback 中相应回调返回
     *  @param updateTime 最新一次记录的群列表服务器更新时间
     *  @return 返回操作id
     */
    -(int)fetchGroups:(long long)updateTime;

示例代码::

    [group fetchGroups:updateTime];

fetchGroups 接口调用后会收到 onFetchGroups 回调
::

    /**
     *  @brief 拉取群列表结果回调
     *  @param operationId 操作表示，由 fetchGroups 接口返回
     *  @param result true 表示获取成功，false 表示获取失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groups 群列表
     *  @param updateTime 服务器更新时间
     *  @param fullUpdate 是否全更新
     *  @see JCGroupReason
     */
    -(void)onFetchGroups:(int)operationId result:(bool)result reason:(JCGroupReason)reason groups:(NSArray*)groups updateTime:(long long)updateTime fullUpdate:(bool)fullUpdate;

示例代码::

    -(void)onFetchGroups:(int)operationId result:(bool)result reason:(JCGroupReason)reason groups:(NSArray*)groups updateTime:(long long)updateTime fullUpdate:(bool)fullUpdate
    {
        if (result) {
            // item 来源于JCGroupItem对象
            if (item.changeState == JCGroupChangeStateAdd) {
                ...
            } else if (item.changeState == JCGroupChangeStateRemove) {
                ...
            } else if (item.changeState == JCGroupChangeStateUpdate) {
                ...
            }
        }
    }


当群列表发生了改变，会收到 onGroupListChange 回调，此时可以调用 fetchGroups 接口获取更新
::

    /**
     *  @brief 群列表更新，调用 JCGroup fetchGroups 获取更新
     */
    -(void)onGroupListChange;


3. 刷新群组信息
::

    /**
     *  @brief 刷新群组信息
     *  @param groupId 群标识
     *  @param updateTime 最新一次记录的该群服务器更新时间
     *  @return 返回操作id
     */
    -(int)fetchGroupInfo:(NSString*)groupId updateTime:(long long)updateTime;

示例代码::

    // groupId 来源于JCGroupItem对象
    [group fetchGroupInfo:groupId updateTime:updateTime];

fetchGroupInfo 接口调用后会收到 onFetchGroupInfo 回调
::

    /**
     *  @brief 拉取群详情结果回调
     *  @param operationId 操作表示，由 fetchGroupInfo 接口返回
     *  @param result true 表示获取成功，false 表示获取失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupItem JCGroupItem 对象
     *  @param members 成员列表
     *  @param updateTime 服务器更新时间
     *  @param fullUpdate 是否全更新
     *  @see JCGroupReason
     */
    -(void)onFetchGroupInfo:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupItem:(JCGroupItem*)groupItem members:(NSArray*)members updateTime:(long long)updateTime fullUpdate:(bool)fullUpdate;

当群信息发生了改变，会收到 onGroupInfoChange 回调，此时可以调用 fetchGroupInfo 接口获取更新
::

    /**
     *  @brief 群信息更新，调用 JCGroup fetchGroupInfo 获取更新
     *  @param groupId 群标识
     */
    -(void)onGroupInfoChange:(NSString*)groupId;


4. 更新昵称
::

    /**
     *  @brief 更新昵称
     *  @param selfInfo JCGroupMember 对象，请传入 groupId，displayName，memberType(保持不变)
     *  @return 返回操作id
     */
    -(int)updateSelfInfo:(JCGroupMember*)selfInfo;

示例代码::

    // item来源于JCGroupItem对象，memberSelf来源于JCGroupMember对象
    JCGroupMember* selfInfo = [[JCGroupMember alloc] init:item.groupId userId:memberSelf.userId displayName:displayName memberType:memberSelf.memberType changeState:JCGroupChangeStateUpdate];
    [group updateSelfInfo:selfInfo];

5. 添加、更新和删除群成员
::

    /**
     *  @brief 操作成员
     *  @param groupId 群标识
     *  @param members JCGroupMemeber 对象列表，通过 changeState 值来表明增加，更新，删除成员操作
     *  @return 返回操作id
     */
    -(int)dealMembers:(NSString*)groupId members:(NSArray*)members;

.. note:: 只有群主才可以删除成员。

其中，群变化状态（JCGroupChangeState）有以下几种：

.. list-table::

   * - 无
     - JCGroupChangeStateNone
   * - 新增
     - JCGroupChangeStateAdd
   * - 更新
     - JCGroupChangeStateUpdate
   * - 删除
     - JCGroupChangeStateRemove

示例代码::

    NSMutableArray* members = [[NSMutableArray alloc] init];
    // item来源于JCGroupItem对象
    [members addObject:[[JCGroupMember alloc] init:item.groupId userId:userId displayName:displayName memberType:JCGroupMemberTypeMember changeState:JCGroupChangeStateAdd]];
    [group dealMembers:item.groupId members:members];


dealMembers 结果回调
::

    /**
     *  @brief dealMembers 结果回调
     *  @param operationId 操作表示，由 dealMembers 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 JCGroupReason
     */
    -(void)onDealMembers:(int)operationId result:(bool)result reason:(JCGroupReason)reason;


6. 更新群
::

    /**
     *  @brief 更新群
     *  @param groupItem JCGroupItem 对象，其中 JCGroupItem 中 changeState 值不影响具体操作
     *  @return 返回操作id
     */
    -(int)updateGroup:(JCGroupItem*)groupItem;

示例代码::

    // item 来源于JCGroupItem对象
    [group updateGroup:[[JCGroupItem alloc] init:item.groupId name:name changeState:JCGroupChangeStateUpdate]];


更新群信息回调
::

    /**
     *  @brief 更新群信息调用回调
     *  @param operationId 操作表示，由 updateGroup 接口返回
     *  @param result true 表示登陆成功，false 表示登陆失败
     *  @param reason 当 result 为 false 时该值有效
     *  @param groupId 群标识
     *  @see JCGroupReason
     */
    -(void)onUpdateGroup:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupId:(NSString*)groupId;

7. 离开群组
::

    /**
     *  @brief 离开群组
     *  @param groupId 群标识
     *  @return 返回操作id
     */
    -(int)leave:(NSString*)groupId;

示例代码::

    // item来源于JCGroupItem对象
    [group leave:item.groupId];

离开群组结果回调
::

    /**
     *  @brief 离开群组回调
     *  @param operationId 操作表示，由 leave 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 JCGroupReason
     *  @param groupId 群标识
     */
    -(void)onLeave:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupId:(NSString*)groupId;


8. 解散群组(群主才能解散群组)
::

    /**
     *  @brief 解散群组，Owner才能解散群组
     *  @param groupId 群标识
     *  @return 返回操作id
     */
    -(int)dissolve:(NSString*)groupId;

示例代码::

    // item来源于JCGroupItem对象
    [group dissolve:item.groupId];

解散结果回调
::

    /**
     *  @brief 解散群组回调
     *  @param operationId 操作表示，由 dissolve 接口返回
     *  @param result true 表示成功，false 表示失败
     *  @param reason 当 result 为 false 时该值有效，参见 JCGroupReason
     *  @param groupId 群标识
     */
    -(void)onDissolve:(int)operationId result:(bool)result reason:(JCGroupReason)reason groupId:(NSString*)groupId;


**收发群组消息**

在群组中发送和接收消息参见 :ref:`消息<消息>` 模块。