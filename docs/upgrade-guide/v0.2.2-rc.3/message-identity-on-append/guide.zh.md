---
kind: upgrade-guide
description: "Session.append 会拒绝消息缺少非空 id 的消息事件，而不是写出下次加载会拒绝的日志。"
---

# Session 追加要求消息带标识

[English](guide.md) | 中文

## 变更

`@deepseek-ai/dsh-session` 的 `Session.append` 现在会拒绝消息缺少非空 `id` 的 `user/message`、`developer/message`、`system/message`、`assistant/message` 或 `tool/result` 事件，并在接受事件时检查消息的角色、来源与内容。此前这类追加会成功，事件被原样持久化，重新打开会话时校验失败并报 `lacks an identified message`，导致整个会话无法读取。受影响的调用方是无类型检查的生产方，通常是手工构造消息的 JavaScript 插件；TypeScript 调用方本来就无法编译。[session README](../../../../packages/core/session/README.zh.md) 描述了追加契约。

## 迁移

1. 把传给 `Session.append` 或从 `agent/pre-step` 返回的手工消息替换为 `@deepseek-ai/dsh-llm` 的 `createUserMessage()`、`createAssistantMessage()`、`createToolResultMessage()` 或 `createMessage()`。无法依赖该包的插件应为每条新消息生成稳定的非空 `id`（例如 `crypto.randomUUID()`）。
2. 改写消息内容或转发来自其他事件的消息时保留原 `id`；只有新的语义消息才生成新 id。
3. 把已经被该缺陷损坏的会话视为拒绝状态：已记录事件不会被自动修复。加载错误会指出事件类型与 `seq`；在尝试任何修复前先备份会话文件。
4. 再次追加该消息并重新打开会话以确认迁移完成。
