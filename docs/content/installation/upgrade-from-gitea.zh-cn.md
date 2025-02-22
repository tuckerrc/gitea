---
date: "2021-09-02T16:00:00+08:00"
title: "从旧版 Gitea 升级"
slug: "upgrade-from-gitea"
sidebar_position: 100
toc: false
draft: false
menu:
  sidebar:
    parent: "installation"
    name: "从旧版 Gitea 升级"
    sidebar_position: 100
    identifier: "upgrade-from-gitea"
---

# 从旧版 Gitea 升级

想要升级 Gitea，只需要下载新版，停止运行旧版，进行数据备份，然后运行新版就好。
每次 Gitea 实例启动时，它都会检查是否要进行数据库迁移。
如果需要进行数据库迁移，Gitea 会花一些时间完成升级然后继续服务。

## 为重大变更检查更新日志

为了让 Gitea 变得更好，进行重大变更是不可避免的，尤其是一些里程碑更新的发布。
在更新前，请 [在 Gitea 博客上阅读更新日志](https://blog.gitea.io/)
并检查重大变更是否会影响你的 Gitea 实例。

## 降级前的备份

Gitea 会保留首二位版本号相同的版本的兼容性 (`a.b.x` -> `a.b.y`)，
这些版本拥有相同的数据库结构，可以自由升级或降级。
其他情况 (`a.b.?` -> `a.c.?`)下，
新版 Gitea 可能将会将数据库升级到与旧版数据库不同的结构。

举个例子:

| 当前 | 目标 | 结果 |
| --- | --- | --- |
| 1.4.0 | 1.4.1 | ✅ |
| 1.4.1 | 1.4.0 | ⚠️ 不建议，后果自负！尽管数据库结构可能不会变更，让它可以正常工作。我们强烈建议降级前进行完全的备份。 |
| 1.4.x | 1.5.y | ✅ 数据库会被自动升级。你可以直接从 1.4.x 升级到最新的 1.5.y。 |
| 1.5.y | 1.4.x | ❌ 数据库已被升级且无法用于旧版本Gitea，使用备份来进行降级。 |

**因为你不能基于升级后的数据库运行旧版 Gitea，所以你应该在数据库升级前完成数据备份。**

如果你在生产环境下使用 Gitea，你应该在升级前做好备份，哪怕只是小版本的补丁更新。

备份步骤:

* 停止 Gitea 实例
* 备份数据库
* 备份 Gitea 配置文件
* 备份 Gitea 在 `APP_DATA_PATH` 中的数据文件
* 备份 Gitea 的外部存储 (例如: S3/MinIO 或被使用的其他存储)

如果你在使用云服务或拥有快照功能的文件系统，
最好对 Gitea 的数据盘及相关资料存储进行一次快照。

## 从 Docker 升级

* `docker pull` 拉取 Gitea 的最新发布版。
* 停止运行中的实例，备份数据。
* 使用 `docker` 或 `docker-compose` 启动较新的 Gitea Docker 容器.

## 从包升级

* 停止运行中的实例，备份数据。
* 使用你的包管理器更新 Gitea 到最新版本。
* 启动 Gitea 实例。

## 从二进制升级

* 下载最新的 Gitea 二进制文件到临时文件夹中。
* 停止运行中的实例，备份数据。
* 将旧的 Gitea 二进制文件覆盖成新的。
* 启动 Gitea 实例。

在 Linux 系统上自动执行以上步骤的脚本可在 [Gitea 的 source tree 中找到 `contrib/upgrade.sh` 来获取](https://github.com/go-gitea/gitea/blob/main/contrib/upgrade.sh).

## 小心你的自定义模板

Gitea 的模板结构与变量可能会随着各个版本的发布发生变化，如果你使用了自定义模板，
你得注意你的模板与你使用的 Gitea 版本的兼容性。

如果自定义模板与 Gitea 版本不兼容，你可能会遇到：
`50x` 服务器错误，页面元素丢失或故障，莫名其妙的页面布局，等等…
移除或更新不兼容的模板，Gitea Web 才可以正常工作。
