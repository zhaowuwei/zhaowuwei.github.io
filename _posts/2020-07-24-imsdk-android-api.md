---
layout: article
title: IMSDK群组功能 Android端api文档
tags: IM
article_header:
  type: cover
  image:
    src: /screenshot.jpg
---

# IMSDK群组功能 Android端api文档

## 创建群组
```java
/**
 * 创建群组
 *
 * @param createGpInfo   群组初始化信息
 * @param ownerName      群组昵称
 * @param initMemberList 初始成员列表
 * @param callBack       结果回调
 */
void createIMGroup(final IMGroupInfo createGpInfo,
                   final String ownerName,
                   final List<CreateGroupMemberInfo> initMemberList,
                   final ICreateGroupResultCallback callBack);
```
## 修改群信息接口
```java
/**
 * 修改群设置和信息
 * 输入参数中除了群id外，只需要传入被修改的信息即可
 * @param groupInfo 修改后的群组信息
 * @param optUid 操作者uid
 * @param callback 结果回调
 */
void modifyIMGroup(IMGroupInfo groupInfo,
                   String optUid,
                   IManageGroupResultCallback callback);

```

## 解散群
```java
/**
 * 解散群组
 * @param groupId 被解散群组的id
 * @param groupType 被解散群组类型
 * @param callback 结果回调
 */
void dismissIMGroup(long groupId,
                    IMGroupConsts.IMGroupType groupType,
                    IManageGroupResultCallback callback);
                    
```
##  退出群
```java
/**
 * 主动退出一个群
 * @param groupId 群id
 * @param optUid 操作者uid
 * @param callback 操作结果回调
 */
void quitIMGroup(long groupId,
                 String optUid,
                 IManageGroupResultCallback callback);
                  
```
##  将某个用户踢出群组
```java
/**
 * 将某个用户踢出群组
 * @param groupId 群id
 * @param kickedUid 被踢出的用户uid
 * @param optUid 发起踢出操作的用户uid
 * @param reason 踢人原因 (可以传空)
 * @param callback 结果回调
 */
void kickGroupMember(long groupId,
                     String kickedUid,
                     String optUid,
                     String reason,
                     IManageGroupResultCallback callback);
```
## 设置/取消群成员为管理员
```java
/**
 * 设置取消群成员为管理员
 * @param groupId 群组id
 * @param memberUidList 成员id集合
 * @param isAdmin 设置或者取消
 * @param optUid 操作者uid
 * @param callback 结果回调
 */
void changeGroupAdminsSetting(long groupId,
                              List<String> memberUidList,
                              boolean isAdmin,
                              String optUid,
                              IManageGroupResultCallback callback);
```
##  从服务器查询群管理员信息
```java
/**
 * 批量查询管理员列表
 * @param groupIdList 查询的群组id集合
 * @param callBack 结果回调
 */
void getMutiGroupsAdminListRemote(List<Long> groupIdList,
                                  IRequestResultCallBack<List<String>> callBack);
```
## 设置/取消 群成员禁言状态
```java
/**
 * 设置/取消群成员禁言
 * @param groupId 群组id
 * @param memberUid 成员uid
 * @param targetSetting 设置的禁言类型
 * @param fordidDuration 禁言时长ms
 * @param optUid 操作人uid
 * @param callback 结果回调
 */
void changeGroupMemberForbidSetting(long groupId,
                                    String memberUid,
                                    IMGroupConsts.IMGroupForbidStatus targe
                                    long fordidDuration,
                                    String optUid,
                                    IManageGroupResultCallback callback);
                                    
```
                                    
##  从服务端查询群禁言成员列表
```java
/**
 * 从服务端查询群禁言成员列表
 * @param groupId 群组id
 * @param callBack 结果回调
 */
void getGroupForbidMemberListRemote(long groupId,
                                    IRequestResultCallBack<List<ForbidMemberInfo>> callBack);
```
## 修改群成员昵称
```java
/**
 * 修改群成员昵称
 * @param groupId 群组id
 * @param memberUid 成员uid
 * @param nickName 昵称
 * @param callback 结果回调
 */
void modifyGroupMemberName(long groupId,
                           String memberUid,
                           String nickName,
                           IManageGroupResultCallback callback);
```
## 查询群成员信息
```java
/**
 * 服务端批量查询群成员信息
 * @param memberList 成员信息集合
 * @param callback 结果回调
 */
void getMultiMemberInfoListRemote(List<SimpleGroupMemberInfo> memberList,
                                  IGetGroupMemberListCallback callback);
```
## 从服务器查询自己所有的群列表
```java
/**
 * 从服务端获取自己的群列表
 * @param uid 用户uid
 * @param callback 结果回调
 */
void getSelfGroupListRemote(String uid,
                            IGetIMGroupListCallback callback);
```

## 设置/ 取消群免打扰
```java
/**
 * 设置取消群勿扰
 * @param groupId 群组id
 * @param targetSetting 群勿扰类型
 * @param optUid 操作人uid
 * @param callback 结果回调
 */
void changeGroupShieldedSetting(long groupId,
                                IMGroupConsts.IMGroupShieldedStatus targetSetting,
                                String optUid,
                                IManageGroupResultCallback callback);
```
## 查询用户免打扰群列表
```java
/**
 * 从服务端查询用户免打扰的群列表
 * @param uid 用户uid
 * @param callBack 结果回调
 */
void getShieldedGroupListRemote(String uid,
                                IRequestResultCallBack<List<Long>> callBack);
```
## 加入或者移除用户到群黑名单
```java
/**
 * 加入或者移除用户到群黑名单
 * @param groupId 群组id
 * @param memberIdList 用户id集合
 * @param isAdd 是否加入黑名单
 * @param optUid 操作人uid
 * @param callback 结果回调
 */
void changeGroupBlackListSetting(long groupId,
                                List<String> memberIdList,
                                boolean isAdd,
                                String optUid,
                                IManageGroupResultCallback callback);
```
## 查询群黑名单列表
```java
/**
 * 服务端获取群黑名单列表
 * @param groupIdList 群id集合
 * @param callBack 结果回调
 */
void getMultiGroupBlackListRemote(List<Long> groupIdList,
                                  IRequestResultCallBack<List<SimpleGroupMemberInfo>> callBack);
```
##  申请加入群
```java
/**
 * 申请加入群组
 * @param groupId 群id
 * @param applyUid 申请人uid
 * @param applyerName 申请人昵称
 * @param applyContent 申请内容
 * @param callBack 结果回调 返回true表示立即进群，false需要等待审批
 */
void applyJoinGroup(long groupId,
                    String applyUid,
                    String applyerName,
                    String applyContent,
                    IRequestResultCallBack<Boolean> callBack);
```
## 查询申请的所有群列表(只查未处理的)
```java
/**
 * 查询群申请列表(只查未处理的)
 * @param groupId 群组id
 * @param applyUid 申请人uid
 * @param callBack 结果回调
 */
void getGroupApplyList(long groupId,
                       String applyUid,
                       IRequestResultCallBack<GroupApplyInfo> callBack);
```
## 同意申请人加入群
```java
/**
 * 批准/拒绝 申请加入群组
 * @param groupId  群id
 * @param applyUid 申请人uid
 * @param optUid   批准操作人uid
 * @param isApproved   是否批准
 * @param callback 结果回调
 */
void replyApplyJoinGroup(long groupId,
                         String applyUid,
                         String optUid,
                         boolean isApproved,
                         IManageGroupResultCallback callback);
```
## 邀请用户加入群聊
```java
/**
 * 邀请 别人 加入群组
 * @param groupId 群id
 * @param invitedUid 被邀请的用户uid
 * @param optUid 发起邀请的用户uid
 * @param roleType 邀请类型，1:邀请管理员; 2:邀请普通成员
 * @param callback 结果回调
 */
void inviteJoinGroup(long groupId,
                     String invitedUid,
                     String optUid,
                     IMGroupConsts.IMGroupRoleType roleType,
                     IManageGroupResultCallback callback);
```

## 同意/拒绝 邀请 别人加入群组

```java
/**
 * 同意/拒绝 邀请 别人加入群组
 * @param groupId 群id
 * @param invitedUid 被邀请的用户uid
 * @param isAgreed 是否同意邀请
 * @param nickname 用户昵称
 * @param callback 结果回调
 */
void replyInviteJoinGroup(long groupId,
                          String invitedUid,
                          boolean isAgreed,
                          String nickname,
                          IManageGroupResultCallback callback);
```

## 服务端批量查询群信息

```java
/**
 * 服务端批量查询群信息
 * @param idList 群组id集合
 * @param callback 结果回调
 */
void getMultiGroupDetailInfosRemote(List<Long> idList,
                                    IGetIMGroupListCallback callback);
```
