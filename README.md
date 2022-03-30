# 重课 Java 开发

[![](https://github.com/lanqiao-labs/java-docs/actions/workflows/sync-lab-docs.yml/badge.svg)](https://github.com/lanqiao-labs/java-docs/actions/workflows/sync-lab-docs.yml)

## 文档同步

- 文件夹中包含了一个课程的所有全部实验文档，文件夹命名规则：`课程ID-课程名称-课程类型`。
- 实验文档文档命名规则：`排序码-章节ID-章节名称.章节类型.md`，例如：`01-1-Linux-入门.sy.md`。

章节唯一 ID 是同步的依据，排序码仅方便章节有序展示，目前支持的自动同步资源有：

- `sy.md`：实验文档内容。
- `tz.md`：挑战文档内容。
- `wd.md`：文档章节内容。
- `sp.md`：视频描述内容。

其他说明：

- 除上述支持的 4 种章节类型外，其他资源需要到教师后台手动更新。
- 排序码在设置时需保证至少有 2 位，即以 01，02 样式依次递增。当章节数量为 100+ 时，排序码应该为 3 位。
- 排序码在设置时可以重复，方便插入新的章节。新插入的章节因章节 ID 较大，会自动排序到相同排序码文档的后一个。

## 更新方式

课程仓库 **不允许 Fork** 操作，更新内容时：

- 先从 master 建立新的分支，**禁止直接向 master 推送内容**。
- 从新分支向 master 提 PR，并交由同事 Review。

仓库添加了持续集成，向 `master` 主分支提交 PR 时，会自动触发格式修正。Prettier 会修正 Markdown 格式并 commit 到个人分支，同时将内容同步上线，以方便线上审核。
