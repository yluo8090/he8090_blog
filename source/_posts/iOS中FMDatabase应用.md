---
title: iOS中FMDatabase应用
date: 2016-10-31 15:56:15
tags:
  - FMDatabase
categories: 技术分享
---
> 使用FMDatabase库管理sqlite3数据库。
<br> 步骤如下：

1、创建本地DB
（1）DB.h
```
#import <Foundation/Foundation.h>
#import "FMDatabase.h"
#import "FMDatabaseQueue.h"
#import "LX_Singleton.h"
#define LXDAO [LX_Dao sharedInstance]
typedef void (^DatabaseBlock)(FMDatabase *db);
typedef void (^TransactionBlock)(FMDatabase *db, BOOL *rollback);
@interface LX_Dao : NSObject
AS_SINGLETON(LX_Dao);
- (void)inDatabase:(DatabaseBlock)block;
- (void)inTransaction:(TransactionBlock)block;
@end
```
（2）DB.m（注意数据库存储路径）
```
#import "LX_Dao.h"
static FMDatabaseQueue *dbQueue;
@implementation LX_Dao
DEF_SINGLETON(LX_Dao);
- (id)init
{
    self = [super init];
    if (self) {
        if (dbQueue == nil) {
            NSString *filePath = [[LX_Sandbox docPath] stringByAppendingPathComponent:@"linxunDb.db"];
            dbQueue = [[FMDatabaseQueue alloc] initWithPath:filePath];
        }
    }
    return self;
}
- (void)inDatabase:(DatabaseBlock)block{
    if (!block) {
        return;
    }
    [dbQueue inDatabase:^(FMDatabase *db) {
        [db setTraceExecution:YES];
        block(db);
    }];
}
- (void)inTransaction:(TransactionBlock)block{
    if (!block) {
        return;
    }
    [dbQueue inTransaction:^(FMDatabase *db, BOOL *rollback) {
        [db setTraceExecution:YES];
        block(db,rollback);
        if (*rollback == YES) {
            LX_DLog(@"数据库出错--->error:%@",[db lastErrorMessage]);
        }
    }];
}
@end
```
2、操作数据库（创建表，添加，删除，查询，修改）
```
- (void)createGroupListTable
{
    __block BOOL success = YES;
    [self inTransaction:^(FMDatabase *db, BOOL *rollback) {

        success = [db executeUpdate:@"CREATE TABLE if not exists group_list_table (_id integer not null primary key autoincrement, group_id text not null, group_name text not null, group_avatar text, create_time text not null, creator_id text not null)"];
        if (!success) {

            *rollback = YES;
        }
    }];
}
- (void)insertGroupWithModel:(GroupInfoModel *)model
{
    __block BOOL success = YES;
    [self inTransaction:^(FMDatabase *db, BOOL *rollback) {

        success = [db executeUpdate:@"INSERT INTO group_list_table (group_id,group_name,group_avatar,create_time,creator_id) VALUES (?,?,?,?,?)", model.groupId, model.groupName, model.groupAvatar, model.createTime, model.creatorId];
        if (!success) {

            *rollback = YES;
        }
    }];
}
- (void)updateGroupNameWithGroupId:(NSString *)groupId groupName:(NSString *)groupName
{
    __block BOOL success = YES;
    [self inTransaction:^(FMDatabase *db, BOOL *rollback) {

        success = [db executeUpdate:@"UPDATE group_list_table set group_name = ? where group_id = ?", groupName, groupId];
        if (!success) {

            *rollback = YES;
        }
    }];
}
- (void)updateGroupAvatarWithGroupId:(NSString *)groupId ImageStr:(NSString *)imageStr
{
    __block BOOL success = YES;
    [self inTransaction:^(FMDatabase *db, BOOL *rollback) {

        success = [db executeUpdate:@"UPDATE group_list_table set group_avatar = ? where group_id = ?", imageStr, groupId];
        if (!success) {

            *rollback = YES;
        }
    }];
}
- (void)deleteGroupWithGroupId:(NSString *)groupId
{
    __block BOOL success = YES;
    [self inTransaction:^(FMDatabase *db, BOOL *rollback) {

        success = [db executeUpdate:@"DELETE FROM group_list_table WHERE group_id = ? and creator_id = ?", groupId, USERID];
        if (!success) {

            *rollback = YES;
        }
    }];
}
```
