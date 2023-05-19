---
title: Oracle crsctl命令笔记
date: 2017-09-21 17:00:00
tags:  
  - Oracle  
  - 开发笔记
categories:
  - Oracle
toc: false
---

crsctl帮助命令
E:\app\11.2.0\grid\BIN>crsctl -h
用法: crsctl add       - 添加资源, 类型或其他实体
       crsctl check     - 检查服务, 资源或其他实体
       crsctl config    - 输出自动启动配置
       crsctl debug     - 获取或修改调试状态
       crsctl delete    - 删除资源, 类型或其他实体
       crsctl disable   - 禁用自动启动
       crsctl enable    - 启用自动启动
       crsctl get       - 获取实体值
       crsctl getperm   - 获取实体权限
       crsctl lsmodules - 列出调试模块
       crsctl modify    - 修改资源, 类型或其他实体
       crsctl query     - 查询服务状态
       crsctl pin       - 在节点列表中固定节点
       crsctl relocate  - 重新定位资源, 服务器或其他实体
       crsctl replace   - 替换表决文件的位置
       crsctl setperm   - 设置实体权限
       crsctl set       - 设置实体值
       crsctl start     - 启动资源, 服务器或其他实体
       crsctl status    - 获取资源或其他实体的状态
       crsctl stop      - 停止资源, 服务器或其他实体
       crsctl unpin     - 在节点列表中取消固定节点
       crsctl unset     - 取消设置实体值, 还原其默认值

开机启用Oracle High Availability Services
crsctl enable has
开机禁用
crsctl disable has
查看状态
crsctl config has

参考 http://blog.chinaunix.net/uid-20687159-id-1895010.html
