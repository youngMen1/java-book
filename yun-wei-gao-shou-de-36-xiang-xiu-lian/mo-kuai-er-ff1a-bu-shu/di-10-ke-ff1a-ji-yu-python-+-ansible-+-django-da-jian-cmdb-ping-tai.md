# 第10课：基于 Python+Ansible+Django 搭建 CMDB 平台

本课时我们主要讲解如何基于 Python、Django、Ansible 开发一套具备 Devops 理念的自动化任务执行和资产管理(CMDB)系统。

## 工程简介

这个可以实现自动化任务执行和资产管理的系统取名为 Imoocc，它是基于 Python、Django、Ansible 实现的，共有两个版本，一个是 Python 2.7 版本（你可以在这个 GitHub 地址上 https://github.com/iopsgroup/imoocc 直接下载），另外一个是 Python 3.6 版本（你可以在本课时末尾的链接中通过附件下载）。



Imoocc 工程可以做到自动化资产收集和自动化任务执行，资产管理的部分会自动把服务器上所有的系统、硬件信息和宿主机子机关联信息收集起来，并做平台化展示。



然后打通自动化任务执行，客户端通过调用 API 接口的方式对搜集的资产信息对象执行自动化任务，对应关系如图所示。

CgpOIF5zJpGAOh_3AAETRSBrz8A133.png