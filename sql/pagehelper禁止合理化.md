## 前言
记录 pagehelper 合理化

## 问题描述

今天在做项目时发现使用 pagehelper 插件进行分页查找时出现第二页无数据也传回了第一页的数据，查找资料才发现是 pagehelper 的合理化导致的。合理化即 PageHelper 在查询完总记录数后，会计算总页数，当发现请求的页数大于总页数时，会将pageNum“合理化“一下，然后重新计算startRow。

## 问题解决
关闭 pagehelper 的合理化

```yml
pagehelper:
    reasonable: false # 禁用合理化时，如果第二页无数据不返回第一页的数据
```