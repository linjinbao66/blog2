---
title: 一条复杂的sql
date: 2021-07-15
---

```sql
USE zentao;
SELECT
 zt_task.id AS '编号',
 zt_projectproduct.product AS '所属产品',
 zt_product.`name` AS '产品名称',
 zt_project.NAME AS '所属项目',
 zt_story.title AS '相关需求',
 zt_task.NAME AS '任务名称',
CASE
  zt_task.STATUS 
  WHEN 'closed' THEN
  '关闭' 
  WHEN 'done' THEN
  '完成' 
  WHEN 'cancel' THEN
  '取消' ELSE '未知' 
 END AS '任务状态',
 zt_task.estimate AS '最初预计',
 zt_task.consumed AS '总消耗',
 zt_task.openedDate AS '创建日期',
 zt_task.finishedDate AS '实际完成' 
FROM
 zt_task
 LEFT JOIN zt_project ON zt_task.project = zt_project.id
 LEFT JOIN zt_story ON zt_task.story = zt_story.id
 LEFT JOIN zt_projectproduct ON zt_project.id = zt_projectproduct.project
 LEFT JOIN zt_product ON zt_projectproduct.product = zt_product.id 
WHERE
 zt_product.`name` LIKE '%数栖%' 
 AND zt_task.openedDate > '2020-04-05 00:00:00'
 AND zt_task.parent =0
```

```sql
-- 导出需求跟踪表
USE zentao;
SELECT
 zt_story.id AS '编号',
 zt_product.`name` AS '所属产品',
 zt_module.`name` AS '所属模块',
 zt_productplan.title AS '计划',
CASE
  zt_story.source 
  WHEN 'market' THEN
  '市场' 
  WHEN 'bug' THEN
  'Bug' 
  WHEN 'other' THEN
  '其他' 
  WHEN 'project' THEN
  '项目' 
  WHEN 'research' THEN
  '内研' ELSE zt_story.source 
 END AS '需求来源',
 zt_story.sourceNote AS '来源备注',
 zt_story.title AS '需求名称',
CASE
  zt_story.`status` 
  WHEN 'changed' THEN
  '已变更' 
  WHEN 'active' THEN
  '激活' 
  WHEN 'draft' THEN
  '草稿' 
  WHEN 'closed' THEN
  '已关闭' ELSE '空' 
 END AS '当前状态',
CASE
  zt_story.stage 
  WHEN 'wait' THEN
  '未开始' 
  WHEN 'planned' THEN
  '已计划' 
  WHEN 'projected' THEN
  '已立项' 
  WHEN 'developing' THEN
  '研发中' 
  WHEN 'developed' THEN
  '研发完毕' 
  WHEN 'testing' THEN
  '测试中' 
  WHEN 'tested' THEN
  '测试完毕' 
  WHEN 'verified' THEN
  '已验收' 
  WHEN 'released' THEN
  '已发布' 
  WHEN 'closed' THEN
  '已关闭' ELSE '空' 
 END AS '所处阶段',
 zt_user.realname AS '由谁创建',
 zt_story.openedDate AS '创建日期',
 zt_story.closedDate AS '关闭日期' 
FROM
 zt_story
 LEFT JOIN zt_product ON zt_story.product = zt_product.id
 LEFT JOIN zt_module ON zt_story.module = zt_module.id
 LEFT JOIN zt_productplan ON zt_story.plan = zt_productplan.id 
 LEFT JOIN zt_user ON zt_story.openedBy = zt_user.account
WHERE
 zt_product.`name` LIKE '%数栖%' 
 AND zt_story.openedDate > '2020-04-05 00:00:00';
```

```sql
-- 导出缺陷跟踪清单
USE zentao;
SELECT
 zt_bug.id AS 'Bug编号',
 zt_product.`name` AS '所属产品',
 zt_module.`name` AS '所属模块',
 zt_project.`name` AS '所属项目',
 zt_bug.title AS 'Bug标题',
CASE
  zt_bug.severity 
  WHEN 1 THEN
  'P1' 
  WHEN 2 THEN
  'P2' 
  WHEN 3 THEN
  'P3' 
  WHEN 4 THEN
  'P4' ELSE zt_bug.severity 
 END AS '严重程度',
CASE
  zt_bug.`status` 
  WHEN 'closed' THEN
  '已关闭' 
  WHEN 'active' THEN
  '激活' 
  WHEN 'resolved' THEN
  '已解决' ELSE zt_bug.`status` 
 END AS 'Bug状态',
CASE
  zt_bug.confirmed 
  WHEN 1 THEN
  '已确认' 
  WHEN 0 THEN
  '未确认' ELSE zt_bug.confirmed 
 END AS '是否确认',
 zt_bug.openedDate AS '创建日期',
 zt_user.realname AS '解决者',
 zt_bug.closedDate AS '解决日期' 
FROM
 zt_bug
 LEFT JOIN zt_product ON zt_bug.product = zt_product.id
 LEFT JOIN zt_module ON zt_bug.module = zt_module.id
 LEFT JOIN zt_project ON zt_bug.project = zt_project.id 
 LEFT JOIN zt_user ON zt_bug.resolvedBy = zt_user.account
WHERE
 zt_product.`name` LIKE '%数栖%' 
 AND zt_bug.openedDate > '2020-04-05 00:00:00'
```
