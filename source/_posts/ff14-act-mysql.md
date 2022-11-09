---
title: FF14 ACT 写 log 到 MySQL
date: 2022-11-09 21:00:00
tags:
- FF14
---

## FF14 ACT 写 log 到 MySQL

谁不想在自己(整天去世)的打本记录上 OLAP 呢？（不是

<!-- more -->

ACT 本身支持了一个通过 ODBC 把打本记录导出的功能。

一开始我天真地以为，Access 甚至 Excel 这种支持 ODBC 的办公软件接受数据就再好不过了，毕竟简单易用嘛，实测以后发现 ODBC 能连通，但是也没有然后了，巨大的 SQL 格式差异导致实际的语句都跑不通，我不折腾这个了，去翻了翻 ACT 作者的网站，发现他自己也是在 MySQL 5.6 上面做的这个功能。那既然这样，我们也来用 MySQL 就好了，何必和自己过不去。

首先在打游戏的 Windows 环境上装一下 MySQL 的 ODBC Client，这个 Client 亲测也可以和 Mariadb 互通的，直接装 MySQL 的就可以。

[Oracle MySQL Connector ODBC](https://dev.mysql.com/downloads/connector/odbc/)

注意一般都是选 `Windows x86 64bit`

然后，准备一个能连通的 MySQL 数据库，这里省略。

装好以后，跟随下图做设置。

{% asset_img act-settings.png %}

首先找到 选项 - 数据输出 - ODBC(SQL) 页面，通过 打开ODBC控制器 这个链接可以打开控制面板里面的 ODBC 设置，这里可以查看 ODBC Connector 有没有正确装好，装好了的话要在这里添加服务器配置，下述连接字符串中的 Driver 部分也要从这里查看。

下面给一个连接字符串示例：

```
Driver={MySQL ODBC 8.0 Unicode Driver};Server=192.168.1.215;Port=3306;User=act;Password=act123456;Database=act;Option=3;
```

填好以后，点测试连接，如果日志打印出 ODBC 连接成功字样，说明以上步骤做对了，我们可以调试写入了。

这里不需要我们提前建表，点验证表设置按钮就会帮我们建表，右边的删除表按钮会清空所有数据，谨慎。

表建好以后要做如下一个 DDL，修正一个字段类型。本来的职业是最大长度为 8 的 VARCHAR，一般的职业缩写都是三个字母，足够了，可是啊，职业列表里混进了一个东西叫做 `Limit Break`。LB 我谢谢你啊。。。。

```SQL
ALTER TABLE combatant_table MODIFY COLUMN job VARCHAR(16);
```

建好表以后就可以用了，可以到 导入/导出 当中测试一下把以往的记录导出一份，看看写入能不能走通，能通的话就可以打开自动导出了！

数据库表深度选项越往下越详细，最详细的级别可能还会碰到 schema 不够用的问题，也要修一下。一般来说前两档随便看看足够了，也就是输出战斗详细数据这个选项，够用了。想跑仔细的分析可以开更高，记录每一个技能。

看看自己死了多少次！

```SQL
SELECT SUM(combatant_table.deaths) FROM combatant_table join encounter_table ON combatant_table.encid = encounter_table.encid WHERE name = 'YOU';
```
