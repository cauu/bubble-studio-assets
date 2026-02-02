# OpenSpec 核心模板集

## 1. Proposal 模板 (proposal.md)

> 目标：定义变更的“为什么”和“是什么”，明确 Capability 边界。

```markdown
## Why

## What Changes

## Capabilities

### New Capabilities

- <name>: <描述> -> 对应创建 specs/<name>/spec.md

### Modified Capabilities

- <name>: <描述变更点> -> 对应更新现有 spec

## Impact

- **技术可行性**: 是否涉及架构调整？依赖项是否有变？
- **业务闭环**: 对现有业务流程（如下单、退款、结算）有何影响？
- **安全性**: 是否涉及敏感数据处理或权限变更？
```

---

## 2. Spec 模板 (specs/<name>/spec.md)

> 目标：定义具体的行为。使用 Delta 模式（仅记录增量）。

## ADDED Requirements

### Requirement: <ID> - <名称>

#### Scenario: <场景名称>

- **WHEN** <触发条件/输入参数/前置状态>
- **THEN** <预期结果/输出/状态变更/异常处理>

## MODIFIED Requirements

### Requirement: <ID> - <名称>

**Original State**: <简述原逻辑> **Updated Requirement**:

## REMOVED Requirements

### Requirement: <ID> - <名称>

**Reason**: <移除原因> **Migration**: <如果涉及数据迁移或接口弃用，请在此说明>

---

## 3. Design 模板 (design.md) - 可选

> 目标：定义“如何实现”。

## Context

## Decisions

- **Decision 1**: <具体决策及理由>
- **Trade-offs**: <权衡了哪些方案？为什么选这个？>

## Implementation Details

- **Schema Changes**: - **API Specs**: ```

---

## 4. Tasks 模板 (tasks.md)

> 目标：执行清单。

```markdown
## 1. Preparation

- [ ] 1.1 环境配置与依赖安装
- [ ] 1.2 数据库迁移脚本编写

## 2. Implementation

- [ ] 2.1 实现 <Capability Name> 的核心逻辑
- [ ] 2.2 单元测试覆盖核心 Scenario

## 3. Verification & Security

- [ ] 3.1 业务闭环链路测试
- [ ] 3.2 权限与输入校验安全扫描
```
