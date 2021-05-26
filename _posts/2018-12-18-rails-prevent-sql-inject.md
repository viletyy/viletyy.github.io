---
layout: post
title: "Rails: Sql注入"
date:  2018-12-18 10:32:00
author: "Viletyy"
header-img: "img/post-bg-rwd.jpg"
tags:
  - Rails
  - Sql
  - 代码安全
---

所谓SQL注入，就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

我们永远不要信任用户的输入，我们必须认定用户输入的数据都是不安全的，我们都需要对用户输入的数据进行过滤处理。

防止SQL注入，我们需要注意以下几个要点：
1. 永远不要信任用户的输入。对用户的输入进行校验，可以通过正则表达式，或限制长度；对单引号和 双"-"进行转换等。
2. 永远不要使用动态拼装sql，可以使用参数化的sql或者直接使用存储过程进行数据查询存取。
3. 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限有限的数据库连接。
4. 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息。
5. 应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装。
6. sql注入的检测方法一般采取辅助软件或网站平台来检测，软件一般采用sql注入检测工具jsky，网站平台就有亿思网站安全平台检测工具。MDCSOFT SCAN等。采用MDCSOFT-IPS可以有效的防御SQL注入，XSS攻击等。

**示例**：

`User.order("#{sort_by} #{sort_direction}")`

如果 查询的是：

`sort_by = "email; DELETE from users; *--"*`

则把user全都删除了

**解决方案**：

```ruby
def index
 @users = User.order(sort_by + "" + direction)
end

private
def sort_by
  %w(email name).include?(params[:sort_by] ? params[:sort_by] : 'name')
end

def direction
  %w(asc desc).include?(params[:direction]) ? params[:direction] : 'asc'
end
```
