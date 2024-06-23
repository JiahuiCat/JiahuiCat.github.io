---
layout:       post
title:        "Microsoft 365与Exchange邮箱之间互通架构图"
author:       "Hy.alva"
header-style: text
catalog:      true
tags:
    - Microsoft 365
    - Exchange
---

> Microsoft 365与Exchange邮箱之间共存架构
##
!(img/20240623-Microsoft 365架构.png)

## 名词解释

1.OWA (Outlook Web Access)
*   Outlook Web Access 是微软提供的一种通过网页浏览器访问和管理 Exchange 邮箱的解决方案。它允许用户在不使用桌面版 Outlook 客户端的情况下，通过网络浏览器收发邮件、管理日历、联系人和任务等。

2.IronPort (邮件反病毒功能)
*   Cisco 旗下安全产品，其中包括针对电子邮件和 Web 安全的解决方案。IronPort AV 特别指的是其电子邮件安全产品中的反病毒（Antivirus）功能。

3.LoadMaster (应用交付控制器-负载均衡)
*   应用交付控制器，通过负载均衡、安全增强和性能优化等功能，确保企业应用和服务的高可用性、可扩展性和安全性。

4.Migration Endpoint (迁移终端) 
*   进行数据或服务迁移时，作为数据迁移目标或源的服务器、存储设备或服务端点。迁移终端在迁移过程中，确保数据或服务能够从一个环境顺利迁移到另一个环境。

5.EOP (Exchange Online Protection)
*   微软云端电子邮件过滤服务，旨在保护用户免受垃圾邮件、恶意软件和其他电子邮件威胁。EOP 可以与 Microsoft 365（以前称为 Office 365）配合使用，也可以与其他本地或云端电子邮件解决方案集成。

6.Azure Active Directory (Azure AD) 
*   微软的基于云的身份和访问管理服务，旨在帮助组织管理用户身份并保护对资源的访问。Azure AD 是 Microsoft 365、Azure 和许多其他 SaaS 应用的基础身份服务，也是混合云和本地环境中身份管理的关键组成部分。
