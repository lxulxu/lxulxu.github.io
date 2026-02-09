---
title: '生成追剧日历'
date: 2026-02-09
tags: [教程]
categories: [生活]
---

通过 [TV Time](https://www.tvtime.com/) 订阅追踪的剧集，自动将更新时间同步到日历应用，支持欧美剧集及部分国产剧，效果如下：

![](C:\Users\lxulx\Documents\我的文档\项目\MyGit\lxulxu.github.io\assets\images\Y2026Q1\1.jpg)



## 导入日历步骤

### 获取用户 ID

1. 登录 TV Time 网页版或应用
2. 进入个人主页
3. 从网页地址栏或个人资料页面获取用户 ID（一串数字）

****

![](C:\Users\lxulx\Documents\我的文档\项目\MyGit\lxulxu.github.io\assets\images\Y2026Q1\2.jpg)

### 构建订阅链接

订阅链接格式：

```
webcal://api.tvtime.com/v1/user/你的UserID/calendar.ics
```

示例：
```
webcal://api.tvtime.com/v1/user/12345678/calendar.ics
```

### 订阅日历

- # 如何订阅日历

  ## 方式一：系统日历应用

  1. 打开手机自带的日历应用
  2. 找到日历订阅链接功能入口(通常在设置或导入选项中)
  3. 输入订阅链接并确认

  > **注:** 不同品牌手机的日历功能存在差异。如果手机不支持直接订阅链接,可以先添加 Google 账号,然后通过账号同步日历(参见方式二)。

  ## 方式二：Google 日历

  1. 打开 Google 日历网页版(移动端暂不支持此功能)
  2. 点击左侧边栏"其他日历"旁的 **+** 图标
  3. 选择"通过网址添加"
  4. 粘贴订阅链接，点击"添加日历"完成订阅
  
------
  
**注意:**订阅日历仅显示未来 30 天内的日程安排
