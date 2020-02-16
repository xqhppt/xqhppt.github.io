---
title: GitFlow实践
date: 2020-02-16 11:23:00
author: xqh
tags:
- 技术管理
- GitFlow
- 流程规范
categories: Git
---

## 什么是GitFlow

就像代码规范一样，Git Flow是一套代码管理时的行为规范。

## 为什么需要GitFlow

### 存在问题

1. 多人开发版本管理混乱
2. 多个功能开发版本管理难
3. 不同环境版本管理混乱
4. 版本分支众多命名混轮
5. 其他等等...


## GitFlow规范

先看流程图如下：

![HashMap关系图](git-flow.png)

### 常用分支

#### master分支

用来保存线上已发布的最新版本，该分支只能从其他分支合并过来不能直接修改。

#### develop分支

主开发分支，包含发布到下一个release版本的代码，主要是从feature分支合并过来不能直接修改。

#### feature分支

新功能开发分支，父分支为develop分支，开发完成后合并到develop分支。

#### release分支

当你需要发布版本时候，基于develop分支创建release分支。

#### hotfix分支

当你线上版本出现Bug后基于master分支创建hotfix分支进行代码修复。

> 上面只是一些基础概念我们来看看实际开发中应该如何进行，分支的创建可以手动也可以是基于运维平台。

### 实际操作

#### 项目创建

1. 默认远程仓库(github或gitlab等)项目分支是master。

#### 开发功能（1.0版本）

1. 基于master分支生成一个develop分支`develop/1.0.0`
2. 同学A从`develop/1.0.0`生成一个`feature/1.0.0`

> 如果有多人开发不同模块代码冲突几率小可以共用一个feature分支，如果协作复杂可以每个人生成一个分支如feature/1.0.0.A

3. 自测完成后把代码合并到`develop/1.0.0`分支进行测试。

> 当有多人开发1.0版本功能时候，按具体项目计划合并自测并提测

4. `develop/1.0.0`分支测试通过后基于此分支创建release分支并进行预发布测试

> 如果开发过程中有其他版本发布必须把master代码合并到此release分支

> 如果在release分支发现bug可以在release分支上修改bug,当然也可以回退到相应的feature分支

#### 发布项目

1. 测试通过后把对应的release分支发布上线，然后线上回归过后把release分支合并回master和develop分支。

> 如果是重要版本可以基于master版本打对应tag

#### 线上BUG修复

1. 如果上线后发现bug需要紧急修复，从master分支拉取hotfix分支并修改测试。
2. 测试通过后把hotfix分支合并回master和develop分支

#### 建议

1. 规范各个分支版本命名规则如feature/xxx方便管理
2. 在合并release之前检查有没有其他版本在开发过程中上线
3. 定期删除无用分支
4. 每次commit都有对应说明可以的话加上bug号
5. 合理划分每个分支权限以及开发人员权限
6. 通过相应的运维平台自动来约定检查发布




