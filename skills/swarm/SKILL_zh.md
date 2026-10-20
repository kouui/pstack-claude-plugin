---
name: swarm
description: "并行派出 N 个 worker，全部收齐后返回一份报告。用于 /swarm、'swarm this'，或并行覆盖、竞速、车轮战和探索。"
disable-model-invocation: true
---

# Swarm

在 Claude Code 中，遵循本 skill 之前，每个会话先阅读一次 [the Claude Code adapter](../../docs/harness/claude-code.md)。它把 pstack skills 中的 Cursor 名称映射到 Claude Code。

## 开始

在启动任何东西之前，打开一个 todolist，每个阶段一条。

1. Frame
2. Fan out
3. Aggregate
4. Report

## 阶段 A：Frame

1. 说明完成判定条件，以及 swarm 必须返回的产物或报告。
2. 选择形态。划分为若干切片，用相同的 brief 竞速 N 个 worker，或两者混合。对于竞速或混合形态，在派生之前声明 `first pass`、`rank all` 或 `best-of`。
4. 从 `~/.claude/rules/pstack-models.md` 中的 `swarm workers` 行选择 worker 模型。如果该规则或该行缺失，使用 `sonnet high`。对于 `auto` 或 `inherit-parent`，省略 `model`，让 worker 使用父模型运行。如果 Task 工具拒绝某个 slug，使用默认值并说明。如果它也拒绝默认值，则从其错误信息中选择同一系列中最接近的有效 slug。对于模型竞速，预先写明每个分支的模型。根据任务为每个 subagent 选择推理级别以及 fast 或 standard 速度，依据 **poteto-mode** skill 的 Subagents 部分。
5. 当 worker 写入时，为每个 worker 提供其独有的可写输出。当 worker 验证或测量 commit 时，每份 brief 写明确切的 SHA。测量类 brief 还要写明方法（样本数量、一个样本是什么、顺序）。worker 在结果中同时记录这两项。

## 阶段 B：Fan out

在一条消息中派出全部 N 个 worker，使用 `subagent_type: generalPurpose`、`isolation: "worktree"`、`run_in_background: true`，并将 `model` 按第 4 步设置；若为 `auto` 或 `inherit-parent`，则保持未设置。每个 worker 都在自己的本地 git worktree 和分支中运行。

当 worker 必须从非默认的已推送分支开始时，让 worker 在它的 worktree 中检出该分支。

每份 brief 都必须独立成立。包含目标、范围、确切的切片或竞速分支、验证方法，以及需要报告的内容。报告使用 `PASS`、`ISSUES` 或 `BLOCKED`，并附带证据。能够证明缺陷存在的 worker 报告 `ISSUES`，并列出它能证明的每一个问题，而不只是第一个。

如果某个 worker 掉线，则以 N-1 继续，并予以记录。

## 阶段 C：Aggregate

读取终态结果。丢弃没有记录其 brief 中所要求的 SHA 和方法的结果，并重新派生该 worker 一次。第二次仍未达标，则记录为缺口。缺口不计为通过。对于覆盖类任务，每个必需切片都必须有一份结果。对于竞速，应用事先声明的选择规则。使用 first pass、rank all 或 best-of。不要粘贴原始的 worker 输出。

## 阶段 D：Report

返回一份统一的聊天内报告，包含表格、问题的一句话概述、缺口或掉线情况，以及使用时的竞速规则。
