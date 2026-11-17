---
kind: upgrade-guide
description: "Session.append rejects a message event whose message has no non-empty id instead of writing a log the next load refuses."
---

# Session append requires identified messages

English | [中文](guide.zh.md)

## Change

`Session.append` from `@deepseek-ai/dsh-session` now rejects a `user/message`, `developer/message`, `system/message`, `assistant/message`, or `tool/result` event whose message lacks a non-empty `id`, and it checks the message's role, source, and content while accepting the event. Previously such an append succeeded, the event was persisted verbatim, and reopening the session failed validation with `lacks an identified message`, which made the whole session unreadable. Affected callers are untyped producers, usually JavaScript plugins that hand-build a message; TypeScript callers already fail to compile. The [session README](../../../../packages/core/session/README.md) describes the append contract.

## Migration

1. Replace a hand-built message passed to `Session.append` or returned from `agent/pre-step` with `createUserMessage()`, `createAssistantMessage()`, `createToolResultMessage()`, or `createMessage()` from `@deepseek-ai/dsh-llm`. A plugin that cannot depend on that package mints a stable non-empty `id` (for example `crypto.randomUUID()`) on every new message.
2. Keep an existing message's `id` when rewriting its content or forwarding a message from another event; only a new semantic message mints a new id.
3. Treat a session that this defect already corrupted as refused: logged events are never repaired automatically. The load error names the event type and `seq`; back up the session file before attempting any repair.
4. Confirm the migration by appending the message again and reopening the session.
